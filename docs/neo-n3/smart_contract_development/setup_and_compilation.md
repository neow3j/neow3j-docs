# Setup & Compilation

The neow3j devpack uses Gradle as its build tool. Thus, the structure of a smart contract project follows the Gradle
convention. You will have a *build.gradle* build file that configures the build behavior and a folder structure that
resembles "src/main/java/your/package/name". For a reference of how a basic smart contract project looks like, fetch
the boilerplate project structure from our [boilerplate](https://github.com/neow3j/neow3j-boilerplate) repository. 
This project contains a very simple smart contract and all necessary Gradle-related files. 

```bash
git clone https://github.com/neow3j/neow3j-boilerplate.git
cd neow3j-boilerplate
```

Of course, you can also initialize a new Gradle project with `gradle init` and then copy the contents from the
boilerplate build file into your `build.gralde`. The source code directory structure needs to follow the conventions of
the Java Gradle plugin, i.e., Java class files go into the directory `src/main/java` and test classes go to
`src/test/java`, plus package related directories.

## The Build File

This section discusses the contents of the `build.gradle` file as found in the boilerplate project. 

The first block in the file applies the necessary Gradle plugins. Neow3j provides its own Gradle plugin that allows
contract compilation via a Gradle task called `neow3jCompile`. The Java plugin is also necessary for contract
compilation.

```groovy
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.15.0"
}
```

Then follows the definition of the project's organization and version.

```groovy
group 'org.example'
version '1.0-SNAPSHOT'
```

Next, we have to set the Java version compatibility, which needs to be Java 1.8 in order for the neow3j compiler to
work.

```groovy
sourceCompatibility = 1.8
targetCompatibility = 1.8
```

Then, we define the artifact repositories in which to find code dependencies. This is by default Maven Central. Neow3j
artifacts are published there. This is followed by the dependencies block. Here we need to define the
`io.neow3j:devpack` module, which is needed for writing smart contract code. Additionally, we use neow3j's
`devpack-test` module and JUnit 5 for testing the smart contract.

> **Note:** For `devpack-test` to work, you need to have Docker installed and the Docker deamon must be running.

```groovy
repositories {
    mavenCentral()
}

dependencies {
    implementation 'io.neow3j:devpack:3.15.0'

    testImplementation 'org.junit.jupiter:junit-jupiter:5.8.0', 
            'io.neow3j:devpack-test:3.15.0'
}
```

For JUnit 5 to work with Gradle, we need to add the following block.

```groovy
tasks.withType(Test) {
    useJUnitPlatform()
}
```

Lastly, there is a block called `neow3jCompiler`, which configures the `neow3jCompile` Gradle task. The contract that we
want to compile is declared by its fully qualified name with the `className` property. The `debug` property determines
if the compilation should produce debugging inforamtion for the Neo Debugger. There is a third property, which is not
shown here, called `outputDir` which specifies a directory for placing the compilation results. By default this is
`build/neow3j`.

```groovy
neow3jCompiler {
    className = "org.example.HelloWorldSmartContract"
    debug = true
}
```

## Compilation

### Using Gradle

With the setup from above, we can now turn to the compilation. The boilerplate project contains a very simple contract.
It has only one method that returns a string. It is the neow3j version of a "hello, world!" program. To compile it, run
`./gradlew neow3jCompile` from the project's root folder. This will compile the smart contract class, and output a NEF
file, contract manifest, and debugging information file to the output directory, by default at `build/neow3j`.
The NEF file is the contract binary containing the NeoVm code. The contract manifest defines the properties of the
contract, for example, what methods are available for invocation. The debugging information file, ending with 
`.nefdbgnfo`, is required by the Neo Debugger to debug your contract. The NEF and the manifest are the artifacts that we
need for deployment.


### Programmatic Compilation

The neow3j compiler can also be invoked programmatically from within another Java program. For this, add the
`io.neow3j:compiler` to your Java project in which you want to make use of the compiler. 

Gradle:

```groovy
implementation 'io.neow3j:compiler:3.15.0',
```

Maven:

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.15.0</version>
</dependency>
```

Then, instantiate the compiler inside your code and call it with the contract class you want to compile. The class must
be specified with its fully qualified name. The neow3j compiler will look for the .class file in the classpath at
runtime. This implies that:

1. Either the smart contract .java file is present in the same project and the whole project was built/compiled with the
Java compiler. Thus, the corresponding contract .class file will be present in the project's output directory and,
therefore, available on the runtime classpath.

2. Or the smart contract was compiled with the Java compiler somewhere else and the output path of the .class file(s) is
added to the neow3j compiler via a `URLClassLoader` 

In the first case the following is engough to compile the contract.

```java
CompilationUnit result = new Compiler().compile("fully.qualified.name.SmartContract");
```

In the second case it would look like the following code.

```java
URL url = new File("/path/to/the/contract/project/build/classes/java/main").toURI().toURL();
URLClassLoader classLoader = new URLClassLoader(new URL[]{url}, this.getClass().getClassLoader());
CompilationUnit result = new Compiler(classLoader)
        .compile("io.neow3j.HelloWorldSmartContract");
```

In either case the `CompilationUnit` will contain a NEF file and the contract manifest, which are both required
for contract deployment.

```java
NefFile nef = result.getNefFile();
ContractManifest manifest = result.getManifest();
```

If you want the compiler to produce debugging information, you need to tell the neow3j compiler where the sources of the
smart contract are. It doesn't matter if the contract's .java files are located in the same project or somewhere else.

```java
File sourceDirectory = new File("/path/to/the/contract/src/java/main");
DirectorySourceContainer sourceContainer = new DirectorySourceContainer(sourceDirectory, false);
CompilationUnit result = new Compiler()
        .compile("io.neow3j.HelloWorldSmartContract", Arrays.asList(sourceContainer));
DebugInfo debugInfo = result.getDebugInfo();
```