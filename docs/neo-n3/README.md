<div style="text-align: center">
    <img src="../images/neow3j-neo3.png" alt="Logo" id="logo">
</div>

<h1 id="cover-header">neow3j <small>3.19.3</small></h1>

Neow3j is a development toolkit that provides easy and reliable tools to build Neo dApps and Smart Contracts using the Java
platform (Java, Kotlin, Android). It is an open-source project developed by the Neo community and maintained by
[AxLabs](https://axlabs.com).


## Features 🚀

* Support for the JSON-RPC API of the Neo node
* Observable pattern for monitoring the Neo blockchain
* Wallet and Account model supporting NEP-6
* Multi-sig address utilities
* Passphrase-protected private keys (NEP-2)
* Mnemonic utilities (BIP-39)
* Building, signing, and sending transactions
* Token transfers
* Smart contract invocations
* Smart contract deployment
* Smart contract development and compilation in Java
* Android support from API 24, which covers [~49%](https://developer.android.com/about/dashboards/) 
of all active Android devices ([~1 billion devices](https://www.youtube.com/watch?v=vWLcyFtni6U#t=2m46s))

## Quickstart 

Neow3j is composed of an **SDK** for dApp development and a **devpack** for smart contract development. 
The following sections describes how to get started with them.

### SDK

To make use of all neow3j SDK features, add the `io.neow3j:contract` dependency to your project.

__Gradle__

```groovy
implementation 'io.neow3j:contract:3.19.3'
```

__Maven__

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>contract</artifactId>
    <version>3.19.3</version>
</dependency>
```

### Devpack

The neow3j devpack is a Java library that provides the Neo-specific functionality required to write smart contracts. If
you want to play around with the devpack, add the `io.neow3j:devpack` dependency to your project.

__Gradle__

```groovy
implementation 'io.neow3j:devpack:3.19.3'
```

__Maven__

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>devpack</artifactId>
    <version>3.19.3</version>
</dependency>
```

For information on how to compile a smart contract hop over to the 
[Setup & Compilation](neo-n3/smart_contract_development/setup_and_compilation.md) section.

## Requirements

Neow3j requires **Java 8** or higher. For Android applications this implies the use of API level >= 24.

Neow3j does **not include a blockchain node**. I.e., it does not synchronize with other nodes nor store or verify
blockchain information locally. It depends on a connection to a Neo node to retrieve blockchain information. If you want
to run your own node check out the [official documentation](https://docs.neo.org/v3/docs/en-us/node/introduction.html) 
on how to set one up.  For local development we recommend using 
[Neo Express](https://github.com/neo-project/neo-express).

## Why the name "neow3j"? 🤔

This project is based on [web3j](https://web3j.io), but focusing on NEO. That's why the suffix "w3j" was added to the "neo" name, forming "neow3j".

Well... then, it was simply natural to imagine [Bongo Cat](https://knowyourmeme.com/memes/bongo-cat) playing on NEO nodes and neow'ing instead of meow'ing, don't you think? :-)

## Versioning

First of all, don't be confused by the number *3* in the name *neow3j*. It is not related to the
version of Neo as explained above.

Versioning of neow3j partially follows an adapted variant of semantic versioning. To be consistent
with Neo's major versions, neow3j keeps the first number fixed to the current Neo version. I.e.,
neow3j 3.x.x for versions of the library that are compatible with Neo N3. The semantics of the
version number is therefore shifted to the right by one position. Incrementing the second number
means that breaking changes were applied. Incrementing the third number implies
backward-compatible features and bug fixes were added.

## Donate 💰

Help the development of neow3j by donating to the following addresses:

| Crypto | Address                                      |
| ------ | -------------------------------------------- |
| NEO    | `AHb3PPUY6a36Gd6JXCkn8j8LKtbEUr3UfZ`         |
| ETH    | `0xe85EbabD96943655e2DcaC44d3F21DC75F403B2f` |
| BTC    | `3L4br7KQ8DCJEZ77nBjJfrukWEdVRXoKiy`         |


## Thanks and Credits 🙏

* [NEO Foundation](https://neo.org/team) & [NEO Global Development (NGD)](https://neo.org/team)
* This project was strongly based on [web3j](https://web3j.io), latest on [this commit](https://github.com/web3j/web3j/commit/2a259ece9736c0338fbb66b1be4c04aba0855254). 
  We are really thankful for it. 😄
