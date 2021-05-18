# Interacting with a Neo Node

## Setting up a Connection
Because neow3j is not a Neo node implementation, interacting with an external node is crucial for any actions that read
from or write to the blockchain.  The centerpiece of the interaction with Neo nodes is the `io.neow3j.protocol.Neow3j` 
class. It provides Java counterparts for all JSON-RPC methods supported by Neo nodes. 
You can instantiate it as follows:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
```

This requires a Neo node to be listening at `http://localhost:40332`. Replace that URL with whatever node you want to
connect to. This way of instantiating a `Neow3j` object will set it up with a default configuration. The right
configuration depends on the network you are connecting to. The default values are tailored to the Neo mainnet, e.g.,
with a 15 seconds block time. If you are connecting to an other network you might need to adapt the configuration and
instantiate the `Neow3j` object with an instance of `Neow3jConfig`. For example:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"), 
                             new Neow3jConfig().setNetworkMagic(769));
```
 
Neow3j uses this configuration internally in a couple of places. E.g., the network magic number is used in the process
of hashing transactions.

The `Neow3jConfig` has a static member and static methods for setting and getting the address version. It is static
because the address version is required in places where no `Neow3j` instance is available. Make sure that the address
version is matching the addresses you are working with and adjust it with `Neow3jConfig.setAddressVersion(byte version)`
if necessary.

Now that we have a `Neow3j` instance set up, we can start exploring possible interactions with the Neo blockchain. Most
methods on `Neow3j` construct and return a `Request` object that defines the request and the expected response format. 
Call `send()` on that request to actually send it to the Neo node. The returned type will be a subclass of `Response`.
To make sure you don't run into unexptected `NullPointerException`s you can call `hasError()`, `getError()`, or
`throwOnError()` on the response object and handle errors smoothly before trying to access any other response data.

Another set of methods on `Neow3j` are based on RxJava and return `Observable`s that you can subscribe to. They are
briefly explored in the next section on monitoring the blockchain.
## Monitoring the Blockchain

One common use case in blockchain-related applications is keeping track of new blocks and their contents.  There are
several methods on `Neow3j` that allow you to catch up and subscribe to new blocks. Any block retrieved from the Neo
node will be passed on to your subscriber. The following example gets all blocks starting at block index 100 and
subscribes to any newly generated blocks. With the boolean parameter you control if you want to receive the complete
transaction data for each block.

```java
neow3j.catchUpToLatestAndSubscribeToNewBlocksObservable(new BigInteger("100"), true)
        .subscribe((blockReqResult) -> {
            System.out.println("blockIndex: " + blockReqResult.getBlock().getIndex());
            System.out.println("hashId: " + blockReqResult.getBlock().getHash());
            System.out.println("confirmations: " + blockReqResult.getBlock().getConfirmations());
            System.out.println("transactions: " + blockReqResult.getBlock().getTransactions());
        });
```

Or if you are not interested in any history, start subscribing from the latest block:

```java
neow3j.subscribeToNewBlocksObservable(true)
        .subscribe((blockReqResult) -> {
            System.out.println("blockIndex: " + blockReqResult.getBlock().getIndex());
            System.out.println("hashId: " + blockReqResult.getBlock().getHash());
            System.out.println("confirmations: " + blockReqResult.getBlock().getConfirmations());
            System.out.println("transactions: " + blockReqResult.getBlock().getTransactions());
        });
```

## Inspecting a transaction

You can retrieve transaction information with the block subscritptions from the last sections but you can also be more
specific and fetch information about single transactions, e.g., if you sent a transaction to a node and now want to
check its state on the blockchain. 

```java
Hash256 txHash = new Hash256("da5a53a79ac399e07c6eea366c192a4942fa930d6903ffc10b497f834a538fee");
NeoGetTransaction response = neow3j.getTransaction(txHash).send();
if (response.hasError()) {
    throw new Exception("Error fetching transaction: " + response.getError().getMessage());
}
Transaction tx = response.getTransaction();
```

This `Transaction` object will contain all the information about the transaction, e.g., the fees paid for it, the block
it was included, or how many blocks have been added since the transadtion was included in a block. If you require the
transaction in its raw byte array form you can use the `getRawTransaction` method instead. It will provide a
Base64-encoded String of the transaction bytes.

```java
NeoGetRawTransaction response = neow.getRawTransaction(txHash).send();
Stribg tx = response.getRawTransaction();
```

Most transactions will have an invocation output that is of interest to the dApp. You can get the results of an
invocation with the `getApplicationLog` method. 

```java
Hash256 txHash = new Hash256("da5a53a79ac399e07c6eea366c192a4942fa930d6903ffc10b497f834a538fee");
NeoGetApplicationLog response = neow.getApplicationLog(txHash).send();
if (response.hasError()) {
    throw new Exception("Error fetching transaction's app log: " + response.getError().getMessage());
}
// Get the first execution. Usually there is only one execution.
NeoApplicationLog.Execution execution = response.getApplicationLog().getExecutions().get(0);
// Check if the execution ended in a Neo VM state FAULT.
if (execution.getState().equals(NeoVMStateType.FAULT)) {
    throw new Exception("Invocation failed");
}
// Get the result stack.
List<StackItem> stack = execution.getStack();
StackItem returnValue = stack.get(0);

// Get the notifications fired by the transaction.
List<NeoApplicationLog.Execution.Notification> notifications = execution.getNotifications();
```

The stack included in the application logs will contain all the stack items that the invocation returned. Usually the
return stack is made up of one return value that is at index 0. You will need to know what type of stack item the
invocation returns to be able to interpret it correctly.  

Next to the return value you can also check what notification have been triggered by the transaction. The applications
log's notifications are one way to track activities of a  smart contract. Currently there is no convenient way to follow
a smart contract. You will need to subscribe to new blocks, go through all transaction's application logs, and check if
the logs contain notifications fired by the contract by comparing the contract hash to `notification.getContract()`.


## Using a Wallet on the Node

If you run your own Neo full node you can make use of wallets that are stored directly on that node. Neow3j covers the
necessary methods to interact and make use of such wallets. 

First a wallet needs to be opened.

```java
NeoOpenWallet response = neow.openWallet("/path/to/wallet.json", "walletPassword").send();
if (response.hasError()) {
    throw new Exception("Failed to open walled. Error message: " + response.getError().getMessage());
}

if (response.getOpenWallet()) {
    System.out.println("Successfully opened wallet.");
} else {
    System.out.println("Wallet not opened.");
}
```

Now, with the open wallet, you can list the accounts in that wallet.

```java
NeoListAddress response = neow.listAddress().send();
if (response.hasError()) {
    throw new Exception("Failed to fetch wallet accounts. Error message: " + response.getError().getMessage());
}
List<NeoAddress> listOfAddresses = response.getAddresses();
```

Check the wallets balances.

```java
NeoGetWalletBalance response = neow.getWalletBalance(NeoToken.SCRIPT_HASH).send();
if (response.hasError()) {
    throw new Exception("Failed to get wallet balance. Error message: " + response.getError().getMessage());
}
String balance = response.getWalletBalance().getBalance();
```

And, in the end, close the wallet.

```java
NeoCloseWallet response = neow.closeWallet().send();
if (response.hasError()) {
    throw new Exception("Failed to close the  wallet. Error message: " + response.getError().getMessage());
}
```