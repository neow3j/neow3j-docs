# neow3j 

[![Build Status](https://travis-ci.org/neow3j/neow3j.svg?branch=master)](https://travis-ci.org/neow3j/neow3j)
![Maven metadata URI](https://img.shields.io/maven-metadata/v/http/central.maven.org/maven2/io/neow3j/core/maven-metadata.xml.svg)
[![Discord](https://img.shields.io/discord/382937847893590016?label=discord)](https://discord.io/neo)

__Neow3j__ is a Java library that aims at providing an easy and reliable integration of Java programs with the NEO blockchain. You can focus on building Java/Android applications that interact with the NEO blockchain and its nodes, without being concerned about the low-level details.

By using **neow3j**, you will happily play with NEO and end up neow'ing around like [Bongo Cat](https://knowyourmeme.com/memes/bongo-cat).

__Neow3j__ is an open-source project developed by the community and maintained by [AxLabs](https://axlabs.com).

## Features

* Support for NEO RPC [API version 2.10.2](https://docs.neo.org/docs/en-us/reference/rpc/latest-version/api.html)
* (see [list of supported RPC methods](/?id=supported-api-methods))
* Observable pattern for monitoring the NEO blockchain
* Asset transfers
* Smart contract invocations
* Smart contract deployments
* Passphrase-protected private keys (NEP-2)
* Wallet and Account model supporting NEP-6
* NEO-compatible Mnemonic utilities (BIP-39)
* Multi-sig address utilities
* Building, signing, and sending raw transactions
* Sync and async interface
* Retry on node errors
* Android support from API 24, which covers [~49%](https://developer.android.com/about/dashboards/) of **all active** Android devices ([~1 billion devices](https://www.youtube.com/watch?v=vWLcyFtni6U#t=2m46s))

## Upcoming Features/Enhancements

* Documentation and example Android apps
* GAS claiming
* NEP5 token contract interaction (already possible via normal contract invocation API)
* Select best seed NEO node based on some metrics (e.g., latency)


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

