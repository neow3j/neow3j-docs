# Tutorial

To compile Java smart contracts, the Java development kit of at least version 8 is required. It is
available at https://adoptopenjdk.net/ and served by different package managers. Additionally, you
need [Gradle](https://gradle.org/install/) as the build tool. We used Gradle version 6.7.1 for this
guide.

A Java smart contract project can have any directory layout, but we recommend using the standard
convention for Java projects introduced by Maven. It will look like the following graph.

```
src
 |-- main
 |     `-- java	
 |          `-- my/package/name
 |                          |
 |                          `-- MyContract.java
 |                          |
 |                          `-- MyHelperClass.java
 |
 `-- test
       `-- java
            `-- my/package/name
                            |
                            `-- MyContractTest.java

```

Create a new directory for the project.

```bash
mkdir HelloWorld
cd HelloWorld
```

Run the Gradle `init` command to initialize the directory as a Gradle project.

```bash
gradle init --type basic --dsl groovy --project-name HelloWorld
```

The project folder will now contain a file called `build.gradle`. This is the build file that
controls Gradle's behavior for this project. Overwrite its contents with the following lines. This
applies the neow3j plugin and adds the devpack as a dependency to the project.

```groovy
plugins {
    id 'java'
    // Apply the neow3j Gradle plugin to the build.
    id 'io.neow3j.gradle-plugin' version "3.5.0"
}

// Required if you have a JDK version higher than 8 installed.
sourceCompatibility = 1.8

repositories {
    // Use jcenter for resolving dependencies.
    jcenter()
}

dependencies {
    // Add the devpack to the dependencies.
    implementation "io.neow3j:devpack:3.+"
     // Use JUnit test framework
    testImplementation 'junit:junit:4.12'
}

neow3jCompiler {
    // Configure the fully qualified name of the contract class you want to compile.
    className = "com.axlabs.helloworld.HelloWorldContract"
}
```

With this setup, Gradle will fetch the neow3j devpack and the plugin, which makes a Gradle task
called `neow3jCompile` available. We will use that task to compile the smart contract later. 

Now, create the direcotry structure mentioned before. We use the package name
`com.axlabs.helloworld`.

```
mkdir -p src/main/java/com/axlabs/helloworld
mkdir -p src/test/java/com/axlabs/helloworld
```

Add the file `HelloWorldContract.java` to the `src/main/java/com/axlabs/helloworld` directory and
insert the following contract code.

```java
package com.axlabs.helloworld;

import io.neow3j.devpack.neo.Storage;
import io.neow3j.devpack.annotations.Features;
import io.neow3j.devpack.annotations.ManifestExtra;

@ManifestExtra(key = "author", value = "AxLabs")
@Features(hasStorage = true)
public class HelloWorldContract {

    public static byte[] sayHello(String key) {
        byte[] val = Storage.get(key);
        Storage.put(key, "Hello, world!");
        return val;
    }
}
```

This simple contract takes a string argument, looks for an entry in the contract's storage,
overwrites that entry with "Hello, World!" and returns the previous value. The contract uses
some of the devpacks API, e.g., all class annotations and the calls to the contract storage.

To compile the contract, first make sure that the following section in the `build.gradle` file
points to the contract class. 

```groovy
neow3jCompiler {
    // Configure the fully qualified name of the contract class you want to compile.
    className = "com.axlabs.helloworld.HelloWorldContract"
}
```

Then, run `./gradlw neow3jCompile` from the project's root folder. This will compile the smart
contract class, and output a NEF file, contract manifest and debugging information file to your
project's build directory, usually `./build/neow3j`. Using `./gradlew` instead of `gradle` after a
project has been initialized is the standard way to use Gradle. The "Gradle Wrapper" `./gradlew`
ensures that everyone working on a code base uses the same Gradle version.

The NEF file is the contract binary containing the neo-vm code. The contract manifest defines the
properties of the contract, for example, what methods are available for invocation. The debugging
information file is required by the Neo Debugger to debug your contract. 

For directions on where to find more information on contract deployment, debugging and other topics
go back to the [Introduction](neo3_guides/compiler_devpack/introduction.md#introduction)).


