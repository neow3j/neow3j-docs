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
or to send the transaction. Therefore, the `TransactionBuilder` is instantiated with a `Neow3j` object.
Once constructed, the builder allows you to set almost all the properties a Neo transaction can have. 
Excluded are fees and witnesses. The system fee, which is the amount of GAS your transaction will consume, and the
network fee, which is based on the size of your transaction and the efforts to verify it, are both fetched automatically
when building the transaction. Although, you do have the option to add an additional network fee
(`additionalNetworkFee(long fee)`) to give your transactions more priority in the network. Witnesses can only be added
to the `Transaction` object and not to the builder because they usually depend on the serialized transaction for
producing a signature.

The properties called *nonce*,  *validUntilBlock*, and *version* will also be set automatically if not set explicitely.
The *nonce* prevents replay attacks in which the exact same transaction is copied and sent again. Its default value is
set at random by the transaction builder. The *validUntilBlock* value determines for how long the transaction will
remain valid before it is included in a block. If it does not get included before that time runs out, it will be
discarded by the network. The default value is set as the maximum supported by the connected network. The transaction
*version* is by default set to 0 which you will most probably not have to change.

The properties that are of most interest are the transation's script and the signers. As mentioned before, the script
determines the actual effects of the transaction on the blockchain state. How to build the script itself is not
discussed here. Neow3j has many classes that will provide you with a `TransactionBuilder` holding a preconfigured
script.
The transaction signers are used to pay for the transaction fees and might be required by the transaction's script for
witness checks. The order of the signers is important because only the first signer will be used to pay for the
transaction fees. You can set it explicitely with the `firstSigner(Hash160 account)` method or use the more general
`signers(Signer... signers)` method. Signers are associated with a witness scope that restricts how their witness on the
transaction can be used in an invocation. For example, if an signer is only needed for fee payment the witness scope can
be reduced to *None*. Checkout the `Signer` class and `WitnessScope` enum for other possible configurations.

The last important property to set on the builder is the wallet. The wallet is required for two reasons. Firstly, the
builder requires the verification scripts of the signers to fetch the correct network fee. If a signer is not available
in the wallet an empty verification script is used instead, which will end up in a wrong network fee and an invalid
transaction.  Secondly, the builder can use the accounts in the wallet to match with the signers and attempt to
automatically generate the necessary signatures for them when calling the `sign()` method. I.e., the wallet will not
show up on the final transaction but provides the private key material to sign the transaction. 

The following is a simple example of how building, signing, and sending a transaction might look.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://127.0.0.1:40332"));
Account account = Account.fromWIF("L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2");
Wallet wallet = Wallet.withAccounts(account);

byte[] script = new ScriptBuilder().contractCall(NeoToken.SCRIPT_HASH, "symbol", null)
                                   .toArray();

Transaction tx = new TransactionBuilder(neow3j)
    .script(script)
    .signers(Signer.calledByEntry(account))
    .wallet(wallet)
    .sign();

tx.send();
```

## Signing Transactions

As you have seen in the last section, the `TransactionBuilder` offers a `sign()` method that attempts to add the correct
signatures according to what it finds in the list of signers and what accounts are in the provided wallet. 
This even works with multi-sig addresses, if all accounts required for the multi-sig address are available in the
wallet. 

But, you can have more control over the signing process in case you need it. Instead of calling the automatic `sign()`
function, you can use `getUnsignedTransaction()` on the builder and retrieve the transaction with an empty list of
witnesses. Now it is up to you to add the necessary witnesses. You can get the serialized transaction bytes and create
a witness from it.

```java
Transaction tx = ...
Account account = Account.fromWIF("L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2");
ECKeyPair keyPair = account.getECKeyPair();
byte[] txBytes = tx.getHashData();
Witness witness = Witness.create(txBytes, keyPair);
tx.addWitness(witness);
tx.send();
```

Maybe you are working with a multi-sig address as a transaction signer but don't have all private keys available. In
that case you will need to share the serialized transaction bytes with all the entities that possess the required keys
and make them create a signature that you can include in witnesses on the transaction.

```java
// All key pairs involved in the multi-sig account.
List<ECKeyPair.ECPublicKey> publicKeys = ...
// The minimumm amount of signatures necessary when using the multi-sig account.
int signingThreshold = ...
Account multiSig = Account.createMultiSigAccount(publicKeys, signingThreshold);
Wallet wallet = Wallet.withAccounts(multiSig);
byte[] script = ...
Transaction tx = new TransactionBuilder(neow3j)
        .script(script)
        .signers(Signer.calledByEntry(multiSig))
        .wallet(wallet)
        .sign();

byte[] txBytes = tx.getHashData();
// Collect the signatures.
List<Sign.SignatureData> signatures = ...
// Create a witness for the multi-sig signer.
Witness witness = Witness.createMultiSigWitness(signatures, multiSig.getVerificationScript());
tx.addWitness(witness);
tx.send();
```

As you can see, the transaction builder still requires a wallet which holds the multi-sig account even though no
automatic signing is happening. The builder needs the multi-sig account's verification script to fetch the network fee.
It gets this verification script from the account in the wallet.


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

If you call `getApplicationLog()` before the transaction is included in a block it simply returns null.