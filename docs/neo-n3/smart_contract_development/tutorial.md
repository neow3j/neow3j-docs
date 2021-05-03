# Tutorial

To quickly get the right project seutp, you can fetch the basic project structure from our
[boilerplate](https://github.com/neow3j/neow3j-boilerplate) repository. 

```bash
git clone https://github.com/neow3j/neow3j-boilerplate.git
cd neow3j-boilerplate
```

This basic project contains the `HelloWorldSmartContract.java` file that we will be compiling.
It also contains the `build.gradle` build file that controls Gradle's behavior for this project. 
First, it applies the neow3j Gradle plugin.

```groovy
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.8.0"
}
```

This makes the Gradle task called `neow3jCompile` available. We will use it to compile the smart
contract later. The task's options are also configured in the build file, in the `neow3jCompiler`
section. We configure the contract to be compiled and if debug information should be produced.

```groovy
neow3jCompiler {
    className = "io.neow3j.HelloWorldSmartContract"
    debug = true
}
```

The build file also imports three dependencies from neow3j. 

```groovy
dependencies {
    implementation 'io.neow3j:contract:3.8.0'
    implementation 'io.neow3j:devpack:3.8.0'
    implementation 'io.neow3j:compiler:3.8.0'
}
```

Properly speaking, we only require the `io.neow3j:devpack:3.8.0` for this tutorial. You can
remove the other two.

Now to the actual contract. The contract in `HelloWorldSmartContract.java` is very simple. It has
only one method that returns a string. The neow3j version of a "hello, world!" contract. To
compile it, run `./gradlw neow3jCompile` from the project's root folder. This will compile the
smart contract class, and output a NEF file, contract manifest, and debugging information file to
your project's build directory, usually `./build/neow3j`.  
The NEF file is the contract binary containing the neo-vm code. The contract manifest defines the
properties of the contract, for example, what methods are available for invocation. The debugging
information file is required by the *Neo Debugger* to debug your contract. 

Congratulations you have setup and compiled your first smart contract!

For directions on where to find more information on contract deployment, debugging and other topics
go back to the [Introduction](smart_contract_development/introduction.md#introduction)).
