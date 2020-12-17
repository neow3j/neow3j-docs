# Beginner's Guide 

A Java smart contract project can have any directory layout, but we recommend using the standard
convention for Java projects introduced by Maven. Because a smart contract doesn't make use of any
resource files, the directory layout will look very simple as shown in the following graph. 

```
src
 |
 |-- main
 |     |
 |     `-- java	
 |
 `-- test
      |
      `-- java
```

Instead of creating this strucutre manually we recommend using the [Gradle](https://gradle.org/)
build tool to initialize the project. You will need Gradle anyway to compile your smart contract.

Create a new directory and enter it.

```bash
mkdir HelloWorld
cd HelloWorld
```

Run the Gradle `init` command to initialize a Gradle project.

```bash
gradle init
```

It will guide you through a couple of steps for the project initialization.
1. In the first step choose `2: application` 
2. Then, choose `3: Java`
3. The third step is about the language used in the build file that we introduce later in this
   section. Choose `1: Groovy`.
4. For the testing framework, choose `1: JUnit 4`
5. Then, choose a project name or press enter to use the directory's name. 
6. Choose a package name for the contract, e.g., `io.neow3j.helloworld` or `com.example.helloworld`.

After the last step the directory layout will be created. Gradle will create some directories and
files we don't need. There will be `resource` directories in `main` and `test` and two Java classes
called `App.java` and `AppTest.java` in the package folders of `main` and `test`, respectively. You
can remove all of them.

In the root folder you will find a file called `build.gradle`. This is the build file that controls
Gradle's behavior for this project. Overwrite its contents with the following lines. This adds the
pluging and the devpack as a dependency to the project.

```groovy
plugins {
    id 'java'
    // Add the neow3j gradle plugin to the plugins section.
    id 'io.neow3j.gradle-plugin' version "3.5.0"
}

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
    // Configure the smart contract class you want to compile.
    className = "fully.qualified.name.HelloWorldContract"
}
```

With this setup, Gradle will fetch the neow3j devpack and the plugin, which makes a Gradle task
`neow3jCompile` available. We will use that task to compile the smart contract later. But first, add
the following class the the java soruces, i.e.
`./src/main/com/example/helloworld/HelloWorldContract.java`.

```java
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

To compile the contract, use the Gradle task `neow3jCompile` that is enabled by 
`io.neow3j.gradle-plugin`. You will find that the plugin is declared in the above mentioned
`gradle.build` file. The build file also contains the following plugin-specific declaration.

```groovy
neow3jCompiler {
    // Configure the smart contract class you want to compile.
    className = "fully.qualified.name.HelloWorldContract"
}
```

Make sure that the `className` attribute reflects the fully qualified name of your contract class.
If you chose `com.example.helloworld` as the root package name, then the class name is
`com.example.helloworld.HelloWorldContract`.

Run `./gradlw neow3jCompile` from the project's root folder. This will compile the smart contract
class, and output a NEF file, contract manifest and debugging information file to your project's
build directory, usually `./build/neow3j`. Using `./gradlew` instead of `gradle` after a project has
been initialized is the standard way to use Gradle. The "Gradle Wrapper" `./gradlew` ensures that
everyone working on a code base uses the same Gradle version.

The NEF file is the contract binary containing the neo-vm code. The contract manifest defines the
properties of the contract, for example, what methods are available for invocation. The debugging
information file is required by the Neo Debugger to debug your contract. 

For directions on where to find more information on contract deployment, debugging and other topics
go back to the [Introduction](neo3_guides/compiler_devpack/introduction.md#introduction)).


