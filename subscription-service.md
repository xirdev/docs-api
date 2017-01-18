# The Subscription Service

When a chatter or turn client creates persistent connection to Xirsys the information for that subscription is stored under the path/topic of interest.

Historically this meeting point between users was denoted by the path /domain/application/room. This is no longer a restriction, but the legacy api enforces the the 3 segment path.

## Create

You can't create subscriptions via HTTP at the moment, this is done using websockets.

## Get

### List Subscription keys for a given topic.

This is a live list of active "chatters". Just the chatter keys

```
TEST: subscriptions list should return 1 item
```

Or more detailed subscription information of you request the values

```
TEST: subscriptions as values should return 1 item
```

Get specific subscription data.

```
TEST: subscriptions get single should get a single subscriber
```

## Delete

### Kicking A Chatter

Because a subscription is available via the \_subs service it may be deleted on demand. When a subscription is deleted it's removed from all neuron routers automatically, so a DELETE to the subs service is the same as kicking an existing user.

```
TEST: subscriptions kick should kick a single subscriber
```

### Kicking a Room

```
curl -XDELETE "https://user:secret@endpoint/_subs/my/path/
```

To remove all subscriptions for a room simply drop the key, and any keys on the path will be deleted.

## Banning

Please see Authorization service for banning.

