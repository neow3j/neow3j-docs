# neow3j 

Neow3j is a development toolkit that provides easy and reliable tools to build Neo dApps and Smart
Contracts using the Java platform (Java, Kotlin, Android).

By using neow3j, you will happily play with Neo and end up neow'ing around like [Bongo
Cat](https://knowyourmeme.com/memes/bongo-cat).

Neow3j is an open-source project developed by the community and maintained by
[AxLabs](https://axlabs.com).

The neow3j development toolkit is composed of:
- neow3j SDK
- neow3j devpack
- neow3j compiler

> Note that there are two versions of neow3j. One for [Neo 2](https://docs.neo.org/docs/en-us/index.html) 
> and one for [Neo 3](https://docs.neo.org/v3/docs/en-us/index.html). It is expected that neow3j for
> Neo 3 will undergo many changes in the near future until a stable version of Neo 3 is released.

## Features

* Support for NEO JSON-RPC API version
    [2.10.3](https://docs.neo.org/docs/en-us/reference/rpc/latest-version/api.html) and
    [3.0.0](https://docs.neo.org/v3/docs/en-us/index.html)
* Observable pattern for monitoring the NEO blockchain
* Sync and async interface
* Token transfers
* Smart contract invocation
* Smart contract deployment
* Building, signing, and sending raw transactions
* Wallet and Account model supporting NEP-6
* Multi-sig address utilities
* Passphrase-protected private keys (NEP-2)
* Mnemonic utilities (BIP-39)
* Android support from API 24, which covers [~49%](https://developer.android.com/about/dashboards/) 
    of all active Android devices ([~1 billion devices](https://www.youtube.com/watch?v=vWLcyFtni6U#t=2m46s))
* Smart contract development and compilation in Java


## Versioning

First of all, don't be confused by the _3_ in neow3j. It is not related to the version of Neo. Its
meaning is explained below.

Versioning of neow3j partially follows an adapted variant of semantic versioning. To be consistent
with Neo's major versions, neow3j keeps its major version fixed to the current Neo version. E.g.
neow3j 3.x.x for Neo 3. The semantics of the version number is therefore shifted to the right.
Incrementing the minor number means that breaking changes were applied. Incrementing the patch
version implies backward-compatible features and bug fixes were added.


## Donate

Help the development of neow3j by donating to the following addresses:

| Crypto | Address                                      |
| ------ | -------------------------------------------- |
| NEO    | `AHb3PPUY6a36Gd6JXCkn8j8LKtbEUr3UfZ`         |
| ETH    | `0xe85EbabD96943655e2DcaC44d3F21DC75F403B2f` |
| BTC    | `3L4br7KQ8DCJEZ77nBjJfrukWEdVRXoKiy`         |


## Why the name "neow3j"?

This project is based on [web3j](https://web3j.io), but focusing on NEO. That's why the suffix "w3j" was added to the "neo" name, forming "neow3j".

Well... then, it was simply natural to imagine [Bongo Cat](https://knowyourmeme.com/memes/bongo-cat) playing on NEO nodes and neow'ing instead of meow'ing, don't you think? :-)


## Thanks and Credits

* [NEO Foundation](https://neo.org/team) & [NEO Global Development (NGD)](https://neo.org/team)
* This project was strongly based on [web3j](https://web3j.io),
a library originally developed by [Conor Svensson](http://conorsvensson.com), latest on [this commit](https://github.com/web3j/web3j/commit/2a259ece9736c0338fbb66b1be4c04aba0855254).
We are really thankful for it. :-)
