# The Namespace Service

Each service can create it's own hierarchy, but typically services will interact around the hierarchy created and managed by the Xirsys account holder via the namespace service.

The namespace is effectively the public tree of the user and provides backwards compatibility with previous versions of Xirsys. Previously namespace were called channels, but can now be used in more scenarios, thus namespace is the more generic term.

The namespace is a hint to all services that the user has a specific hierarchy of names they want to work with, it is then up to the service if that namespace can be used in its context - if so, better, as it provides more context for the user.

**Note: In the current version of Xirsys there is no "referential integrity" between layers, so paths in one layer are not tied to paths in others - only by convention.
**
## Create a new path in the public tree …

```
curl -X PUT "https://user:secret@endpoint/_ns/my/path"
```
## Delete Path ...

```
curl -X DELETE "https://user:secret@endpoint/_ns/my/path"
```


Note: This deletes all paths under the provided.

## Get

All tree nodes under the specified path for the _ns service …

```
curl "https://user:secret@endpoint/_ns/my/path"
```


All nodes to the given depth under the current path …

```
curl "https://user:secret@endpoint/_ns/my/path?depth=2"
```


To list your entire namespace to segment depth 10 …

```
curl "https://user:secret@endpoint/_ns/depth=10
```