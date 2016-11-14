# Authorization Service

The authorization service allows you to manage IP addresses per node that will disallow connections from the given IP range.

## Deny

`curl -XPUT -H "Content-type: application/json" "http://async:secret@endpoint/_auth/my/path" -d '{"k": "IPADDRESS"}'`

The value is unimportant, as we simply utilise the key here.

All services will now ignore connections from the given IP address, on the given path or below.

## Allow

`curl -DELETE -H "http://async:secret@endpoint/_auth/my/path?k=IPADDRESS'`

This service is in it's infancy, and can provide a lot more going forward.

# Host Service

_host provides details about servers currently operating in the system. Currently you can query turn and signals hosts.

`curl -s "https://async:secret@endpoint/_host/best/signal" | jq .                                                                                                                 {"v": "wss://euro3.xirsys.com:4005/ws","s": "ok"}`

/_host/best provides the best load balanced signals machine from the current cluster. This is equivalent to the APIV2 /wsList.

Similarly for _hosts/best/turn.