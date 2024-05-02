# Interacting with a Neo Node

## Setting up a Connection

Since neow3j is not a Neo node implementation, interacting with an external node is crucial for any actions that involve reading from or writing to the blockchain. The primary component for interacting with Neo nodes is the `io.neow3j.protocol.Neow3j` class. This class offers Java equivalents for all JSON-RPC methods supported by Neo nodes.
You can instantiate it as follows:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
```

This assumes a Neo node is listening at `http://localhost:40332`. Replace that URL with the address of the node you want to connect to. By instantiating a `Neow3j` object in this way, it will be configured with default settings. However, the appropriate configuration depends on the network you are connecting to. The default values are set for the Neo mainnet, including a 15-second block time. If you are connecting to a different network, you may need to customize the configuration and instantiate the `Neow3j` object with an instance of `Neow3jConfig`. For example:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"),
                             new Neow3jConfig().setNetworkMagic(769));
```

Neow3j internally utilizes this configuration in several instances. For example, the network magic number is employed in the transaction hashing process.

The `Neow3jConfig` class includes a static member and static methods for setting and retrieving the address version. It is designed as static because the address version is needed in contexts where no `Neow3j` instance is accessible. Ensure that the address version matches the type of addresses you are working with, and adjust it using `Neow3jConfig.setAddressVersion(byte version)` if necessary. This ensures compatibility with the specific address format you are interacting with.

Now that we have set up a `Neow3j` instance, we can begin exploring potential interactions with the Neo blockchain. Most methods on `Neow3j` construct and return a `Request` object that specifies the request and the expected response format. Use `send()` on this request to actually send it to the Neo node. The returned type will be a subclass of `Response`.

To avoid encountering unexpected `NullPointerException`s, you can use methods like `hasError()`, `getError()`, or `throwOnError()` on the response object to handle errors smoothly before accessing any other response data. These methods help ensure robust error handling when interacting with the Neo blockchain through neow3j.

Another set of methods available on `Neow3j` are based on RxJava and return `Observable`s that you can subscribe to. These methods will be briefly explored in the next section, which covers monitoring the blockchain. This approach enables you to asynchronously observe and react to events and data from the Neo blockchain using neow3j.

## Monitoring the Blockchain

A common use case in blockchain applications involves tracking new blocks and their contents. `Neow3j` offers several methods for catching up and subscribing to new blocks. Any block retrieved from the Neo node will be forwarded to your subscriber. The following example demonstrates fetching all blocks starting from block index 100 and subscribing to newly generated blocks. The boolean parameter controls whether you want to receive complete transaction data for each block.

```java
neow3j.catchUpToLatestAndSubscribeToNewBlocksObservable(new BigInteger("100"), true)
        .subscribe((blockReqResult) -> {
            System.out.println("blockIndex: " + blockReqResult.getBlock().getIndex());
            System.out.println("hashId: " + blockReqResult.getBlock().getHash());
            System.out.println("confirmations: " + blockReqResult.getBlock().getConfirmations());
            System.out.println("transactions: " + blockReqResult.getBlock().getTransactions());
        });
```

If you're not interested in historical blocks and prefer to start subscribing from the latest block:

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

You can retrieve transaction information with block subscriptions from the previous sections, but you can also be more specific by fetching information about individual transactions. For example, if you sent a transaction to a node and now want to check its status on the blockchain.

```java
Hash256 txHash = new Hash256("da5a53a79ac399e07c6eea366c192a4942fa930d6903ffc10b497f834a538fee");
NeoGetTransaction response = neow3j.getTransaction(txHash).send();
if (response.hasError()) {
    throw new Exception("Error fetching transaction: " + response.getError().getMessage());
}
Transaction tx = response.getTransaction();
```

The `Transaction` object will contain all the information about the transaction, such as the fees paid for it, the block it was included in, or how many blocks have been added since the transaction was included in a block. If you need the transaction in its raw byte array form, you can use the `getRawTransaction` method instead. This method will provide a Base64-encoded string representing the transaction bytes.

```java
NeoGetRawTransaction response = neow.getRawTransaction(txHash).send();
Stribg tx = response.getRawTransaction();
```

For most transactions, the invocation output is of interest to the dApp. You can retrieve the results of an invocation using the `getApplicationLog` method.

```java
Hash256 txHash = new Hash256("da5a53a79ac399e07c6eea366c192a4942fa930d6903ffc10b497f834a538fee");
NeoGetApplicationLog response = neow.getApplicationLog(txHash).send();
if (response.hasError()) {
    throw new Exception("Error fetching transaction's app log: " + response.getError().getMessage());
}
// Get the first execution. Usually there is only one execution.
NeoApplicationLog.Execution execution = response.getApplicationLog().getExecutions().get(0);
// Check if the execution ended in a NeoVM state FAULT.
if (execution.getState().equals(NeoVMStateType.FAULT)) {
    throw new Exception("Invocation failed");
}
// Get the result stack.
List<StackItem> stack = execution.getStack();
StackItem returnValue = stack.get(0);

// Get the notifications fired by the transaction.
List<NeoApplicationLog.Execution.Notification> notifications = execution.getNotifications();
```

The stack included in the application logs will contain all the stack items returned by the invocation. Typically, the return stack consists of one return value located at index 0. It's important to know the type of stack item that the invocation returns in order to interpret it correctly.

Next to the return value, you can also check for notifications triggered by the transaction. Notifications in the application log provide a way to track activities of a smart contract. Currently, there isn't a straightforward method to monitor a smart contract directly. Instead, you'll need to subscribe to new blocks, inspect transaction application logs, and check for notifications fired by the contract by comparing the contract hash to `notification.getContract()`.

## Using a Wallet on the Node

If you are running your own Neo full node, you can utilize wallets that are stored directly on that node. Neow3j provides the necessary methods to interact with and use these wallets.

First, you need to open a wallet:

```java
NeoOpenWallet response = neow3j.openWallet("/path/to/wallet.json", "walletPassword").send();
if (response.hasError()) {
    throw new Exception("Failed to open wallet. Error message: " + response.getError().getMessage());
}

if (response.getOpenWallet()) {
    System.out.println("Successfully opened wallet.");
} else {
    System.out.println("Wallet not opened.");
}
```

Once the wallet is open, you can list the accounts in that wallet:

```java
NeoListAddress response = neow3j.listAddress().send();
if (response.hasError()) {
    throw new Exception("Failed to fetch wallet accounts. Error message: " + response.getError().getMessage());
}
List<NeoAddress> listOfAddresses = response.getAddresses();
```

Check the wallet balances:

```java
NeoGetWalletBalance response = neow3j.getWalletBalance(NeoToken.SCRIPT_HASH).send();
if (response.hasError()) {
    throw new Exception("Failed to get wallet balance. Error message: " + response.getError().getMessage());
}
String balance = response.getWalletBalance().getBalance();
```

Finally, close the wallet:

```java
NeoCloseWallet response = neow3j.closeWallet().send();
if (response.hasError()) {
    throw new Exception("Failed to close the wallet. Error message: " + response.getError().getMessage());
}
```

## Neo-Express

The class `io.neow3j.protocol.Neow3jExpress` extends the methods of `Neow3j` with methods specific to neo-express. [Neo-express](https://github.com/neo-project/neo-express) is a developer tool that facilitates a streamlined workflow, providing tools for managing and configuring private networks for development purposes.
Neo-express exposes additional RPC methods compared to normal Neo nodes, accessible through `Neow3jExpress`. Use `Neow3jExpress` similarly to `Neow3j`, but with a URL pointing to a neo-express instance.

```java
Neow3jExpress neow3j = Neow3jExpress.build(new HttpService("http://localhost:40332"));
```

This API is particularly useful for developers creating tools for neo-express as part of a local Neo network setup.
