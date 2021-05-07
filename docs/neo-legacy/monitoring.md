# Monitoring the blockchain 

For the retrieval of information about the blockchain and the network's state, neow3j relies on NEO's RPC nodes. You
can initialize the connection to such a node with the following line.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed1.ngd.network:10332"));
```

Depending on what RPC methods you want to use you have to make sure that the node has the appropriate plugins installed.
E.g. the 

* Get all blocks starting from, e.g. `2889367`, and subscribe to also get newly generated NEO blocks:

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
<!-- Mention that certain calls require plugins or sufficient node version -->
