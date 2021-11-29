# Devpack

The neow3j devpack provides classes, methods and annotations required for writing smart contracts in
Java. For example, if your smart contract needs to verify a transaction signature, the devpack offers
a method for that. Or, if you want to publish detailed information about the contract in its
manifest, you can use one of the devpack's annotations.
The following sections describe the devpack's API, frequently used concepts and constructs. 


## Neo Smart Contract API

The biggest part of the devpack is the Neo smart contract API. It provides many functionalities, for
example, retrieving blockchain information, access a contract's storage area, and interact with the
execution environment in which a contract is run. This API is the same for all Neo smart contract devpacks, e.g.
[neo-boa](https://github.com/CityOfZion/neo-boa) or [neo-go](https://github.com/nspcc-dev/neo-go).
You can find the documentation on it in the official 
[Neo docs](https://docs.neo.org/v3/docs/en-us/reference/scapi/fw/dotnet/neo.html). 


## Hashes

As with neow3j SDK, the devpack provides special types for hashes too. Use `Hash160` for contract and account hashes and
`Hash256` for transaction and block hashes. The underlying stack item of both of these types is a NeoVM byte string,
thus, changing to and from `ByteString` doesn't require an actual conversion. Though, when you use the constructors
`Hash160(ByteString value)` an `Hash256(ByteString value)` the devpack inserts checks that make sure the value is a
valid hash of the respective size. If it is not, the VM will stop in a FAULT state.  
You can use `Hash160.isValid(Object obj)`/`Hash256.isValid(Object obj)` before using the constructor to check if the
object is a valid hash. That way you don't risk a VM halt.

## Storage

Every smart contract on the Neo blockchain has its own key-value storage. This storage is accessed
via a so called storage context. The context is the gateway to the contract's storage. This
additional concept between you and the storage potentially allows you to pass the context to another
contract, which could then access your contract's storage directly. 

In the devpack, the storage context is represented by the `io.neow3j.devpack.neo.StorageContext` class. The pivotal
class related to contract storage is `io.neow3j.devpack.neo.Storage`. It provides many `put(...)` and `get(...)` methods
for writing to the storage and reading from it. Each of theses methods requires the storage context as an argument.
Thus, it makes sense to retrieve the `StorageContext` once with `Storage.getStorageContext()`, store it in a static
class variable and reuse it every time the storage is accessed. This will save GAS in contract invocations.

Note that the size for storage keys and values is limited to 64 bytes and 65535 bytes, respectively. When using the
`StorageMap` class, the map prefix counts towards the key size.


<!-- Storing and loading objects to and from a contract's storage is possible by using the `StdLib`
native contract. Use `StdLib.serialize(Object obj)` before writting the object to storage and
`StdLib.deserialize(byte[] bytes)` after fetching the object from storage. -->


## Smart Contract Interfaces

Your smart contract can call other contracts via the `Contract.call(Hash160 scriptHash, String method, byte callFlags,
Object[] arguments)` method in an adhoc way. But, there is a another way of doing this, and that is by using what we
call a contract interface. In the context of the devpack, contract interfaces are classes that provide an
interface to deployed contracts. The word interface is not used in Java's meaning but in the meaning of a gateway to the
actual contract instance on the blockchain. For example, the `NeoToken` class holds all methods of the native NeoToken
contract and allows you to call it from within your own smart contract. These contract interface classes are located in
the `io.neow3j.devpack.contracts` package. All of Neo's native contracts are represented here, plus some other classes,
e.g., interfaces to access token contracts.

If you need to call a method of a native contract, e.g., get the hash of the latest block, use the corresponding static
method on the contract interface. An overview of the functionality of the native contracts is described in 

```java
Hash256 blockHash = LedgerContract.currentHash();
```

Additionally to the existing contract interfaces, you can define your own. The requirements for a valid contract interface
are:
- extend the `io.neow3j.devpack.contracts.ContractInterface` abstract class. 
- annotated the class with the `io.neow3j.devpack.annotations.ContractHash` annotation.

A minimal version of a custom contract interface looks like this.

```java
@ContractHash("0xef4073a0f2b305a38ec4050e4d3d28bc40ea63f5")
class MyContract extends ContractInterface {
}
```

In this minimal form the class only provides access to the contract's hash via the `getHash()` method inherited from
`ContractInterface`. Any other contract methods have to be added according to their signature in the contract's
manifest. Assuming the contract has a method `findElement` with a `ByteString` parameter and a `ByteString` return type,
you would need to add the following method. Note that the method needs to be static and native. A method body
implementation is not necessary, since this is only an interface to an actual contract instance on the blockchain.

```java
public static native ByteString findElement(ByteString key);
```

The devpack provides abstract contract interfaces that already the API of contracts following a standard. For example,
if you want to establish a contract interface to a fungible token contract extend the `FungibleToken` class. All methods
of a NEP-17 token contract are already available and the only thing you need to add is the contract hash annotation with
the hash of the target contract. If that contract has some extra methods, simply add them as shown before.

```java
@ContractHash("0xef4073a0f2b305a38ec4050e4d3d28bc40ea63f5")
class MyTokenContract extends FungibleToken {

    public static native int someCustomMethod(String arg);

}
```

Checkout the `io.neow3j.devpack.contracts` package for more such contract interfaces.

## Native Contracts

### StdLib

When using the `StdLib.jsonSerialize(Object o)` method for a value or object that contains a byte string or byte array
(e.g., `ByteString`, `byte[]`, or `Hash160`) make sure to first Base64-encoded that value. Otherwise, it will be
interpreted as a UTF-8 encoded string, which might lead to errors. It will not be presented in the JSON as a hexadecimal
string.

### Neo Name Service

The Neo Name Service contract (NNS) is technically not a native contract but is maintained and issued by the Neo
Foundation. The devpack provides a contract interface to the NNS with the class
`io.neow3j.devpack.contracts.NeoNameService`. Note, that this class does not have a fixed script hash, since it is not a
native contract. If you want to use the class in your contract you have to extend it and annotate the extending class
with the `@ContractHash` annotation.

```java
@ContractHash("a92fbe5bf164170a624474841485b20b45a26047")
class MyNeoNameService extends NeoNameService { }
```

Make sure that the script hash is equal to the current script hash of the NNS contract.

## Events

Neo smart contracts can fire events. They appear, for example, in the [application
logs](https://docs.neo.org/v3/docs/en-us/reference/rpc/latest-version/api/getapplicationlog.html) of a contract
invocation. The events that a contract can fire are listed in its manifest. The JSON below shows how this could look.

```json
"events": [
    {
        "name": "transfer",
        "parameters": [
            {
                "name": "arg1",
                "type": "Integer"
            },
            {
                "name": "arg2",
                "type": "String"
            }
        ]
    }
]
```

An event is defined by its name and the state parameters that are passed with it. The devpack allows you to define and
use events with up to 16 state parameters. The classes representing these events are located in the
[`io.neow3j.devpack.events`](https://javadoc.io/doc/io.neow3j/devpack/latest/io/neow3j/devpack/events/package-summary.html)
package. 

Events are declared in static contract variables as shown in the following code snippet. They cannot
be declared inside of a method body or in classes that are not the main contract class.

```java
    @DisplayName("mint")
    private static Event1Arg<Integer> onMint;

    @DisplayName("transfer")
    private static Event2Args<Integer, String> onTransfer;
```

It is not necessary to initialize the variables with an actual instance. This is counter-intuitive for a Java developer,
but, the variables are not meant to have an actual value. They are only definitions, with a name and the number and
types of state parameters. All event classes follow the naming schema `Event[n]Args`, where `n` is the number of state
parameters the event takes. The `@DisplayName` annotation is optional and can be used to define a different name for the
event than the variable name. If it is not used, the variable name is the event name.

Once an event is declared, it can then be used in contract methods by calling its `fire(...)` method. 

```java
    public static boolean transfer() throws Exception {
        ...
        onTransfer.fire(transferAmount, "tokens transferred!");
        ...
        return true;
    }
```

>**Note:** Events are not allowed to be fired in the [verify method](neo-n3/smart_contract_development/devpack.md?id=verify).

## Special Contract Methods

There are a couple of contract methods that have a special purpose. The neow3j devpack provides annotations to mark them
in your contract code. Using the annotations will make it easier to spot the methods in your code and allows the
compiler to make checks that help finding errors in these methods faster.

### _deploy

This method is called right after a contract is deployed or updated. More precisely, the *ContractManamgement* contract
will call this method on your contract when you invoke `deploy` or `update` on the *ContractManagement*. You can use it
to setup and configure your contract at deploy-time. 
Use the devpack's `io.neow3j.devpack.annotations.OnDeployment` annotation on the designated method. Your method's name
does not have to be `_deploy`, but can be anything. Although, in the contract manifest it will show up under the name
`_deploy`. The method's signature must be `void methodName(Object data, boolean isUpdate)`.

### verify

This method is called if your contract is invoked with the verification trigger. For example, when a contract owns
tokens and you issue a withdraw transaction that transfers those tokens to another account/contract, the contract
is called with the verification trigger. In other words the contract's `verify` method is called. Most often the
`verify` method contains a simple witness check on the owner of the contract.
Use the devpack's `io.neow3j.devpack.annotations.OnVerification` annotation on the designated method. Your method's name
does not have to be `verify`, but can be anything. Although, in the contract manifest it will show up under the name
`verify`. The method must return a boolean and can have any number of parameters.

>**Note:** The Neo node does not allow the verify method to fire any event. The compiler will throw an exception
>if an event is fired within this method.

### onNEP17Payment

Your contract requires this method to be able to receive tokens from NEP-17 contracts, i.e., fungible tokens like NEO or
GAS. Any contract that follows the NEP-17 standard will call this method on your contract if some of its tokens are
transferred to your contract. 
Use the devpack's `io.neow3j.devpack.annotations.OnNEP17Payment` annotation on the designated method. Your method's name
does not have to be `onNEP17Payment`, but can be anything. Although, in the contract manifest it will show up under the
name `onNEP17Payment`. The method's signature must be `void methodName(Hash160 sender, int amount, Object data)`.

### onNEP11Payment

Your contract requires this method to be able to receive tokens from NEP-17 contracts, i.e., non-fungible tokens. Any
contract that follows the NEP-11 standard will call this method on your contract if some of its tokens are transferred
to your contract. Use the devpack's `io.neow3j.devpack.annotations.OnNEP11Payment` annotation on the designated method.
Your method's name does not have to be `onNEP11Payment`, but can be anything. Although, in the contract manifest it will
show up under the name `onNEP11Payment`. The method's signature must be `void methodName(Hash160 sender, int amount,
ByteString tokenId, Object data)`.


## Permissions, Trusts, Groups, Safe Methods, and Call Flags

The basis of authorization on the Neo blockchain are cryptographic signatures. Users attach signatures to contract
invocations to proove that they are authorized to perform certain actions in a smart contract. This authorization can be
misused by smart contracts. A malicious contract can use the signature to perform a token transfer unintended by the
user. To prevent that, Neo applies witness scopes that allow the user to restrict the use of their witness/signature. By
default a witness is only valid in the contract that is the entry point of an invocation. The scope can be extended to
specific contracts, groups of contracts or to a global scope.

### Groups

Smart contract groups are designated by an EC public key. The contract manifest contains the affiliation of a contract
with a group as shown below.

```json
"groups": [
    {
        "pubkey":"033a4d051b04b7fc0230d2b1aaedfd5a84be279a5361a7358db665ad7857787f1b",
        "signature":"DEA2r+67KlQc/dIvEdOXMIGCm7x+V5vXT5ZRtGRwOlDxBuqzur/OU8OSitiYn5f6tQywog8FziGp5S2VI2/5Wnxj"
    }
],
```

The public key identifies the group and the signature is proof that the originator of the contract was in possession of
the corresponding private key material. The signature is created from the contract's hash and needs to be
Base64-encoded. Thus, if you want to add your contract to a group you first need to compile it, calculate the contract
hash, create the signature over that hash and extend the contract manifest with the group's public key and the produced
signature. Note, that the contract hash depends on the account used to deploy the contract. I.e., you need to know in
advance which account you will use to deploy the contract. To retrieve the contract hash and produce the signature you
can use the neow3j SDK as in the following example code.

```java
Hash160 sender = ...;
ContractManifest manifest = ...;
NefFile nefFile = ...;
Hash160 contractHash = SmartContract.calcContractHash(sender, nefFile.getCheckSumAsInteger(), manifest.getName());

ECKeyPair keyPair = ...;
Sign.SignatureData sig = Sign.signMessage(contractHash.toArray(), keyPair);
String encSig = Base64.encode(sig.getConcatenated());
```

You will have to modify the contract manifest JSON file and add the produced encoded signature and the public
key to the `groups` section manually.

### Permissions 

Besides witness scopes, smart contract security is improved by a system of permissions and trusts that a contract
developer can define for her contract. Permissions define which contracts your contract is permitted to call. They are
actively enforced, meaning that once defined in the contract's manifest, any calls from within your contract to
contracts and methods not contained in the permissions will fail. To define permissions use the 
`io.neow3j.devpack.annotations.Permission` annotation on class level of your contract class. By default your contract
will have no permissions.

The following is an example configuration. It allows your contract to call any method of the contract
with hash `726cb6e0cd8628a1350a611384688911ab75f51b`, the methods `getBalance` and `transfer` of the contract with hash
`d2a4cff31913016155e38e474a2c06d08be276cf`, and the method `commonMethodName` of any contract in the group with public key
`033a4d051b04b7fc0230d2b1aaedfd5a8  4be279a5361a7358db665ad7857787f1b`.

```java
@Permission(contract = "726cb6e0cd8628a1350a611384688911ab75f51b", methods = "*")
@Permission(contract = "d2a4cff31913016155e38e474a2c06d08be276cf", methods = {"getBalance", "transfer"})
@Permission(contract = "033a4d051b04b7fc0230d2b1aaedfd5a84be279a5361a7358db665ad7857787f1b", methods = "commonMethodName")
public class MyContract {
```

To set a permission for a native contract, you can instead use the `nativeContract` field in the annotation together with
the enum `NativeContract`. As you may have noticed, the second permission in the example code above refers to the native
GasToken contract. As it is a native contract, you can instead also use the following annotation, which results in the
exact same outcome.

```java
@Permission(nativeContract = NativeContract.GasToken, methods = {"getBalance", "transfer"})
```

If you want to allow your contract to call any other contract, use the wildcard permission 
`@Permission(contract = "*", methods = "*")`.

### Trusts

Trusts define what contracts can call your contract, but, in contrast to permissions they are not enforeced. I.e., you
cannot deter other contracts from calling yours. Trusts are only a definition that wallets and other dApps can use to
tell the user when a contract is invoked that doesn't trust the calling contract.

By default the trust property is empty, i.e., no contracts are trusted. Use the `io.neow3j.devpack.annotations.Trust`
annotation to define trusts like in the following example. The first entry is based on a single smart contract hash and
the second one on a public key of a contract group.

```java
@Trust(contract = "acce6fd80d44e1796aa0c2c625e9e4e0ce39efc0")
@Trust(contract = "033a4d051b04b7fc0230d2b1aaedfd5a84be279a5361a7358db665ad7857787f1b")
public class MyContract {
```

To trust a native contract, instead of adding its hash to the `contract` attribute, you can use the `nativeContract`
attribute the same way as it is used in the `@Permission` annotation as specified
[above](neo-n3/smart_contract_development/devpack.md#Permissions). In the following example, the native StdLib is trusted:

```java
@Trust(nativeContract = NativeContract.StdLib)
public class MyContract {
```

If you want to trust any contract use the wildcard option `@Trust("*")`.

### Safe Methods

Methods that don't change state of a contract and don't fire events can be safely invoked in a read-only mode. To signal
that to the Neo network, you can use the `io.neow3j.devpack.annotations.Safe` annotation on method-level. The method
will be tagged as safe in the contract's manifest.

If your `@Safe`-annotate a method does change state or fire an event, invocations of that method will fail.


### Call Flags

Call flags allow you to restrict the actions of a contract you call within your contract. For example, you can deny
further calls to other contracts, changing blockchain state, or firing events.

The possible flags are defined and docuemented in `io.neow3j.devpack.constants.CallFlags` and are ment to be used in the
`call` method of the `io.neow3j.devpack.Contract` class.


## Placeholder Substitution

The devpack offers the possibility to substitute strings used in a contract before compiling it. Any string literal in
the main contract class can be substituted, even annotation values. Currently, this feature is not supported in
auxiliary classes used in the contract class.

You have to compile the contract programmatically to be able to use this feature. Checkout
[this](neo-n3/smart_contract_development/setup_and_compilation.md#programmatic-compilation) section for information on
how to compile programmatically. The `Compiler` provides a `compile` method that takes a `Map<String, String>`
parameter. This is the substitution map that tells the compiler which strings are placeholder strings (the map's keys)
and which values they should be replaced with (the map's values).

```java
    Map<String, String> substitutionMap = new HashMap<>();
    substitutionMap.put("<contract_hash>", "da65b600f7124ce6c79950c1772a36403104f2be");
    substitutionMap.put("<event_name", "TheEvent");
    ...
    CompilationUnit res = new Compiler().compile(YourSmartContract.class.getName(), substitutionMap);
```

Here's an example of how a smart contract using placeholders might look.

```java
    @Permission(contract = "<contract_hash>", methods = "*")
    static class PlaceholderSubstitutionContract {

        static final String aString = "<a_string>";
        static final Hash160 OWNER = StringLiteralHelper.addressToScriptHash("<account_address>");

        @DisplayName("<event_name>")
        static Event1Arg<String> event;

        public static String method() {
            String s2 = "<another_string>";
            return s1 + s2;
        }

        public static Hash160 getOwner() {
            return OWNER;
        }
    }
```

Even though the example consistently uses "<>" for placeholder strings, this is not mandatory. You can use any format,
but be careful that you don't replace strings by accident.

The placeholder substitution feature works in contract tests too. See
[this](neo-n3/smart_contract_development/testing.md#deployment-configuration) section for more information.



<!-- ## Annotations

The devpack provides several annotations to be used on smart contract classes and methods. All annotation are contained
in the [`io.neow3j.devpack.annotations`](https://javadoc.io/doc/io.neow3j/devpack/latest/io/neow3j/devpack/annotations/package-summary.html)
package. Check the Javadocs for more information on what the annotations do. -->