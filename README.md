# Beta Introduction

Xirsys 3.0 introduces a completely rearchitected Xirsys platform, based around Docker! The new platform is a general purpose cloud application server, which happens to do TURN.

Through an extensible REST API Xirsys 3.0 provides custom analytics and data storage for customers. Xirsys 3.0 supports the now legacy V2 API.

This document describes the V3 API.

# Nomenclature

The Xirsys platform uses a cellular analogy to describe it's topology and function, the API docs refer to "neurons", "cells" and so forth, so we define the terms here.

* *Stems *are containers which provide a specific runtime, e.g. Erlang, Node.js, .NET Core etc.

* *Cells* are applications which are loaded from a Git Repository (Github) into a stem to provide a differentiated cell (container) with a specific function.

* *Neurons* are the base Xirsys functionality carried by all nodes in a Xirsys network. 

* *Memory, *we provide an integral 4D data store accessible via REST/Websockets/TCP (tbd).
