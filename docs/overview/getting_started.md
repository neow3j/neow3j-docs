# Getting started

## SDK

To make use of all neow3j SDK features, add `io.neow3j:contract` project to your dependencies.
Neow3j is split into tow modules, so you can also depend on just the core functionality by adding
`io.neow3j.core` to your project.

Gradle:

```groovy
compile 'io.neow3j:contract:3.+'
```

Maven:

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>contract</artifactId>
    <version>[3.0.0,)</version>
</dependency>
```

Releases are available for Neo Legacy and Neo N3. The example above shows the newest release of neow3j for
Neo N3. To use the latest release for Neo Legacy, use the version `2.4.0`.

## Smart Contract Development

For smart contract development you require the `io.neow3j:devpack`. It provides all the Neo-related
utilities that are needed in a smart contracts. If you want to play around with the devpack add the
following dependency to your project.

Gradle:

```groovy
implementation 'io.neow3j:devpack:3.+',
```

Maven:

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>[3.0.0,)</version>
</dependency>
```

For information on how to compile a smart contract hop over to the contract
[Compilation](smart_contract_development/compilation.md#compilation) section.

Note that the devpack and compiler are only available for Neo N3. Thus, Java cannot be used to
compile smart contracts that are compatible with Neo Legacy.
