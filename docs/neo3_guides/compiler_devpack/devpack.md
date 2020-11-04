# Devpack


## Neo Smart Contract API

The Neo's smart contract API provides methods to retrieve blockchain information, access a contract's
storage area, and interact with the execution environment in which a contract is run.
The API is the same for all Neo developer packs, e.g.
[neo-boa](https://github.com/CityOfZion/neo-boa) or [neo-go](https://github.com/nspcc-dev/neo-go).
You can find the documentation on it at the official 
[Neo docs](https://docs.neo.org/v3/docs/en-us/reference/scapi/fw/dotnet/neo.html). 

Neow3j provides this API in the packages `io.neow3j.devpack.neo` and `io.neow3j.devpack.system`
following the same naming and structure as described in the Neo docs.


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





## Working with Account Addresses and Script Hashes 

The `devpack` provides a few methods that allow you to conveniently define an account script hash,
a byte array, or an integer via string literals.




