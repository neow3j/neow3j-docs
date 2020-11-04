# Requirements

## System Requirements

Neow3j is based on Java 8 and therefore requires applications to use Java version >= 8. For Android
applications, this implies the use of API level >= 24.

## RPC Nodes

Neow3j is not a blockchain network node. It does not synchronizes with other nodes nor store
or verify blockchain information locally. It depends on Neo nodes and their JSON-RPC interface to
retrieve block and transaction information.

Publicly reachable nodes can be found [here](http://monitor.cityofzion.io/).
If you want to run your own node check out the [official
documentation](https://docs.neo.org/v3/docs/en-us/node/introduction.html).

## Logging

Neow3j uses the Simple Logging Facade for Java [SLF4J](http://www.slf4j.org/) for logging. To procude any logs, a dependency that
binds to the SLF4J needs to be added.

E.g. the [Logback Classic](http://logback.qos.ch/index.html) library can be used.

```groovy
dependencies {
    implementation 'io.neow3j:contract:3.+'
    implementation 'io.neow3j:devpack:3.+'
    implementation 'io.neow3j:compiler:3.+'
    implementation 'ch.qos.logback:logback-classic:1.2.3'
    testImplementation group: 'junit', name: 'junit', version: '4.12'
}
```
