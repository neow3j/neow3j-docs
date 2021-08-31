# Smart Contracts

To deploy, invoke or just retrieve information about any contract's state on the blockchain, the class `SmartContract`
can be used. It is the most generic class in the neow3j SDK that models a contract. All other contract classes are
subclasses of this one. On one hand, `SmartContract` offers methods to invoke a contract on the blockchain with the
result of an actual state change. On the other hand, you can use it to simulate an invocation without actually inducing
state chagnes. The latter is also used when the invocation only performs read actions. 

The only method that creates an actual transaction is `invokeFunction(...)`. It builds a script based on the provided
function name and contract parameters and constructs a `TransactionBuilder` with it. The returned transaction builder
can then be configured, built, and the transaction send as described in the Section
[Transactions](neo-n3/dapp_development/transactions.md). This process is visualized in the following figure.

```
         Build script           ->      Configure transaction       ->   Tx ready to sign and send

        ---------------                  --------------------                -------------
       | SmartContract |        ->      | TransactionBuilder |      ->      | Transaction |
        ---------------                  --------------------                -------------
```


The methods that don't change blockchain state all start with the word *call* to indicate that the performed invocation
is only a call to the Neo JSON-RPC API's `invokefunction` or `invokescript` methods. They don't produce transaction
builders or transactions.  There are several of such *call* methods, with the basic one being `callInvokeFunction(...)`.
If you know what type the contract invocation will return, you can use one of the more specific call methods that
already unpack the invocation result value, e.g., `callFunctionReturningScriptHash(...)`.

Because every contract has a contract manifest, `SmartContract` offers the method `getContractManifest()` that will
fetch the manifest.

## Contract Parameters

When invoking a smart contract, you will need method parameters. The neow3j SDK, represents parameters via the
`ContractParameter` class. It provides many static construction methods that cover all possible parameter types. If you
use those methods, neow3j will make sure that the parameter is sent to the contract in the correct encoding and the
correct type declaration. For example, if you need to pass a script hash of a NEO address as a parameter, you can use
the method `ContractParameter.hash160(...)`. It converts the script hash to the expected byte array.

If you invoke a contract that takes an object as a parameter, you need to use a contract parameter of type `Array`.
As an example, assume the that the `Bongo` class below is the expecte method parameter. 

```java
public class Bongo {

    public String lowNote;
    public String highNote;

    public Bongo(String lowNote, String highNote) {
        this.lowNote = lowNote;
        this.highNote = highNote;
    }
}
```

Using the neow3j SDK, you would have to construct the following parameter representing a `Bongo` instance.

```java
ContractParameter.array(
    ContractParameter.string("C2"), 
    ContractParameter.string("C5")));
```

The same applies when the object is used as a return type. In other words, expect a return value of
type `Array` that will hold the object's variables in the order they appear in the class.


## Contract Invocation

First, you have to specify which contract and which function you want to invoke. Use the `io.neow3j.contract.Hash160`
class for the contract's script hash and a simple string for the function.

```java
Hash160 scriptHash = new Hash160("0x1a70eac53f5882e40dd90f55463cce31a9f72cd4");
String function = "register";
```

Then you need to define the parameters that will be passed to the contract. In this example, we are invoking a name
service contract and call the `register` function with a domain name and an address that should be registered under that
domain name. Observe that the account to register is not passed as an address string but as a Hash160 parameter. That's
what the contract in this example expects. The developer needs to be aware of what parameter type a contract expects.

```java
Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
ContractParameter domainParam = ContractParameter.string("neo.com");
ContractParameter accountParam = ContractParameter.hash160(account.getScriptHash());
```

With the contract hash, the function, and the parameters, we can construct an invocation as follows. This doesn't yet
send a transaction but returns a transaction builder for further configuration.

```java
TransactionBuilder txBuilder = new SmartContract(scriptHash, neow3j)
        .invokeFunction(function, domainParam, accountParam);
```

Here's the complete code with the configuration of the transaction builder. The transaction is signed by the same
account that is registered under the domain name "neo.com".

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));

Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
Wallet wallet = Wallet.withAccounts(account);

Hash160 scriptHash = new Hash160("0x1a70eac53f5882e40dd90f55463cce31a9f72cd4");
String function = "register";

ContractParameter domainParam = ContractParameter.string("neo.com");
ContractParameter accountParam = ContractParameter.hash160(account.getScriptHash());

NeoSendRawTransaction response = new SmartContract(scriptHash, neow3j)
        .invokeFunction("register", domainParam, accountParam)
        .signers(AccountSigner.calledByEntry(account))
        .wallet(wallet)
        .sign()
        .send();
```

Of course, it is also possible to call a smart contract function that doesn't take any parameters, e.g., a
contract that simply increments a number every time it gets called.

```java
NeoSendRawTransaction response = new SmartContract(contract, neow3j)
        .invokeFunction("increment")
        .signers(AccountSigner.calledByEntry(account))
        .wallet(wallet)
        .sign()
        .send();
```

## Testing the Invocation

If you need to know the effect of your invocation before actually propagating it through the network, you can do a test
invocation first. This also calls the RPC node but only simulates the execution without any effects on the blockchain.
Continuing the above example of the domain name contract, a test invocation would look like the following. Notice that
depending on what call you perform, you also need to add signers even though no blockchain state is changed. In this
specific example the signer is needed because in the called contract it will be verified if the account that is
registered is also the sender of the transaction.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));

Hash160 scriptHash = new Hash160("0x1a70eac53f5882e40dd90f55463cce31a9f72cd4");
String function = "register";

Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
ContractParameter domainParam = ContractParameter.string("neo.com");
ContractParameter accountParam = ContractParameter.hash160(account.getScriptHash());
List<ContractParameter> params = Arrays.asList(domainParam, accountParam);

NeoInvokeFunction response = new SmartContract(scriptHash, neow3j)
        .callInvokeFunction(funtion, params, AccountSigner.calledByEntry(account));
```

The `NeoInvokeFunction` holds information about the GAS amount consumed in the contract execution, the VM exit state
(e.g. HALT or FAULT), and the VM's stack, i.e. the return value.

## Contract Interfaces

There are several subclasses of `SmartContract` that implement a sort of interface to Neo's native contracts and
contracts that follow token standards. Here, the word interface means that these classes provide an interface to
interact with the deployed contracts. Token contracts are discussed separately in Section [Token
Contracts](neo-n3/dapp_development/token_contracts.md). They include fungible and non-fungible tokens such as `NeoToken`
and `GasToken`. The other available contract classes are discussed below. 

### ContractManagement

The `ContractManagement` contract is a Neo native contract that, as it's name suggests, can be used to manage other
contracts. More precisely, it allows you to deploy, update, and delete a contract. In the neow3j SDK the update and
delete methods cannot be used, because they can only be called from within another contract. But, the deploy method is
available and allows you to deploy new contracts with neow3j. For example:

```java
Transaction tx = new ContractManagement(neow3j)
        .deploy(nef, manifest)
        .signers(calledByEntry(account1.getScriptHash()))
        .wallet(w)
        .sign();
```

Producing the necessary NEF (Neo Executable Format) file and contractd manifest is not discussed here but are part of
the Contract Development [section]().

There are two other methods on `ContracManagement`. They are concerned with the minimum deployment fee. The getter
`getMinimumDeploymentFee()` can be used by anyone. But, the setter `setMinimumDeploymentFee(...)` can only successfully
be used if the transaction is signed by committee members. I.e., it will be of no use to most developers.


### PolicyContract

The `PolicyContract` holds information about several settings of the Neo network. You can retrieve information like GAS
fee per transaction byte, the GAS price per byte of contract storage, or if a certain account is blacklisted.

The contract also provides setters for all of these values, though, these can only successfully be used if the
transaction is signed by committee members. 


### RoleManagement

The `RoleManagament` contract is used to assign roles to nodes in the network. A node can be a state validator, an
oracle node, or a NeoFS "Alphabet" node (respondible for consensus on NeoFS sidechain). 

The designation of a node to a role can only be done via the Neo committee. But you can check the role assignments with
the `getDesignatedRole(...)` method.


### NeoNameService

The `NeoNameService` is not a native contract but managed by the Neo core team. The script hash of this contract is not
known to neow3j and has to be provided by the developer when constructing an instance of `NeoNameService`. As it's name
suggest the name service contract provides the possibility to map a name to an owner account. You can read more about it
in the official [Neo Docs](https://docs.neo.org/docs/en-us/reference/nns.html).

The `NeoNameService` class in the neow3j SDK provides you with all the methods of the contract. So you can check
registered names and register your own name-to-address mappings. Some of them can only be called by the Neo committee,
e.g., `addRoot(...)` or `setPrice(...)` methods.