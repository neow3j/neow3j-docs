# Concepts and Structure

The Neow3j SDK attempts to provide a high abstraction layer that reliefs the developer from dealing
with technical details of the Neo blockchain. At the same time, it gives the ability for more
detailed configuration for cases where more control is needed.

You can, for example, use the `NeoToken` class to conveniently make a NEO transfer with a single
method call, use the `Invocation` class to configure your transfer in more detail, or use the
`Transaction` class to skip all the abstractions and construct a transaction "manually".


## Modules

Neow3j SDK is separated into two modules _core_ and _contract_ that separate the lower level core functionalities
from higher level transaction building. The _core_ module is a dependency of the _contract_ module, so
if you add _contract_ as a dependency to your project you will get the _core_ module as well.

<!-- TODO: Think about changing the `gradle.build`s to use `implementation` instead of `compile`. In that case a dev would have to import every module separately and cannot simply import `io.neow3j:contract` and thereby get all other modules. -->

```
core <- contract
```

In the following subsection, the two modules are briefly described.


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
