# Upgrading from 3.24.x to 3.25.1

Update the SDK, devpack, test tools, and Gradle plugin dependencies you use to `3.25.1`. The following changes affect source code written for 3.24.x.

## Submitting transactions

`Transaction.send()` now returns `TransactionSubmission`, not `NeoSendRawTransaction`. Keep the submission to inspect the RPC response, transaction ID, inclusion, and execution result:

```java
TransactionSubmission submission = builder.sign().send();
if (submission.hasError()) {
    throw new IllegalStateException(submission.getError().getMessage());
}
Hash256 txId = submission.getTxId();
submission.await();
NeoApplicationLog log = submission.getApplicationLog(); // May be null if unavailable.
```

`track()` and `getApplicationLog()` moved from `Transaction` to `TransactionSubmission`. The raw `NeoSendRawTransaction` is available through `submission.getSendResponse()`. `await()` can take a `Duration` timeout; it confirms inclusion, so inspect the log or call `hasHalted()` to check execution. See [Transactions](neo-n3/dapp_development/transactions.md#tracking-transactions).

RPC polling helpers in `Await` and the submission's blocking wait and lookup methods now declare checked `IOException` for communication failures. Catch it or propagate it from your method. Missing logs return `null`, while unexpected RPC errors are raised.

## Hash strings and contract storage

`Hash160.toString()` and `Hash256.toString()` now include `0x`. Use `toUnprefixedString()` where a bare hexadecimal value is needed; JSON serialization continues to use bare hexadecimal strings.

The devpack removed `StorageContext` and the storage methods that take a context. Use `Storage.get(key)`, `Storage.put(key, value)`, `Storage.delete(key)`, or `new StorageMap(prefix)` instead. See [Devpack storage](neo-n3/smart_contract_development/devpack.md#storage).

## Token amounts and additions

Use `toBaseUnits` to convert a decimal token amount to an integer amount, and `fromBaseUnits` for the reverse conversion. These replace the deprecated conversion names. Instance methods fetch the token's decimal precision; static overloads accept an explicit precision. See [Token Contracts](neo-n3/dapp_development/token_contracts.md).

You can now use `Neow3j.subscribeToEvents(contractHash)` to receive new contract notifications and `io.neow3j.contract.StdLib` to invoke native StdLib functions from an SDK application. These are distinct from the devpack's contract-side StdLib interface. See [Monitoring the Blockchain](neo-n3/dapp_development/interacting_with_a_node.md#monitoring-the-blockchain) and [Native StdLib](neo-n3/dapp_development/smart_contracts.md#native-stdlib).

For `devpack-test`, 3.25.1 moves the bundled Neo Express `rpc.MaxGasInvoke` and `rpc.MaxFee` settings to the root `settings` object. Move them there too if you copied the older file into your tests. See [Testing](neo-n3/smart_contract_development/testing.md#chain-configuration).
