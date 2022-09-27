# Transactions

Transactions are the main protagonists when changing blockchain state. Every change is induced by a transaction. Neo's
transactions are very generic. There are no transaction types. What determines their impact is the attached script. A
script is a sequence of NeoVM instructions and can therefore take many forms, though, transaction scripts usually
contain one or more calls to smart contracts. The full definition of a Neo transaction is described
[here](https://docs.neo.org/docs/en-us/basic/concept/transaction.html) in the official Neo documentation.

The neow3j SDK represents Neo transactions in its `io.neow3j.transaction.Transaction` class. It provides all necessary
functionality to serialize to a byte array and send the transaction to a Neo node. But, constructing a valid transaction
manually would be a tedious task. Therefore, neow3j provides the `io.neow3j.contract.TransactionBuilder`, which takes
care of building a valid transaction and points out wrong configurations as early as possible. 

## Building Transactions

The `TransactionBuilder` and the `Transaction` require a connection to a Neo node for fetching information, like fees
or to send the transaction. Therefore, the `TransactionBuilder` is instantiated with a `Neow3j` object. Once
constructed, the builder allows you to set almost all the properties a Neo transaction can have. The system fee, which
is the amount of GAS your transaction will consume, and the network fee, which is based on the size of your transaction
and the efforts to verify it, are both fetched automatically when building the transaction.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://127.0.0.1:40332"));
TransactionBuilder builder = new TransactionBuilder(neow3j);
```

The properties called *nonce*, *validUntilBlock*, and *version* will also be set automatically if not set explicitly.
The *nonce* prevents replay attacks in which the exact same transaction is copied and sent again. Its default value is
set at random by the transaction builder. The *validUntilBlock* value determines for how long the transaction will
remain valid before it is included in a block. If it does not get included before that time, it will be discarded by
the network. The default value is set as the maximum supported by the connected network. The transaction *version* is
set to 0 by default which you will most probably not have to change.

The properties that are of most interest are the transaction's script and its signers. As mentioned before, the script
determines the actual effects of the transaction on the blockchain state. How to build the script itself is not
discussed here. Neow3j offers many classes in the `contract` module that will provide you with a `TransactionBuilder`
holding a preconfigured script.

```java
builder.script(script);
```

The use of transaction signers is twofold. The first signer in the list specifies the account that will pay for the
transaction fees - it is called the `sender` of the transaction. Secondly, all signers - including the first in the
list - are used to allow specific actions of the script that are depending on the signature of an account. For example,
if a token should be transferred from account a to account b, usually this requires the approval of account a.

You can set the signers of a transaction with the method `signers(Signer... signers)` in the transaction builder. Note,
that the first signer in the provided parameters is used as the transaction's sender. In case you have set all signers,
but want to change the sender to another signer, you can use the method `firstSigner(Hash160 account)` to set it
explicitly.

```java
Account account = Account.fromWIF("L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2");
builder.signers(AccountSigner.calledByEntry(account));
```

Signers are associated with a witness scope that restricts how the signer's witness* can be used in an invocation.
For example, if a signer is only needed for paying the transaction fee, the witness scope can be set to *None*. There
are two signer classes to be aware of - the `AccountSigner` and the `ContractSigner`. Use `AccountSigner` for signers
that are backed by an account. It provides static builder methods for all witness scopes.

Use `ContractSigner` if the signer is a smart contract. This kind of signer doesn't require a signature, but will call
the contract's `verify(...)` method when the transaction is executed. Since contract signers cannot pay the transaction
fees, the class `ContractSigner` does not provide a builder method for the *None* witness scope. Further, the
`ContractSigner`'s builder methods can take contract parameters in case the contract's `verify(...)` method has extra
parameters.

**Note:** A transaction requires at least one `AccountSigner` that will pay for the fees.

> *A witness is a script pair that contains an invocation and a verification script. Both scripts are simply a
> sequence of NeoVM instructions. In the basic case of an account signer, the invocation script consists of the
> instruction to push the signature data on the NeoVM stack, whereas in the verification script, the corresponding
> public key data is pushed on the NeoVM stack followed by an instruction to check its signature provided in the
> invocation script.
>
> Check out the [Medium article by NeoSPCC](https://neospcc.medium.com/thou-shalt-check-their-witnesses-485d2bf8375d)
> to learn more about witnesses and witness scopes.

Witnesses can only be added to the `Transaction` object and not to the builder because they usually depend on the
serialized transaction for producing a signature. In case you are using `single-sig` accounts in a transaction's
signers, the `sign()` method will produce the witnesses automatically for you.

The following is a simple example of how building, signing, and sending a transaction might look.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://127.0.0.1:40332"));
Account account = Account.fromWIF("L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2");

byte[] script = new ScriptBuilder().contractCall(NeoToken.SCRIPT_HASH, "symbol", null)
                                   .toArray();

Transaction tx = new TransactionBuilder(neow3j)
    .script(script)
    .signers(AccountSigner.calledByEntry(account))
    .sign();

NeoSendRawTransaction response = tx.send();
```

For manually adding witnesses to a transaction continue reading the next section.

## Signing Transactions

As you have seen in the last section, the `TransactionBuilder` offers a `sign()` method. It attempts to add the correct
signatures according to what it finds in the list of signers. If you add an `AccountSigner` that holds a private key,
the `sign()` method can automatically create the signature/witness for that signer. If the account doesn't have a key,
e.g., because it is a multi-sig account, you will need to provide the witness manually.
Witnesses for `ContractSigners` are also added automatically with `sign()`.

To sign manually, use `getUnsignedTransaction()` on the builder and retrieve the transaction with an empty list of
witnesses. Now it is up to you to add the necessary witnesses. You can get the serialized transaction bytes and create
a witness from it as shown below.

```java
Transaction tx = txBuilder.getUnsignedTransaction();
Account account = Account.fromWIF("L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2");
ECKeyPair keyPair = account.getECKeyPair();
byte[] txBytes = tx.getHashData();
Witness witness = Witness.create(txBytes, keyPair);
tx.addWitness(witness);
tx.send();
```

If your use case is more advanced and requires witnesses of `multi-sig` accounts, you can create an unsigned
`Transaction` with `getUnsignedTransaction()` and append the witnesses manually to the transaction with
one of the `addMultiSigWitness(...)` convenience methods. The following example shows how such an implementation could
look like with a multi-sig account of three participants that requires the signature of two of them.


```java
Neow3j neow3j = Neow3j.build(new HttpService("http://127.0.0.1:40332"));

ECKeyPair.ECPublicKey pubKey1 = ...;
ECKeyPair.ECPublicKey pubKey2 = ...;
ECKeyPair.ECPublicKey pubKey3 = ...;
int threshold = 2;

// The multi-sig account holding its verification script.
Account multiSigAccount = Account.createMultiSigAccount(Arrays.asList(pubKey1, pubKey2, pubKey3), threshold);

byte[] script = ...;

// Create and get unsigned transaction.
Transaction tx = new TransactionBuilder(neow3j)
    .script(script)
    .signers(AccountSigner.calledByEntry(multiSigAccount))
    .getUnsignedTransaction();

// Externally sign the transaction's hash data (tx.getHashData()) with e.g., Sign.signMessage(txHash, ecKeyPair);
// Then, parse the raw signature data into the Sign.SignatureData object.
byte[] rawSig1 = ...;
Sign.SignatureData sigData1 = Sign.SignatureData.fromByteArray(rawSig1);
byte[] rawSig2 = ...;
Sign.SignatureData sigData2 = Sign.SignatureData.fromByteArray(rawSig2);

// Create a map to specify what signature data belongs to which public key.
HashMap<ECKeyPair.ECPublicKey, Sign.SignatureData> signatureMap = new HashMap<>();
signatureMap.put(pubKey1, sigData1);
signatureMap.put(pubKey2, sigData2);

// Creates and adds a multi-sig witness to the transaction based on the verification script and the signatures.
tx.addMultiSigWitness(multiSigAccount.getVerificationScript(), signatureMap);

NeoSendRawTransaction response = tx.send();
```

If you need to sign a transaction manually and also require a witness for a contract signer, you can use the method
`Witness.createContractWitness(List<ContractParameter> verifyParams)` to create a witness for the contract signer. Then,
add it with `addWitness(Witness witness)` to the transaction.

## Tracking Transactions

To track an issued transaction, you can use the method `track()` on the transaction instance.  As soon as the
transaction is included in a block, you can retrieve its application log with the method `getApplicationLog()`.

```java
tx.track().subscribe(blockIndex -> {
    System.out.println("Transaction included in block " + blockIndex + ".");
    NeoApplicationLog log = tx.getApplicationLog();
    System.out.println("Transaction exited with state " + log.getExecutions().get(0).getState() + ".");
});
```

If you call `getApplicationLog()` before the transaction is included in a block it throws an `RpcResponseErrorException`
with an error message, that the transaction is unknown or could not be found.

## Adding additional Network Fees

There are two kind of fees one has to pay for a transaction, the system fee and the network fee. The system fee is the
cost of resources consumed by the execution of a script on the NeoVM. It depends on the number and type of instructions
executed in the script. The network fee is paid for the size of the transaction and the effort needed for signature
verification. Adding a higher network fee than needed gives the transaction priority in the network. Neow3j
automatically fetches the necessary system and network fees for a transaction, but you can add an additional network fee
for priority. Use the method `additionalNetworkFee()` as in the example below.

```java
Transaction tx = new TransactionBuilder(neow3j)
        .script(script)
        .signers(AccountSigner.calledByEntry(acc))
        .additionalNetworkFee(1_000_000L)
        .sign();
```

This adds 1'000'000 GAS fractions to the network fee, which is 1'000'000 / 10^8 = 0.01 GAS.
