# Smart Contracts

To deploy, invoke, or retrieve information about any contract's state on the blockchain, you can use the `SmartContract` class. It is the most general class in the neow3j SDK that models a contract. All other contract classes are subclasses of this one. On one hand, `SmartContract` offers methods to invoke a contract on the blockchain resulting in an actual state change. On the other hand, you can use it to simulate an invocation without actually changing the state. The latter is also used when the invocation only performs read actions.

The only method that creates an actual transaction is `invokeFunction(...)`. It builds a script based on the provided function name and contract parameters and constructs a `TransactionBuilder` with it. The returned transaction builder can then be configured, built, and the transaction sent as described in the [Transactions](neo-n3/dapp_development/transactions.md) section. This process is visualized in the following figure.

```
         Build script           ->      Configure transaction       ->   Tx ready to sign and send

        ---------------                  --------------------                -------------
       | SmartContract |        ->      | TransactionBuilder |      ->      | Transaction |
        ---------------                  --------------------                -------------
```

The methods that don't change blockchain state all start with the word "call" to indicate that the performed invocation is only a call to the Neo JSON-RPC API's `invokefunction` or `invokescript` methods. They don't produce transaction builders or transactions. There are several of such "call" methods, with the basic one being `callInvokeFunction(...)`. If you know what type the contract invocation will return, you can use one of the more specific call methods that already unpack the invocation result value, e.g., `callFunctionReturningScriptHash(...)`.

Contrary to the return type, each method of a contract takes specific parameter types as input. To know what methods exist in a smart contract and what parameter types it requires, every contract has a manifest that provides you with this and much more information about the contract. In the following sections, we'll show you how to create contract parameters, get the manifest, and figure out what parameters you need to pass to a function you want to invoke.

## Contract Parameters

When invoking a smart contract, you will need parameters. The neow3j SDK represents parameters via the `ContractParameter` class. It provides many static construction methods that cover all possible parameter types. If you use those methods, neow3j will make sure that the parameter is sent to the contract in the correct encoding and the correct type declaration. For example, if you need to pass a script hash of a NEO address as a parameter, you can use the method `ContractParameter.hash160(...)`. It converts the script hash to the expected byte array.

If you invoke a contract that takes an object as a parameter, you need to use a contract parameter of type `Array`. As an example, assume that the `Bongo` struct class below is the expected method parameter.

```java
@Struct
public class Bongo {

    public String lowNote;
    public String highNote;

    public Bongo(String lowNote, String highNote) {
        this.lowNote = lowNote;
        this.highNote = highNote;
    }
}
```

Using the neow3j SDK, you would need to construct the following parameter to represent a `Bongo` instance.

```java
ContractParameter.array(
    ContractParameter.string("C2"),
    ContractParameter.string("C5"));
```

The same concept applies when the object is used in a return type. In other words, expect a return value of type `Array` that holds the object's variables in the order they appear in the class.

## Contract Invocation

First, specify which contract and which function you want to invoke. Use the `io.neow3j.contract.Hash160`
class for the contract's script hash and create a `SmartContract` along with a `Neow3j` instance.

```java
Hash160 scriptHash = new Hash160("0x1a70eac53f5882e40dd90f55463cce31a9f72cd4");
SmartContract smartContract = new SmartContract(scriptHash, neow3j);
```

Then, define the parameters that will be passed to the contract. To determine the required parameters, you need information about its methods. You can obtain this information by reading the contract's ABI in the manifest. The following example shows how to retrieve all methods of a smart contract with their names, parameters, and return type to understand what parameters are needed for a function and what you can expect to be returned:

```java
List<ContractManifest.ContractABI.ContractMethod> methods = smartContract.getManifest().getAbi().getMethods();
```

In this example, we are invoking a name service contract and calling the `register` function with a domain name and an
address that should be registered under that domain name.

```java
Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
ContractParameter domainParam = ContractParameter.string("myname.neo");
ContractParameter accountParam = ContractParameter.hash160(account.getScriptHash());
```

With the smart contract instance, the function, and the parameters, we can construct an invocation as follows. This
does not yet send a transaction, but it builds the correct script and instantiates a transaction builder with it for
further configuration.

```java
String function = "register";
TransactionBuilder txBuilder = smartContract.invokeFunction(function, domainParam, accountParam);
```

Here's the complete code with the configuration of the transaction builder. The transaction is signed by the same
account that is registered under the domain name "myname.neo".

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
Hash160 scriptHash = new Hash160("0x1a70eac53f5882e40dd90f55463cce31a9f72cd4");
SmartContract smartContract = new SmartContract(scriptHash, neow3j);

Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
ContractParameter domainParam = ContractParameter.string("myname.neo");
ContractParameter accountParam = ContractParameter.hash160(account.getScriptHash());
String function = "register";

NeoSendRawTransaction response = new SmartContract(scriptHash, neow3j)
        .invokeFunction(function, domainParam, accountParam)
        .signers(AccountSigner.calledByEntry(account))
        .sign()
        .send();
```

Of course, it is also possible to call a smart contract function that doesn't take any parameters, for example, a contract that simply increments a number every time it gets called.

```java
NeoSendRawTransaction response = new SmartContract(contract, neow3j)
        .invokeFunction("increment")
        .signers(AccountSigner.calledByEntry(account))
        .sign()
        .send();
```

## Testing the Invocation

If you need to know the effect of your invocation before actually propagating it through the network, you can do a test invocation first. This also calls the RPC node but only simulates the execution without any effects on the blockchain. Continuing the above example of the domain name contract, a test invocation would look like the following. Notice that depending on what call you perform, you also need to add signers even though no blockchain state is changed. In this specific example, the signer is needed because the called contract will verify if the account that is registered is also the sender of the transaction.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));

Hash160 scriptHash = new Hash160("0x1a70eac53f5882e40dd90f55463cce31a9f72cd4");
String function = "register";

Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
ContractParameter domainParam = ContractParameter.string("neo.com");
ContractParameter accountParam = ContractParameter.hash160(account.getScriptHash());
List<ContractParameter> params = Arrays.asList(domainParam, accountParam);

NeoInvokeFunction response = new SmartContract(scriptHash, neow3j)
        .callInvokeFunction(function, params, AccountSigner.calledByEntry(account));
```

The `NeoInvokeFunction` holds information about the GAS amount consumed in the contract execution, the VM exit state (e.g., HALT or FAULT), and the VM's stack, i.e., the return value.

## Contract Interfaces

There are several subclasses of `SmartContract` that implement a type of interface to Neo's native contracts and contracts that follow token standards. Here, the term "interface" means that these classes provide a way to interact with the deployed contracts. Token contracts, including fungible and non-fungible tokens such as `NeoToken` and `GasToken`, are discussed separately in Section [Token Contracts](neo-n3/dapp_development/token_contracts.md). The other available contract classes are discussed below.

### ContractManagement

The `ContractManagement` contract is a native Neo contract that, as its name suggests, can be used to manage other contracts. More specifically, it allows you to deploy, update, and delete a contract. In the neow3j SDK, the update and delete methods cannot be used because they can only be called from within another contract. However, the deploy method is available and allows you to deploy new contracts with neow3j. For example:

```java
Transaction tx = new ContractManagement(neow3j)
        .deploy(nef, manifest)
        .signers(AccountSigner.calledByEntry(account1)))
        .sign();
```

Producing the necessary NEF (Neo Executable Format) file and contract manifest is not discussed here but are part of the Contract Development [section]().

There are two other methods on `ContractManagement`. They are concerned with the minimum deployment fee. The getter `getMinimumDeploymentFee()` can be used by anyone. However, the setter `setMinimumDeploymentFee(...)` can only successfully be used if the transaction is signed by committee members, meaning it will be of no use to most developers.

### PolicyContract

The `PolicyContract` holds information about several settings of the Neo network. You can retrieve information like GAS fee per transaction byte, the GAS price per byte of contract storage, or if a certain account is blacklisted.

The contract also provides setters for all of these values, though they can only successfully be used if the transaction is signed by committee members.

### RoleManagement

The `RoleManagement` contract is used to assign roles to nodes in the network. A node can be a state validator, an oracle node, or a NeoFS "Alphabet" node (responsible for consensus on NeoFS sidechain).

The designation of a node to a role can only be done via the Neo committee. However, you can check the role assignments with the `getDesignatedRole(...)` method.

### NeoNameService

The `NeoNameService` is not a native contract but managed by the Neo core team. The script hash of this contract is not known to neow3j and has to be provided by the developer when constructing an instance of `NeoNameService`. As its name suggests, the name service contract provides the possibility to map a name to an owner account. You can read more about it in the official [Neo Docs](https://docs.neo.org/docs/n3/Advances/neons/index.html).

The `NeoNameService` class in the neow3j SDK provides you with all the methods of the contract. So you can check registered names and register your own name-to-address mappings. Some of them can only be called by the Neo committee, for example, `addRoot(...)` or `setPrice(...)` methods.
