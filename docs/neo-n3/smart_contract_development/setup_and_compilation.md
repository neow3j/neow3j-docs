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

If you want to initialize an empty Gradle project, use `gradle init --dsl groovy --type basic` and then copy the
contents from the boilerplate build file into your `build.gralde`. The source code directory structure needs to follow
the conventions of the Java Gradle plugin, i.e., Java source files go into the directory `src/main/java` plus the
package subdirectories.

## The Build File

This section discusses the contents of the `build.gradle` file as found in the boilerplate project. 

The first block in the file applies Gradle plugins. Neow3j provides its own Gradle plugin that allows contract
compilation via a Gradle task called `neow3jCompile`. Applying this plugin adds the task to your project.  
The Java plugin is necessary for compilation as well and it demands that the contract source files are placed in the 
directory `src/main/java`.

```groovy
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.11.2"
}
```

The next three parts we set:
- The project's organization and version. 
- The compatibility of the source code and the compilation result, i.e., class files. This needs to be set to Java 1.8
  in order to work properly with neow3j's compiler.
- The artifact repository in which to find project dependencies. This is by default Maven Central because neow3j
  artifacts are published there.

```groovy
group 'io.neow3j'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenCentral()
}
```

Next comes the dependencies block. Here we need to define the `io.neow3j:devpack` module, which is needed for writing
smart contract code. 

```groovy
dependencies {
    implementation 'io.neow3j:devpack:3.11.2'
}
```

Lastly, there is a block called `neow3jCompiler`, which configures the `neow3jCompile` task. The contract that we want
to compile is declared by its fully qualified name with the `className` property. In the boilerplate project's case that
is the class `io.neow3j.HelloWorldSmartContract`.  The `debug` property determines if the compilation should produce
debugging inforamtion for the Neo Debugger. There is a third property, which is not shown here, called `outputDir` which 
specifies a directory for placing the compilation results. By default this is `build/neow3j`.

```groovy
neow3jCompiler {
    className = "io.neow3j.HelloWorldSmartContract"
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
implementation 'io.neow3j:compiler:3.11.2',
```

Maven:

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.11.2</version>
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