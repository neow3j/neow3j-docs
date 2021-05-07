# Input Selection Strategy

When you create an `AssetTransfer`, a `ContractInvocation` or a `ContractDeployment` with neow3j, you usually have some
NEO or GAS involved. I.e. you need to spend NEO or GAS to execute the transaction. Because Neo Legacy follows the UTXO model,
there is no single account balance to go to for checking the sufficiency of funds. Rather, we have to look for one or
multiple UTXOs that together make up the required amount.

If there are enough UTXOs available from a certain account, then the next question is which of these UTXOs should be
used in the transaction.
For this selection, different strategies can be applied. For example:
- Order the UTXOs from oldest to youngest and select as many UTXOs necessary starting with the oldest.
- Order the UTXOs from smallest to largest (with respect to the amount of NEO/GAS) and select as many UTXOs as
necessary, starting with the smallest.

Neow3j aims at providing multiple such strategies that implement the `InputCalculationStrategy` interface. Currently,
only one strategy is implemented. It simply takes a list of UTXO and traverses it "from left to right", selecting UTXOs
until the required amount is fulfilled.

The strategy can be defined when building a new transaction with the classes `AssetTransfer`, `ContractInvocation`, and
`ContractDeployment`. For example:

```java
AssetTransfer transfer = new AssetTransfer.Builder(neow3j)
        .account(fromAccount)
        .output(NEOAsset.HASH_ID, 10, "AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y")
        .inputCalculationStrategy(new LeftToRightInputCalculationStrategy())
        .build()
        .sign()
        .send();
```

This example explicitly sets the strategy which is also the default strategy. If you don't explicitly define a strategy
in this way, this is the one that will be used.
