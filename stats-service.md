# The Stats Service

The Stats service provides a unified interface for services to take advantage of recording values against measurements, and then aggregating those measurements over various time periods to find sum/max/min and count stats.

The Stats service uses the path/key/layer construct of KV4 in a slightly different manner to the other services. Typically services intersect path/layer as entity/service, however, it's best to think of stats as intersecting path/layer as entity/measurement where the key is the user/device accumulating the data.

Given this intersection, the time series feature of KV4 comes into play. Each value put to an entity/measurement/key (path,layer,key) is always retained.

* An entity is any useful object a service wants to accumulate against. (path)

* A measurement is some measurement against the object (layer).

* Key is the user/device accumulating the stat.

An important point here is an entity is something in the system that could have multiple measurements against it, e.g. a user, or a host, a path.

Important entities for the user are typically the paths created in their public hierarchy, as this is where connections for turn and signals are created. Thus these are good candidate entities for recording measurements against. The system provides the following measurements for paths (new services will add new measurements).

<table>
  <tr>
    <td>measurement</td>
    <td>entity</td>
    <td>description</td>
  </tr>
  <tr>
    <td>router:sub</td>
    <td>namespace path</td>
    <td>a subscription to the router</td>
  </tr>
  <tr>
    <td>router:pkt</td>
    <td>namespace path</td>
    <td>the number/size of packets passing through the router</td>
  </tr>
  <tr>
    <td>turn:sess</td>
    <td>namespace path</td>
    <td>turn sessions created</td>
  </tr>
  <tr>
    <td>turn:data</td>
    <td>namespace path</td>
    <td>turn data transferred</td>
  </tr>
  <tr>
    <td>stun:sess</td>
    <td>namespace path</td>
    <td>stun sessions created</td>
  </tr>
  <tr>
    <td>stun:data</td>
    <td>namespace path</td>
    <td>stun data transferred</td>
  </tr>
  <tr>
    <td>api:3</td>
    <td>/service/METHOD</td>
    <td>get api call statistics per method and service</td>
  </tr>
</table>


For example, here is a measure of router packets on the system bus, the entity /runtime/hosts/active is only available to the sys user, but substitute your own creds and path in to retrieve the stats for the packets transferred over the given path/topic (i.e. the measurement (router:pkt) can be reused over multiple entities).

curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt"  | jq .{"v": {"sum": 30561,"min": 48,"max": 51,"count": 604},"s": "ok"}The interpretation is; there have been 604 packets published by the router(s) on the /runtime/host/active channel; the accumulated size of the packets was 30561 bytes, the smallest of which was 48 bytes, the largest of which was 51 bytes.

## The Stats Call ....

`curl "https://user:secret@endpoint/_stats/my/path?l=measurement&gs=y:m:d:h:min&ge=y:m:d:h:min&break=1"`

The measurement is provided using the l (layer) parameter. This is actually, very precise, as the kv4 layer is used to record the measurement axis.

Note: Typically you will be measuring against user defined namespace entities, as this is the historical use of the system, but other entities that various services find useful could be exposed. So, the stats entity (path) is not limited to namespace paths.

The gs and ge params are group start and group end respectively. Although typically used for dates, I've refrained from "ds" and "de" and a date syntax, as this query mechanism is not restricted to dates.

You may enter any level of the grouping, e.g.

    gs=2016:2&ge=2016:3

and be provided with the sum/count/max/min of the measurement over (in this instance) the given time period, at the level of the grouping. So, you get a single object back for the sum/count/max/min of the measurement over the time period.

However, if you add the break=1 option

    gs=2016:2&ge=2016:3&break=1

You'll get a list of two items over the given period, one for February and 1 for March.

If you take you're grouping down a level,

    gs=2016:2:4&ge=2016:3:10&break=1

You'll get a list of sum/count/max/min for each day, or without break=1 the totals between the periods given. You may take this to the minute level.

**Note: Stats are accumulated and posted every 15 secs to the database, so you won't get instant response_**

## Limitations

Stats is currently limited in it's breakdowns as the underlying database engine only provides 1 set of group breakdowns at a time, and currently we've plumped for grouping by time to the minute level. For example, being able to break down by layer tags, and partial paths, simultaneous to time, would totally rock, but this can't be achieved efficiently atm.

## Create

**You may create stats (example required)**

## Delete

You may not delete stats.

## Examples

### Router Packets

`curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:3&ge=2016:3:16:6&break=1"  | jq .{"v": {  "2016:3:16:5": {    "sum": 25349,    "min": 48,    "max": 51,    "count": 501  }},"s": "ok"}`

Here we've added grouping to the hour level, the query was between 3 - 6 hours, but the only data was for the 5th hour.

Let's break down the 5th hour packet and find stats over the first 10 mins of the hour ...

`curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:5:0&ge=2016:3:16:5:10"  | jq .{"v": {  "sum": 560,  "min": 50,  "max": 51,  "count": 11},"s": "ok"}`

There were 11 packets in the first 10 mins of the 5 hour in total, but we can break that down further into which minutes ...

`curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:5:0&ge=2016:3:16:5:10&break=1"  | jq .{"v": {  "2016:3:16:5:9": {    "sum": 50,    "min": 50,    "max": 50,    "count": 1  },  "2016:3:16:5:10": {    "sum": 510,    "min": 51,    "max": 51,    "count": 10  }},"s": "ok"}`

### API Calls

api:3 is a good example of a measurement, measured against some entity not created by namespace, but rather, stats creates the path /service/METHOD for you to query against, e.g. how many GETs have I done against the data service?

`curl -s "http://user:secret@endpoint/_stats/data/GET?k=user&l=api:3"  |jq .{"v": {  "sum": 5,  "min": 1,  "max": 1,  "count": 5  },  "s": "ok"}`

or how many calls against the stats service …

`curl -s "http://user:secret@endpoint/_stats/stats/GET?k=user&l=api:3"  |jq .{ "v": {   "sum": 23,   "min": 1,   "max": 1,   "count": 23 }, "s": "ok" }`

which increases by 1 every time I call it.

`curl -s "http://user:secret@endpoint/_stats/?l=api:3&gs=2016:3:6&ge=2016:3:8&break=0"  |jq .                                                                                       {"v": {    "2016:3:8": {      "sum": 93,      "min": 1,      "max": 1,      "count": 93    },    "2016:3:7": {      "sum": 57,      "min": 1,      "max": 1,      "count": 57    },    "2016:3:6": {      "sum": 9,      "min": 1,      "max": 1,      "count": 9    }  },  "s": "ok"}`

In this example, only the measurement is present (l=api:3), no key or path exists thus we get a full breakdown of calls over api:v3, totals calls by hour.

However we can be more specific and ask for the GETS to a specific service

....

Notice the key group has extended in the results to include the hour of the /data/GET

Finally you may add a key, the key will perform the breakdown for a given user. Typically the user is you. However, if you have added admins into your system then you can retrieve usage for each.

# Grafana Datasource

Grafana ([http://grafana.org](http://grafana.org)) is a terrific real time data visualization tool with lot’s of plugins/apps/datasources available from various vendors.

Xirsys leverages Grafana visualization by providing a data source that uses the standard Xirsys REST Api as a datasource.

![](/assets/image_2.png)

Simply point Grafana at one of our many cluster entry urls **using your own credentials** and you may access your data in the measurement/entity configuration of the Xirsys stats service.
![](/assets/image_3.png)

For example,
![](/assets/image_4.png)

Notice the metrics, turn:recv, turn:sent with the path that the series applies too.

From the Grafana editor you may select the metrics and paths available via drop down lists (note: this applies to any stat recorded in the system, including user defined).

![](/assets/image_5.png)

## Flexibility

* You may create multiple data sources, and mix them on the same chart for comparisons.
* You may run your own Grafana servers (using Docker is very simple).
