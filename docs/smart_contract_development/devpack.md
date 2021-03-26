# Devpack

The neow3j devpack provides classes, methods and annotations required for writing smart contracts in
Java. For example, if your smart contract needs to verify a transaction signature, the devpack offers
a method for that. Or, if you want to publish detailed information about the contract in its
manifest, you can use one of the devpack's annotations.

The following sections describe parts of the devpack's API and frequently used concepts and constructs. It is
not exshausting, so, checkout the [Javadoc](https://javadoc.io/doc/io.neow3j/devpack) too.


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