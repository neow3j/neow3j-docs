# Getting started

## neow3j SDK

To make use of all neow3j SDK features, add `io.neow3j:contract` project to your dependencies.
Neow3j is split into multiple modules, so you can also depend on a subset of the functionality. 
For example, if you only need the passphrase-protected private key feature (NEP-2), you can add
`io.neow3j:crtypo` to your project.

Gradle:
```groovy
compile 'io.neow3j:contract:3.2.0'
```

Maven:
```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>contract</artifactId>
    <version>3.2.0</version>
</dependency>
```

Releases are available for Neo 2 and Neo 3. The example above shows the newest release of neow3j for
Neo 3.


## neow3j Devpack and Compiler

Neow3j supports implementation of Neo smart contracts in Java with the `io.neow3j:devpack`.
It provides all the Neo-specific utilities that can be used in a smart contracts. Additionally to
the devpack, the neow3j compiler is required for compiling Java classes to NeoVM code. 

Note that the devpack and compiler are only available for Neo 3, i.e. Java cannot be used to compile
to Neo 2 compatible smart contracts.


### Devpack

If you only want to play around with the devpack add the following dependencies to your project.

Gradle:
```groovy
implementation 'io.neow3j:devpack:3.1.0',
```

Maven:
```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.1.0</version>
</dependency>
```


### Compiler 

To compile a contract, there are currently two options.

__Programmatically__ 

Add the `io.neow3j:compiler` to your smart contract project and call the compiler programmatically.

```java
CompilationResult res = new Compiler().compileClass("fully.qualified.name.SmartContract");
```

Gradle:
```groovy
implementation 'io.neow3j:compiler:3.1.0',
```

Maven:
```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.1.0</version>
</dependency>
```

__Gradle Plugin__

The `io.neow3j.gradle-plugin` can be used inside your Gradle project to launch contract compilation
with Gradle. Add the following lines to your `build.gradle`.

Add the neow3j gradle plugin to the plugins section.

```groovy
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.2.0"
}
```

Add the devpack to the dependencies.

```groovy
dependencies {
    compile("io.neow3j:devpack:3.2.0")
}
```

Configure the fully qualified name of the class you want to compile.

```groovy
neow3jCompiler {
    className = "fully.qualified.name.SmartContract"
}
```


## Snapshots

If you would like to test `SNAPSHOT` versions, which are snapshots of the development branch, use
the following repository and append `-SNAPSHOT` to the version number. E.g.
`io.neow3j:contract:3.3.0-SNAPSHOT`.

### SDK, Devpack, Compiler

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

### Gradle Plugin

If you want to use a `SNAPSHOT` version of the neow3j gradle plugin you must add the following to
your `settings.gradle`.

```groovy
pluginManagement {
    repositories {
        maven {
            url 'http://oss.sonatype.org/content/repositories/snapshots'
        }
        gradlePluginPortal()
    }
}
```

And then use the following in the plugins section in your `build.gradle`.
```groovy
id 'io.neow3j.gradle-plugin' version "3.3.0-SNAPSHOT"
``` 