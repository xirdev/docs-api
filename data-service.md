# The Data Service

The data service provides for the creation of user data on the tree in the data layer.

## Create

```
TEST: data put 1 should add user data at path
```

## Delete

### Deletes a key in the data layer

```
TEST: data delete should delete 1 element
```

### Deletes path in the data layer

```

```

### Get a Value

```

```

### Get list of keys

```
TEST: data list should list two elements at same path
```

### Get timeseries for a key

```
curl "https://user:secret@endpoint/_data/my/path?time_series=1"
```

Time series may be queried using the same syntax for stats queries, where gs/ge are group start/group end resp, and broken into year:month:day:hour:minute

```
curl -s  "http://user:secret@endpoint/_data/my/path?k=mykey&time_series=1&gs=2016:3:16:3:4&ge=2016:3:16:3:20"  | jq . 

{
 "v": [
   [
     1458109254,
     "user",
     "myvalue"
   ],
   [
     1458109242,
     "user",
     "myvalue"
   ]
 ],
 "s": "ok"
}
```