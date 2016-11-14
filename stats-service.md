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

curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt"  | jq .

## The Stats Call ....

curl "https://user:secret@endpoint/_stats/my/path?l=measurement&gs=y:m:d:h:min&ge=y:m:d:h:min&break=1"

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

curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:3&ge=2016:3:16:6&break=1"  | jq .

Here we've added grouping to the hour level, the query was between 3 - 6 hours, but the only data was for the 5th hour.

Let's break down the 5th hour packet and find stats over the first 10 mins of the hour ...

curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:5:0&ge=2016:3:16:5:10"  | jq .

There were 11 packets in the first 10 mins of the 5 hour in total, but we can break that down further into which minutes ...

curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:5:0&ge=2016:3:16:5:10&break=1"  | jq .

### API Calls

api:3 is a good example of a measurement, measured against some entity not created by namespace, but rather, stats creates the path /service/METHOD for you to query against, e.g. how many GETs have I done against the data service?

curl -s "http://user:secret@endpoint/_stats/data/GET?k=user&l=api:3"  |jq .

or how many calls against the stats service …

curl -s "http://user:secret@endpoint/_stats/stats/GET?k=user&l=api:3"  |jq .

which increases by 1 every time I call it.

curl -s "http://user:secret@endpoint/_stats/?l=api:3&gs=2016:3:6&ge=2016:3:8&break=0"  |jq .                                                                                       

In this example, only the measurement is present (l=api:3), no key or path exists thus we get a full breakdown of calls over api:v3, totals calls by hour.

However we can be more specific and ask for the GETS to a specific service

....

Notice the key group has extended in the results to include the hour of the /data/GET

Finally you may add a key, the key will perform the breakdown for a given user. Typically the user is you. However, if you have added admins into your system then you can retrieve usage for each.

# Grafana Datasource

Grafana ([http://grafana.org](http://grafana.org)) is a terrific real time data visualization tool with lot’s of plugins/apps/datasources available from various vendors.

Xirsys leverages Grafana visualization by providing a data source that uses the standard Xirsys REST Api as a datasource.

![image alt text](image_2.png)

Simply point Grafana at one of our many cluster entry urls **using your own credentials** and you may access your data in the measurement/entity configuration of the Xirsys stats service.

![image alt text](image_3.png)

For example,

![image alt text](image_4.png)

Notice the metrics, turn:recv, turn:sent with the path that the series applies too.

From the Grafana editor you may select the metrics and paths available via drop down lists (note: this applies to any stat recorded in the system, including user defined).

![image alt text](image_5.png)

## Flexibility

* You may create multiple data sources, and mix them on the same chart for comparisons.

* You may run your own Grafana servers (using Docker is very simple).

* Xirsys may run Grafana as a standard service on all of our hosts.

# Aside: Partitioning of Data

Before examining account API it’s worth mentioning how data is partitioned and indexed within CouchDB.

The first important point to note is that data pertaining to a given user is contained within it’s own CouchDB bucket. Each bucket has it’s own indexes. Access to raw CouchDB data could be provided directly for given users without compromising any other data in the system. Obviously, this could be very useful in lot’s of enterprise contexts we can’t think of - e.g. replication scenarios.

There is a top level account structure, for example, when you subscribe to Xirsys you get a top level account. However, each top level account can create subsequent accounts for themselves. From the context of Xirsys operation, this provides us with the potential to have "resellers". Each reseller may use the API to manage sub accounts, and create their own turn/???? Service. Each sub account of a reseller is also partitioned into their own CouchDB bucket. Access to this structure is handled by the API we have already investigated by overloading the user:secret parameters when calling the API.

In Xirsys there is 1 top level account called $sys. $sys has it’s own CouchDB bucket named kv4_$sys_data, this bucket contains all top level accounts. In addition it contains all system metadata, host subscriptions, and all other data that makes the system tick. It is also available via the REST api, so can be queried and updated in real time.

Thus when a top level account is authenticated it’s authenticated in the $sys bucket.

Whenever an account in $sys creates data (using the API) the information is stored in that accounts bucket. For example, if you have the account name "cool", then your data is stored in bucket kv4_cool_data.

If "cool" is a reseller, and adds it’s own sub-account, say “woot”, then that sub-account is authenticated from the cool bucket, but it’s data is saved in kv4_cool)woot_data.

This hierarchy is only two levels deep.

Within each of these buckets, at any level of account, you will find KV4 layers corresponding to the services that that particular account is using. If it doesn’t use a given service the data won’t exist.