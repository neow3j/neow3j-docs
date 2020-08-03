# Smart Contract Development

The tools for developing smart contracts with neow3j are currently in their early stages. Neow3j
offers a devpack and a compiler. The devpack is a Java library that contains Neo-specific
annotations and methods required when developing a smart contract for Neo. The compiler, as its name
points out, is used to compile a smart contract written in Java.

To use them, set up a Gradle or Maven project with the following dependencies.

__Gradle__

```groovy
compile 'io.neow3j:devpack:3.1.0',
        'io.neow3j:compiler:3.1.0'
```

__Maven__

```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>devpack</artifactId>
    <version>3.1.0</version>
</dependency> 
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.1.0</version>
</dependency> 
```

### "Hello, world!" smart contract

Below is a simple Java smart contract that can be compiled to NeoVM code. Many Java language
features are currently not supported and are part of future releases. A list of supported features
will be added soon.

```java
@ManifestExtra(key = "name", value = "ExampleContract")
@ManifestFeature(hasStorage = true)
public class SmartContract {

    @EntryPoint
    public static byte[] entryPoint(String key) {
        Runtime.log("Hello, world!");
        return Storage.get(key);
    }
}
```

### Compiling the contract

Currently there is no command line tool or plugin for compiling Java smart contracts, the compiler
has to be used programmatically. For this, put the smart contract class into the same project as
the following code that will compile it.

```java
public static void main(String[] args) throws IOException {
    CompilationResult res = new Compiler().compileClass("fully.qualified.name.SmartContract");
    String nefFilePath = "/path/to/nef/file/test.nef";
    String manifestFilePath = "/path/to/manifest/file/test.manifest.json";
    try (FileOutputStream s = new FileOutputStream(nefFilePath)) {
        s.write(res.getNef().toArray());
    }
    try (FileOutputStream s = new FileOutputStream(manifestFilePath)) {
        ObjectMapperFactory.getObjectMapper().writeValue(s, res.getManifest());
    }
    // Use the `ScriptReader` to convert a binary NeoVM script into human-readable instructions.
    System.out.println(ScriptReader.convertToOpCodeString(res.getNef().getScript()));
}
```

After compiling you can deploy the NEF and manifest file on a Neo node of version Neo3-preview2.