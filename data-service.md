# The Data Service

The data service provides for the creation of user data in the data layer.

## Create An Arbitrary Key/Value

```
TEST: data put 1 should add user data at path
```

## A Note on Locking

All Xirsys services return a version id \(_ver_\) when you create or get a value.

This can be ignored unless you're going to update the value. Xirsys employs **optimistic locking** to make sure you don't overwrite someone elses changes. When you update a value make sure you include the _ver_ in your updated object - you'll be warned if you forget with a **\_ver\_missing** error. If a new version of the object has been created since you received your version, you'll get a **version\_conflict** error returned. If someone is in process of updating the object,  then a **locked** error will be returned.

Although, you're required to supply the up to date version id, Xirsys, doesn't update in place. Instead we always save your old object, which you may retrieve via the time series. By never updating in place, Xirsys maintains an audit trail on all data.

## Delete

### Delete The Key

```
TEST: data delete should delete 1 element
```

On successful delete you receive an ok, with the numer of elements deleted. For a single key delete this is always 1. However, by omitting the k, you can do a full path delete.

```
TEST: data delete path
```

_Note: Paths are not stored indepently of keys, except for the \_ns service. Therefore if you delete all keys in path the path itself is removed._

### Get a Value

```
TEST: data get should get a value
```

### Get list of keys

```
TEST: data list should list two elements at same path
```

### Get timeseries for a key

```
curl "https://user:secret@endpoint/_data/my/path?time_series=1"
```

Time series may be queried using the same syntax for stats queries, where gs/ge are group start/group end respectively, and broken into year:month:day:hour:minute:second

```
TEST: data time series should return all action at this path
```

The array structure returned has the following structure

\[timestamp,whoperformedoperation,value\]

