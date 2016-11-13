# The Virtual System Layer

A VSL is a base runtime which knows how to talk to other nodes in the cluster, and full access to up to date and cached KV4 data.

This property means that a single node can be the entire system.

We only need 1 node in a cluster to provide all **base** services.

The base services offered by the current VSL are:

1. KV4 Access
2. Websockets
2. HTTP API
3. Dashboard
4. Node Bus - Pubsub between VSLs as they come on/off line. VSLs know about each other.
4. High level abstractions for Accounts, Sessions, Configurations, Logging, Subscriptions.

Other services, e.g. turn, are treated as external services.

Each node is also redundant and offers the base system functionality. Thus each node is a system.

## Source Layout

| MicroApp | Purpose                                              |
| ---------| -----------------------------------------------------|
| xs_core  | Utilities                                            |
| xs_kv4   | KV4 implementation and base api                      |
| xs_data  | High level data abstractions on top of KV4           |
| xs_sys   | Composition of abstractions/signaling into a runtime |
| xs_end   | Endpoints into the runtime                           |
| xs_dash  | Xirsys Dashboard                                     |
| xs_vsl   | Main running app                                     |

