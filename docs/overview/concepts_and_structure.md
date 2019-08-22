# Concepts and Structure

Neow3j attempts to provide a high abstraction layer that reliefs the developer
from tricky details of the NEO blockchain. At the same time it gives the
possibility to use less abstract concepts for sepcial case where more control is
needed.

The abstract activites in which neow3j supports you are:
- Transfering assets like NEO and GAS
- Transfering tokens
- Invoking smart contracts
- Deploying smart contracts
- Claiming GAS (work in progress)
- Monitoring the blockchain


## Modules

Neow3j is separated into multiple modules to make commonly useful functionality
separately available without imposing dependency on more specific 
functionionality. The "top-level" modules that provide the most abstract and
convenient API are _wallet_ and _contract_. In fact, if you add _contract_
as a dependency you will get all other modules as well because it has all of 
them as dependencies. The whole dependency graph between the modules is depicted 
here.

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

In the following subsection each module is briefly described.


### io.neow3j:model

The _model_ module contains mostly enums that define types like the different
transaction types that exist in NEO 2. This module is used in all other modules.


### io.neow3j:utils

The _utils_ module holds utility methods.
The most notable classes here are:
- `Numeric`: Holds many methods for converting back and forth between 
  hexadecimal strings and their byte array or integer represenations. It also 
  contains conversion methods for Fixed8 numbers.
- `BigIntegers`: Provides methods for converting between BigIntegers and byte 
  arrays. Always with the endianess precisely defined.
- `Keys`: Provides methods related to cryptograhpic keys. E.g. derivation of
  script hash or address form a public keys or conversion between integer and 
  byte array representations of keys.

<!-- 
In neow3j version 2 there are a few classes in this module that are here only 
for backward-compatability but should semantically be in another module. 
These classess are 
- `NeoConstants`, which holds many constants that are related to the NEO 
  blockchain and its specifications.
- `OpCode`, which is an enum containing some of the NEO VM operation codes.
- `ContractParameter`, which models a contracts input parameters. 
-->


### io.neow3j:crypto

The _crypto_ module provides the basics for the cryptography used in neow3j.
Everything related to key pairs (class `ECKeyPairs`) and signatures (class 
`Sign`) resides in this module.


### io.neow3j:core

The _core_ module implements the basics for interacting with the NEO 
blockchain. The most prominent class here is `Neow3j`, which is used for making 
calls to an RPC node and monitoring the blockchain (See the 
[guides](guides/guides.md) section on how to use it). _Core_, therefore, defines
all the JSON objects expected in the responses of an RPC node.  

It also holds specific classes representing some of NEO's transaction types 
(`ContractTransaction`, `InvocationTransaction`, `ClaimTransaction`).


### io.neow3j:wallet

The _wallet_ module introduces the concept of accounts and wallets. It for 
example provides functionality to read and write from NEP-6 wallet files and 
create new accounts and wallets which are the basis for making transactions.

This module also provides the `AssetTransfer` class, which does everything
related to transfering global assets like NEO and GAS.


### io.neow3j:contract

The _contract_ module contains functionality related to smart contracts. It
handles smart contract deployment, invocation, and loading of a contract's ABI.


### Module dependency

