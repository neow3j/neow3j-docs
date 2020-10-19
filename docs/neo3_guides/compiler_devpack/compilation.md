# Compilation

To compile a contract, there are currently two options.

## Programmatically

Add the `io.neow3j:compiler` to your smart contract project and call the compiler programmatically.

Gradle:

```groovy
implementation 'io.neow3j:compiler:3.3.0',
```

Maven:

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.3.0</version>
</dependency>
```

Then call the compiler inside your project to compile your smart contract class. The compiler will
look for the class in the projects classpath.

```java
CompilationUnit compUnit = new Compiler().compileClass("fully.qualified.name.SmartContract");
```

The `CompilationUnit` will contain the NEF and the contract manifest which are both required for
contract deployment.

## Gradle Plugin

The `io.neow3j.gradle-plugin` can be used inside your Gradle project to launch contract compilation
as a Gradle task. Add the following few lines to your `build.gradle`.

```groovy
// Add the neow3j gradle plugin to the plugins section.
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.3.0"
}

// Add the devpack to the dependencies.
dependencies {
    compile("io.neow3j:devpack:3.3.0")
}

//Configure the fully qualified name of the class you want to compile.
neow3jCompiler {
    className = "fully.qualified.name.SmartContract"
}
```

With this setup, Gradle should fetch the plugin and a task `neow3jCompile` becomes available.
Running that task with `./gradlw neow3jCompile` from the project root will compile the smart
contract class, and output the NEF and contract manifest file to your project's build directory,
e.g.,`./build/neow3j`.

<!-- **Plugin Snapshot Versions**

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
``` -->
