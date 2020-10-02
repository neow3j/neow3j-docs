# Getting started

## SDK

To make use of all neow3j SDK features, add `io.neow3j:contract` project to your dependencies.
Neow3j is split into multiple modules, so you can also depend on a subset of the functionality.
For example, if you only need the passphrase-protected private key feature (NEP-2), you can add
`io.neow3j:crtypo` to your project.

Gradle:

```groovy
compile 'io.neow3j:contract:3.2.2'
```

Maven:

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>contract</artifactId>
    <version>3.2.2</version>
</dependency>
```

Releases are available for Neo 2 and Neo 3. The example above shows the newest release of neow3j for
Neo 3. To use the latest release for Neo 2, use the version `2.4.0`.

## Devpack and Compiler

Neow3j supports implementation of Neo smart contracts in Java with the `io.neow3j:devpack`.
It provides all the Neo-specific utilities that can be used in a smart contracts. Additionally to
the devpack, the neow3j compiler is required for compiling Java classes to NeoVM code.

Note that the devpack and compiler are only available for Neo 3, i.e. Java cannot be used to compile
to Neo 2 compatible smart contracts.

### Devpack

If you only want to play around with the devpack add the following dependency to your project.

Gradle:

```groovy
implementation 'io.neow3j:devpack:3.2.2',
```

Maven:

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.2.2</version>
</dependency>
```

### Compiler

Hop over to the [Contract Development](neo3_guides/compiler_devpack/compilation.md)
section for information on how to use the neow3j compiler.

## Snapshots

If you would like to test `SNAPSHOT` versions, which are snapshots of the development branch, use
the following repository and append `-SNAPSHOT` to the version number. E.g.
`io.neow3j:contract:3.3.0-SNAPSHOT`.

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
```
