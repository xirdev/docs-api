# The Data Service

The data service provides for the creation of user data on the tree in the data layer.

## Create

`curl -XPOST -H 'Content-type: application/json'  "http://user:secret@endpoint/_data/my/path" -d '{"k": "mykey","v": "myvalue"}'`

## Delete

### Deletes a key in the data layer

`curl -X DELETE "https://user:secret@endpoint/_data/my/path?k=key"`

### Deletes path in the data layer

`curl -X DELETE "https://user:secret@endpoint/_data/my/path"`
### Get a Value

`curl "https://user:secret@endpoint/_data/my/path?k=key"`

### Get list of keys

`curl "https://user:secret@endpoint/_data/my/path"`

### Get timeseries for a key

`curl "https://user:secret@endpoint/_data/my/path?time_series=1"`

Time series may be queried using the same syntax for stats queries, where gs/ge are group start/group end resp, and broken into year:month:day:hour:minute

`curl -s  "http://user:secret@endpoint/_data/my/path?k=mykey&time_series=1&gs=2016:3:16:3:4&ge=2016:3:16:3:20"  | jq . 

{ "v": [   [     1458109254,     "user",     "myvalue"   ],   [     1458109242,     "user",     "myvalue"   ] ], "s": "ok"}`