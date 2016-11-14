# The Subscription Service

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

curl -s  "https://user:secret@endpoint/_subs/richie.com/default/default/topic" | jq .{"v": ["rt"],"s": "ok"}

Or more detailed subscription information of you request the values

curl -s  "https://user:secret@endpoint/_subs/richie.com/default/default/topic?as=values" | jq .{ "v": [   {     "topic": "topic",     "service": "xs_turn",     "path": "/richie.com/default/default",     "origin": "euro_euro-core",     "meta": null,     "key": "rt",     "ident": "user"   } ], "s": "ok"}Probably the most interesting piece of data there is the origin server that the subscription logged in from.

## Get specific subscription data.

curl  "https://user:secret@endpoint/_subs/my/path/topic?k=chatter
