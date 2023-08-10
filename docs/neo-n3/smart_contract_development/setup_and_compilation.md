# Setup & Compilation

There are two ways of getting started with neow3j. You can either setup your local environment or you can use GitHub
codespaces to get started quickly. If you want to set up your local environment you can just skip the GitHub Codespace
setup steps.

Use the [boilerplate](https://github.com/neow3j/neow3j-boilerplate) repository for a basic project setup on which you
can build your contracts. You can use it as a template to create your own project repository.

<img src="https://axlabs.fra1.digitaloceanspaces.com/neow3j-files%2Ftutorials%2F20230330_neow3j-boilerplate-1-clone-from-template.gif" width="80%"/>

If you want to develop locally, skip the next section and continue [here](#the-build-file).

## Setup in GitHub Codespace

You can get started quickly by using our boilerplate in a GitHub codespace. This allows you to get started quickly
without having to setup and manage your local environment. Just follow the following few steps:

Start a new codespace in your newly created boilerplate repository. This might take a while because it downloads and
runs multiple images.

<img src="https://axlabs.fra1.digitaloceanspaces.com/neow3j-files%2Ftutorials%2F20230330_neow3j-boilerplate-2-start-codespace.gif" width="80%"/>

Once the VSCode IDE is showing up in your browser, you should see the `main` and `test` source sets. The `main`
contains a smart contract class, and `test` holds a file to test it. Once a green arrow shows up in the test class, the
IDE is fully loaded and ready.

> **Note:** There is also a third sourceset `deploy` present which is used for productive deployment configurations.

<img src="https://axlabs.fra1.digitaloceanspaces.com/neow3j-files%2Ftutorials%2F20230330_neow3j-boilerplate-3-show-main-test-sourcesets.gif" width="80%"/>

Now, you can run the tests by clicking on the green arrow. The test class uses neow3j's test framework which runs a
Neo node in the background where your smart contract is deployed and invoked.

<img src="https://axlabs.fra1.digitaloceanspaces.com/neow3j-files%2Ftutorials%2F20230330_neow3j-boilerplate-4-run-tests.gif" width="80%"/>

## The Build File

The neow3j devpack uses [Gradle](https://gradle.org/) as its build tool. Thus, the structure of a smart contract project
follows the Gradle convention. This section discusses the contents of the `build.gradle` file as found in the
boilerplate project.

The first block in the file applies the necessary Gradle plugins. Neow3j provides its own Gradle plugin that allows
contract compilation via a Gradle task called `neow3jCompile`. The Java plugin is also necessary.

```groovy
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.20.2"
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
    implementation 'io.neow3j:devpack:3.20.2'

    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.0',
            'io.neow3j:devpack-test:3.20.2',
            'ch.qos.logback:logback-classic:1.2.11'

    deployImplementation 'io.neow3j:compiler:3.20.2',
            'ch.qos.logback:logback-classic:1.2.11'
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
