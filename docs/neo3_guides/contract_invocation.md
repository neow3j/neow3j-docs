# Invoking Smart Contracts

> **Consider that there is no testnet for Neo 3 yet!** To use neow3j versions `3.+`, you need a local node of Neo 3 running. You can find one [here](http://github.com/axlabs/neo3-privatenet-docker).

> For invocations of NEP-5 contracts, check out the documentation in the [next Section](neo3_guides/token_transfer.md#transferring-tokens-assets).

To deploy, invoke or just retrieve information about any contract's state on the blockchain, the
class `SmartContract` can be used. For example, by calling `invokeFunction(...)` a script is built
based on the provided parameters and is handed to a `TransactionBuilder`. In the
`TransactionBuilder` signers can be configured, a sender can be set or an additional network fee
can be added. The transaction can then be signed and built, and the resulting `Transaction` can
be sent to a neo-node. Alternatively, an unsigned `Transaction` can be created for later signing
(e.g. when using a multi-sig account, see
[here](neo3_guides/token_transfer.md#transfer-from-a-multi-sig-account)).

The above process is visualized in the following figure.

```
         Build script           ->      Configure transaction       ->   Tx ready to sign and send

        ---------------                  --------------------                -------------
       | SmartContract |        ->      | TransactionBuilder |      ->      | Transaction |
        ---------------                  --------------------                -------------
```

To invoke any contract, you will need a connection to an RPC node via a `Neow3j` instance.
```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
```


## Contract Parameters

When invoking a smart contract, you will most likely use parameters, e.g. the method of the smart contract that is to be
invoked. If that method takes arguments, then we add these arguments as additional parameters to the invocation.

For parameter definition in neow3j, use the `ContractParameter` class. It provides many static construction methods that
cover all possible parameter types. If you use those methods, neow3j will make sure that the parameter is sent to the
contract in the correct encoding and the correct type declaration.

For example, if you need to pass a script hash of a NEO address as a parameter, you can use the method
`ContractParameter.hash160(...)`. It converts the script hash to the required byte array form.

When creating a parameter of the type byte array, make sure to read the method documentation which indicates with which
endianness the value has to be provided.

## Basic Contract Invocation

First, you have to specify which contract you want to invoke. Use the `ScriptHash` class for this and pass it the script
hash of the contract you want to call.

```java
ScriptHash scriptHash = new ScriptHash("0x1a70eac53f5882e40dd90f55463cce31a9f72cd4");
```

Then you need to define the parameters that will be passed to the contract. In this example, the method `register` is
called with a domain name and an address that should be registered under that domain name.

```java
Account account = Account.createAccount();
ContractParameter functionArg1 = ContractParameter.string("neo.com");
ContractParameter functionArg2 = ContractParameter.hash160(account.getScriptHash());
```

Observe that the address to register is not passed to the contract as a string but as a hash160, that's what the contract expects.
The static creation method `ContractParameter.hash160()` does that for you.

Here's the complete code.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));

Account account = Account.createAccount();
Wallet wallet = Wallet.withAccounts(account);

ScriptHash scriptHash = new ScriptHash("0x1a70eac53f5882e40dd90f55463cce31a9f72cd4");
ContractParameter functionArg1 = ContractParameter.string("neo.com");
ContractParameter functionArg2 = ContractParameter.hash160(account.getScriptHash());

NeoSendRawTransaction response = new SmartContract(scriptHash, neow3j)
        .invokeFunction("register", functionArg1, functionArg2)
        .wallet(wallet)
        .signers(Signer.calledByEntry(account.getScriptHash()))
        .sign()
        .send();
```

To make it more clear how the classes `SmartContract`, `TransactionBuilder` and `Transaction` are used, we split the above invocation
in their individual parts below. The method `invokeFunction()` builds an invocation script and passes it to a `TransactionBuilder`,
there the wallet and the signers are specified, the transaction is signed and the built `Transaction` is returned, which is then sent.

```java
SmartContract contract = new SmartContract(contract, neow3j);

TransactionBuilder builder = contract.invokeFunction("register", functionArg1, functionArg2);

Transaction tx = builder.wallet(wallet)
        .signers(Signer.calledByEntry(account.getScriptHash()))
        .sign();

NeoSendRawTransaction response = tx.send();
```

> **Note:** When the method `sign()` is called in the `TransactionBuilder`, the following two steps are executed:
>
> - A `Transaction` object is constructed.
> - For each signer, its corresponding account is fetched from the wallet, the signature for the
>   invocation script is created and added to the `Transaction` object.

## Testing the Invocation before propagating it

If you need to know the effect of your invocation before actually propagating it through the network, you can do a test
invocation first. This also calls the RPC node but only simulates the execution without any effects on the blockchain.
For this use the method `callInvokeFunction()` instead of `invokeFunction()` in the `SmartContract` class.

To do so, the contract parameters have to be packed in a list and the signers are passed to the method directly, since no
transaction is actually built.

```java
List<ContractParameter> params = Arrays.asList(functionArg1, functionArg2);
NeoInvokeFunction response = new SmartContract(contract, neow3j)
        .callInvokeFunction("register", params, Signer.calledByEntry(account.getScriptHash()));
```

The `NeoInvokeFunction` holds information about the GAS amount consumed in the contract execution, the VM exit state
(e.g. HALT or FAULT) and the VM's stack, i.e. the return values.

## Calling a Contract without Parameters

Of course, it is also possible to call a smart contract function that doesn't take any parameters. For example, a contract method
that simply increments a number every time it gets called.

```java
NeoSendRawTransaction response = new SmartContract(contract, neow3j)
        .invokeFunction("increment")
        .wallet(wallet)
        .signers(Signer.calledByEntry(account.getScriptHash()))
        .sign()
        .send();
```

## Adding additional network Fees

There are two kind of fees. System fees and network fees. The system fees are the cost of resources consumed by the transaction
execution in NeoVM. The total system fee depends on the number and type of instructions executed in the smart contract script.

Then there are network fees that will give the transaction priority in the network. To add a network fee, use the method
`additionalNetworkFee()` as in the example below.

```java
NeoSendRawTransaction response = new SmartContract(contract, neow3j)
        .invokeFunction("increment")
        .wallet(wallet)
        .additionalNetworkFee((long) 0.1)
        .signers(Signer.calledByEntry(account.getScriptHash()))
        .sign()
        .send();
```

You can check how much fees a transaction will cost with a test invocation call as described
[above](neo3_guides/contract_invocation.md#testing-the-invocation-before-propagating-it).

> **Note:** The node only returns one number for the amount of consumed gas, which corresponds to the sum of the system and the network fee.

<!-- ## Adding Transaction Attributes and Scripts
Extend as soon as transaction attributes are defined for Neo 3 -->

## Manually signing a Transaction

> **Important:** For multi-sig accounts this is done differently. See the next section.

If you don't have all private keys required to sign the transaction, you can still build the transaction by using the method
`getUnsignedTransaction()` in the `TransactionBuilder` and then manually add the signature/witness.

To do so, you can get the raw transaction byte array with the method `getHashData()`. Then you can sign this data by using the
static method `createWitness()` of the `Witness` class and add the created `Witness` to the `Transaction` with `addWitness()`.

An example scenario for this is when invoking a smart contract with a multi-sig account.

> **Note:** You still need to add the `Signer(s)` when building the `Transaction`. The accounts' addresses are still necessary for
> the building process. This means that in the `TransactionBuilder` only the signers and the optional network fee have to be specified
> (the wallet is only used for creating the signature).

In the following example the same transaction as in previous examples is created with manually adding the signature.

```java
Transaction tx = new SmartContract(contract, neow3j)
        .invokeFunction("register", functionArg1, functionArg2);
        .wallet(wallet)
        .signers(Signer.calledByEntry(account.getScriptHash()))
        .getUnsignedTransaction();

byte[] txBytes = tx.getHashData();
ECKeyPair keyPair account.getECKeyPair();

tx.addWitness(Witness.createWitness(txBytes, keyPair));
```

## Signing a Transaction with a multi-sig Account

Multi-sig accounts are usually not controlled by one single entity. Meaning the private keys of the involved key pairs
are not all available to sign a transaction locally. So this scenario is different in the signing step.

In the example below, a multi-sig account made up of two accounts (the account used in previous examples and a new `account2`)
is used. Its address is "ALK7evGaofZciCZu86K8bhXsUdpa5FcdJs". This account does not own the key material to properly sign
a transaction. Neow3j can only fetch the account's balances. Signing the transaction is up to you. It is the raw transaction
byte array that needs to be signed by the required number of keys. Then the signatures are combined in a witness script with
the static method `createMultiSigWitness()`.

When the witness is created, it can be added to the transaction and it is ready to be sent.

```java
Account account2 = Account.fromWIF("L45BGYyybk91pvwH3Mj1CfDZ11GGQLVPr6qfzpWugeP4WeJZyfki");
Account multiSigAccount = Account.fromAddress("ALK7evGaofZciCZu86K8bhXsUdpa5FcdJs");

functionArg2 = ContractParameter.hash160(multiSigAccount.getScriptHash());

Transaction tx = new SmartContract(contract, neow3j)
        .invokeFunction("register", functionArg1, functionArg2);
        .signers(Signer.calledByEntry(multiSigAccount.getScriptHash()))
        .getUnsignedTransaction();

byte[] unsignedTxHex = tx.getHashData();

List<ECPublicKey> keys = Arrays.asList(account1.getECKeyPair().getPublicKey(), account2.getECKeyPair().getPublicKey());
SignatureData sig1 = Sign.signMessage(unsignedTxHex, account1.getECKeyPair());
SignatureData sig2 = Sign.signMessage(unsignedTxHex, account2.getECKeyPair());

Witness witness = Witness.createMultiSigWitness(2, Arrays.asList(sig1, sig2), keys);

tx.addWitness(witness);
```

> **Note:** It is important that the raw transaction byte array is fetched with the method `getHashData()` and not
> `toArray()`! The part of the script that is excluded in the former is exactly what is created and appended with the witness.

Of course, in reality this is an unlikely example, since this means that you were in possession of both keypairs
(more specific - the private keys), which makes the usage of multi-sig accounts obsolete. But it shows the basic
requirements to make this transaction happen.

When an external signature is needed (i.e. only one private key for a multi-sig account with threshold 2 is held in the wallet),
you can communicate the raw byte array (`unsignedTxHex` in the example above) with a party that holds the private key for the
second account that can then sign it and send back the signature data in form of a byte array with the method `getConcatenated()`.

```java
SignatureData externalSig2 = Sign.signMessage(unsignedTxHex, account2.getECKeyPair());
byte[] signatureBytes = externalSig2.getConcatenated();
```

From these signature bytes, a `SignatureData` object can then be recreated with its static method `fromByteArray()`.

```java
SignatureData sig2 = SignatureData.fromByteArray(signatureBytes);
```

The last steps are again the same as in the first example in this section. The signatures are combined in a witness script,
added to the transaction and then the transaction can be sent.

```java
Witness witness = Witness.createMultiSigWitness(2, Arrays.asList(sig1, sig2), keys);

tx.addWitness(witness);
tx.send();
```
