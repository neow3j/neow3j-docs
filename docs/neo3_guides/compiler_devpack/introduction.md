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
More examples of Java smart contracts can be found in the `io.neow3j.examples.sc.contracts` package
in the [neow3j-examples repository](https://github.com/neow3j/neow3j-examples-java).

```java
import io.neow3j.devpack.neo.Storage;
import io.neow3j.devpack.annotations.Features;
import io.neow3j.devpack.annotations.ManifestExtra;

@ManifestExtra(key = "name", value = "ExampleContract")
@ManifestExtra(key = "author", value = "neow3j")
@Features(hasStorage = true)
public class ExampleContract {

    public static byte[] sayHello(String key) {
        byte[] val = Storage.get(key);
        Storage.put(key, "Hello, world!");
        return val;
    }
}
```

