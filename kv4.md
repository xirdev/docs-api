# KV4

KV4 and it's understanding is essential to understand the system. Why 4D?

### Dimensional Progression

| Dimension | Description                                                                  |
|-----------|------------------------------------------------------------------------------|
| 1D        | Not much to say about this a hash table is a 1D k/v store                    |
| 2D        | Index a hash table by path. Each path has a hash at each segment junction.   |
| 3D        | Index a hash table by path / layer. Each segment/layer has a has a hash      |
| 4D        | An 3D hash hash where each K/V is a time series, that is, nothing is deleted.|


e.g. KV4 essentially boils down to ...

``` 
KV4.put("/accounts/","ritchie",{"last_name": "turner"},"default") 
KV4.get("/accounts","ritchie","default") => {"last_name": "turner"}

KV4.get_timeseries("/accounts","ritchie","default") => [{time_stamp,who,{"last_name": "turner"}]
```

## Scaling

KV4 is based upon CouchDB, and takes advantage of a fairly unique property of CouchDB, it's changes feed.

VSL's provide an up to date distributed view of any KV4 data from a "centralized" CouchDB. Centralized is in double quotes as CouchDB data may be replicated between data centers, so each regional cluster maintains a local CouchDB, which all local VSLs may talk to, but these CouchDBs are then replicated between data centers. This property of CouchDB is very well documented, and highly reliable.

Thus, a Xirsys platform cluster, consists of multiple VSL each of which maintains a dynamic cache of up to date KV4 key/values. The VSL manages it's cache consistency via the CouchDB changes feed.

After the first hit on the db (couchdb) a k/v is in the VSL cache. An Erlang supervisor managed connection to the change feed on the source of the data is created. This is a real-time feed, which is consumed by all VSL. Thus if any change is made to the db all caches, in all VSL are updated with great reliability. Because the entire system is based on this mechanism, we have scaleable and fast access from all nodes. Indeed, all VSL's are equal.

Keys are not loaded on a node if not requested on that node. However if a given key is requested it's guaranteed to be up to date automatically, without any polling by the client. You can thus within tight loops refer to DB keys knowing that you'll have the latest value even if it came from a different cluster. That is, new values are pushed transparently to nodes and cached without any intervention.

This single idea powers most of Xirsys functionality, and note that it is not tied to any given service, just to the data model.


