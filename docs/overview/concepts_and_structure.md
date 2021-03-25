# Concepts and Structure

The Neow3j SDK attempts to provide a high abstraction layer that reliefs the developer from dealing
with technical details of the Neo blockchain. At the same time, it gives the ability for more
detailed configuration for use cases where more control is needed.
You can, for example, use the `NeoToken` class to conveniently construct a NEO transfer
transaction with a single method call, or use the `TransactionBuilder` class to configure your
transaction manually.


## Modules

The neow3j SDK is separated into two modules _core_ and _contract_ that separate the lower level core
functionalities from higher level transaction building. The _core_ module is a dependency of the
_contract_ module, so if you add _contract_ as a dependency to your project you will get the
_core_ module as well.

The neow3j devpack is separated into three modules *devpack*,  *compiler*, and *gradle-plugin*. 
The *devpack* module is the main dependency for developing smart contracts. The *compiler* can
also be imported if you want to do programmatic compilation of contracts. The *gradle-plugin* is
not meant as a dependency. It contains a Gradle plugin that is served via the Gradle Plugin
Repository.

In the following the modules are briefly described.


### io.neow3j:core

The _core_ module contains all the core functionalities:
  - Basic definitions and enums
  - Utility methods of which the most notable classes are:
    - `Numeric`: Holds many methods for converting back and forth between hexadecimal strings and their byte array
    or integer representations. It also contains conversion methods for Fixed8 numbers.
    - `BigIntegers`: Provides methods for converting between BigIntegers and byte arrays.
    Always with the endianness precisely defined.
  - Cryptographic methods related to key pairs (class `ECKeyPair`) and signatures (class `Sign`)
  - Wallet and account management i.e. the handling of key pairs or reading from and writing to NEP-6 wallet files.
  - Interaction with the NEO blockchain i.e. the communication with the JSON-RPC interface of a Neo node. The most prominent
  class for this is `Neow3j`, which is used for making calls to a Neo node, e.g., for monitoring the blockchain.

### io.neow3j:contract

The _contract_ module contains functionality related to smart contracts. It provides wrapper classes for interfaces like fungible
or non-fungible token contracts and native contracts that support building invocation scripts to create a transaction.

### io.neow3j:devpack

The _devpack_ module provides classes, methods and annotations required for writing smart
contracts in Java. For example, if your smart contract needs to verify a transaction signature
the devpack offers a method for that. Or, if you want to publish detailed information about the
contract in its manifest, you can use one of the devpack's annotations.

### io.neow3j:compiler

The _compiler_ module contains all the logic to compile a Java smart contract code to neo-vm byte
code. It can be used for compilation of smart contracts in a programmatic manner. I.e., calling
the compiler from within Java code.

### io.neow3j:gradle-plugin

The _gradle-plugin_ module contains a plugin for usage within Gradlew projects. The plugin allows
you to run a Gradle task that compiles a Java smart contract. This module depends on the
_compiler_ package for the compilation logic. As a developer, you will not depend on this module
as a Java dependency but use it in Gradle build files.
