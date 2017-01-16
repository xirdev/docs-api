# The Data Service

The data service provides for the creation of user data in the data layer.

## Create An Arbitrary Key/Value

```
TEST: data put 1 should add user data at path
```

## Delete

### Delete The Key

```
TEST: data delete should delete 1 element
```

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

[timestamp,whoperformedoperation,value]
