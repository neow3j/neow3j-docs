# Smart Contract Development

The tools for developing smart contracts with neow3j are currently under development. But you can
already play around with the features available so far. The neow3j smart contract toolkit contains a
devpack and the compiler.

The `io.neow3j:devpack` is a Java library that contains Neo-specific annotations and methods
required when developing a smart contract for Neo. You will use this library, e.g., to fetch
information about the current block inside of your smart contract, or to send out a notification.

The `io.neow3j:compiler` contains the contains the compiler that compiles your Java smart contract
class to NeoVM code. Instead of using this package you can make use of the neow3j Gradle plugin that
lets you compile a smart contract using Gradle.

Check out the [Getting Started](overview/getting_started.md?id=neow3j-devpack-and-compiler) section
for information on how to setup a project for smart contract development.

The following shows a basic smart contract making use of some of the devpack features.

```java
import io.neow3j.devpack.framework.Storage;
import io.neow3j.devpack.framework.annotations.EntryPoint;
import io.neow3j.devpack.framework.annotations.Features;
import io.neow3j.devpack.framework.annotations.ManifestExtra;

@ManifestExtra(key = "name", value = "ExampleContract")
@ManifestExtra(key = "author", value = "neow3j")
@Features(hasStorage = true)
public class ExampleContract {

    @EntryPoint
    public static byte[] sayHello(String key) {
        byte[] val = Storage.get(key);
        Storage.put(key, "Hello, world!");
        return val;
    }
}
```


## Smart Contract API

Neo's smart contract API provides methods to retrieve blockchain information, access a contract's
storage area, and interact with the execution environment in which a contract is run.
That API is reproduced in the `io.neow3j:devpack`.
For the API documentation, refer to the [Neo
documentation](https://docs.neo.org/v3/docs/en-us/reference/scapi/fw/dotnet/neo.html).


## Annotations

Besides the smart contract API, the devpack offers several annotations that can be used on a smart
contract class.

### @Appcall



## Limitations





## Compilation

To compile a contract, there are currently two options. 

### Programmatically

Add the `io.neow3j:compiler` to your smart contract project and call the compiler programmatically.

Gradle:
```groovy
implementation 'io.neow3j:compiler:3.2.0',
```

Maven:
```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.2.0</version>
</dependency>
```

Then call the compiler inside your project to compile your smart contract class. The compiler will
look for the class in the projects classpath.

```java
CompilationResult res = new Compiler().compileClass("fully.qualified.name.SmartContract");
```

The `CompilationResult` will contain the NEF and the contract manifest which are both required for
contract deployment.


### Gradle Plugin

> Note that the plugin is in an early development stage and currently doesn't output a contract 
> manifest. This prevents deployment of the compiled contract when using the plugin for compilation.
 
The `io.neow3j.gradle-plugin` can be used inside your Gradle project to launch contract compilation
as a Gradle task. Add the following few lines to your `build.gradle`.

```groovy
// Add the neow3j gradle plugin to the plugins section.
plugins {
    id 'java'
    id 'io.neow3j.gradle-plugin' version "3.2.0"
}

// Add the devpack to the dependencies.
dependencies {
    compile("io.neow3j:devpack:3.2.0")
}

//Configure the fully qualified name of the class you want to compile.
neow3jCompiler {
    className = "fully.qualified.name.SmartContract"
}
```

With this setup Gradle should fetch the plugin and a task `neow3jCompile` becomes available.
Running that task with `./gradlw neow3jCompile` from the project root will compile the smart
contract class and output a NEF file the directory to `./build/neow3j`. 

__Plugin Snapshot Versions__

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
``` 


## Deployment

### Using a Neo Node

If you have control over a Neo node, you can use that node directly to deploy smart contracts. The
information on how to do so, can be found in the [Neo
documentation](https://docs.neo.org/v3/docs/en-us/sc/deploy/deploy.html).  
But to get the NEF and manifest produced by the neow3j compiler onto the node, they must first be
written to a file. The following snippet shows how this could be done.

```java
CompilationResult res = new Compiler().compileClass("fully.qualified.name.SmartContract");
String nefFilePath = "/path/to/nef/file/test.nef";
String manifestFilePath = "/path/to/manifest/file/test.manifest.json";
try (FileOutputStream s = new FileOutputStream(nefFilePath)) {
    s.write(res.getNef().toArray());
}
try (FileOutputStream s = new FileOutputStream(manifestFilePath)) {
    ObjectMapperFactory.getObjectMapper().writeValue(s, res.getManifest());
}
```


### Using neow3j

Neow3j offers a `SmartContract` class that can be instantiated with a NEF and manifest file for
deployment purposes. To use this feature add the `io.neow3j:contract` package to your project as
described in the [Gettig Started](overview/getting_started.md?id=sdk) section.

The following snippet shows how the `SmartContract` class can be used for deployment.  
The `Neow3j` object is needed to send the deployment transaction to a Neo node of your choice. 
An account with enough GAS to cover the deployment fees is required. In the example such an account
is recovered from a WIF. The `Wallet` created from the account is handed to the deployment builder.
Without any other configurations the added account is automatically used to cover the fees and sign
the transaction.

```java
Neow3j neow = Neow3j.build(new HttpService("http://localhost:40332"));

File nefFile = new File("/path/to/ExampleContract.nef");
File manifestFile = new File("/path/to/ExampleContract.manifest.json");
SmartContract sc = new SmartContract(nefFile, manifestFile, neow);

Account a = Account.fromWIF( "L1WMhxazScMhUrdv34JqQb1HFSQmWeN2Kpc1R9JGKwL7CDNP21uR")
Wallet w = Wallet.withAccounts(a);
NeoSendRawTransaction response = sc.deploy()
        .withWallet(w)
        .build()
        .sign()
        .send();
```

Depending on the network you are connecting to, you need to make sure that you use the same magic
number in neow3j as the network is using. You can set the magic number globally on the `NeoConfig`
object.

```java 
NeoConfig.setMagicNumber(new byte[]{0x01, 0x03, 0x00, 0x0}); // Magic number 769, little-endian
```