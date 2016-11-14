# Host Service

_host provides details about servers currently operating in the system. 

Currently you can query turn and signals hosts.

`curl -s "https://async:secret@endpoint/_host/best/signal" | jq .                                                                                                                 {"v": "wss://euro3.xirsys.com:4005/ws","s": "ok"}`

/_host/best provides the best load balanced signals machine from the current cluster. This is equivalent to the APIV2 /wsList.

Similarly for _hosts/best/turn.