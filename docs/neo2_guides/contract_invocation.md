# Invoking Smart Contracts

Smart contract invocations are handled by the `ContractInvocation` class. In any scenario, you will need a connection to
an RPC node via a `Neow3j` instance.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
```

If a fee is added to an invocation, neow3j needs to select appropriate transaction inputs to cover that fee. Therefore,
the RPC node in use needs to have the `RpcSystemAssetTrackerPlugin` installed (see the
[Requirements](overview/requirements?id=rpc-nodes) section).


## Contract Parameters

When invoking a smart contract, you will most likely use parameters. For example, when invoking a contract that follows
the NEP-3 standard, we will need to provide the desired method that we want to call as a parameter. If that method takes
arguments, then we add these arguments in an additional parameter to the invocation. 

For parameter definition in neow3j, use the `ContractParameter` class. It provides many static construction methods that
cover all possible parameter types. If you use those methods neow3j will make sure that the parameter is sent to the
contract in the correct encoding and the correct type declaration.

When creating a parameter of the type byte array, make sure to read the method documentation which indicates with which
endianness the value has to be provided.

If you need to pass a script hash of a NEO address as a parameter, you can use the method
`ContractParameter.byteArrayFromAddress(...)`. It converts the address to its script hash in byte array form.


## Basic Contract Invocation

First, you have to specify which contract you want to invoke. Use the `ScriptHash` class for this and pass it the script
hash of the contract you want to call.

```java
ScriptHash scriptHash = new ScriptHash("1a70eac53f5882e40dd90f55463cce31a9f72cd4");
```

Then you need to define the parameters that will be passed to the contract.  In this example, the method `register` is
called with a domain name and an address that should be registered under that domain name. The method arguments have to
be packed together in an array parameter.

```java
ContractParameter methodParam = ContractParameter.string("register");
ContractParameter argumetnsParam = ContractParameter.array(
            ContractParameter.string("neo.com"),
            ContractParameter.byteArrayFromAddress("AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y"));
```
Observe that the address to register is not passed to the contract as it is (i.e. as a string), but is converted to a
byte array. More precisely, it is the script hash of that address that is sent to the contract. That's what the contract
expects. The static creation method `ContractParameter.byteArrayFromAddress(...)` does all that for you.

Finally, you need to specify which account should be used to sign the invocation. The example simply uses a newly
created account. The account is needed so that neow3j can automatically attach the needed signature when calling the
`InvocationTransaction.sign()` method.

Here's the complete code.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
ScriptHash scriptHash = new ScriptHash("1a70eac53f5882e40dd90f55463cce31a9f72cd4");
ContractParameter methodParam = ContractParameter.string("register");
ContractParameter argumetnsParam = ContractParameter.array(
            ContractParameter.string("neo.com"),
            ContractParameter.byteArrayFromAddress("AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y"));
Account account = Account.createAccount();

ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(scriptHash)
        .parameter(methodParam)
        .parameter(argumentsParam)
        .account(account)
        .build()
        .sign()
        .invoke()
```

The `invoke()` method does not return information about the state of the invocation, but it will throw an exception in
case the invocation ran into an error while executing on the RPC node. More information about the error can be retrieved
from the thrown `ErrorResponseException`.


## Testing the Invocation before propagating it

If you need to know the effect of your invocation before actually propagating it through the network, you can do a test
invocation first. This also calls the RPC node but only simulates the execution without any effects on the blockchain.

The setup of the `ContractInvocation` object is the same as in a normal invocation.

```java
ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(scriptHash)
        .parameter(methodParam)
        .parameter(argumentsParam)
        .account(account)
        .build()
        .sign();

InvocationResult result = invoc.testInvoke();

// Inspect the invocation result and then sign and invoke for real.

invoc.invoke();
```

The `InvocationResult` holds information about the GAS amount consumed in the contract execution, the VM exit state
(e.g. HALT or FAULT) and the VM's stack, i.e. the return values.

!> At the time of writing, the test invocation might not work properly in NEO 2. The problem lies in the
`CheckWitness()` method used in smart contracts. If the test invocation runs through a `CheckWitness()` call, it will
fail because the witness is not correctly validated. Refer to the related
[Github issue](https://github.com/neo-project/neo/pull/335).


## Calling a Contract without Parameters

Of course, it is also possible to call a smart contract that doesn't take any parameters. For example, a contract that
simply increments a number every time it gets called. The invocation calls the main method (entry point) of the smart
contract.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));

ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(new ScriptHash("bff561a41a780fa0a4771d03bcc924e90c04fc8e"))
        .account(Account.createAccount())
        .build()
        .sign()
        .invoke();
```


## Calling a specific Contract Function 

Usually, a smart contract provides a standard entry point that takes a parameter that specifies which function of the
contract should be called. If this is not the case you can also call a function directly.
Use `ContractInvocation.function(...)` for this.

The parameters that are passed to the function are defined differently in comparison to the
[basic example](guides/contract_invocation?id=basic-contract-invocation). They are not combined in an array but added
separately. This is correct if the specific function takes two separate parameters. You need to know the method
signatures of the contract well, to be able to construct valid invocations.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));

ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(new ScriptHash("1a70eac53f5882e40dd90f55463cce31a9f72cd4"))
        .function("register")
        .parameter(ContractParameter.string("neo.com"))
        .parameter(ContractParameter.byteArrayFromAddress("AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y"))
        .account(Account.createAccount())
        .build()
        .sign()
        .invoke();
```


## Adding Transaction Fees

As in any other transaction, you can add a network fee to your invocation. This will give it priority in the network.
Additionally, you might need to add a system fee. The system fee depends on the GAS consumption of your invocation.
When you invoke a contract, the contract execution consumes a certain amount of GAS. You can check that amount with a
test invocation as described [above](guides/contract_invocation?id=testing-the-invocation-before-propagating-it). 

In each invocation, the first 10 GAS is for free. If more GAS is consumed, the additional amount has to be added in the
form of the system fee as shown in the example below.

If any fee is attached, the account used in the invocation needs to provide the required transaction inputs. So you
might need to update the account's balances before invoking.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));

Account account = Account.createAccount();
account.updateAssetBalances(neow3j);

ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(new ScriptHash("1a70eac53f5882e40dd90f55463cce31a9f72cd4"))
        .account(account)
        .networkFee(0.1)
        .systemFee(2.3)
        .build()
        .sign()
        .invoke();
```


## Adding Transaction Attributes and Scripts

If your contract expects certain other information to be provided along with the invocation, you might need to add
attributes and scripts.

Neow3j already adds some attributes automatically if your invocation does not include any outputs (e.g. fee payments).
This includes a script attribute that tells the NEO VM which address it should use for validating the transaction's
witness. The second attribute is a remark attribute consisting of some random value. It makes the transaction unique and
therefore repeatable.

Custom attributes, as well as witnesses, can be added when building a `ContractInvocation` as shown below.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account account = Account.createAccount();

RawTransactionAttribute attr = new RawTransactionAttribute(TransactionAttributeUsageType.SCRIPT, account.getScriptHash().toArray())

RawScript script = new RawScript(new byte[]{0x0a, 0x21}, new byte[]{0x3a, 0x04});

ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(new ScriptHash("1a70eac53f5882e40dd90f55463cce31a9f72cd4"))
        .account(account)
        .attribute(attr)
        .witness(script)
        .build()
        .sign()
        .invoke();
```


## Manually signing the Invocation

If you can't set up the `ContractInvocation` with an `Account` object that contains the private key material to
automatically sign the transaction, you can manually add the signature/witness. An example scenario for this is when
invoking with a multi-sig account.

You still need to add the `Account` when building the `ContractInvocation`. The account's address is still necessary for
the building process. After building, instead of calling the `sign()` method, you need to call the `addWitness(...)`
method with a custom witness. You can obtain the raw transaction array for signing after building the
`ContractInvocation`.

The example below uses a multi-sig account made up of two accounts (`keyPair1` and `keyPair2`). Its address is
"ATcWffQV1A7NMEsqQ1RmKfS7AbSqcAp2hd". The contract invocation is simple and does not use any parameters. From the
`ContractInvocation` object the raw transaction is generated and a witness is created from it.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account multiSigAcct = Account.fromAddress("ATcWffQV1A7NMEsqQ1RmKfS7AbSqcAp2hd").build();
ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(new ScriptHash("1a70eac53f5882e40dd90f55463cce31a9f72cd4"))
        .account(multiSigAcct)
        .build();

byte[] unsignedTxHex = at.getTransaction().toArrayWithoutScripts();
SignatureData sig1 = Sign.signMessage(unsignedTxHex, keyPair1);
SignatureData sig2 = Sign.signMessage(unsignedTxHex, keyPair2);
RawScript witness = RawScript.createMultiSigWitness(2, Arrays.asList(sig1, sig2), keys);

at.addWitness(witness).invoke();
```
