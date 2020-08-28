# Concepts and Structure

The Neow3j SDK attempts to provide a high abstraction layer that reliefs the developer from dealing
with technical details of the Neo blockchain. At the same time, it gives the ability for more
detailed configuration for cases where more control is needed.

You can, for example, use the `NeoToken` class to conveniently make a NEO transfer with a single
method call, use the `Invocation` class to configure your transfer in more detail, or use the
`Transaction` class to skip all the abstractions and construct a transaction "manually".


## Modules

Neow3j SDK is separated into multiple modules to make commonly useful functionality separately
available without imposing dependencies on more specific functionality. The modules that provide the
highest abstractions _wallet_ and _contract_. If you add _contract_ as a dependency to your project
you will get all other modules as well because it has all of them as dependencies. The whole
dependency graph between the modules is depicted here.

<!-- TODO: Think about changing the `gradle.build`s to use `implementation` instead of `compile`. In that case a dev would have to import every module separately and cannot simply import `io.neow3j:contract` and thereby get all other modules. -->

```
model 
  ^
  |--- utils 
  |      ^
  |      |
  |------+--- crypto 
  |      |      ^
  |      |      |
  |------+------+--- core 
  |      |      |      ^
  |      |      |      |
  |------+------+------+--- wallet 
  |      |      |      |      ^
  |      |      |      |      |
  `------+------+------+------+--- contract 
```

In the following subsection, each module is briefly described.


### io.neow3j:model

The _model_ module contains enums and definitions required by all other modules.


### io.neow3j:utils

The _utils_ module holds utility methods. The most notable classes here are:
- `Numeric`: Holds many methods for converting back and forth between hexadecimal strings and their
  byte array or integer representations. It also contains conversion methods for Fixed8 numbers.
- `BigIntegers`: Provides methods for converting between BigIntegers and byte arrays. Always with
  the endianness precisely defined.
- `Keys`: Provides methods related to cryptographic keys. E.g. derivation of script hash or address form a public key or conversion between integer and byte array representations of keys.


### io.neow3j:crypto

The _crypto_ module provides the basics for the cryptography used in neow3j. Everything related to key pairs
(class `ECKeyPairs`) and signatures (class `Sign`) resides in this module.


### io.neow3j:core

The _core_ module implements the basics for interacting with the NEO blockchain. The most prominent
class here is `Neow3j`, which is used for making calls to an Neo node, e.g., for monitoring the
blockchain. The _core_, therefore, defines all the DTO objects required for communicating with
JSON-RPC interface of a Neo node.


### io.neow3j:wallet

The _wallet_ module introduces the concept of accounts and wallets. For example, it provides the
functionality to read and write from NEP-6 wallet files and create new accounts and wallets which
are the basis for making transactions.


### io.neow3j:contract

The _contract_ module contains functionality related to smart contracts. It handles smart contract
deployment and invocation.
