# Introduction

XirSys is known as a world leader in TURN server provision. However, the underlying platform, and XirSys long term focus, is not solely linked to WebRTC and TURN.

The XirSys platform consists of an Enterprise level application development and distribution/clustering framework based on Docker. All core functions are written in Elixir/Erlang.

 We use a cellular analogy to describe the system:

* *Stems *are containers which provide a specific runtime, e.g. Erlang, Node.js, .NET Core etc.* *Cells* are applications which are loaded from a Git Repository (Github) into a stem to provide a differentiated cell (container) with a specific function.

* *Neurons* are the base Xirsys functionality carried by all nodes in a Xirsys network.

* *Memory, *we provide an integral 4D data store accessible via REST/Websockets/TCP (tbd).

For example, our Turn service is an Elixir application, loaded into an Elixir Stem, in real time, from Github. This is an additional take on typical Docker container management which deals with images only. We use images to create a runtime environment then inject application code dynamically into the Stem.

This provides a very natural development cycle for testing and deploying new code.

1. Push To Github
2. Differentiate to Stems
3. Launch on Stems

For script language (JS) work this is very desirable. **Heavy up front compilation environments can still use the typical Docker repo services, and build entire containers before deployment.

Currently we use this method to deploy applications across 5 clusters and 20 hosts.

The Xirsys platform is very easily installed on a cluster, and a fully running system can be installed locally for development purposes.

# Neurons

All Xirsys nodes are Neurons. The Neuron contains the coordination primitives of the system; Data, Api, and Websocket support. All Neurons are equal and are aware of each other. If a neuron fails to ping the others on a regular basis the others tidy up connections/data belonging to the failed neuron on their host.

All other applications (cells), e.g. Turn, are clients to their local Neurons, that is, Turn containers and Neuron containers run on the same host. Turn containers use the REST api to talk to the local Neuron.

By this method, any server side app can be a client to it’s local Neuron, and have fast local access to data (more later), via http,websocket or **TCP (tbd)**.

## Data Persistence

Shared data is stored externally to neurons but pervasively cached in each neuron, in what we call KV4 - a 4 dimensional key/value store

All neurons contain materialized views of cached data based on change feeds from a central CouchDB database. This means that once data is loaded into the cache it maintains the latest value as set from any other neuron in the network. If a Neuron updates/deletes a key in CouchDB it’s update/deleted from all other Neuron caches via the CouchDB change feed.

Everything in the system is based on this mechanism, and is the basis of data distribution over multiple nodes. KV4 provides distributed Etcd like hierarchical services, categorized by path/layer/key/time (4d)

## Messaging

Each Neuron knows about all users attached to a given channel (path) via Websockets. As subscriptions are stored by path in KV4 all neurons know where a user/device is logged in and where to message it. Each neuron then acts as a router to a given channel, and may send messages directly to the node where the peer is logged in. Thus there is no central router - as mentioned all neurons are equal - they all contain cached subscription data updated reliably from CouchDB change feed. Interneuron messaging is secure HTTPS.

# Appendix 1: A Note on Xirsys Cluster Management

Xirsys Cluster Management (XCM) is still a work in progress, but currently manages 4 clusters and 20 hosts for Xirsys.

XCM is designed for simple ops management by a small team. Thus there are 2 requirements, docker requires to be installed on all machines participating, and a single CouchDB instance is required to tie all hosts together. So there are 2 moving parts. Usefully, Cloudant can be utilized instead of your own CouchDB installation which is a very useful option, XCM can also download and boot a CouchDB instance off of hub.docker.io as an alternative.

We like to think of XCM as a cloud based application server, as it’s easily deployed, and integrated directly into a typical Git based development workflow, removing for the most part (depending on application) the "docker build" phase from the development cycle.

Running in a container itself, XCM is easily deployed to the cloud, intranet or desktop for local development. Once XCM is installed, and hosts are available it can bootstrap the entire Xirsys Brand Turn services and required database from a simple text based menu system (webapp gui is in progress)

Based on our cellular model, described in the introduction, docker-machine is utilised to manage and create hosts, then we build stems (runtime environments) and then differentiate those stems to specific cell types by injecting code directly from Git(Hub).

Of course all meta data for host allocation is stored in KV4, which XCM directly accesses.

To be clear, stems can be any runtime environment; node.js, .net core, python. Xirsys Turn brand happens to be entirely Elixir/Erlang, but XCM can run node.js apps just as well.

# Appendix 2: A Note of the Role of CouchDB

CouchDB is the only external server required by the entire Xirsys system.

It has 2 main roles, obviously as a data store, but more importantly, it’s change feeds are used to bind the system together via Neuron caching.

Every neuron has an (erlang ets based) cache, which caches all data referenced via that neuron from client applications. When a kv4 reference is accessed (path/layer/key) the data is loaded and then kept up to date automatically in the cache by the change feeds. Thus if any other node changes that reference not just the data in Couchdb but all caches are updated automatically. Sequences ids are followed so that if a change feed is dropped for any reason it can restart and retrace from the correct sequence to get all the changes missed.

This method provides distributed fast access to cached data for all neurons.

Note, only neurons (1 per host) get a change feed, not client applications. Client applications access the neuron, via the REST API/websockets etc. Client applications will not access CouchDB directly, although it could be for replication and other ops scenarios.That this method is so useful, the base system itself uses it for system metadata, subscriptions and so on.

So, a Xirsys system is defined by the nodes that the underlying CouchDB manages. You could run multiple Xirsys systems in parallel using multiple CouchDBs.

CouchDB’s may also be setup to replicate over geographical regions.