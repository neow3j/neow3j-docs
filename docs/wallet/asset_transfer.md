# Asset Transfers

An transfer of global assets can be done by using the `AssetTransfer` class. It
can also be done manually by constructing a `ContractTransaction` and sending it
to an RPC node but the `AssetTransfer` works at a higher abstraction layer.

In the simplest case you have an `Account` with a public and private key, e.g.
obtained from a fresh key pair (`ecKeyPair` in the example). 
You will also need an RPC node that has the RpcSystemAssetTrackerPlugin
installed. It is needed for retrieving an accounts unspent transactino outputs
(UTXOs).

```java
Account fromAccount = Account.fromECKeyPair(ecKeyPair).build();
Neow3j neow3j = Neow3j.build(new HttpService("http://nucbox.axlabs.com:30333"));
```

In order that Neow3j knows the balances of your account you need to update them.
Use the following line. This makes a call to the RPC node. It is up to the
developer to make this updates. Neow3j will not do them on its own. This way the
developer has full control over how often these balance updates should happen. 

```java
fromAccount.updateAssetBalances(neow3j);
```

Now specify what you want to send to whom. This is done by creating a
Transactino Output. In the example we send 10 NEO.

```java
String toAddress = "AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y";
RawTransactionOutput output = new RawTransactionOutput(NEOAsset.HASH_ID, "10", toAddress);
```

All that remains is building the AssetTransfer object with the above arguments
and sending it. 
You can voluntarily specify a fee for your transaction, for higher priority.

```java
AssetTransfer transfer = new AssetTransfer.Builder()
        .neow3j(neow3j)
        .account(fromAccount)
        .output(output)
        .fee(new BigDecimal("0.001")
        .build();

transfer.send();
```
