# Beta Introduction

Xirsys 3.0 introduces a completely rearchitected Xirsys platform, based around Docker! The new platform is a general purpose cloud application server, which happens to do TURN.

Through an extensible REST API Xirsys 3.0 provides custom analytics and data storage for customers. Xirsys 3.0 still supports the V2 API.

This document describes the V3 API.

# Service Overview

Xirsys is based around a dynamically expanding set of services. Basic services include; accounts, namespace, data, stats, and authorization. Extended services include; turn and video processing, with more to come.

Each service manages a layer of the KV4 tree and will create or recognise various paths \(entities\) in the tree that are significant to it. **The service name is the data layer name.**

A service provides some entity which is identified by path and key, .e.g,

* The account service recognizes "/\_acc/accounts?k=myident"

* The subscriptions service recognizes "/\_subs/domain/app/room/topic?k=chatter"

The API is "soft" in that a new service may be added which may interact with other data in the system - typically for the same path.

**Internal** services are those contained within the neuron itself. Currently recognised internal services are …

token and host services are virtual, they don't correspond to any persistent KV4 layer.

Xirsys also supports services external to the neuron, like turn and video processing; Although they run on the same host as the neuron, they communicate only via the public APIs. **In fact, external services may be arbitrary, server side apps.**

This model then, allows a base set of Xirsys system hosts that fulfill infrastructure requirements yet allows clean integration of external service hosts and their data.

External services, require to be registered with the base system which then proxies requests, after authorization etc, to the "best" external service host based on load/location etc. This requires that prefix names \(e.g. \_turn etc \) are always unique.

Subsequent versions of Xirsys will come equipped with a \_service service where new external services can be added dynamically.

# The Shape of the Service API

Using the Xirsys HTTP API, you can GET/PUT/POST/DELETE to any service using the following syntax \(depending on applicability\). The basic shape of all the commands is the same, and references to keys, layers are consistent, there are however parameter variations when required.

Note, your username and secret are required. All your data resides in it's own database \(bucket\) within the Xirsys system.

## GET

A value for a specific key ...

```
curl "https://user:secret@endpoint/_servicename/my/path?k=key"
```

All keys at node

```
curl "https://user:secret@endpoint/_servicename/my/path"
```

## POST/PUT

```
curl -XPOST -H 'Content-type: application/json' "https://user:secret@endpoint/_servicename/my/path" -d '{"k": "mykey","v": "myvalue"}'
```

You typically PUT to create new entities, and POST to update, e.g. you may create and update accounts respectively.

## DELETE

Delete a key.

```
curl -XDELETE "https://user:secret@endpoint/_servicename/my/path?k=key"
```

Delete all keys at path.

```
curl -XDELETE "https://user:secret@endpoint/_servicename/my/path"
```

**Note, that the prefix \(\_servicename\) used in the API is the layer that the data resides in.**

Again this is the general shape of the API, now please check the specifics for each service.

