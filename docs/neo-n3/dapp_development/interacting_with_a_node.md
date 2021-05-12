# Interacting with a Neo Node

Because neow3j is not a Neo node implementation, interacting with an external node is crucial for any actions that read
from or write to the blockchain.  The central interface that covers the interaction with Neo nodes is
`io.neow3j.protocol.Neow3j`. This class provides Java counterparts for all JSON-RPC methods supported by Neo nodes. 
You can instantiate it as follows:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
```

This expects a Neo node to be listening at `http://localhost:40332` and configures the `Neow3j` object with the default
configuration. The necessary configuration depends on the network you are connecting to. The default values are tailored
to the Neo mainnet, e.g., 15 seconds block time. If you are connecting to an other network you might need to adapt the
configuration and instantiate the `Neow3j` object with an instance of `Neow3jConfig`. For example:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"), 
                             new Neow3jConfig().setNetworkMagic(769));
```
 
Neow3j uses this configuration internally in a couple of places. E.g., the network magic number is used in the process
of hashing transactions.

The `Neow3jConfig` has a static member and methods for the address version. Because the address version is required 
without a `Neow3j` instance available, it is static and not an instance variable. Make sure that the address version is
matching the addresses you are working with and adjust it with `Neow3jConfig.setAddressVersion(byte version)` if
necessary.


## Monitoring the Blockchain


- Get all blocks starting from, e.g. `2889367`, and subscribe to also get newly generated NEO blocks:

```java
neow3j.catchUpToLatestAndSubscribeToNewBlocksObservable(new BlockParameterIndex(2889367), true)
        .subscribe((blockReqResult) -> {
            System.out.println("#######################################");
            System.out.println("blockIndex: " + blockReqResult.getBlock().getIndex());
            System.out.println("hashId: " + blockReqResult.getBlock().getHash());
            System.out.println("confirmations: " + blockReqResult.getBlock().getConfirmations());
            System.out.println("transactions: " + blockReqResult.getBlock().getTransactions());
        });
```

Or, you can just subscribe to the newly generated NEO blocks:

```java
neow3j.catchUpToLatestAndSubscribeToNewBlocksObservable(BlockParameterName.LATEST, true)
        .subscribe((blockReqResult) -> {
            System.out.println("#######################################");
            System.out.println("blockIndex: " + blockReqResult.getBlock().getIndex());
            System.out.println("hashId: " + blockReqResult.getBlock().getHash());
            System.out.println("confirmations: " + blockReqResult.getBlock().getConfirmations());
            System.out.println("transactions: " + blockReqResult.getBlock().getTransactions());
        });
```

