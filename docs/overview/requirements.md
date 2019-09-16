# Requirements

## System Requirements

Neow3j is based on Java 8 and therefore requires applications to use Java version >= 8. For Android applications, this implies the use of API level >= 24.


## RPC Nodes

Neow3j is not a blockchain network node. It does not synchronizes with other nodes or store blockchain information locally or propagate new transactions to peers. It depends on RPC nodes as proxies to the network.

NEO nodes can have different configurations and therefore will expose different sets of features. The major characteristic to look out for is the API version number of the node. The newest version of neow3j will always depend on the newest NEO node API version. So, you need to make sure that the node that you use runs an up-to-date API (see [NEO API documentation](https://docs.neo.org/docs/en-us/reference/rpc/latest-version/api.html)).

The second characteristic to look out for, are the plugins of a node. Certain features are only available if the associated plugin is installed on the node. For example, if you need information about unspent transaction outputs of any address your node will have to run the `RpcSystemAssetTracker` plugin. You can query an RPC node's plugins with the `listplugins` method.