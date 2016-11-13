# Xirsys API

These docs are for internal purposes only and contain some information that we probably don't generally want diseminated, e.g. the $sys user.
  
## Background: KV4, a 4D K/V Data Model

The Xirsys API provides a comprehensive restful API into all facets of Xirsys.

Xirsys is based around KV4 our 4 dimensional data model, where data is indexed primarily by 3 coordinates [path,key,layer]. The 4th dimension is time, each 'put' to a [p,k,l] is recorded with timestamp and the entire time-series on a given key may be retrieved and queried when required.

You may think of the KV4 data space as a hash table at each tree node, and then a projection of the entire tree into the 3rd dimension with the layer. In addition to the primary key, secondary indexes may also be contructed per path/layer.

Internally  Xirsys uses the same data model,

The Xirsys system allows the user to create a public tree structure and then various services add layers in the tree for their own data. Thus, there  is an intersection of entity (path/key) and service (layer). The namespace service has a special importance as it allows the creation of the public tree.

## Scaling and Synchronization

KV4 and Xirsys application nodes can be deployed over multiple synchronized nodes over multiple clusters each PUT to any service is replicated to every node in the entire group of machines. To have a node participate in a "group", the node simply has to have a reference to a participating CouchDB node.
  
## Service Overview

Xirsys is based around a dynamically expanding set of services. Basic services include; accounts, namespace, data, stats, and authorization. Extended services include; turn and video processing, with more to come.

Each service manages a layer of the KV4 tree and will create or recognise various paths (entities) in the tree that are significant to it.

A service provides some entity which is identified by path and key, .e.g, 

- account recognizes "/accounts?k=myident" 
- subscriptions recognizes "/domain/app/room/topic?k=chatter"

The API is "soft" in that a new service may be added which may interact with other data in the system - typically for the same entity.

Currently recognised _internal_ services are ...
  
  
| Service       | Prefix | Layer  |
|---------------|--------|--------|
| accounts      | _acc   | *acc   |
| namespace     | _ns    | *ns    |
| subscriptions | _subs  | *subs  |
| authorization | _auth  | *auth  |
| stats         | _stats | *stats |
| user data     | _data  | *data  |
| host          | _host  | n/a    |
| token         | _token | n/a    |
| discovery     | _ds    | *ds    |


token and host services are virtual, they don't correspond to any persistent KV4 layer.

Xirsys also supports _external_ services, like turn and video processing;  external in the sense that they talk to the core system hosts using this API,  but they manage their own data layers in the tree.

| External Service | Prefix | Layer  |
|------------------|--------|--------|
| turn             | _turn  | *turn  |
| video            | _video | *video |

This model then, allows a base set of Xirsys system hosts that fullfill infrastructure requirements and then fully integrated external service hosts, may participate and add to the data model in clean way.

Note that new services can be introduced into this api dynamically, simply by booting up and signaling their presense on the host bus. The system will then select the desired service host based on host load when requested.

Using the Xirsys HTTP API, you can GET/PUT/POST/DELETE to any service using the following syntax (depending on applicability). The basic shape of all the commands is the same, and references to keys, layers etc are consistent, there are however parameter variations when required.

### GET
    
A value for a specific key ...

```
curl "http://user:secret@endpoint.xirsys.com/_servicename/my/path?k=key"
```

All keys at node

```
curl  "http://user:secret@endpoint.xirsys.com/_servicename/my/path"
```


### POST/PUT


```
curl -XPOST -H 'Content-type: application/json'  "http://user:secret@endpoint.xirsys.com/_servicename/my/path" -d '{"k": "mykey","v": "myvalue"}'
```

You typically PUT to create new entities, and POST to update, e.g. you may create and update accounts respectively.


### DELETE

Delete a key.

```
curl  -XDELETE "http://user:secret@endpoint.xirsys.com/_servicename/my/path?k=key"
```


Delete all keys at path.

```
curl  -XDELETE "http://user:secret@endpoint.xirsys.com/_servicename/my/path"
```

### Discovery

An expanding set of services requires a means of discovering them. For this purpose, Xirsys provides the _ds service.

Typically you always want to know which layers are available first:

```
[ritchie@mini ~]$ xirsys get /_ds/
{
  "v": [
    "*acc",
    "*logs:debug",
    "*logs:info",
    "*logs:warn",
    "*stats:cache:hit",
    "*stats:cache:miss",
    "*stats:cache:write",
    "*stats:router:pkt",
    "*stats:router:sub",
    "*subs:local_mini",
    "default"
  ],
  "s": "ok"
}

```

This is the output for the $sys user, normal users won't get the same set of layers, which is exactly the point. Note, all layers for the given user are presented.

Once you have the layers, you want to find the paths within each layer, once you have the paths you can then query the layer/paths for keys.

For example, now that I know I have the *acc layer, I can use _ds to query for it's paths

```
[ritchie@mini ~]$ xirsys get "/_ds?a=paths&l=*acc"
{
  "v": [
    "/accounts"
  ],
  "s": "ok"
}

```

As you can see there is one path in the layer *acc.

Similarly, for the *stats:router:pkt

```
[ritchie@mini ~]$ xirsys get "/_ds?a=paths&l=*stats:router:pkt"
Using creds: $sys:secret
{
  "v": [
    "/runtime/hosts/active"
  ],
  "s": "ok"
}
```

There is a path which contains the packet info for /runtime/hosts/active, remember the path can contain multiple keys. In this instance every machine on the Xirsys network is recording it's packet usage to this path, keyed on the node id.

I'm running my Xirsys test environment with 1 host, contributing to the *stats:router:pkt layer

```
[ritchie@mini ~]$ xirsys get "/_ds/runtime/hosts/active?a=keys&l=*stats:router:pkt"
{
  "v": [
    "mini:xs_vsl"
  ],
  "s": "ok"
}
```

Similarly, for the *subs layer. *subs:local_mini is the subscription layer for my local machine, there is currently 1 subscription




# The Namespace Service

Each service can create it's own hierarchy, but typically services will interact around the hierarchy created and managed by the Xirsys account holder via the namespace service.

The namespace is effectively the public tree of the user and provides backwards compatibility with previous versions of Xirsys.

## Create
   
### Create a new path in the public tree ...

```
curl -X PUT "https://async:secret@euro-service.xirsys.com/_ns/my/path"
```


## Delete
   
   
### Deletes path ...

```
curl -X DELETE "https://async:secret@euro-service.xirsys.com/_ns/my/path"
```

Note: This deletes all paths under the provided.

## Get
### All tree nodes under the specified path for the _ns service ...

```
curl "https://async:secret@euro-service.xirsys.com/_ns/my/path"
```

### All nodes to the given depth under the current path ...

```
curl "https://async:secret@euro-service.xirsys.com/_ns/my/path?depth=2"
```

### To list your entire namespace to segment depth 10 ...

```
curl "https://async:secret@euro-service.xirsys.com/_ns/depth=10
```

## Namespace and other Services

It's important to note that although _ns shapes the public tree, services aren't bound to it.

So for example, with the _data service you could create an entirely different tree of data to the ns tree, however, for sanity, overlap is advised.
   
   
# The Subscription Services

When a chatter or turn client creates peristent connection to Xirsys the information for that subscription is stored under the path/topic of interest. 

Historically this meeting point between users was denoted by the path /domain/application/room. This is no longer a restriction, but the legacy api defaults to the 3 segment path.

## Topics

Topics are extra classification that are created specifically by the _subs service.

The user may have constructed a path /my/path using the namespace service, but the _subs service provides more granularity, by extending topic nodes from the existing path /my/path, e.g.

   - /my/path/topic1
   - /my/path/topic2

Each topic node then contains subscription keys/values of chatters.

This data is stored in the #subs layer so there is no overlap with the base namespace, thus the base namespace is shadowed. This is probably the best example of a service adorning the public namespace of the user with service specific information contained in a separate layer.

### V2 API (Legacy)

From an V2 API perspective the addition of a topic is a further classification of room to topics, e.g.

    /domain/application/room/topic

and for the V2 API the system defaults to a default topic of "topic".


## Create

You can't create subscriptions via HTTP at the moment, this is done using websockets typically.


## Delete

### Kicking A Chatter

Because a subscription is available via the _subs service it may be deleted on demand. When a subscription is deleted it's removed from the router automatically, so a DELETE to the subs service is the same as kicking an
existing user.

```
curl -XDELETE "https://async:secret@euro-service.xirsys.com/_subs/my/path/topic?k=chatter"
```


### Kicking a Room

```
curl -XDELETE "https://async:secret@euro-service.xirsys.com/_subs/my/path/topic"
```


## Banning

Please see Authorization service for banning.

## Get

### List Subscription keys for a given topic. 

This is a live list of active "chatters". Just the chatter keys

```
[ritchie@bdog ~]$ curl -s  "https://async:secret@euro-service.xirsys.com/_subs/richie.com/default/default/topic" | jq .
{
"v": [
"rt"
],
"s": "ok"
}
```

Or more detailed subscription infomation of you request the values

```
[ritchie@bdog ~]$ curl -s  "https://async:secret@euro-service.xirsys.com/_subs/richie.com/default/default/topic?as=values" | jq .
{
 "v": [
   {
     "topic": "topic",
     "service": "xs_turn",
     "path": "/richie.com/default/default",
     "origin": "euro_euro-core",
     "meta": null,
     "key": "rt",
     "ident": "async"
   }
 ],
 "s": "ok"
}

```

   Probably the most interesting piece of data there is the origin server that the subscription logged in from.

## Get specific subscription data.

```
curl  "https://async:secret@euro-service.xirsys.com/_subs/my/path/topic?k=chatter
```

# The Data Service

  The data service provides for the creation of user data on the tree in the
  data layer.

## Create

```
curl -XPOST -H 'Content-type: application/json'  "http://async:secret@euro-service.xirsys.com/_data/my/path" -d '{"k": "mykey","v": "myvalue"}'
```


## Delete

### Deletes a key in the #data layer

```
curl -X DELETE "https://async:secret@euro-service.xirsys.com/_data/my/path?k=key"
```


### Deletes path in the #data layer ...

   ```
   curl -X DELETE "https://async:secret@euro-service.xirsys.com/_data/my/path"
   ```
    
## Get

### Get a Value

```
curl ""https://async:secret@euro-service.xirsys.com/_data/my/path?k=key"
```

### Get list of keys

```
curl ""https://async:secret@euro-service.xirsys.com/_data/my/path"
```

### Get timeseries for a key

```
curl ""https://async:secret@euro-service.xirsys.com/_data/my/path?time_series=1"
```

Timeseries may be queried using the same syntax for stats queries, where gs/ge are group start/group end resp, and broken into year:month:day:hour:minute

```
curl -s  "http://async:secret@euro-service.xirsys.com/_data/my/path?k=mykey&time_series=1&gs=2016:3:16:3:4&ge=2016:3:16:3:20"  | jq . 
{
 "v": [
   [
     1458109254,
     "async",
     "myvalue"
   ],
   [
     1458109242,
     "async",
     "myvalue"
   ]
 ],
 "s": "ok"
}
```

# The Stats Service

The Stats service provides a unified interface for services to take advantage of recording values against measurements, and then aggregating those measurements over various time periods to find sum/max/min and count stats.

The Stats service uses the path/key/layer construct of KV4 in a slightly different manner to the other services. Typically services intersect path/layer as entity/service, however, it's best to think of stats as intersecting path/layer as entity/measurement where the key is the user who is accumulating the data.

Given this intersection, the time series feature of KV4 comes into play. Each value put to an entity/measurement/key (path,layer,key) is always retained.

  - An entity is any useful object a service wants to accumulate against. (path)
  - A measurement is some measurement against the object (layer).
  - Key is the user/entity accumulating the stat.

An important point here is an entity is something in the system that could have _multiple_ measurements against, e.g. a user, or a host, a path.

Important entities for the user are typically the paths created in their public hierarchy, as this is where connections for turn and signals are created. Thus these are good candidate entities for recording measurements
against. The system provides the following measurements for paths, new services will add new measurements.

| measurement | entity          | description                                           |
|-------------|-----------------|-------------------------------------------------------|
| router:sub  | namespace path  | a subscription to the router                          |
| router:pkt  | namsepace path  | the number/size of packets passing through the router |
| turn:sess   | namespace path  | turn sessions created                                 |
| turn:data   | namespace path  | turn data transferred                                 |
| stun:sess   | namespace path  | stun sessions created                                 |
| stun:data   | namespace path  | stun data transferred                                 |
| api:3       | /service/METHOD | get api call statistics per method and service        |


For example, here is a measure of router packets on the system bus, the entity /runtime/hosts/active is only available to the sys user, but substitute your own creds and path in to retrieve the stats for the packets transferred over the given path/topic (i.e. the measurement (router:pkt) can be reused over multiple entities).

```
curl -s  "http://%24sys:secret@localhost:4006/_stats/runtime/hosts/active?l=router:pkt"  | jq .
{
"v": {
"sum": 30561,
"min": 48,
"max": 51,
"count": 604
},
"s": "ok"
}
```

The interpretation is; there have been 604 packets published by the router(s) on the /runtime/host/active channel; the accumulated size of the packets was 30561 bytes, the smallest of which was 48 bytes, the largest of which was 51 bytes.


## The Stats Call ....

```
curl "https://async:secret@euro-service.xirsys.com/_stats/my/path?l=measurement&gs=y:m:d:h:min&ge=y:m:d:h:min&break=1"
```


The measurement is provided using the l (layer) parameter. This is actually, very precise, as the kv4 layer is used to record the measurement axis.

Note: Typically you will be measuring against user defined namespace entities, as this is the historical use of the system, but other entities that various services find useful could be exposed. So, the stats entity (path) is not limited to namespace paths.

The gs and ge params are group start and group end respectively. Although typically used for dates, I've refrained from "ds" and "de" and a date syntax, as this query mechanism is not restricted to dates.

You may enter any level of the grouping, e.g. 

```
gs=2016:2&ge=2016:3
```

and be provided with the sum/count/max/min of the measurement over (in this instance) the given time period, at the level of the grouping. So, you get a single object back for the sum/count/max/min of the measurement over the time period.

However, if you add the break=1 option

```
gs=2016:2&ge=2016:3&break=1
```

You'll get a list of two items over the given period, one for February and 1 for March.

If you take you're grouping down a level,

```
gs=2016:2:4&ge=2016:3:10&break=1
```

You'll get a list of sum/count/max/min for each day, or without break=1 the totals between the periods given.

You may take this to the minute level.
    
Note: Stats are now easy to add and the interface to them is uniform so if you think of other useful stats pls let me know.
  
Note: Stats are accumlated and posted every 15 secs to the database, so you won't get instant response_

## Limitations

Stats is currently limited in it's breakdowns as the underlying database engine only provides 1 set of group breakdowns at a time, and currently we've plumped for grouping by time to the minute level. For example, being able to break down by layer tags, and partial paths, simultaneous to time, would totally rock, but this can't be achieved efficiently atm.

A future feature will be introspection that report which entities/measurements are available.

## Create

You may not create stats at this time. This restriction will be removed in the future.


## Delete

You may not delete stats.


## Examples

### Router Packets

```
curl -s  "http://%24sys:secret@localhost:4006/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:3&ge=2016:3:16:6&break=1"  | jq .
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
```

Here we've added grouping to the hour level, the query was between 3 - 6 hours, but the only data was for the 5th hour.

Let's break down the 5th hour packet and find stats over the first 10 mins of the hour ...


```
curl -s  "http://%24sys:secret@localhost:4006/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:5:0&ge=2016:3:16:5:10"  | jq .
{
"v": {
  "sum": 560,
  "min": 50,
  "max": 51,
  "count": 11
},
"s": "ok"
}
```

There were 11 packets in the first 10 mins of the 5 hour in total, but we can break that down further into which minutes ...

```
curl -s  "http://%24sys:secret@localhost:4006/_stats/runtime/hosts/active?l=router:pkt&gs=2016:3:16:5:0&ge=2016:3:16:5:10&break=1"  | jq .
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
```


### API Calls


api:3 is a good example of a measurement, measured against some entity not created by namespace, but rather, stats creates the path /service/METHOD for you to query against, e.g. how many GETs have I done against the data
service?
  
```
curl -s "http://async:secret@localhost:4006/_stats/data/GET?k=async&l=api:3"  |jq .
{
"v": {
  "sum": 5,
  "min": 1,
  "max": 1,
  "count": 5
  },
  "s": "ok"
}
```
    
or how many calls against the stats service ...

```
curl -s "http://async:secret@localhost:4006/_stats/stats/GET?k=async&l=api:3"  |jq .
{
 "v": {
   "sum": 23,
   "min": 1,
   "max": 1,
   "count": 23
 },
 "s": "ok"
 }
```
   
which increases by 1 every time I call it.

```
curl -s "http://async:secret@localhost:4006/_stats/?l=api:3&gs=2016:3:6&ge=2016:3:8&break=0"  |jq .                                                                                       
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
```

In this example, only the measurement is present (l=api:3), no key or path exists thus we get a full breakdown of calls over api:v3, totals calls by hour.

However we can be more specific and ask for the GETS to a specific service

....

Notice the key group has extended in the results to include the hour of the /data/GET

Finally you may add a key, the key will perform the breakdown for a given user. Typically the user is you. However, if you have added admins into your system then you can retrieve usage for each.

    
# The Accounts Service

  The accounts service provides 2 functions:

  1. New logins to the base Xirsys account, that may be subsquently used in API
     interactions, i.e. other admin credentials. All facilities on your account
     are available to the new admin, and each change to the data is stamped with
     the admins credentials. You can see changes to data by using the
     time_series=1 option on most GET calls.

  2. You may create full Xirsys sub-accounts, that are manipulated via the
     standard API. 

## Administrators
     
### Create an Admin

   Post a login name and password to /users/admins, for an administrator
   login. 

```
curl -H "Content-type: application/json" "http://async:secret@euro-service.xirsys.com/_acc/users/admins" -d '{"k": "key", "v": {"password": "password"}}'
```

### Delete an Admin
```
curl -X DELETE "http://async:secret@euro-service.xirsys.com/_acc/users/admins?k=key"
```

### List Admins

```
curl "http://async:secret@euro-service.xirsys.com/_acc/users/admins"
```

### Get Admin

```
curl "http://async:secret@euro-service.xirsys.com/_acc/users/admins?k=key"
```

### Time Series

```
curl "http://async:secret@euro-service.xirsys.com/_acc/users/admins?k=key&time_series=1"
```

### Usage
    
    
To login as an administrator, just use the following syntax in the login credentials.

```
curl "https://user:admin_key#admin_password@euro-service.xirsys.com/_data/my/path"
```

Notice that the secret is now loaded with 2 fields of information separated by a "#".

We need to have the main Xirsys account holder username (ident), and once we have that we can authenticate the admin within that context.

Currently all admins have the same rights, role based permissions will be available at some stage.

Note: Time series are returned with the time, login and value, so you can see who made changes to the data.
    
## Account Proxy

The _acc service allows the creation of full Xirsys sub-accounts that you may manage using the standard API.

This gives you fully self contained stats and services for your sub-accounts. For example, a sub-account could require it's own propietary namespace.

### Create 

Simply PUT to the _acc services /accounts path to add a new sub-account.

```
curl -H "Content-type: application/json" "http://async:secret@euro-service.xirsys.com/_acc/accounts" -d '{"k": "username", "v": {"password": "password","email": "email","first_name": "first_name","last_name": "last_name"}}'
```

### Delete

This will delete the sub-account and all it's data. 

Note: Sub-account data is completely self contained in it's own 'table', the entire 'table' is removed with this action.

```
curl -XDELETE "http://async:secret@euro-service.xirsys.com/_acc/accounts?k=username'
```

### Get

To get the details of a sub-account ...

```
curl "http://async:secret@euro-service.xirsys.com/_acc/accounts?k=username'
```
    

### Usage

Usage of sub-accounts is driven entirely through the overloading of the authorization headers. This allows the API to remain consistent, and to allow Xirsys V2 services to use the feature without extension.

The authorization of a sub-account must take place in the context of the primary Xirsys account holder (you), thus the primary username must always be present. The authorization header looks like this

primary_username#subaccount_username:subaccount_secret

Or more easily remembered, you must prefix primary_username# to any sub-account call.

For example to add a namespace to the sub-account

```
curl -s -XPUT "https://primary_username#subacount_username:subaccount_secret@euro-service.xirsys.com/_ns/my/path"
```

Effectively this syntax redirects the call to a different database 'table', and this is consistent across all services.

For primary accounts executing requests for sub-accounts looking up a sub-account secret is not efficient, so primary account holders may authenticate with their own secret.

To indicate primary account holder secret prefix the secret with "!".

```
curl -s -XPUT "https://primary_username#subacount_username:!primaryaccount_secret@euro-service.xirsys.com/_ns/my/path"
```

## Walkthrough

Here's a walk thru of using accounts, which includes creation of a top level account,via the sys user; I've included that so that you can see how your account creation is exactly the same as the system's

Here's the creation of a top level primary account by sys...

```
curl -s -XPUT -H 'Content-type: application/json' "http://%24sys:secret@localhost:4006/_acc/accounts" -d '{"k": "elena", "v": {"email": "elena@async.cl","password": "woot"}}'
{"v":{"username":"elena","term":"monthly","secret":"8af2a710-eba7-11e5-b4a0-9cdd20b4c6b5","plan":"free","password":"woot","last_name":null,"first_name":null,"email":"elena@async.cl","customer_id":null,"created":1458154052,"company":null,"active":true},"s":"ok"}
```

Notes:
   
   - the key is the "ident", added to the body as "username" automatically
   - email is required, and is added as a secondary index for the path
     "/accounts"
   - active is true, controls if the user may login in or not

So, now, Elena is a top level account holder, and sys can query for her in 2 ways by ident(key), and by email, like this ...

```
 curl -s  "http://%24sys:secret@localhost:4006/_acc/accounts?k=elena" | jq .
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
```

or because a secondary key was added automatically on email ...

```
curl -s  "http://%24sys:secret@localhost:4006/_acc/accounts?k2=elena@async.cl" | jq .
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
```

Notice we're querying on k2.

### Aside: Secondary Keys

KV4 has 4 keys in total you can use to index into your objects _per_path_;k, k2, k3 and k4. So, accounts objects in /accounts could be looked up in 4 different ways, but the same is true for any other path you could create.
These 4 keys are also integrated into the caching system so GETs on any of these are fast. 4 keys is not a hard limit, so more work could go into this to get more out of it. 4 keys was chosen as KV4 should have 4 indexes, we
can add a fifth for kv5 :).

Please see Secondary Keys for more info on creation (TBD).

### Primary Account Holder Creates Secondary Account

So, account holder Elena wishes to create a secondary account. She can do so by PUTing to her /accounts, like this ...

```
curl -s -XPUT -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@localhost:4006/_acc/accounts" -d '{"k": "lily", "v": {"email": "lily@async.cl","password": "woot2"}}' | jq .
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
```

So Elena PUTs sub account lily to her /accounts path in the _acc service. Notice the complete symmetry between the sys user putting a new account and Elena.

### Aside: Storage Model

Elena's data is stored in CouchDB in the bucket kv4_$elena_data, the sys users data is stored as kv4_$sys_data. So CouchDB buckets are named by "ident", and contain all of the users data, whether it's stats, accounts,
subscriptions or namespace. The ice thing with this model is we can see at a glance customer usage, replicate the customer, place a customer in their own couch instance as a "sys" user. 

Further, the cache system caches per bucket, so each user effectively has their own cache per VSL, these caches are expired and cleaned up on no usage.

### Querying Sub Accounts

Just as sys would do to get a list of it's accounts, Elena may query her /accounts path,

```
 curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@localhost:4006/_acc/accounts" | jq .
 {
 "v": [
   "lily"
  ],
  "s": "ok"
}
```

so now we know lily is a sub account holder we can query her for more details ...

```
curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@localhost:4006/_acc/accounts?k=lily" | jq .
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
```

### Setting Sub Account Properties

You may alter sub account properties by POSTing new attribute values to the key. For example, let's say you want to change the "active" property ...


```
curl -s -XPOST -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@localhost:4006/_acc/accounts" -d '{"k": "lily", "v": {"active": false}}' | jq .
{
  "s": "ok"
}
```

and to check ...
    
```
curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@localhost:4006/_acc/accounts?k=lily" | jq .
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
```

You may post new properties to the object using this means too. The system (should not) does not allow you to change email or username at this time.


### Lily as a Stand Alone User

Now that Lily has been created by Elena, she may want to add new namespaces for herself, so the new Lily account is completely self contained. So this may seem reasonable ...

```
curl -s -XPUT "http://lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@localhost:4006/_ns/a/path/that/lily/created" | jq .
{
  "v": "unauthorized",
  "s": "error"
}
```
    
however, this would look Lily up in the top level accounts database (sys database), where it does not exist. The Xirsys top level database, only knows about Elena, not her secondary accounts. Thus, we need to perform the action in the context of Elena, so this seems reasonable ...

```
curl -s -XPUT "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@localhost:4006/_ns/a/path/that/lily/created" | jq .
{
  "v": "unauthorized",
  "s": "error"
}
```

This would typically work, however, above, I deactivated the account, and the activation is checked in all authorizations, so, let's fix that ...

```
curl -s -XPOST -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@localhost:4006/_acc/accounts" -d '{"k": "lily", "v": {"active": true}}' | jq .
{
  "s": "ok"
}
```

now ...
    
```
curl -s -XPUT "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@localhost:4006/_ns/a/path/that/lily/created" | jq .
{
"s": "ok"
}
```

and to check to the 5th level ...

```
curl -s "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@localhost:4006/_ns/?depth=5" | jq .
{
  "v": [
    "a/path/that/lily/created"
  ],
  "s": "ok"
}
```


### Aside: Storage Model

We've already seen that Elena as a top level user gets her own bucket kv4_$elena_data, where are secondary accounts data stored? When I created the namespace above where did it go?

Each secondary account, is a full account in the context of the primary user, so is stored in a new bucket, which has the following name format

```
kv4_$elena)lily_data
```

CouchDB has a limited number of characters we can use for bucket names so )is reasonable.

Again, this means that all namespace, subscriptions and so on are self contained for this user.

It's plausible with a flexible enough storage naming scheme that the system could continue to create recursive secondary accounts, but for sanity, 1 level deep is good enough for now.


### Usage - A Proxy Service.

The secondary account system purpose is to allow proxy usage of Xirsys. Elena can update her own accounts and data, but now she has a secondary account holder Lily, who she wants to create a Xirsys service for.

Thus a means for Elena to create namespaces, subscriptions and so on in the context of Lily is required.

Elena doesn't want to check Lily's secret each time, so she wants to do the authorization as herself with her secret, this may seem reasonable ...

```
curl -s -XPUT "http://elena#lily:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@localhost:4006/_ns/a/path/that/elena/created/for/lily" | jq .
{
"v": "unauthorized",
"s": "error"
}
```

Here we have Elena trying to add a path for Lily but with her own secret, and it fails. Obviously, as above, the system assumes, that the secret provided is the user's (Lily's) secret and not Elenas. So to indicate that
it's the primary account holders secret and not the secondary, we prefix secret with a "!", like so ...

```
curl -s -XPUT 'http://elena#lily:!9a3b4066-ec19-11e5-930a-2aee0ed40b9c@localhost:4006/_ns/a/path/that/elena/created/for/lily' | jq .
{
  "s": "ok"
}
```

Please note here, I changed the " to ' quotes for bash as it will do a substitution on the ! otherwise.

The data although authorized on Elena's account, does in fact go into Lily's data bucket. Here is Lily authorizing with Lily's secret, to find the new data deposited by the primary account holder ...

```
curl -s "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@localhost:4006/_ns/?depth=7" | jq .
{
  "v": [
    "a/path/that/elena/created/for/lily",
    "a/path/that/lily/created"
  ],
  "s": "ok"
}
```

All other services will authorize in the same way.

# Authorization Service

The authorization service allows you to manage IP addresses per node that will disallow connections from the given IP range.

## Deny

```
curl -XPUT -H "Content-type: application/json" "http://async:secret@euro-service.xirsys.com/_auth/my/path" -d '{"k": "IPADDRESS"}'
```

The value is unimportant, as we simply utilise the key here.

All services will now ignore connections from the given IP address, on the given path or below.

## Allow

```
curl -DELETE -H "http://async:secret@euro-service.xirsys.com/_auth/my/path?k=IPADDRESS'
```

This service is in it's infancy, and can provide a lot more going forward.

# Host Service

_host provides details about servers currently operating in the system. Currently you can query turn and signals hosts. 

```
[ritchie@bdog ~]$ curl -s "https://async:secret@euro-service.xirsys.com/_host/best/signal" | jq .                                                                                                                 
{
"v": "wss://euro3.xirsys.com:4005/ws",
"s": "ok"
}
```


/_host/best provides the best load balanced signals machine from the current cluster. This is equivalent to the APIV2 /wsList

Similarly for _hosts/best/turn.


# Token Service

_token provides a token for signals authentication.

You may create a token specifically for a path or a universal token. The V2 API requires a full path (/domain/application/room) while the V3 API requires just requires a token.

To add a chatter key add k=chatter_key.

```
[ritchie@bdog ~]$ curl -s -XPUT "https://async:secret@euro-service.xirsys.com/_token/my/path?k=chatter_key&expire=20" | jq .
{
"v": "ZG1enoNPMqSLf93auDFjMD84jTVRMsoor7nHHMGoG_4HsH3ogWlNUmMwIFekwe44FEWYoBQMBAFfiFluzOL2K6QZJ2Qc4H1-bCaYcYpuRB9Tvcqyv9l53RHG1SUHdL7tCEjxrDDFDMtDrRgs5A",
"s": "ok"
}
```

This example, generates a API V2 compatible token which includes the path specified, and only allows login to the specified path. It is equivalent to the V2 API /getToken url.

```
[ritchie@bdog ~]$ curl -s -XPUT "https://async:secret@euro-service.xirsys.com/_token?expire=20" | jq .
{
"v": "tAbfpN-kmrxtRIV6JBgjUVA1ZDeLtypCx-x1MuIHSmdsA2NRDXCwtZmgzrV5t_TBEByXUh-nXhZt8Mvof2YP9VkL9KJmg7X-2U2DzHZ31GJ9ZW3CxgJMcaYHrC0v689RA25fusceIsBJSGoN4ggupeqp",
"s": "ok"
}
```

Generates a token without a path requirement.

An expiration of 0 is no expiration.

