# How to build a claim transaction

> Since neow3j v1.0.10

Even though the RPC call `claimgas` allows for convenient claiming of GAS, it is
not always an option. If the caller does not have an open wallet on the RPC
node, this method cannot be used. Instead, a raw transaction has to be prepared
on the client and sent to the RPC node.

In the following we'll go through the steps needed to create a raw transaction
for claiming GAS.  
The prerequisites are:

* a private key in [WIF](https://en.bitcoin.it/wiki/Wallet_import_format)
  from an [ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
  key pair
* a transaction output (of type NEO) that you have already spent with the above
  key pair

GAS is claimable for NEO that you own. But before the GAS becomes available, the
NEO has to be spent in a transaction. So if you know of a transaction output
that makes you the owner of 100 NEO, you have to use that output in a new
transaction. The amount of NEO and the number of blocks between receiving and
spending it determine the amount of GAS that you can claim for that output. Of
course, the NEO don't have to be sent to another account, the transaction can
simply send it back to your account. The only effect then is, that the
transaction output has been used and GAS becomes available for claiming. A more
detailed description of GAS claiming can be found in the ClaimTransaction
section in this
[article](https://docs.neo.org/developerguide/en/articles/tx_execution.html#claimtransaction)
in the NEO documentation.

Because neow3j does not *yet* provide its own wallet index to keep track of
transaction outputs and claimable GAS, those outputs have to be retrieved in
another way. For example, you can use the neoscan API. Call
`https://api.neoscan.io/api/main_net/v1/get_claimable/<address>` with your
address.
You will get a list of claims like the following:

```json
{
    "value": 1,
    "unclaimed": 0.00000651,
    "txid": "4ba4d1f1acf7c6648ced8824aa2cd3e8f836f59e7071340e0c440d099a508cff",
    "sys_fee": 0,
    "start_height": 3697834,
    "n": 0,
    "generated": 0.00000651,
    "end_height": 3697927
},
```

From this you will need the properties `txid`, `n` and `unclaimed`. The first
two together specify the transaction output that can be used for claiming GAS.
The last one is the amount of claimable GAS corresponding to the output. The
other properties show the amount of NEO (`value`) that produced the GAS in the
time from receiving the NEO (`start_height`) up to spending them in a transaction
(`end_height`).

With that information, we can start creating the raw transaction.

First we need to obtain the ECDSA key pair from the private key WIF. The key
pair is used later for signing the transaction.

```java
String wif = "KxDgvEKzgSBPPfuVfw67oPQBSjidEiqTHURKSDL1R7yGaGYAeYnr";
ECKeyPair ecKeyPair = ECKeyPair.create(WIF.getPrivateKeyFromWIF(wif));
String adr = Keys.getAddress(ecKeyPair.getPublicKey()); // "AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y"
```

Using the claim example from above we can create a `Claim` object with neow3j,
and with it setup a `ClaimTransaction`. The `ClaimTransaction` actually only
relies on the `claimValue`, `txId` and `index` of the `Claim` object for
creating a valid transaction. 
Multiple `Claims` can be input to a new `ClaimTransaction`.

```java
String txId = "4ba4d1f1acf7c6648ced8824aa2cd3e8f836f59e7071340e0c440d099a508cff";
int index = 0;
Claim claim = new Claim(BigDecimal.valueOf(0.00000651), txId, index, BigInteger.valueOf(1), 3697834, 3697927);
ClaimTransaction tx = ClaimTransaction.fromClaims(Arrays.asList(claim), adr);
```

What's left to do is the transaction signing, i.e. creating the verification and
invocation scripts and adding them to the transaction. Check out [this
article](https://medium.com/neoresearch/understanding-multisig-on-neo-df9c9c1403b1)
for more information on the two scripts.

```java
byte[] rawTxUnsignedArray = tx.toArray();
// Create the Invocation Script
List<RawInvocationScript> rawInvocationScriptList = Arrays.asList(
        new RawInvocationScript(Sign.signMessage(rawTxUnsignedArray, ecKeyPair)));
// Create the Verification Script
byte[] publicKeyByteArray = Numeric.hexStringToByteArray(
        Numeric.toHexStringNoPrefix(ecKeyPair.getPublicKey()));
RawVerificationScript verificationScript = Keys.getVerificationScriptFromPublicKey(publicKeyByteArray);

tx.addScript(rawInvocationScriptList, verificationScript);

byte[] rawTxSignedArray = tx.toArray();
String rawTransactionHexString = Numeric.toHexStringNoPrefix(rawTxSignedArray);
System.out.println("rawTransactionHexString: " + rawTransactionHexString);
```

The raw transaction hex string can now be sent to a NEO JSON-RPC node using the
`sendrawtransactoin` method.

```json
{
  "jsonrpc": "2.0",
  "method": "sendrawtransaction",
  "params":["020001ff8c509a090d440c0e3471709ef536f8e8d32caa2488ed8c64c6f7acf1d1a44b0000000001e72d286979ee6cb1b7e65dfddfb2e384100b8d148e7758de42e4168b71792c600060d020a900000023ba2703c53263e8d6e522dc32203339dcd8eee90141400c40efd5f4a37b09fb8dca3e9cd6486c1b2d46c0319ac216c348f546ff44bb5fc3a328a43f2f49c9b2aa4cb1ce3f40327fd8403966e117745eb5c1266614f7d42321031a6c6fbbdf02ca351745fa86b9ba5a9452d785ac4f7fc2b7548ca2a46c4fcf4aac"],
  "id": 1
}
```
