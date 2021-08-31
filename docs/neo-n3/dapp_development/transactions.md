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
(`additionalNetworkFee(long fee)`) to give your transactions more priority in the network.  
Witnesses can only be added to the `Transaction` object and not to the builder because they usually depend on the
serialized transaction for producing a signature.

The properties called *nonce*, *validUntilBlock*, and *version* will also be set automatically if not set explicitely.
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
transaction can be used in an invocation. For example, if a signer is only needed for fee payment the witness scope can
be reduced to *None*. There are two signer classes to be aware of. The `AccountSigner` and the `ContractSigner`. Use
`AccountSigner` for signers that are backed by an account. It provides static builder methods for all witness scopes. 
A transaction requires at least one `AccountSigner` that will pay for the fees.
Use `ContractSigner` if the signer is a smart contract. This kind of signer doesn't require a signature for the witness
but will call the contract's `verify(...)` method when the transaction is executed. A contract signer cannot pay the
transaction fees and therefore does not provide a builder method for the *None* witness scope. The `ContractSigner`'s
builder methods can take contract parameters in case the contract's `verify(...)` method has extra parameters.

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

When you are working with mutli-sig accounts you can use one of the `addMultiSigWitness(...)` convenience methods.
For example:

```java
// The multi-sig account holding its verification script.
Account multiSig = Account.createMultiSigAccount(publicKeys, signingThreshold);

// Collect signatures
Sign.SignatureData[] signatures = ...

// Create and get unsigned transaction. Then, add a witness for the multi-sig account.
Transaction tx = new TransactionBuilder(neow3j)
        .script(script)
        .signers(AccountSigner.calledByEntry(multiSig))
        .getUnsignedTransaction()
        .addMultiSigWitness(multiSig.getVerificationScript(), signatures)
        .send();
```

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


## Adding additional Network Fees

There are two kind of fees one has to pay for a transaction, the system fee and the network fee. The system fee is the
cost of resources consumed by the execution of a script on the neo-vm. It depends on the number and type of instructions
executed in the script. The network fee is paid for the size of the transaction and the effort needed for signature
verification. Adding a higher network fee than needed gives the transaction priority in the network. Neow3j
automatically fetches the necessary system and network fees for a transaction, but you can add an additional network fee
for priority. Use the method `additionalNetworkFee()` as in the example below.

```java
Transaction tx = new TransactionBuilder(neow3j)
        .script(script)
        .signers(AccountSigner.calledByEntry(acc))
        .additionalNetworkFee(10_000L)
        .sign();
```

This adds 10'000 GAS fractions to the network fee, which is 10'000 / 10^8 = 0.0001 GAS.
