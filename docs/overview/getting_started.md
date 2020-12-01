# Getting started

## SDK

To make use of all neow3j SDK features, add `io.neow3j:contract` project to your dependencies.
Neow3j is split into multiple modules, so you can also depend on a subset of the functionality.
For example, if you only need the passphrase-protected private key feature (NEP-2), you can add
`io.neow3j:crtypo` to your project.

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

Releases are available for Neo 2 and Neo 3. The example above shows the newest release of neow3j for
Neo 3. To use the latest release for Neo 2, use the version `2.4.0`.

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
[Compilation](neo3_guides/compiler_devpack/compilation.md#compilation) section.

Note that the devpack and compiler are only available for Neo 3. Thus, Java cannot be used to
compile to Neo 2 compatible smart contracts.


<!-- ## Snapshots

If you would like to test `SNAPSHOT` versions, which are snapshots of the development branch, use
the following repository and append `-SNAPSHOT` to the version number. E.g.
`io.neow3j:contract:3.4.0-SNAPSHOT`.

Gradle:

```groovy
repositories {
    maven {
        url 'http://oss.sonatype.org/content/repositories/snapshots'
    }
    mavenCentral()
}
```

Maven:

```xml
<repositories>
    <repository>
        <id>sonatype-snapshots</id>
        <name>OSS Sonatype Snapshots</name>
        <url>http://oss.sonatype.org/content/repositories/snapshots</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
``` -->
