## Transferring Assets

Asset transfers are handled by the `AssetTransfer` class. In any scenario you 
will need a connection to an RPC node via a `Neow3j` instance.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
```

Because in an asset transfer neow3j needs to pick approproate inputs from an
account according to your intended outputs, the connected RPC node needs to have
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

In order that neow3j can pick the appropriate inputs for your transfer, you need
to update your account's balances before building the asset transfer. Use the
`Account.updateAssetBalances()` method for this.

What remains is to specify how much of what asset you want to transfer to whom.
These three things together represent a transaction output. You can specify 
multiple outputs in the same transfer, they will all show up in the same 
transaction.

After building the `AssetTransfer` object you can sign and send it. 

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));

Account fromAccount = Account.createAccount();
fromAccount.updateAssetBalances(neow3j);

AssetTransfer transfer = new AssetTransfer.Builder(neow3j)
        .account(fromAccount)
        .output(NEOAsset.HASH_ID, 10, "AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y")
        .output(NEOAsset.HASH_ID, 5, "APLJBPhtRg2XLhtpxEHd6aRNL7YSLGH2ZL")
        .networkFee(0.001)
        .build()
        .sign()
        .send();
```

The `send()` method does not return information about the state of the
transaction, but it will throw an exception in case the RPC node returns an
error.

In the example a fee is also added to the transfer. This is the network fee
that give the transfer priority in the network.


### Asset Transfer with specific Inputs

Instead of letting neow3j automatically select the inputs for an asset transfer 
you can also tell it which inputs it should use. Use the `Utxo` class to specify 
which unspent transaction outputs (UTXOs) you want to be consumed by the transfer. 

```java
Utxo utxo1 = new Utxo(NEOAsset.HASH_ID, "ea8f4ea77370f317c3ea1529e10c60869d7ac9193b953e903a91e3dbeb188ac5", 0, 10);
Utxo utxo2 = new Utxo(NEOAsset.HASH_ID, "fdd33c5ee319101311dd0485950a902eb286eff4d3cd164c13337e0be154e268", 0, 10);
```

Of course, these UTXOs must belong to the account that you will use in the 
transfer. Only then will the signed transaction be valid. The cummulative amount
of the UTXOs has to be equal or greater than the amount of the intended outputs.
Neow3j will not look for other UTXOs if the provided UTXOs do not suffice the
required output amount. The transfer will simply fail in that case. UTXOs that
have not been fully consumed are transferred back to the account used in the 
transfer.

It is not necessary to update the accounts balances with
`fromAccount.updateAssetBalances(neow3j)` in this scenario.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account fromAccount = Account.createAccount();
AssetTransfer at = new AssetTransfer.Builder(neow3j)
        .account(fromAccount)
        .output(NEOAsset.HASH_ID, "15", "AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y")
        .utxos(utxo1, utxo2)
        .build()
        .sign()
        .send();
```


### Asset Transfer from a Smart Contract

Instead of transferring assets directly from one of your accounts you can
'withdraw' assets from a smart contract. Of course, your account must be
authorized to do so. This is specified in the smart contract itself and not part
of this guide.

The only thing you have to add to the asset transfer is the smart contract's
script hash, which is done with the following line.
```
  .fromContract(new ScriptHash("d994605e4f3960ba8d7422c4c8b1e94d48960a8d"))
```

It is not necessary to update the accounts balances with
`fromAccount.updateAssetBalances(neow3j)` in this scenario, because it is the 
balances of the contract that are needed here. Neow3j automatically retrieves 
does in the process.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account fromAccount = Account.createAccount();
AssetTransfer at = new AssetTransfer.Builder(neow3j)
        .account(fromAccount)
        .output(NEOAsset.HASH_ID, 1, "AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y")
        .fromContract(new ScriptHash("d994605e4f3960ba8d7422c4c8b1e94d48960a8d"))
        .build()
        .sign()
        .send();
```


### Asset Transfer from Multi-sig address

Multi-sig addresses are usually not controlled by one single entity. Meaning the
private keys of the involved key pairs are not all available to sign a 
transaction locally. So this scenario is different in the signing step.

The account used in the asset transfer must be constructed from the public keys
involved in the multi-sig address.
```java
List<BigInteger> keys = Arrays.asList(publicKey1, publicKey2);
Account multiSigAcct = Account.fromMultiSigKeys(keys, 2).build();
```
If those are not available you can also instantiate the account from just the
multi-sig address.
```java
Account multiSigAcct = Account.fromAddress("ATcWffQV1A7NMEsqQ1RmKfS7AbSqcAp2hd").build();
```

Obviously, this account does not own the key material to properly sign a
transation. Neow3j can only fetch the address's/account's balances and pick the 
appropriate UTXOs for the transfer. Signing the transaction is up to you. It is
the raw transaction byte array that needs to be signed by the required number of 
keys. Then the signatures are combined in a witness script. 

```java
byte[] unsignedTxHex = at.getTransaction().toArrayWithoutScripts();
SignatureData sig1 = Sign.signMessage(unsignedTxHex, keyPair1);
SignatureData sig2 = Sign.signMessage(unsignedTxHex, keyPair2);
RawScript witness = RawScript.createMultiSigWitness(2, Arrays.asList(sig1, sig2), keys);
```

This is an unlikely example because it means that you where in possession of
both key pairs which makes the usage of a multi-sig address obsolete. But it
shows the basic requirements to make this transfer happen. Neow3j does not yet 
provide a mechanism to collect signatures from differnt parties.

It is important that the raw transaction byte array is fetched with the
`toArrayWithoutScripts()` method and not `toArray()`. The scripts part that is
excluded in the former is exactly what is created and appended with the witness.

The created witness can now be added to the asset transfer. Then the transfer
can be executed. The whole example code looks as follows:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account multiSigAcct = Account.fromAddress("ATcWffQV1A7NMEsqQ1RmKfS7AbSqcAp2hd").build();
multiSigAcct.updateAssetBalances(neow3j);
AssetTransfer at = new AssetTransfer.Builder(neow3j)
        .account(multiSigAcct)
        .output(NEOAsset.HASH_ID, 1, "AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y")
        .build();

byte[] unsignedTxHex = at.getTransaction().toArrayWithoutScripts();
SignatureData sig1 = Sign.signMessage(unsignedTxHex, SampleKeys.KEY_PAIR_1);
SignatureData sig2 = Sign.signMessage(unsignedTxHex, SampleKeys.KEY_PAIR_2);
RawScript witness = RawScript.createMultiSigWitness(2, Arrays.asList(sig1, sig2), keys);

at.addWitness(witness).send();
```