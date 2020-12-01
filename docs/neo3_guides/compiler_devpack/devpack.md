# Devpack

The neow3j devpack provides classes, methods and annotations required for writing smart contracts in
Java. For example, if your smart contract needs to verify a transaction signature the devpack offers
a method for that. Or, if you want to publish detailed information about the contract in its
manifest, you can use one of the devpack's annotations.

The following sections describe parts of the devpack's API and frequently used concepts and
constructs.


## Neo Smart Contract API

The biggest part of the devpack is the Neo smart contract API. It provides many functionalities, for
example, to retrieve blockchain information, access a contract's storage area, and interact with the
execution environment in which a contract is run. This API is the same for all Neo developer packs, e.g.
[neo-boa](https://github.com/CityOfZion/neo-boa) or [neo-go](https://github.com/nspcc-dev/neo-go).
You can find the documentation on it int the official 
[Neo docs](https://docs.neo.org/v3/docs/en-us/reference/scapi/fw/dotnet/neo.html). 

Neow3j provides this API in the packages `io.neow3j.devpack.neo` and `io.neow3j.devpack.system`
following the same naming and structure as described in the Neo docs.


## Storage
<!-- TODO: Document on how to use storage. -->
Coming soon.


## Annotations

The `io.neow3j:devpack` provides several annotations to be used on smart contract classes and
methods. 

The first three annotations below - `@Features`, `@ManifestExtra`, and `@SupportedStandards` - are
on contract-level, meaning that they provide additional information about your contract. If your
contract's functionality is spread over multiple files make sure that you use these annotations only
on one single class.


#### @Features

The `@Features` annotation is a contract-level annotation and allows you to specify certain features
of the smart contract. This currently only includes if the contract needs storage and if it is
payable. The latter signifies if tokens can be sent to the contract. This annotation can only be
used once on a contract.

Here's an example usage.
```
@Features(hasStorage = true, payable = true)
public class MySmartContract {
    ...
}
```

If you don't explicitely use `@Features`, the contract will by default not have any storage and will
not be payable.


#### @ManifestExtra

The `@ManifestExtra` annotation allows you to specify information that will be placed in the `extra`
field of the contract manifest. This annotation can be added multiple times, each item containing a
key-value pair. For example:

```
@ManifestExtra(key = "name", value = "My Smart Contract")
@ManifestExtra(key = "author", value = "neow3j")
public class MySmartContract {
    ...
}
```

#### @SupportedStandards

`@SupportedStandards` is a contract-level annotation used to declare standards (e.g., NEP-5) that
the contract implements. It will show up in the contract manifest in the `supportedstandards` field.
The standards are simply added as strings.

```
@SupportedStandards({"NEP5", "NEP10"})
public class MySmartContract {
    ...
}
```

#### @Contract

#### @Instruction

#### @Syscall



## Events

Neo smart contracts can trigger events. They appear, for example, in the [application
logs](https://docs.neo.org/v3/docs/en-us/reference/rpc/latest-version/api/getapplicationlog.html) 
of a contract invocation.

In order that users of a smart contract (dApps or other smart contracts) know what events are to be
expected, a contract's events should be listed in its contract manifest. The below JSON shows how
this could look.

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
you to define and use events with up to 16 state parameters. The classes representing these events
are located in the `io.neow3j.devpack.events` package. 

Events are declared in static contract variables as shown in the following code snippet. They cannot
be declared inside of a method body.

```java
    @DisplayName("mint")
    private static Event1Arg<Integer> onMint;

    @DisplayName("transfer")
    private static Event2Args<Integer, String> onTransfer;
```

It is not necessary to initialize the variables with an actual instance. This is counter-intuitive
for a Java developer, but remember, we are not developing for the JVM but for the neo-vm. The
variables are simply a definition of an event, with a name, the number and type of state parameters
that the event can take. All event classes follow the naming schema `Event[n]Args`, where `n` is the
number of state parameters the event takes. The `@DisplayName` annotation is optional and can be
used to define a different name for the event than the variable name. If it is not used, the
variable name is the events name.

Defined events can then be used in contract methods by calling their `notify(...)` method. 

```java
    public static boolean transfer() throws Exception {
        ...
        onTransfer.notify(transferAmount, "tokens transferred!");
        ...
        return true;
    }
```

<!-- TODO: Add information about the `Runtime.noitfy(Object... objects)` method. This method is 
currently not compilable. See issue #275 -->


## Working with Account Addresses and Script Hashes 

The `devpack` provides a few methods that allow you to conveniently define an account script hash,
a byte array, or an integer via string literals.

<!-- TODO: Write this section -->




