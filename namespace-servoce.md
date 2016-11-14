# The Namespace Service

Each service can create it's own hierarchy, but typically services will interact around the hierarchy created and managed by the Xirsys account holder via the namespace service.

The namespace is effectively the public tree of the user and provides backwards compatibility with previous versions of Xirsys, previously the namespace were called channels, but now could be used in a variety of scenarios, so "namspace" is more generic.

The namespace is a hint to all services, that the user has a specific hierarchy of names they want to work with, it is then up to the service if that namespace can be used in its context - if so, better, as it provides more context for the user.

**Note: In the current version of Xirsys there is no "referential integrity" between layers, so paths in one layer are not tied to paths in others - only by convention.
**
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

curl -s "https://user:secret@endpoint/_subs/richie.com/default/default/topic" | jq .{"v": ["rt"],"s": "ok"}

Or more detailed subscription information of you request the values

curl -s "https://user:secret@endpoint/_subs/richie.com/default/default/topic?as=values" | jq .{ "v": [ { "topic": "topic", "service": "xs_turn", "path": "/richie.com/default/default", "origin": "euro_euro-core", "meta": null, "key": "rt", "ident": "user" } ], "s": "ok"}Probably the most interesting piece of data there is the origin server that the subscription logged in from.

## Get specific subscription data.

curl "https://user:secret@endpoint/_subs/my/path/topic?k=chatter

# The Data Service

The data service provides for the creation of user data on the tree in the data layer.

## Create

curl -XPOST -H 'Content-type: application/json' "http://user:secret@endpoint/_data/my/path" -d '{"k": "mykey","v": "myvalue"}'

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

curl -s "http://user:secret@endpoint/_data/my/path?k=mykey&time_series=1&gs=2016:3:16:3:4&ge=2016:3:16:3:20" | jq . { "v": [ [ 1458109254, "user", "myvalue" ], [ 1458109242, "user", "myvalue" ] ], "s": "ok"}
