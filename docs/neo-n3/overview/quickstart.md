# Quickstart 


Neow3j is divided into an SDK for dApp development and a devpack for smart contract development. The following describes
how to get started with them.

## SDK

To make use of all neow3j SDK features, add the `io.neow3j:contract` dependency to your project.

__Gradle__

```groovy
implementation 'io.neow3j:contract:3.9.+'
```

__Maven__

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>contract</artifactId>
    <version>[3.9.0,)</version>
</dependency>
```

Releases are available for Neo Legacy and Neo N3. The example above shows the newest release of neow3j for
Neo N3. To use the latest release for Neo Legacy, use the version `2.4.0`.

## Devpack

The neow3j devpack is a Java library that provides the Neo-specific functionality required to write smart contract. If
you want to play around with the devpack, add the `io.neow3j:devpack` dependency to your project.

__Gradle__

```groovy
implementation 'io.neow3j:devpack:3.9.+'
```

__Maven__

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>devpack</artifactId>
    <version>[3.9.0,)</version>
</dependency>
```

For information on how to compile a smart contract hop over to the contract
[Compilation](neo-n3/smart_contract_development/compilation.md#compilation) section.