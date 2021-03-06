# ![image alt text](image_0.png)

# XirSys v3 Enterprise API and Documentation

#### Created: 13th October, 2016
Last updated: 18th October, 2016

#### Contributors: Ritchie Turner, Lee Sylvester

[[TOC]]

# Introduction

XirSys is known as a world leader in TURN server provision.  However, the underlying platform, and XirSys long term focus, is not solely linked to WebRTC and TURN.

The XirSys platform consists of an Enterprise level application development and distribution/clustering framework based on Docker. All core functions are written in Elixir/Erlang.

 We use a cellular analogy to describe the system:

* *Stems *are containers which provide a specific runtime, e.g. Erlang, Node.js, .NET Core etc.

* *Cells* are applications which are loaded from a Git Repository (Github) into a stem to provide a differentiated cell (container) with a specific function.

* *Neurons* are the base Xirsys functionality carried by all nodes in a Xirsys network. 

* *Memory, *we provide an integral 4D data store accessible via REST/Websockets/TCP (tbd).

For example, our Turn service is an Elixir application, loaded into an Elixir Stem, in real time, from Github. This is an additional take on typical Docker container management which deals with images only. We use images to create a runtime environment then inject application code dynamically into the Stem.

This provides a very natural development cycle for testing and deploying new code. 

1. Push To Github

2. Differentiate to Stems

3. Launch on Stems

For script language (JS) work this is very desirable. **Heavy up front compilation environments can still use the typical Docker repo services, and build entire containers before deployment (tbd).**

Currently we use this method to deploy applications across 5 clusters and 20 hosts.

The Xirsys platform is very easily installed on a cluster, and a fully running system can be installed locally for development purposes.

# Neurons

All Xirsys nodes are Neurons. The Neuron contains the coordination primitives of the system; Data, Api, and Websocket support. All Neurons are equal and are aware of each other. If a neuron fails to ping the others on a regular basis the others tidy up connections/data belonging to the failed neuron on their host.

All other applications (cells), e.g. Turn, are clients to their local Neurons, that is, Turn containers and Neuron containers run on the same host. Turn containers use the REST api to talk to the local Neuron.

By this method, any server side app can be a client to it’s local Neuron, and have fast local access to data (more later), via http,websocket or **TCP (tbd)**.

## Data Persistence

Shared data is stored externally to neurons but pervasively cached in each neuron, in what we call KV4 - a 4 dimensional key/value store

All neurons contain materialized views of cached data based on change feeds from a central CouchDB database. This means that once data is loaded into the cache it maintains the latest value as set from any other neuron in the network. If a Neuron updates/deletes a key in CouchDB it’s update/deleted from all other Neuron caches via the CouchDB change feed.

Everything in the system is based on this mechanism, and is the basis of data distribution over multiple nodes. KV4 provides distributed Etcd like hierarchical services, categorized by path/layer/key/time (4d)

## Messaging

Each Neuron knows about all users attached to a given channel (path) via Websockets. As subscriptions are stored by path in KV4 all neurons know where a user/device is logged in and where to message it. Each neuron then acts as a router to a given channel, and may send messages directly to the node where the peer is logged in. Thus there is no central router - as mentioned all neurons are equal - they all contain cached subscription data updated reliably from CouchDB change feed. Interneuron messaging is secure HTTPS.

## Service API

Each Neuron provides a consistent and comprehensive REST API into all aspects of the system, based on service. 

# KV4 Data

Before proceeding into the structure of services, it’s worthwhile examining the structure of KV4 data as all other services key off it’s influence. Understanding KV4 is essential to understand the system. Why 4D?

### Dimensional Progression

<table>
  <tr>
    <td>Dimension</td>
    <td>Description</td>
  </tr>
  <tr>
    <td>1D</td>
    <td>Not much to say about this a hash table is a 1D k/v store</td>
  </tr>
  <tr>
    <td>2D</td>
    <td>Index a hash table by path. Each path has a hash at each segment junction.</td>
  </tr>
  <tr>
    <td>3D</td>
    <td>Index a hash table by path / layer. Each segment/layer has a hash</td>
  </tr>
  <tr>
    <td>4D</td>
    <td>An 3D hash hash where each K/V is a time series, that is, nothing is deleted.</td>
  </tr>
</table>


This can be viewed as a tree with multiple layers.

![image alt text](image_1.png)

# Service Overview

Xirsys is based around a dynamically expanding set of services. Basic services include; accounts, namespace, data, stats, and authorization. Extended services include; turn and video processing, with more to come. 

Each service manages a layer of the KV4 tree and will create or recognise various paths (entities) in the tree that are significant to it. **The service name is the data layer name.**

A service provides some entity which is identified by path and key, .e.g,

* The account service recognizes "/_acc/accounts?k=myident"

* The subscriptions service recognizes "/_subs/domain/app/room/topic?k=chatter"

The API is "soft" in that a new service may be added which may interact with other data in the system - typically for the same path.

**Internal** services are those contained within the neuron itself. Currently recognised internal services are …

<table>
  <tr>
    <td>Internal Service</td>
    <td>Prefix</td>
    <td>Layer</td>
  </tr>
  <tr>
    <td>accounts</td>
    <td>_acc</td>
    <td>*acc</td>
  </tr>
  <tr>
    <td>namespace</td>
    <td>_ns</td>
    <td>*ns</td>
  </tr>
  <tr>
    <td>subscriptions</td>
    <td>_subs</td>
    <td>*subs</td>
  </tr>
  <tr>
    <td>authorization</td>
    <td>_auth</td>
    <td>*auth</td>
  </tr>
  <tr>
    <td>stats</td>
    <td>_stats</td>
    <td>*stats</td>
  </tr>
  <tr>
    <td>user data</td>
    <td>_data</td>
    <td>*data</td>
  </tr>
  <tr>
    <td>host</td>
    <td>_host</td>
    <td>n/a</td>
  </tr>
  <tr>
    <td>token</td>
    <td>_token</td>
    <td>n/a</td>
  </tr>
</table>


token and host services are virtual, they don't correspond to any persistent KV4 layer.

Xirsys also supports services external to the neuron, like turn and video processing; Although they run on the same host as the neuron, they communicate only via the public APIs. **In fact, external services may be arbitrary, server side apps.**

<table>
  <tr>
    <td>External Service</td>
    <td>Prefix</td>
    <td>Layer</td>
  </tr>
  <tr>
    <td>turn</td>
    <td>_turn</td>
    <td>*turn</td>
  </tr>
  <tr>
    <td>video</td>
    <td>_video</td>
    <td>*video</td>
  </tr>
</table>


This model then, allows a base set of Xirsys system hosts that fulfill infrastructure requirements yet allows clean integration of external service hosts and their data.

External services, require to be registered with the base system which then proxies requests, after authorization etc, to the "best" external service host based on load/location etc. This requires that prefix names (e.g. _turn etc ) are always unique.

Subsequent versions of Xirsys will come equipped with a  _service service where new external services can be added dynamically.

# The Shape of the Service API

Using the Xirsys HTTP API, you can GET/PUT/POST/DELETE to any service using the following syntax (depending on applicability). The basic shape of all the commands is the same, and references to keys, layers etc are consistent, there are however parameter variations when required.

## GET

A value for a specific key ...

curl "https://user:secret@endpoint/_servicename/my/path?k=key"

All keys at node

curl  "https://user:secret@endpoint/_servicename/my/path"

## POST/PUT

curl -XPOST -H 'Content-type: application/json'  "https://user:secret@endpoint/_servicename/my/path" -d '{"k": "mykey","v": "myvalue"}'

You typically PUT to create new entities, and POST to update, e.g. you may create and update accounts respectively.

## DELETE

Delete a key.

curl  -XDELETE "https://user:secret@endpoint/_servicename/my/path?k=key"

Delete all keys at path.

curl  -XDELETE "https://user:secret@endpoint/_servicename/my/path"

**Note, that the prefix (_servicename) used in the API is the layer that the data resides in.**

# The Namespace Service

Each service can create it's own hierarchy, but typically services will interact around the hierarchy created and managed by the Xirsys account holder via the namespace service. 

The namespace is effectively the public tree of the user and provides backwards compatibility with previous versions of Xirsys. The Namespace service allows the creation of Channels

The namespace is a hint to all services, that the user has a specific hierarchy of names they want to work with, it is then up to the service if that namespace can be used in its context - if so, better, as it provides more context for the user.

Note: In the current version of Xirsys there is no "referential integrity" between layers, so paths in one layer are not tied to paths in others - only by convention.

## Create a new path in the public tree …

curl -X PUT "https://user:secret@endpoint/_ns/my/path"

## Delete Path ...

curl -X DELETE "https://user:secret@endpoint/_ns/my/path"

Note: This deletes all paths under the provided.

## Get

All tree nodes under the specified path for the _ns service …

curl "https://user:secret@endpoint/_ns/my/path"

All nodes to the given depth under the current path …

curl "https://user:secret@endpoint/_ns/my/path?depth=2"

To list your entire namespace to segment depth 10 …

curl "https://user:secret@endpoint/_ns/depth=10

# The Subscription Services

When a chatter or turn client creates persistent connection to Xirsys the information for that subscription is stored under the path/topic of interest.

Historically this meeting point between users was denoted by the path /domain/application/room. This is no longer a restriction, but the legacy api enforces the the 3 segment path.

## Topics

Topics are extra classification that are created specifically by the _subs service.

The user may have constructed a path /my/path using the namespace service, but the _subs service provides more granularity, by extending topic nodes from the existing path /my/path, e.g.

* /my/path/topic1

* /my/path/topic2

Each topic node then contains subscription keys/values of chatters.

This data is stored in the subs layer so there is no overlap with the base namespace, thus the base namespace is shadowed. This is probably the best example of a service adorning the public namespace of the user with service specific information contained in a separate layer.

### V2 API (Legacy)

From an V2 API perspective the addition of a topic is a further classification of room to topics, e.g.

/domain/application/room/topic

and for the V2 API the system defaults to a default topic of "topic".

## Create

You can't create subscriptions via HTTP at the moment, this is done using websockets typically.

## Delete

### Kicking A Chatter

Because a subscription is available via the _subs service it may be deleted on demand. When a subscription is deleted it's removed from all neuron routers automatically, so a DELETE to the subs service is the same as kicking an existing user.

curl -XDELETE "https://user:secret@endpoint/_subs/my/path/topic?k=chatter"

### Kicking a Room

curl -XDELETE "https://user:secret@endpoint/_subs/my/path/topic"

## Banning

Please see Authorization service for banning.

## Get

### List Subscription keys for a given topic.

This is a live list of active "chatters". Just the chatter keys

curl -s  "https://user:secret@endpoint/_subs/richie.com/default/default/topic" | jq .
{
"v": [
"rt"
],
"s": "ok"
}

Or more detailed subscription information of you request the values

curl -s  "https://user:secret@endpoint/_subs/richie.com/default/default/topic?as=values" | jq .
{
 "v": [
   {
     "topic": "topic",
     "service": "xs_turn",
     "path": "/richie.com/default/default",
     "origin": "euro_euro-core",
     "meta": null,
     "key": "rt",
     "ident": "user"
   }
 ],
 "s": "ok"
}
Probably the most interesting piece of data there is the origin server that the subscription logged in from.

## Get specific subscription data.

curl  "https://user:secret@endpoint/_subs/my/path/topic?k=chatter

# The Data Service

The data service provides for the creation of user data on the tree in the data layer.

## Create

curl -XPOST -H 'Content-type: application/json'  "http://user:secret@endpoint/_data/my/path" -d '{"k": "mykey","v": "myvalue"}'

## Delete

### Deletes a key in the data layer

curl -X DELETE "https://user:secret@endpoint/_data/my/path?k=key"

### Deletes path in the data layer …

### curl -X DELETE "https://user:secret@endpoint/_data/my/path"

### Get a Value

curl "https://user:secret@endpoint/_data/my/path?k=key"

### Get list of keys

curl "https://user:secret@endpoint/_data/my/path"

### Get timeseries for a key

curl "https://user:secret@endpoint/_data/my/path?time_series=1"

Time series may be queried using the same syntax for stats queries, where gs/ge are group start/group end resp, and broken into year:month:day:hour:minute

curl -s  "http://user:secret@endpoint/_data/my/path?k=mykey&time_series=1&gs=2016:3:16:3:4&ge=2016:3:16:3:20"  | jq . 
{
 "v": [
   [
     1458109254,
     "user",
     "myvalue"
   ],
   [
     1458109242,
     "user",
     "myvalue"
   ]
 ],
 "s": "ok"
}

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
{
"v": {
"sum": 30561,
"min": 48,
"max": 51,
"count": 604
},
"s": "ok"
}
The interpretation is; there have been 604 packets published by the router(s) on the /runtime/host/active channel; the accumulated size of the packets was 30561 bytes, the smallest of which was 48 bytes, the largest of which was 51 bytes.

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
{
"v": {
  "2016:3:16:5": {
    "sum": 25349,
    "min": 48,
    "max": 51,
    "count": 501
  }
},
"s": "ok"
}

Here we've added grouping to the hour level, the query was between 3 - 6 hours, but the only data was for the 5th hour.

Let's break down the 5th hour packet and find stats over the first 10 mins of the hour ...

curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:5:0&ge=2016:3:16:5:10"  | jq .
{
"v": {
  "sum": 560,
  "min": 50,
  "max": 51,
  "count": 11
},
"s": "ok"
}

There were 11 packets in the first 10 mins of the 5 hour in total, but we can break that down further into which minutes ...

curl -s  "http://%24sys:secret@endpoint/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:5:0&ge=2016:3:16:5:10&break=1"  | jq .
{
"v": {
  "2016:3:16:5:9": {
    "sum": 50,
    "min": 50,
    "max": 50,
    "count": 1
  },
  "2016:3:16:5:10": {
    "sum": 510,
    "min": 51,
    "max": 51,
    "count": 10
  }
},
"s": "ok"
}

### API Calls

api:3 is a good example of a measurement, measured against some entity not created by namespace, but rather, stats creates the path /service/METHOD for you to query against, e.g. how many GETs have I done against the data service?

curl -s "http://user:secret@endpoint/_stats/data/GET?k=user&l=api:3"  |jq .
{
"v": {
  "sum": 5,
  "min": 1,
  "max": 1,
  "count": 5
  },
  "s": "ok"
}

or how many calls against the stats service …

curl -s "http://user:secret@endpoint/_stats/stats/GET?k=user&l=api:3"  |jq .
{
 "v": {
   "sum": 23,
   "min": 1,
   "max": 1,
   "count": 23
 },
 "s": "ok"
 }

which increases by 1 every time I call it.

curl -s "http://user:secret@endpoint/_stats/?l=api:3&gs=2016:3:6&ge=2016:3:8&break=0"  |jq .                                                                                       
{
"v": {
    "2016:3:8": {
      "sum": 93,
      "min": 1,
      "max": 1,
      "count": 93
    },
    "2016:3:7": {
      "sum": 57,
      "min": 1,
      "max": 1,
      "count": 57
    },
    "2016:3:6": {
      "sum": 9,
      "min": 1,
      "max": 1,
      "count": 9
    }
  },
  "s": "ok"
}

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

# The Accounts Service

The accounts service provides 2 functions:

1. New logins to the base Xirsys account, that may be subsequently used in API interactions, i.e. other admin credentials. All facilities on your account are available to the new admin, and each change to the data is stamped with the admins credentials. You can see changes to data by using the time_series=1 option on most GET calls.

2. You may create full Xirsys sub-accounts, that are manipulated via the standard API.

## Administrators

### Create an Admin

Post a login name and password to /users/admins, for an administrator login.

curl -H "Content-type: application/json" "http://user:secret@endpoint/_acc/users/admins" -d '{"k": "key", "v": {"password": "password"}}'

### Delete an Admin

curl -X DELETE "http://user:secret@endpoint/_acc/users/admins?k=key"

### List Admins

curl "http://user:secret@endpoint/_acc/users/admins"

### Get Admin

curl "http://user:secret@endpoint/_acc/users/admins?k=key"

### Time Series

curl "http://user:secret@endpoint/_acc/users/admins?k=key&time_series=1"

### Usage

To login as an administrator, just use the following syntax in the login credentials.

curl "https://user:admin_key#admin_password@endpoint/_data/my/path"

Notice that the secret is now loaded with 2 fields of information separated by a "#".

We need to have the main Xirsys account holder username (ident), and once we have that we can authenticate the admin within that context.

Currently all admins have the same rights, role based permissions will be available at some stage.

Note: Time series are returned with the time, login and value, so you can see who made changes to the data.

## Account Proxy

The _acc service allows the creation of full Xirsys sub-accounts that you may manage using the standard API.

This gives you fully self contained stats and services for your sub-accounts. For example, a sub-account could require it's own proprietary namespace.

### Create

Simply PUT to the _acc services /accounts path to add a new sub-account.

curl -H "Content-type: application/json" "http://user:secret@endpoint/_acc/accounts" -d '{"k": "username", "v": {"password": "password","email": "email","first_name": "first_name","last_name": "last_name"}}'

### Delete

This will delete the sub-account and all it's data.

Note: Sub-account data is completely self contained in it's own 'table', the entire 'table' is removed with this action.

curl -XDELETE "http://user:secret@endpoint/_acc/accounts?k=username'

### Get

To get the details of a sub-account ...

curl "http://user:secret@endpoint/_acc/accounts?k=username'

### Usage

Usage of sub-accounts is driven entirely through the overloading of the authorization headers. This allows the API to remain consistent, and to allow Xirsys V2 services to use the feature without extension.

The authorization of a sub-account must take place in the context of the primary Xirsys account holder (you), thus the primary username must always be present. The authorization header looks like this

primary_username#subaccount_username:subaccount_secret

Or more easily remembered, you must prefix primary_username# to any sub-account call.

For example to add a namespace to the sub-account

curl -s -XPUT "https://primary_username#subacount_username:subaccount_secret@endpoint/_ns/my/path"

Effectively this syntax redirects the call to a different database 'table', and this is consistent across all services.

For primary accounts executing requests for sub-accounts looking up a sub-account secret is not efficient, so primary account holders may authenticate with their own secret.

To indicate primary account holder secret prefix the secret with "!".

curl -s -XPUT "https://primary_username#subacount_username:!primaryaccount_secret@endpoint/_ns/my/path"

## Walkthrough

Here's a walk thru of using accounts, which includes creation of a top level account,via the sys user; I've included that so that you can see how your account creation is exactly the same as the system's

Here's the creation of a top level primary account by sys…

curl -s -XPUT -H 'Content-type: application/json' "http://%24sys:secret@endpoint/_acc/accounts" -d '{"k": "elena", "v": {"email": "elena@user.cl","password": "woot"}}'
{"v":{"username":"elena","term":"monthly","secret":"8af2a710-eba7-11e5-b4a0-9cdd20b4c6b5","plan":"free","password":"woot","last_name":null,"first_name":null,"email":"elena@async.cl","customer_id":null,"created":1458154052,"company":null,"active":true},"s":"ok"}

Notes:

* the key is the "ident", added to the body as "username" automatically

* email is required, and is added as a secondary index for the path "/accounts"

* active is true, controls if the user may login in or not

So, now, Elena is a top level account holder, and sys can query for her in 2 ways by ident(key), and by email, like this …

curl -s  "http://%24sys:secret@endpoint/_acc/accounts?k=elena" | jq .
 {
   "v": {
     "username": "elena",
     "last_name": null,
     "first_name": null,
     "email": "elena@async.cl",
     "created": 1458203041,
     "company": null,
     "active": true
   },
   "s": "ok"
 }

or because a secondary key was added automatically on email …

curl -s  "http://%24sys:secret@endpoint/_acc/accounts?k2=elena@async.cl" | jq .
{
   "v": {
     "username": "elena",
     "last_name": null,
     "first_name": null,
     "email": "elena@async.cl",
     "created": 1458203041,
     "company": null,
     "active": true
   },
   "s": "ok"
}

Notice we're querying on k2.

### Aside: Secondary Keys

KV4 has 4 keys in total you can use to index into your objects per_path;k, k2, k3 and k4. So, accounts objects in /accounts could be looked up in 4 different ways, but the same is true for any other path you could create. These 4 keys are also integrated into the caching system so GETs on any of these are fast. 4 keys is not a hard limit, so more work could go into this to get more out of it. 4 keys was chosen as KV4 should have 4 indexes, we can add a fifth for kv5 :).

Please see Secondary Keys for more info on creation (TBD).

### Primary Account Holder Creates Secondary Account

So, account holder Elena wishes to create a secondary account. She can do so by PUTing to her /accounts, like this …

curl -s -XPUT -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts" -d '{"k": "lily", "v": {"email": "lily@async.cl","password": "woot2"}}' | jq .
{
  "v": {
    "username": "lily",
    "term": "monthly",
    "secret": "068564e6-ec1e-11e5-b3d6-09c98c1c18d8",
    "plan": "free",
    "password": "woot2",
    "last_name": null,
    "first_name": null,
    "email": "lily@async.cl",
    "customer_id": null,
    "created": 1458204940,
    "company": null,
    "active": true
  },
  "s": "ok"
}

So Elena PUTs sub account lily to her /accounts path in the _acc service. Notice the complete symmetry between the sys user putting a new account and Elena.

### Querying Sub Accounts

Just as sys would do to get a list of it's accounts, Elena may query her /accounts path,

curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts" | jq .
 {
 "v": [
   "lily"
  ],
  "s": "ok"
}

so now we know lily is a sub account holder we can query for more details …

curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts?k=lily" | jq .
{
  "v": {
    "username": "lily",
    "last_name": null,
    "first_name": null,
    "email": "lily@async.cl",
    "created": 1458204940,
    "company": null,
    "active": true
  },
  "s": "ok"
}

### Setting Sub Account Properties

You may alter sub account properties by POSTing new attribute values to the key. For example, let's say you want to change the "active" property ...

curl -s -XPOST -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts" -d '{"k": "lily", "v": {"active": false}}' | jq .
{
  "s": "ok"
}

and to check …

curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts?k=lily" | jq .
{
  "v": {
    "username": "lily",
    "last_name": null,
    "first_name": null,
    "email": "lily@async.cl",
    "created": 1458204940,
    "company": null,
    "active": false
  },
  "s": "ok"
}

You may post new properties to the object using this means too. The system (should not) does not allow you to change email or username at this time.

### Lily as a Stand Alone User

Now that Lily has been created by Elena, she may want to add new namespaces for herself, so the new Lily account is completely self contained. So this may seem reasonable …

curl -s -XPUT "http://lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/a/path/that/lily/created" | jq .
{
  "v": "unauthorized",
  "s": "error"
}

however, this would look Lily up in the top level accounts database (sys database), where it does not exist. The Xirsys top level database, only knows about Elena, not her secondary accounts. Thus, we need to perform the action in the context of Elena, so this seems reasonable …

curl -s -XPUT "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/a/path/that/lily/created" | jq .
{
  "v": "unauthorized",
  "s": "error"
}

This would typically work, however, above, I deactivated the account, and the activation is checked in all authorizations, so, let's fix that …

curl -s -XPOST -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts" -d '{"k": "lily", "v": {"active": true}}' | jq .
{
  "s": "ok"
}

now ...

curl -s -XPUT "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/a/path/that/lily/created" | jq .
{
"s": "ok"
}

and to check to the 5th level …

curl -s "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/?depth=5" | jq .
{
  "v": [
    "a/path/that/lily/created"
  ],
  "s": "ok"
}

### Usage - A Proxy Service.

The secondary account system purpose is to allow proxy usage of Xirsys. Elena can update her own accounts and data, but now she has a secondary account holder Lily, who she wants to create a Xirsys service for.

Thus a means for Elena to create namespaces, subscriptions and so on in the context of Lily is required.

Elena doesn't want to check Lily's secret each time, so she wants to do the authorization as herself with her secret, this may seem reasonable …

curl -s -XPUT "http://elena#lily:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_ns/a/path/that/elena/created/for/lily" | jq .
{
"v": "unauthorized",
"s": "error"
}

Here we have Elena trying to add a path for Lily but with her own secret, and it fails. Obviously, as above, the system assumes, that the secret provided is the user's (Lily's) secret and not Elenas. So to indicate that it's the primary account holders secret and not the secondary, we prefix secret with a "!", like so …

curl -s -XPUT 'http://elena#lily:!9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_ns/a/path/that/elena/created/for/lily' | jq .
{
  "s": "ok"
}

Please note here, I changed the " to ' quotes for bash as it will do a substitution on the ! otherwise.

The data although authorized on Elena's account, does in fact go into Lily's data bucket. Here is Lily authorizing with Lily's secret, to find the new data deposited by the primary account holder …

curl -s "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/?depth=7" | jq .
{
  "v": [
    "a/path/that/elena/created/for/lily",

    "a/path/that/lily/created"
  ],
  "s": "ok"
}

**All other services will authorize in the same way.**

# Authorization Service

The authorization service allows you to manage IP addresses per node that will disallow connections from the given IP range.

## Deny

curl -XPUT -H "Content-type: application/json" "http://async:secret@endpoint/_auth/my/path" -d '{"k": "IPADDRESS"}'

The value is unimportant, as we simply utilise the key here.

All services will now ignore connections from the given IP address, on the given path or below.

## Allow

curl -DELETE -H "http://async:secret@endpoint/_auth/my/path?k=IPADDRESS'

This service is in it's infancy, and can provide a lot more going forward.

# Host Service

_host provides details about servers currently operating in the system. Currently you can query turn and signals hosts.

curl -s "https://async:secret@endpoint/_host/best/signal" | jq .                                                                                                                 
{
"v": "wss://euro3.xirsys.com:4005/ws",
"s": "ok"
}

/_host/best provides the best load balanced signals machine from the current cluster. This is equivalent to the APIV2 /wsList.

Similarly for _hosts/best/turn.

# **Token Service**

_token provides a token for signals authentication.

You may create a token specifically for a path or a universal token. The V2 API requires a full path (/domain/application/room) while the V3 API requires just requires a token.

To add a chatter key add k=chatter_key.

curl -s -XPUT "https://user:secret@endpoint/_token/my/path?k=chatter_key&expire=20" | jq .
{
"v": "ZG1enoNPMqSLf93auDFjMD84jTVRMsoor7nHHMGoG_4HsH3ogWlNUmMwIFekwe44FEWYoBQMBAFfiFluzOL2K6QZJ2Qc4H1-bCaYcYpuRB9Tvcqyv9l53RHG1SUHdL7tCEjxrDDFDMtDrRgs5A",
"s": "ok"
}

This example, generates a API V2 compatible token which includes the path specified, and only allows login to the specified path. It is equivalent to the V2 API /getToken url.

curl -s -XPUT "https://user:secret@endpoint/_token?expire=20" | jq .
{
"v": "tAbfpN-kmrxtRIV6JBgjUVA1ZDeLtypCx-x1MuIHSmdsA2NRDXCwtZmgzrV5t_TBEByXUh-nXhZt8Mvof2YP9VkL9KJmg7X-2U2DzHZ31GJ9ZW3CxgJMcaYHrC0v689RA25fusceIsBJSGoN4ggupeqp",
"s": "ok"
}

Generates a token without a path requirement.

An expiration of 0 is no expiration.

# Appendix 1: A Note on Xirsys Cluster Management

Xirsys Cluster Management (XCM) is still a work in progress, but currently manages 4 clusters and 20 hosts for Xirsys. 

XCM is designed for simple ops management by a small team. Thus there are 2 requirements, docker requires to be installed on all machines participating, and a single CouchDB instance is required to tie all hosts together. So there are 2 moving parts. Usefully, Cloudant can be utilized instead of your own CouchDB installation which is a very useful option, XCM can also download and boot a CouchDB instance off of hub.docker.io as an alternative.

We like to think of XCM as a cloud based application server, as it’s easily deployed, and integrated directly into a typical Git based development workflow, removing for the most part (depending on application) the "docker build" phase from the development cycle.

Running in a container itself, XCM is easily deployed to the cloud, intranet or desktop for local development. Once XCM is installed, and hosts are available it can bootstrap the entire Xirsys Brand Turn services and required database from a simple text based menu system (webapp gui is in progress)

Based on our cellular model, described in the introduction, docker-machine is utilised to manage and create hosts, then we build stems (runtime environments) and then differentiate those stems to specific cell types by injecting code directly from Git(Hub).

Of course all meta data for host allocation is stored in KV4, which XCM directly accesses.

To be clear, stems can be any runtime environment; node.js, .net core, python. Xirsys Turn brand happens to be entirely Elixir/Erlang, but XCM can run node.js apps just as well.

# Appendix 2: A Note of the Role of CouchDB

CouchDB is the only external server required by the entire Xirsys system.

It has 2 main roles, obviously as a data store, but more importantly, it’s change feeds are used to bind the system together via Neuron caching.

Every neuron has an (erlang ets based) cache, which caches all data referenced via that neuron from client applications. When a kv4 reference is accessed (path/layer/key) the data is loaded and then kept up to date automatically in the cache by the change feeds. Thus if any other node changes that reference not just the data in Couchdb but all caches are updated automatically. Sequences ids are followed so that if a change feed is dropped for any reason it can restart and retrace from the correct sequence to get all the changes missed.

This method provides distributed fast access to cached data for all neurons.

Note, only neurons (1 per host) get a change feed, not client applications. Client applications access the neuron, via the REST API/websockets etc. Client applications will not access CouchDB directly, although it could be for replication and other ops scenarios.

That this method is so useful, the base system itself uses it for system metadata, subscriptions and so on.

So, a Xirsys system is defined by the nodes that the underlying CouchDB manages. You could run multiple Xirsys systems in parallel using multiple CouchDBs.

CouchDB’s may also be setup to replicate over geographical regions.

