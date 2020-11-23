# Compilation

To compile a contract, there are currently two options. You can use the neow3j Gradle plugin or do
it programmatically.

## Gradle Plugin

The `io.neow3j.gradle-plugin` can be used inside your Gradle project to launch contract compilation
as a Gradle task. Add the following few lines to your `build.gradle`.

```groovy
// Add the neow3j gradle plugin to the plugins section.
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.4.0"
}

// Add the devpack to the dependencies.
dependencies {
    compile("io.neow3j:devpack:3.+")
}

//Configure the fully qualified name of the class you want to compile.
neow3jCompiler {
    className = "fully.qualified.name.SmartContract"
}
```

With this setup, Gradle will fetch the neow3j plugin and a task `neow3jCompile` becomes available.
Running that task with `./gradlew neow3jCompile` from the project root will compile the smart
contract class, and output the NEF file and contract manifest to your project's build directory,
usually `./build/neow3j`.

By default the plugin also generates debugging information to be used with the Neo Debugger in
VSCode. This is a file with the suffix `.nefdbgnfo` that is placed next to the NEF and manifest.
If you do not want this to happen, you can set the `debug` flag in your `build.gradle` as follows.

```groovy
neow3jCompiler {
    className = "fully.qualified.name.SmartContract"
    debug = false
}
```

For more information on debugging go to the
[Debugging](neo3_guides/compiler_devpack/debugging.md#debugging) section.

## Programmatically

Add the `io.neow3j:compiler` to your smart contract project and call the compiler programmatically.

Gradle:

```groovy
implementation 'io.neow3j:compiler:3.+',
```

Maven:

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.4.0</version>
</dependency>
```

Then call the compiler inside your project to compile your smart contract class. The compiler will
look for the class in the projects classpath.

```java
CompilationUnit compUnit = new Compiler().compileClass("fully.qualified.name.SmartContract");
NefFile nef = compUnit.getNefFile();
ContractManifest manifest = compUnit.getManifest();
```

The `CompilationUnit` will contain the NEF file and the contract manifest, which are both required
for contract deployment.

If you want the compiler to produce debugging information, provide the absolute path to the
contract's source file. The compilation result will then contain a `DebugInfo` object that can be
written to disk and used in VSCode to debug the contract.

```java
String classFile = "fully.qualified.name.SmartContract";
String sourceFile = "/absolute/path/to/contract/source/file/SmartContract.java";
CompilationUnit unit = new Compiler().compileClass(classFile, sourceFile);
DebugInfo dbg = unit.getDebugInfo();
```

For more information on debugging go to the
[Debugging](neo3_guides/compiler_devpack/debugging.md#debugging) section.
