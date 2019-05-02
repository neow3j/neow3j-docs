# How-to

## Build a raw asset transaction

Even though the RPC call `sendtoaddress` allows for convenient and intuitive
sending of NEo or GAS to an address, it is not always an option. If the caller
does not have an open wallet on the RPC node this method cannot be used. Instead
a raw transaction has to be prepared on the client directly and then send to
the RPC node.

In the following we'll go through the steps needed to create a raw transaction
with Neow3j.  
The prerequisites are:

* a private key in [WIF](https://en.bitcoin.it/wiki/Wallet_import_format) format
  from an [ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
  key pair
* an [unspent transaction output
  (UTXO)](https://docs.neo.org/developerguide/en/articles/blockchain/utxo.html)
  that can be used with the private key
* an address to send assets to

The UTXOs corresponding to an address are usually tracked by a wallet running on
the client or a service running in a remote server. At the time of writing such
wallet functionality is not *yet* part of Neow3j. But one can use neoscan.io's
[API](https://neoscan.io/docs/index.html) to fetch the UTXOs of an address. 
Simply call `https://api.neoscan.io/api/main_net/v1/get_balance/<address>` with
your address. For each asset (e.g. NEO) you will get a list of UTXOs as in the
below example:

```json
{
  "value": 2,
  "txid": "9c77393c44ajf27i1f46l4ceddd6316e55871ca4b5aef2d133204c8006c4a683",
  "n": 1
}
```

The `n` denotes the index of the UTXO. In the example the second output of the
transaction with `txid` makes you the owner of 2 NEO.

With that, we can start creating the raw transaction.

First we need to obtain the ECDSA key pair from the private key WIF. The key
pair is later used for signing the transaction.

```java
String wif = "KxDgvEKzgSBPPfuVfw67oPQBSjidEiqTHURKSDL1R7yGaGYAeYnr";
ECKeyPair ecKeyPair = ECKeyPair.create(WIF.getPrivateKeyFromWIF(wif));
```

Given the transaction ID and the unspent output index from the above example we
can declare the inputs for our new transaction. The inputs will be consumed fully
once the transaction is put on a block. Multiple inputs could be added.

```java
String txId = "9c77393c44ajf27i1f46l4ceddd6316e55871ca4b5aef2d133204c8006c4a683";
int index = 1;
List<RawTransactionInput> transactionInputs = Arrays.asList(
    new RawTransactionInput(txId, index));
```

Then we can specify how we want to spent the input. In this case we will send 1
NEO to some other address given in `toAdr` and direct the second NEO that is
leftover back to our address.

```java
String toAdr = "AZ81H31DMWzbSnFDLFkzh9vHwaDLayV7fU";
String changeAdr = Keys.getAddress(ecKeyPair.getPublicKey());
List<RawTransactionOutput> transactionOutputs = Arrays.asList(
    new RawTransactionOutput(0, NEOAsset.HASH_ID, 1, toAddress),
    new RawTransactionOutput(1, NEOAsset.HASH_ID, 1, fromAddress));
```

With the inputs and outputs defined we can create the raw transaction object.
Here we call the `createContractTransaction(...)` because the asset transaction
that we want to make is of the `ContractTransaction` type.

```java
RawTransaction rawTx = RawTransaction.createContractTransaction(
    null, null, transactionInputs, transactionOutputs);
```

What follows is the creation and attaching of the witness to the transaction.
The witness consists of the invocation script and the verification script.
The invocation script is basically transaction hash signed with the senders
private key (i.e. the signature). This is needed to prove that the owner of the
private key intended this transaction to happen. The verification script is
created from the public key and can be used to verify the signature.
Check out [this
article](https://medium.com/neoresearch/understanding-multisig-on-neo-df9c9c1403b1)
for more information on the two scripts.

```java
// serialize the base raw transaction
byte[] rawTxUnsignedArray = rawTx.toArray();

// Create the Invocation Script
List<RawInvocationScript> rawInvocationScriptList = Arrays.asList(
    new RawInvocationScript(Sign.signMessage(rawTxUnsignedArray, ecKeyPair)));

// Create the Verification Script
byte[] publicKeyByteArray = Numeric.hexStringToByteArray(
        Numeric.toHexStringNoPrefix(ecKeyPair.getPublicKey()));
RawVerificationScript verificationScript = Keys.getVerificationScriptFromPublicKey(publicKeyByteArray);

// give the invocation and verification script to the raw transaction:
rawTx.addScript(rawInvocationScriptList, verificationScript);
```

What remains is to convert our complete transaction object to a byte array and
print it so that we can send it to a RPC node.

```java
byte[] rawTxSignedArray = rawTx.toArray();
String rawTransactionHexString = Numeric.toHexStringNoPrefix(rawTxSignedArray);
System.out.println("rawTransactionHexString: " + rawTransactionHexString);
```