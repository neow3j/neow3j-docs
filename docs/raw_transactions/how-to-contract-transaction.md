# How to build a contract transaction

Even though the RPC call `sendtoaddress` allows for convenient and intuitive
sending of NEO or GAS to an address, it is not always an option. If the caller
does not have an open wallet on the RPC node, this method cannot be used.
Instead, a raw transaction has to be prepared on the client directly and then
sent to the RPC node.

In the following we'll go through the steps needed to create a raw transaction
manually for transferring NEO.  
The prerequisites are:

* a private key in [WIF](https://en.bitcoin.it/wiki/Wallet_import_format)
  from an [ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
  key pair
* an address to send assets to
* an [unspent transaction output
  (UTXO)](https://docs.neo.org/developerguide/en/articles/blockchain/utxo.html)
  that can be used with the private key

The UTXOs can be retieved by calling `getUnspents(String address)` on a `Neow3j`
object (see ["Using the Newow3j Object"](using_the_neow3j_object.md)).
The response will contain the UTXOs of the provided address per global asset
(e.g. NEO). In this example it is assumed that one NEO UTXO is available for the
account of which you possess the private key in WIF.

First we need to obtain the ECDSA key pair from the private key WIF. The key
pair is later used for signing the transaction.

```java
String wif = "KxDgvEKzgSBPPfuVfw67oPQBSjidEiqTHURKSDL1R7yGaGYAeYnr";
ECKeyPair ecKeyPair = ECKeyPair.create(WIF.getPrivateKeyFromWIF(wif));
```

Then, with the UTXO retrieved before, we can declare the inputs for our new 
transaction. The inputs will be consumed fully once the transaction is put on a
block. Multiple inputs could be added but we only add one.

```java
String txId = "9c77393c44ajf27i1f46l4ceddd6316e55871ca4b5aef2d133204c8006c4a683";
int index = 1;
List<RawTransactionInput> transactionInputs = Arrays.asList(
    new RawTransactionInput(txId, index));
```

Then we can specify how we want to spent the input. In this case we will send 1
NEO to some other address given in `toAdr` and direct the second NEO back to our
address `changeAdr`.

```java
String toAdr = "AZ81H31DMWzbSnFDLFkzh9vHwaDLayV7fU";
String changeAdr = Keys.getAddress(ecKeyPair.getPublicKey());
List<RawTransactionOutput> transactionOutputs = Arrays.asList(
    new RawTransactionOutput(NEOAsset.HASH_ID, "1", toAdr),
    new RawTransactionOutput(NEOAsset.HASH_ID, "1", changeAdr));
```

With the inputs and outputs defined we can create the raw transaction object.
We use the `ContractTransaction` because that is the type of asset transactions.

```java
ContractTransaction rawTx = new ContractTransaction.Builder()
        .inputs(transactionInputs)
        .outputs(transactionOutputs)
        .build();
```

What follows is the creation and attaching of the witness to the transaction.
The witness consists of the invocation script and the verification script. The
invocation script is basically the transaction's hash signed with the senders
private key (i.e. the signature). This is needed to prove that the owner of the
private key intended this transaction to happen. The verification script is
created from the public key and can be used to verify the signature. Check out
[this
article](https://medium.com/neoresearch/understanding-multisig-on-neo-df9c9c1403b1)
for more information on the two scripts.

```java
// serialize the base raw transaction
byte[] rawTxUnsignedArray = rawTx.toArray();

// Create the Invocation Script
List<RawInvocationScript> rawInvocationScriptList = Arrays.asList(
    new RawInvocationScript(Sign.signMessage(rawTxUnsignedArray, ecKeyPair)));

// Create the Verification Script
RawVerificationScript verificationScript = 
    Keys.getVerificationScriptFromPublicKey(ecKeyPair.getPublicKey());

rawTx.addScript(rawInvocationScriptList, verificationScript);
```

The transaction is complete now and could be send to an RPC node by calling the
`sendrawtransaction` method. The transaction needs to be converted to a
hexadecimal string for that.

```java
byte[] rawTxSignedArray = rawTx.toArray();
String rawTransactionHexString = Numeric.toHexStringNoPrefix(rawTxSignedArray);
```