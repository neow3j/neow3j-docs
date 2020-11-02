**Consider that there is no testnet for Neo 3 yet!** To use neow3j versions `3.+`, you need a local node of Neo 3 running. You can find one [here](http://github.com/axlabs/neo3-privatenet-docker).

# Monitoring the Blockchain

For the retrieval of information about the blockchain and the network's state, neow3j relies on NEO's RPC nodes. You
can initialize the connection to such a node with the following line.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
```

## Monitoring Blocks

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

## Tracking a Transaction

To track an issued transaction, you can use the method `track()` of your transaction instance.

As soon as the transaction is included in a block, you can retrieve its application log with the
method `getApplicationLog()`.

```java
Observable<Long> obs = transaction.track().doOnComplete(() -> {
    System.out.println("Observable completed.");
});

Disposable disp = obs.subscribe(blockIndex -> {
    NeoApplicationLog log = tx.getApplicationLog();
    System.out.println("Found tx on block " + blockIndex + ". Tx exited with state " + log.getState() + ".");
    neow.shutdown();
})
```

> **Note:** `getApplicationLog()` returns null if the log could not be fetched or the transaction is not
> yet included in a block.

<!-- Mention that certain calls require plugins or sufficient node version
Depending on what RPC methods you want to use you have to make sure that the node has the appropriate plugins installed. -->
