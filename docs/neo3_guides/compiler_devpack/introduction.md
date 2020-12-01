# Smart Contract Development

Neow3j contains two modules that are responsible for smart contract development, `io.neow3j:devpack`
and `io.neow3j:compiler`.  

The `io.neow3j:devpack` is a library that contains Neo-specific annotations, methods, and classes
required when developing a smart contract. You will use this library in your contract, for example,
to fetch information about the current block, or to send out a notification.  
The `io.neow3j:compiler` contains the compiler that compiles your smart contract class to neo-vm
code. It is not necessary to use this module directly. You can instead make use of the neow3j Gradle 
plugin that lets you compile contracts using Gradle.

The following section will guide you through the necessary setup for the implementation and
compilation of a simple "Hello, World!" contract. From there you can continue to the sections listed
below for more detailed information:

- The basic principles and differences of developing in Java for the neo-vm versus Java for the JVM
are documented in the section 
[Java Smart Contracts](neo3_guides/compiler_devpack/java_smart_contracts.md#java-smart-contracts).

- In-depth documentation on the devpack's API can be found in the
[Devpack](neo3_guides/compiler_devpack/devpack.md#Devpack) section.

- The possibilities of how to compile a Java smart contract are documented in the 
[Compilation](neo3_guides/compiler_devpack/compilation.md#compilation) section.

- For information on debugging go to the
[Debugging](neo3_guides/compiler_devpack/debugging.md#debugging) section.

- For information on contract deployment check out the 
[Deployment](neo3_guides/compiler_devpack/deployment.md#Deployment) section. 

- More Java smart contract examples can be found in the 
[neow3j-examples](https://github.com/neow3j/neow3j-examples-java) repository.


## "Hello, World!" Contract

<!-- In this section we will work through the necessary setup for the implementation and compilation of a
simple "Hello, world!" contract.  -->

We use Gradle as the build tool because neow3j offers a Gradle plugin for smart contract
compilation. First, setup a new Gradle project either manually or with the CLI command 
`gradle init`. Then, put the following few lines into your `build.gradle` file. This adds the
pluging and the devpack to the project.

```groovy
plugins {
    id 'java'
    // Add the neow3j gradle plugin to the plugins section.
    id 'io.neow3j.gradle-plugin' version "3.5.0"
}

dependencies {
    // Add the devpack to the dependencies.
    implementation "io.neow3j:devpack:3.+"
}

neow3jCompiler {
    // Configure the smart contract class you want to compile.
    className = "fully.qualified.name.HelloWorldContract"
}
```

With this setup, Gradle will fetch the neow3j devpack and the plugin, which makes a Gradle task
`neow3jCompile` available. We will use that task to compile the smart contract. But first, add the
following class into the Java sources directory.

```java
import io.neow3j.devpack.neo.Storage;
import io.neow3j.devpack.annotations.Features;
import io.neow3j.devpack.annotations.ManifestExtra;

@ManifestExtra(key = "name", value = "HelloWorld")
@ManifestExtra(key = "author", value = "neow3j")
@Features(hasStorage = true)
public class HelloWorldContract {

    public static byte[] sayHello(String key) {
        byte[] val = Storage.get(key);
        Storage.put(key, "Hello, world!");
        return val;
    }
}
```

This simple contract takes a string argument, looks for an entry in the contract's storage,
overwrites that entry with "Hello, World!" and returns the previous value. Already this small
contract makes heavy use of the devpacks API, e.g., all class annotations and the calls to the
contract storage.

The contract can now be compiled with the `neow3jCompile` Gradle task. Make sure that the correct
fully qualified class name is set in the `neow3jCompiler` of the `build.gradle`. It is the name of
the contract you want to compile. Then, run `./gradlw neow3jCompile` from the project's root folder.
This will compile the smart contract class, and output a NEF file, contract manifest and debugging
information to your project's build directory, usually `./build/neow3j`.

The NEF file is the contract binary containing neo-vm code. The contract manifest defines the
properties of the contract, for example, what methods are available for invocation. The NEF and the
manifest file are ready to be deployed on the Neo blockchain. 


