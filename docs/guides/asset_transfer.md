## Asset Transfers

Asset transfers are handled by the `AssetTransfer` class. In any scenario you 
will need a connection to an RPC node via a `Neow3j` instance.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
```

Because in an asset transfer neow3j needs to pick approproate inputs from an
account according to the intended outputs, the connected RPC node needs to have
the `RpcSystemAssetTrackerPlugin` installed (see the 
[Requirements](overview/requirements.md) section).
<!-- TODO: Finish the Requiremnts section. -->


### Simple Asset Transfer

In the simplest scenario you have an `Account` with a public and private key,
e.g. a newly created one, as in the example below, or one from a wallet that you
loaded from a NEP-6 file. If you instantiated the wallet from a NEP-6 file, make
sure that the account you want to use has been decrypted either with
`Account.decryptPrivateKey(...)` or `Wallet.decryptAllAccounts(...)`. This is
necessary for automatically creating the transaction signature.

In order that neow3j can pick the apporpriate inputs for your transfer you need
to update your account's balances before building the asset transfer Use the
`Account.updateAssetBalances()` method for this.

What remains is to specify how much of what asset you want to transfer to whom.
These three things together represent a transaction output.

After building the `AssetTransfer` object you can sign and send it. The `send()` 
method does not return any information about the state of the transaction, but
it will throw an exception in case the RPC node returns an error.

In the example below a fee is added to the transfer. This is the network fee
which helps give the transfer priority in the network.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));

Account fromAccount = Account.createAccount();
fromAccount.updateAssetBalances(neow3j);

AssetTransfer transfer = new AssetTransfer.Builder(neow3j)
        .account(fromAccount)
        .output(NEOAsset.HASH_ID, 10, "AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y")
        .networkFee(0.001)
        .build()
        .sign()
        .send()
```

### Asset Transfer from multi-sig account



