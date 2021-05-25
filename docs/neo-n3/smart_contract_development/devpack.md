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


## Storage

Every smart contract on the Neo blockchain has its own key-value storage. This storage is accessed
via a so called storage context. The context is the gateway to the contract's storage. This
additional concept between you and the storage potentially allows you to pass the context to another
contract, which could then access your contract's storage directly. 

In the devpack, the storage context is represented by the `io.neow3j.devpack.neo.StorageContext` class. The pivotal
class related to contract storage is `io.neow3j.devpack.neo.Storage`. It provides many `put(...)` and `get()` methods
for writing to the storage and reading from it. Each of theses methods requires the storage context as an argument.
Thus, it makes sense to retrieve the `StorageContext` once with `Storage.getStorageContext()`, store it in a static
class variable and reuse it every time the storage is accessed. This will save GAS in contract invocations.


## Annotations

The devpack provides several annotations to be used on smart contract classes and methods. All annotation are contained
in the [`io.neow3j.devpack.annotations`](https://javadoc.io/doc/io.neow3j/devpack/latest/io/neow3j/devpack/annotations/package-summary.html)
package. Check the Javadocs for more information on what the annotations do.


## Events

Neo smart contracts can fire events. They appear, for example, in the [application
logs](https://docs.neo.org/v3/docs/en-us/reference/rpc/latest-version/api/getapplicationlog.html) 
of a contract invocation. In order that others can know what events a contract can fire, they should be
listed in the manifest. The below JSON shows how this could look.

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

An event is defined by its name and the state parameters that are passed with it. The devpack allows
you to define and use events with up to 16 state parameters. The classes representing these events are located in the
[`io.neow3j.devpack.events`](https://javadoc.io/doc/io.neow3j/devpack/latest/io/neow3j/devpack/events/package-summary.html)
package. The interface `EventInterface` is only used as a marker for all the available event classes. It is not meant
for usage in a contract.

Events are declared in static contract variables as shown in the following code snippet. They cannot
be declared inside of a method body.

```java
    @DisplayName("mint")
    private static Event1Arg<Integer> onMint;

    @DisplayName("transfer")
    private static Event2Args<Integer, String> onTransfer;
```

It is not necessary to initialize the variables with an actual instance. This is counter-intuitive
for a Java developer, but remember, we are not developing for the JVM here, but for the neo-vm. The
variables are simply a definition of an event, with a name, the number and type of state parameters
that the event can take. All event classes follow the naming schema `Event[n]Args`, where `n` is the
number of state parameters the event takes. The `@DisplayName` annotation is optional and can be
used to define a different name for the event than the variable name. If it is not used, the
variable name is the event name.

Once an event is declared, it can then be used in contract methods by calling its `fire(...)` method. 

```java
    public static boolean transfer() throws Exception {
        ...
        onTransfer.fire(transferAmount, "tokens transferred!");
        ...
        return true;
    }
```


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
method on the contract interface. 

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

The devpack provides abstract contract interfaces, e.g., for contracts that follow a token standard. So, if you want to
establish a contract interface to a fungible token contract extend the `FungibleToken` class. All methods of a NEP-17
token contract are already available and the only thing you need to add is the contract hash annotation with the hash of
the targeted contract. If that contract has some extra methods, simply add them as shown before.

```java
@ContractHash("0xef4073a0f2b305a38ec4050e4d3d28bc40ea63f5")
class MyTokenContract extends FungibleToken {

    public static native int someCustomMethod(String arg);

}
```