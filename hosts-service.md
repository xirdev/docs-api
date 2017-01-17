Host Service

```
https://ident:secret@endpoint/_host?type=signal&k=name
```

By default this request provides the least loaded host on the endpoint referenced.

**Example**

```
TEST: host query signals successfully
```

If you wish to find the best loaded server on the best located cluster between sender and recipient, then add the path and key

[https://ident:secret@endpoint/\\_host/my/path?k=ritchie&type=signal](https://ident:secret@endpoint/\_host/my/path?k=ritchie&type=signal)

what this does is lookup the peer's subscription by name in the given channel and finds their IP address, it does the same for the issuer of the call. It then finds the best located cluster to the midpoint of the IPs.