<div style="text-align: center">
    <img src="../images/neow3j-neo3.png" alt="Logo" id="logo">
</div>

<h1 id="cover-header">neow3j <small>3.9.0</small></h1>

Neow3j is a development toolkit that provides easy and reliable tools to build Neo dApps and Smart
Contracts using the Java platform (Java, Kotlin, Android).

The toolkit can be divided into the neow3j SDK, which is used for dApp development, and the
neow3j devpack, which is used for smart contract development. We use the term dApp development
for activities around the blockchain, e.g., building a web page that interacts with the
blockchain, but exclude the implementation of smart contracts. Of course, smart contracts will
naturally be part of a running dApp.

Neow3j is an open-source project developed by the community and maintained by
[AxLabs](https://axlabs.com).


## Features 🚀

* Support for Neo's JSON-RPC API.
* Observable pattern for monitoring the Neo blockchain
* Wallet and Account model supporting NEP-6
* Multi-sig address utilities
* Passphrase-protected private keys (NEP-2)
* Mnemonic utilities (BIP-39)
* Token transfers
* Smart contract invocations
* Building, signing, and sending raw transactions
* Smart contract deployment
* Smart contract development and compilation in Java
* Android support from API 24, which covers [~49%](https://developer.android.com/about/dashboards/) 
    of all active Android devices ([~1 billion devices](https://www.youtube.com/watch?v=vWLcyFtni6U#t=2m46s))


## Versioning

First of all, don't be confused by the number *3* in the name *neow3j*. It is not related to the
version of Neo. Its meaning is explained in [*Why the name "neow3j"?*](#why-the-name-quotneow3jquot).

Versioning of neow3j partially follows an adapted variant of semantic versioning. To be consistent
with Neo's major versions, neow3j keeps the first number fixed to the current Neo version. I.e.,
neow3j 3.x.x for versions of the library that are compatible with Neo N3. The semantics of the
version number is therefore shifted to the right by one position. Incrementing the second number
means that breaking changes were applied. Incrementing the third number implies
backward-compatible features and bug fixes were added.


## Why the name "neow3j"? 🤔

This project is based on [web3j](https://web3j.io), but focusing on NEO. That's why the suffix "w3j" was added to the "neo" name, forming "neow3j".

Well... then, it was simply natural to imagine [Bongo Cat](https://knowyourmeme.com/memes/bongo-cat) playing on NEO nodes and neow'ing instead of meow'ing, don't you think? :-)


## Donate 💰

Help the development of neow3j by donating to the following addresses:

| Crypto | Address                                      |
| ------ | -------------------------------------------- |
| NEO    | `AHb3PPUY6a36Gd6JXCkn8j8LKtbEUr3UfZ`         |
| ETH    | `0xe85EbabD96943655e2DcaC44d3F21DC75F403B2f` |
| BTC    | `3L4br7KQ8DCJEZ77nBjJfrukWEdVRXoKiy`         |


## Thanks and Credits 🙏

* [NEO Foundation](https://neo.org/team) & [NEO Global Development (NGD)](https://neo.org/team)
* This project was strongly based on [web3j](https://web3j.io),
a library originally developed by [Conor Svensson](http://conorsvensson.com), latest on [this commit](https://github.com/web3j/web3j/commit/2a259ece9736c0338fbb66b1be4c04aba0855254).
We are really thankful for it. 😄
