# **Token Service**


This provides a token which allows a given user to connect to our websocket signal servers on the specified path \(created by \_ns\)

The idea is you get a token for your user, who can then login directly with it, this is something that should be done by your server, so as not to expose your credentials.

```
PUT https://ident:secret@endpoint/_token/path/to/channel?k=user&expire=0
```

| Param | Notes |
| :--- | :--- |
| k | Provide a name for your user. This name will be used for display purposes, and referencing the user in the given "channel". Note, if you omit the k parameter, a random userid will be generated |
| expire | Expire the token ability to connect after X seconds. Omiting expire, expires the token after 60 seconds. If expire is 0, then token does not expire. |

Http PUT is used as you're creating something new. The namespace you create a token for must exist,

### Token Creation Failure

```
TEST: token create no namespace should return {error,no_namespace}
```

### Token Creation Success

```
TEST: token create namespace exists should return {ok,token}
```
