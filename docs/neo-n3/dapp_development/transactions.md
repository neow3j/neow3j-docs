# Transactions

Transactions are essential for changing the blockchain state. Every change is initiated by a transaction. Neo's transactions are quite generic; they do not have specific types. Instead, their impact is determined by the attached script. A script is a sequence of NeoVM instructions and can therefore take various forms, although transaction scripts usually involve one or more calls to smart contracts. You can find the complete definition of a Neo transaction described [here](https://docs.neo.org/docs/en-us/basic/concept/transaction.html) in the official Neo documentation.
Transactions are essential for changing the blockchain state. Every change is initiated by a transaction. Neo's transactions are quite generic; they do not have specific types. Instead, their impact is determined by the attached script. A script is a sequence of NeoVM instructions and can therefore take various forms, although transaction scripts usually involve one or more calls to smart contracts. You can find the complete definition of a Neo transaction described [here](https://docs.neo.org/docs/en-us/basic/concept/transaction.html) in the official Neo documentation.

The neow3j SDK represents Neo transactions in its `io.neow3j.transaction.Transaction` class. It provides all the necessary functionality to serialize into a byte array and send the transaction to a Neo node. However, constructing a valid transaction manually would be a tedious task. Therefore, neow3j provides the `io.neow3j.contract.TransactionBuilder`, which simplifies the process of building a valid transaction and identifies incorrect configurations as early as possible.
The neow3j SDK represents Neo transactions in its `io.neow3j.transaction.Transaction` class. It provides all the necessary functionality to serialize into a byte array and send the transaction to a Neo node. However, constructing a valid transaction manually would be a tedious task. Therefore, neow3j provides the `io.neow3j.contract.TransactionBuilder`, which simplifies the process of building a valid transaction and identifies incorrect configurations as early as possible.

## Building Transactions

The `TransactionBuilder` and the `Transaction` require a connection to a Neo node to fetch information such as fees or to send the transaction. Therefore, the `TransactionBuilder` is instantiated with a `Neow3j` object. Once constructed, the builder allows you to set almost all the properties a Neo transaction can have. The system fee, which is the amount of GAS your transaction will consume, and the network fee, which is based on the size of your transaction and the effort required to verify it, are both fetched automatically when building the transaction.
The `TransactionBuilder` and the `Transaction` require a connection to a Neo node to fetch information such as fees or to send the transaction. Therefore, the `TransactionBuilder` is instantiated with a `Neow3j` object. Once constructed, the builder allows you to set almost all the properties a Neo transaction can have. The system fee, which is the amount of GAS your transaction will consume, and the network fee, which is based on the size of your transaction and the effort required to verify it, are both fetched automatically when building the transaction.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://127.0.0.1:40332"));
TransactionBuilder builder = new TransactionBuilder(neow3j);
```

The properties called _nonce_, _validUntilBlock_, and _version_ will also be set automatically if not set explicitly. The _nonce_ prevents replay attacks, where the exact same transaction is copied and sent again. Its default value is set randomly by the transaction builder. The _validUntilBlock_ value determines how long the transaction will remain valid before it needs to be included in a block. If it is not included before that time, it will be discarded by the network. The default value is set as the maximum supported by the connected network. The transaction _version_ is set to 0 by default, which you will most probably not need to change.
The properties called _nonce_, _validUntilBlock_, and _version_ will also be set automatically if not set explicitly. The _nonce_ prevents replay attacks, where the exact same transaction is copied and sent again. Its default value is set randomly by the transaction builder. The _validUntilBlock_ value determines how long the transaction will remain valid before it needs to be included in a block. If it is not included before that time, it will be discarded by the network. The default value is set as the maximum supported by the connected network. The transaction _version_ is set to 0 by default, which you will most probably not need to change.

The properties of most interest are the transaction's script and its signers. As mentioned earlier, the script determines the actual effects of the transaction on the blockchain state. The process of building the script itself is not discussed here. Neow3j offers many classes in the `contract` module that will provide you with a `TransactionBuilder` holding a preconfigured script.
The properties of most interest are the transaction's script and its signers. As mentioned earlier, the script determines the actual effects of the transaction on the blockchain state. The process of building the script itself is not discussed here. Neow3j offers many classes in the `contract` module that will provide you with a `TransactionBuilder` holding a preconfigured script.

```java
builder.script(script);
```

The usage of transaction signers serves a dual purpose. The first signer in the list specifies the account that will pay for the transaction fees, known as the `sender` of the transaction. Additionally, all signers, including the first one in the list, are used to authorize specific actions within the script that depend on the signature of an account. For example, transferring a token from account A to account B typically requires approval from account A.
The usage of transaction signers serves a dual purpose. The first signer in the list specifies the account that will pay for the transaction fees, known as the `sender` of the transaction. Additionally, all signers, including the first one in the list, are used to authorize specific actions within the script that depend on the signature of an account. For example, transferring a token from account A to account B typically requires approval from account A.

You can set the signers of a transaction using the `signers(Signer... signers)` method in the transaction builder. Note that the first signer provided in the parameters will be used as the transaction's sender. If you have specified all the signers but want to change the sender to another signer, you can use the `firstSigner(Hash160 account)` method to set it explicitly.
You can set the signers of a transaction using the `signers(Signer... signers)` method in the transaction builder. Note that the first signer provided in the parameters will be used as the transaction's sender. If you have specified all the signers but want to change the sender to another signer, you can use the `firstSigner(Hash160 account)` method to set it explicitly.

```java
// Import the Account class from the io.neow3j.wallet package
import io.neow3j.wallet.Account;

// Instantiate the Account class as follows
// Import the Account class from the io.neow3j.wallet package
import io.neow3j.wallet.Account;

// Instantiate the Account class as follows
Account account = Account.fromWIF("L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2");
builder.signers(AccountSigner.calledByEntry(account));
```

Signers are associated with a witness scope that restricts how the signer's witness* can be used during invocation. For instance, if a signer is only needed for paying the transaction fee, the witness scope can be set to *None\*. There are two signer classes to be aware of - `AccountSigner` and `ContractSigner`. Use `AccountSigner` for signers backed by an account, which provides static builder methods for all witness scopes.
Signers are associated with a witness scope that restricts how the signer's witness* can be used during invocation. For instance, if a signer is only needed for paying the transaction fee, the witness scope can be set to *None\*. There are two signer classes to be aware of - `AccountSigner` and `ContractSigner`. Use `AccountSigner` for signers backed by an account, which provides static builder methods for all witness scopes.

Use `ContractSigner` if the signer is a smart contract. This type of signer does not require a signature but calls the contract's `verify(...)` method during transaction execution. Since contract signers cannot pay transaction fees, the `ContractSigner` class does not provide a builder method for the _None_ witness scope. Furthermore, the `ContractSigner`'s builder methods can accept contract parameters if the contract's `verify(...)` method has extra parameters.
Use `ContractSigner` if the signer is a smart contract. This type of signer does not require a signature but calls the contract's `verify(...)` method during transaction execution. Since contract signers cannot pay transaction fees, the `ContractSigner` class does not provide a builder method for the _None_ witness scope. Furthermore, the `ContractSigner`'s builder methods can accept contract parameters if the contract's `verify(...)` method has extra parameters.

**Note:** A transaction requires at least one `AccountSigner` to pay for the fees.
**Note:** A transaction requires at least one `AccountSigner` to pay for the fees.

> A witness is a script pair that consists of an invocation and a verification script. Both scripts are sequences of NeoVM instructions. In the case of an account signer, the invocation script includes instructions to push the signature data onto the NeoVM stack. In the verification script, the corresponding public key data is pushed onto the NeoVM stack, followed by an instruction to verify the signature provided in the invocation script.
>
> For more information on witnesses and witness scopes, refer to the [Medium article by NeoSPCC](https://neospcc.medium.com/thou-shalt-check-their-witnesses-485d2bf8375d).
> For more information on witnesses and witness scopes, refer to the [Medium article by NeoSPCC](https://neospcc.medium.com/thou-shalt-check-their-witnesses-485d2bf8375d).

Witnesses can only be added to the `Transaction` object, not to the builder, because they typically rely on the serialized transaction to produce a signature. If you are using `single-sig` accounts in a transaction's signers, the `sign()` method will automatically generate the witnesses for you.
Witnesses can only be added to the `Transaction` object, not to the builder, because they typically rely on the serialized transaction to produce a signature. If you are using `single-sig` accounts in a transaction's signers, the `sign()` method will automatically generate the witnesses for you.

Here's a simple example of how to build, sign, and send a transaction:
Here's a simple example of how to build, sign, and send a transaction:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://127.0.0.1:40332"));
Account account = Account.fromWIF("L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2");

byte[] script = new ScriptBuilder().contractCall(NeoToken.SCRIPT_HASH, "symbol", null)
                                   .toArray();

TransactionBuilder builder = new TransactionBuilder(neow3j)
TransactionBuilder builder = new TransactionBuilder(neow3j)
    .script(script)
    .signers(AccountSigner.calledByEntry(account));
    .signers(AccountSigner.calledByEntry(account));

// Build the transaction
Transaction tx = builder.sign()
                        .send();
// Build the transaction
Transaction tx = builder.sign()
                        .send();
```

To manually add witnesses to a transaction, continue reading the next section.
To manually add witnesses to a transaction, continue reading the next section.

## Signing Transactions

As demonstrated in the section on [Building Transactions](#/neo-n3/dapp_development/transactions?id=building-transactions), the `TransactionBuilder` offers a `sign()` method to add the correct signatures based on the provided signers. If you use an `AccountSigner` with a private key, the `sign()` method will automatically generate the required signature/witness. However, if the account does not have a key (e.g., for a multi-sig account), you'll need to provide the witness manually.

`To sign manually, use `getUnsignedTransaction()` on the builder to retrieve the transaction without witnesses. Then, add the necessary witnesses by creating a witness from the serialized transaction bytes:

```java
Transaction tx = builder.getUnsignedTransaction();
Transaction tx = builder.getUnsignedTransaction();
Account account = Account.fromWIF("L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2");
ECKeyPair keyPair = account.getECKeyPair();
byte[] txBytes = tx.getHashData();
Witness witness = Witness.create(txBytes, keyPair);
tx.addWitness(witness);
tx.send();
```

For more advanced use cases that require witnesses from multi-sig accounts, you can create an unsigned `Transaction` and manually append the witnesses using `addMultiSigWitness(...)` methods. Here's an example with a multi-sig account of three participants that requires the signature of two:
For more advanced use cases that require witnesses from multi-sig accounts, you can create an unsigned `Transaction` and manually append the witnesses using `addMultiSigWitness(...)` methods. Here's an example with a multi-sig account of three participants that requires the signature of two:

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://127.0.0.1:40332"));

ECKeyPair.ECPublicKey pubKey1 = ...;
ECKeyPair.ECPublicKey pubKey2 = ...;
ECKeyPair.ECPublicKey pubKey3 = ...;
int threshold = 2;

// Create the multi-sig account with its verification script
// Create the multi-sig account with its verification script
Account multiSigAccount = Account.createMultiSigAccount(Arrays.asList(pubKey1, pubKey2, pubKey3), threshold);

byte[] script = ...; // Define your script
byte[] script = ...; // Define your script

// Create and get an unsigned transaction
// Create and get an unsigned transaction
Transaction tx = new TransactionBuilder(neow3j)
    .script(script)
    .signers(AccountSigner.calledByEntry(multiSigAccount))
    .getUnsignedTransaction();

// Sign the transaction's hash data externally
byte[] rawSig1 = ...; // Signature from participant 1
byte[] rawSig2 = ...; // Signature from participant 2

// Sign the transaction's hash data externally
byte[] rawSig1 = ...; // Signature from participant 1
byte[] rawSig2 = ...; // Signature from participant 2

Sign.SignatureData sigData1 = Sign.SignatureData.fromByteArray(rawSig1);
Sign.SignatureData sigData2 = Sign.SignatureData.fromByteArray(rawSig2);

// Map signature data to public keys
// Map signature data to public keys
HashMap<ECKeyPair.ECPublicKey, Sign.SignatureData> signatureMap = new HashMap<>();
signatureMap.put(pubKey1, sigData1);
signatureMap.put(pubKey2, sigData2);

// Add a multi-sig witness to the transaction based on the verification script and signatures
// Add a multi-sig witness to the transaction based on the verification script and signatures
tx.addMultiSigWitness(multiSigAccount.getVerificationScript(), signatureMap);

// Send the transaction
// Send the transaction
NeoSendRawTransaction response = tx.send();
```

If you need to sign a transaction manually and require a witness for a contract signer, use `Witness.createContractWitness(List<ContractParameter> verifyParams)` to create a witness for the contract signer and add it to the transaction using `addWitness(Witness witness)`.
If you need to sign a transaction manually and require a witness for a contract signer, use `Witness.createContractWitness(List<ContractParameter> verifyParams)` to create a witness for the contract signer and add it to the transaction using `addWitness(Witness witness)`.

## Tracking Transactions

To track an issued transaction, use the `track()` method on the transaction instance. You can then retrieve the application log as soon as the transaction is included in a block using `getApplicationLog()`:
To track an issued transaction, use the `track()` method on the transaction instance. You can then retrieve the application log as soon as the transaction is included in a block using `getApplicationLog()`:

```java
tx.track().subscribe(blockIndex -> {
    System.out.println("Transaction included in block " + blockIndex + ".");
    NeoApplicationLog log = tx.getApplicationLog();
    System.out.println("Transaction exited with state " + log.getExecutions().get(0).getState() + ".");
});
```

If you call `getApplicationLog()` before the transaction is included in a block, it will throw an `RpcResponseErrorException` indicating that the transaction is unknown or could not be found.
If you call `getApplicationLog()` before the transaction is included in a block, it will throw an `RpcResponseErrorException` indicating that the transaction is unknown or could not be found.

## Adding Additional Network Fees

## Adding Additional Network Fees

Transactions involve two types of fees: system fee and network fee. The system fee covers the resources consumed by script execution on the NeoVM, while the network fee is based on the transaction size and effort required for signature verification. You can add an additional network fee for priority using `additionalNetworkFee()`:

```java
Transaction tx = new TransactionBuilder(neow3j)
        .script(script)
        .signers(AccountSigner.calledByEntry(acc))
        .additionalNetworkFee(1_000_000L) // Adds 1,000,000 GAS fractions (0.01 GAS)
        .additionalNetworkFee(1_000_000L) // Adds 1,000,000 GAS fractions (0.01 GAS)
        .sign();
```

This example adds 1,000,000 GAS fractions to the network fee for transaction priority.
This example adds 1,000,000 GAS fractions to the network fee for transaction priority.
