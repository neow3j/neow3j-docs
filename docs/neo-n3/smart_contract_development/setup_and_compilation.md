# Setup & Compilation

The neow3j devpack uses [Gradle](https://gradle.org/) as its build tool. Thus, the structure of a smart contract project
follows the Gradle convention. Clone the [boilerplate](https://github.com/neow3j/neow3j-boilerplate) repository for a
basic project setup on which you can build your contracts.

```bash
git clone https://github.com/neow3j/neow3j-boilerplate.git
cd neow3j-boilerplate
```

## The Build File

This section discusses the contents of the `build.gradle` file as found in the boilerplate project. 

The first block in the file applies the necessary Gradle plugins. Neow3j provides its own Gradle plugin that allows
contract compilation via a Gradle task called `neow3jCompile`. The Java plugin is also necessary.

```groovy
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.19.0"
}
```

Then follows the definition of the project's organization and version.

```groovy
group 'com.axlabs'
version '1.0-SNAPSHOT'
```

Next, we have to set the Java version compatibility, which needs to be Java 1.8 in order for the neow3j compiler to
work.

```groovy
sourceCompatibility = 1.8
targetCompatibility = 1.8
```

Then, we define the artifact repositories in which to find code dependencies. This is by default Maven Central. Neow3j
artifacts are published there. 

```groovy
repositories {
    mavenCentral()
}
```

An extra source set is established next. It is meant for code concerning contract deployment. This adds a source set
`deploy` additional to `main` and `test`. 

```groovy
sourceSets {
    deploy {
        compileClasspath += sourceSets.main.output
        runtimeClasspath += sourceSets.main.output
    }
}
```

Then we need to define the dependencies, which are `io.neow3j:devpack` for writing smart contract code, `io.neow3j:devpack-test` and JUnit 5 for wirting contract tests, and `io.neow3j:compiler` for compiling contracts inside of Java code.

```groovy
dependencies {
    implementation 'io.neow3j:devpack:3.19.0'

    testImplementation 'org.junit.jupiter:junit-jupiter:5.8.2',
            'io.neow3j:devpack-test:3.19.0',
            'ch.qos.logback:logback-classic:1.2.10'

    deployImplementation 'io.neow3j:compiler:3.19.0',
            'ch.qos.logback:logback-classic:1.2.10'
}
```

For JUnit 5 to work with Gradle, we need to add the following block.

```groovy
tasks.withType(Test) {
    useJUnitPlatform()
}
```

Lastly, there is a block called `neow3jCompiler`, which configures the `neow3jCompile` Gradle task. The contract that we
want to compile is declared by its fully qualified name with the `className` property. There are two hidden properties
not used here. The first is the `debug` property, which determines if the compilation should produce debugging
inforamtion for the Neo Debugger. It is true by default. The second is `outputDir`, which specifies a directory for
placing the compilation results. It is `build/neow3j` by default.

```groovy
neow3jCompiler {
    className = "com.axlabs.boilerplate.HelloWorldSmartContract"
}
```

## Compilation

Contract compilation can happen in two ways. Either you use neow3j's Gradle task or you invoke the compiler from within
Java code. The next two sections explain the two approaches.

### Using Gradle

With the setup from above, we can now turn to the compilation. The boilerplate project contains a very simple contract.
To compile it, run `./gradlew neow3jCompile` from the project's root folder. This will compile the smart contract class,
and output a NEF file, contract manifest, and debugging information file to the output directory `build/neow3j`. The NEF
file is the contract binary containing the NeoVm code. The contract manifest defines the properties of the contract, for
example, what methods are available for invocation. The debugging information file, ending with `.nefdbgnfo`, is
required by the Neo Debugger to debug your contract. The NEF and the manifest are the artifacts that we need for
deployment.


### Programmatic Compilation

The neow3j compiler can be invoked from within another Java program. For this, we require the `io.neow3j:compiler`
module which is already included in the boilerplate project.

In the class `com.axlabs.boilerplate.Deployment`, you can see an example usage of the neow3j compiler.

```java
CompilationUnit res = new Compiler().compile(
               HelloWorldSmartContract.class.getCanonicalName(),
               substitutions);
```

The compile requires the fully qualified name of the contract class to compile and will look for that class file in the
project. After compilation the results are accessible in the `CompilationUnit`. It contains the NEF file, contract
manifest, and other information.

```java
NefFile nef = result.getNefFile();
ContractManifest manifest = result.getManifest();
```

If you want the compiler to produce debugging information, you need to tell the neow3j compiler where the sources of the
smart contract are. It doesn't matter if the contract's Java files are located in the same project or somewhere else.

```java
File sourceDirectory = new File("/path/to/the/contract/src/java/main");
DirectorySourceContainer sourceContainer = new DirectorySourceContainer(sourceDirectory, false);
CompilationUnit result = new Compiler()
        .compile("com.axlabs.boilerplate.HelloWorldSmartContract", Arrays.asList(sourceContainer));
DebugInfo debugInfo = result.getDebugInfo();
```
