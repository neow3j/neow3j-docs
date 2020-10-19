# Capabilities

?>  More content coming soon!

## Smart Contract API

Neo's smart contract API provides methods to retrieve blockchain information, access a contract's
storage area, and interact with the execution environment in which a contract is run.
The API is documented in the official [Neo
documentation](https://docs.neo.org/v3/docs/en-us/reference/scapi/fw/dotnet/neo.html). Neow3j
provides smart contract developers access to the API via the devpack (`io.neow3j:devpack`).


## Annotations

The `io.neow3j.devpack` provides several annotations to be used on smart contract classes and
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




## Debugging Information

The neow3j compiler produces debugging information that can be used with the [Neo
Debugger](https://github.com/neo-project/neo-debugger). For this to work, you need to provide Java
source files instead of already compiled class files.  
At the moment debugging is only supported for the source files/classes provided to the compiler. If
other code is invoked in the contract the debugger will not be able to step through that code. 



