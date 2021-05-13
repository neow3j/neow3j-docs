# dApp Development

In our definition, dApp development comprises the implementation of systems that interact and are based on a blockchain. 
Although, smart contracts are also part of a dApp, we separate contract development and dApp development because
neow3j's libraries are cleanly separable into these two development activities. In this part we introduce the neow3j SDK
which is concerned with dApp development.

The neow3j SDK attempts to provide a high abstraction layer that relieves the developer from dealing
with technical details of the Neo blockchain. At the same time, it gives the ability for more
detailed configuration for use cases where more control is needed.
You can, for example, use the `NeoToken` class to conveniently construct a NEO transfer
with a single method call, or construct a transaction from scratch using the `TransactionBuilder`.

The neow3j SDK is divided into two modules `io.neow3j:core` and `io.neow3j:contract` that separate the lower level core
functionalities, like signing, from more abstract functionality, like building contract-specific transactions. The
_core_ module is a dependency of the _contract_ module, so if you add _contract_ as a dependency to your project you
will get the _core_ module as well.

**`io.neow3j:core`** contains
  - Basic definitions and enums
  - Utility methods of which the most notable classes are, e.g., for handling hex strings. 
  - Cryptographic methods related to key pairs (class `ECKeyPair`) and signatures (class `Sign`)
  - Wallet and account management, i.e., the handling of key pairs or reading from and writing to NEP-6 wallet files.
  - Interaction with the Neo blockchain, i.e., communication via JSON-RPC with a Neo node. 
  - The `Transaction` and `ScriptBuilder` classes that are fundamental for constructing contract invocations of any
    kind.

**`io.neow3j:contract`** contain
- Functionality related to smart contracts. 
- Classes that allow easy interaction with native contracts.
- Classes that represent and allow easy interaction with contracts following a certain standard, like the NEP-17 token
  standard.
- for interfaces like fungible or non-fungible token contracts and native contracts that support building invocation
  scripts to create a transaction.

The following sections explain the SDK's capabilities based on common concepts and activities used in dApp
development.