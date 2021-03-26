# Requirements

## System Requirements

Neow3j requires Java 8 or higher. For Android applications this implies the use of API level >= 24.

## RPC Nodes

Neow3j is not a blockchain node. It does not synchronizes with other nodes nor store or verify
blockchain information locally. It depends on Neo nodes and their RPC interface to retrieve block
and transaction information.

If you want to run your own node check out the [official
documentation](https://docs.neo.org/v3/docs/en-us/node/introduction.html) on how to set one up.

For local development we recommend using [Neo Express](https://github.com/neo-project/neo-express).

## Logging

Neow3j uses the Simple Logging Facade for Java [SLF4J](http://www.slf4j.org/) for logging. To
produce any logs, a dependency that binds to the SLF4J needs to be added.

E.g., the [Logback Classic](http://logback.qos.ch/index.html) library can be used.

```groovy
dependencies {
    ...
    implementation 'ch.qos.logback:logback-classic:1.2.3'
   ...
}
```
