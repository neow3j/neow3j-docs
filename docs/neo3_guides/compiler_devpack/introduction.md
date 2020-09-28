# Smart Contract Development

The tools for developing smart contracts with neow3j are currently under development. But you can
already play around with the features available so far. The neow3j smart contract toolkit contains a
devpack and the compiler.

The `io.neow3j:devpack` is a Java library that contains Neo-specific annotations and methods
required when developing a smart contract for Neo. You will use this library, e.g., to fetch
information about the current block inside of your smart contract, or to send out a notification.

The `io.neow3j:compiler` contains the contains the compiler that compiles your Java smart contract
class to NeoVM code. Instead of using this package you can make use of the neow3j Gradle plugin that
lets you compile a smart contract using Gradle.

Check out the [Getting Started](overview/getting_started.md?id=neow3j-devpack-and-compiler) section
for information on how to setup a project for smart contract development.

The following shows a basic smart contract making use of some of the devpack features.

```java
import io.neow3j.devpack.framework.Storage;
import io.neow3j.devpack.framework.annotations.EntryPoint;
import io.neow3j.devpack.framework.annotations.Features;
import io.neow3j.devpack.framework.annotations.ManifestExtra;

@ManifestExtra(key = "name", value = "ExampleContract")
@ManifestExtra(key = "author", value = "neow3j")
@Features(hasStorage = true)
public class ExampleContract {

    @EntryPoint
    public static byte[] sayHello(String key) {
        byte[] val = Storage.get(key);
        Storage.put(key, "Hello, world!");
        return val;
    }
}
```

## Smart Contract API

Neo's smart contract API provides methods to retrieve blockchain information, access a contract's
storage area, and interact with the execution environment in which a contract is run.
That API is reproduced in the `io.neow3j:devpack`.
For the API documentation, refer to the [Neo
documentation](https://docs.neo.org/v3/docs/en-us/reference/scapi/fw/dotnet/neo.html).

<!-- ## Annotations

Besides the smart contract API, the devpack offers several annotations that can be used on a smart
contract class.

### @Appcall -->
