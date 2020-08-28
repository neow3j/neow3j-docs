# Requirements

## System Requirements

Neow3j is based on Java 8 and therefore requires applications to use Java version >= 8. For Android
applications, this implies the use of API level >= 24.


## RPC Nodes

Neow3j is not a blockchain network node. It does not synchronizes with other nodes nor store
or verify blockchain information locally. It depends on Neo nodes and their JSON-RPC interface to
retrieve block and transaction information.

Publicly reachable nodes can be found [here](http://monitor.cityofzion.io/).
If you want to run your own node check out the [official
documentation](https://docs.neo.org/v3/docs/en-us/node/introduction.html).