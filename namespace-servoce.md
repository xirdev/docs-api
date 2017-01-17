## The Namespace Service \(\_ns\)

In the V2.0 API the concept of /domain/application/room exists. In V3.0 API these are generalized to multi segment paths, and go by the general term "namespaces". They are not called specifically "channels" as these paths can be used by many services; they are in fact a hint to services of the paths that the user wants to expose publicly. Thus, they are not tied to simply signals \(and hence channel nomenclature\).

However, in the context of signals and turn as known by V2, the namepaces are exactly equivalent to /domain/application/room.

You are free to create namespaces to any depth with the V3 api.

### Create a new namespace

To create a new namespace simply PUT a path to the \_ns service.

```
TEST: namespace create should return ok
```

For convenience you don't need to put an object, just the path is required. Http PUT is used as you're creating something new.

### Delete a namespace

```
TEST: namespace delete should delete the test user
```

### Listing Namespaces

```
TEST: namespace list should return 1 item
```