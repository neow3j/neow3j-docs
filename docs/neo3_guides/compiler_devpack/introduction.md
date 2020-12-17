# Smart Contract Development

Neow3j contains two modules that are responsible for smart contract development, `io.neow3j:devpack`
and `io.neow3j:compiler`.  

The `io.neow3j:devpack` is a library that contains Neo-specific annotations, methods, and classes
required when developing a smart contract. You will use this library in your contract, for example,
to fetch information about the current block, or to send out a notification.  
The `io.neow3j:compiler` contains the smart contract compiler. It produces neo-vm code from Java
classes. It is not necessary to use this module directly. You can instead make use of the neow3j
Gradle plugin that lets you compile contracts using Gradle.

The documentation on smart contract development is divided into the following sections:

- If you are new to smart contracts in Java start with the [Beginner's
Guide](neo3_guides/compiler_devpack/beginners-guide.md#beginners-guide). It will guide you through
the necessary setup and how to implement and compile a simple contract.

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


