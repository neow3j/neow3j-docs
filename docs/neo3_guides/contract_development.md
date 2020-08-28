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
implementation 'io.neow3j:compiler:3.1.0',
```

Maven:
```xml
<dependency>
    <groupId>io.neow3j</groupId>
    <artifactId>compiler</artifactId>
    <version>3.1.0</version>
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
contract class and output a NEF file the directory at `./build/neow3j`. 
The plugin does not produce the contract manifest at the moment.

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

<!-- TODO: Describe how to deploy with neow3j SDK -->




## Contract structure

Every Smart Contract inherits the `SmartContract` base class which is in the Neo framework and provides some basic methods.

The `NEO` namespace is the API provided by the Neo blockchain, providing a way to access the block-chain data and manipulate the persistent store. These APIs are divided into two categories:

- Blockchain ledger. The contract can access all the data on the entire blockchain through interops layer, including complete blocks and transactions, as well as each of their fields.
- Persistent store. Each application contract deployed on Neo has a storage space that can only be accessed by the contract itself. These methods provided can access the data in the contract.

## Contract property

Inside the contract class, the property defined with `static readonly` or `const` is the contract property which can be used as constants and can not be changed. For instance, when we want to define a Owner of that contract or the factor number which will be used in the later asset transfer, we can define these constants in this way:

```c#
// Represents onwner of this contract, which is a fixed address. Usually should be the contract creator
public static readonly byte[] Owner = "ATrzHaicmhRj15C3Vv6e6gLfLqhSD2PtTr".ToScriptHash();

// A constant number
private const ulong factor = 100000000;
```

These properties defined in contract property are usually constants that can be used inside the methods of smart contract and every time the smart contract is running on any instance, these properties keep the same value.

In addition, developer can define static method  in contract and return a constant, which is exposing the method  out of the contract and let end-user can call the method to get the fixed value when they try to query the smart contract. For instance, when you create you own token, you have to define a name which you may want everyone use you contract can check he name with this method.

```c#
public static string Name() => "name of the token";
```

## Storage property

When you develop the smart contract, you have to store your application data on the blockchain. When a Smart Contract is created or when a transaction awakens it, the Contract’s code can read and write to its storage space. All data stored in the storage of the smart contract are automatically persisted between invocations of the smart contract. Full nodes in the blockchain store the state of every smart contract on the chain.

Neo has provided data access interface based on key-value pairs. Data records may be read or deleted from or written to the smart contracts using keys. Besides, smart contracts may retrieve and send their storage contexts to other contracts, thereby entrusting other contracts to manage their storage areas. In C# development, smart contract can use the `Storage` Class to read/write the persistent storage  The `Storage` class is a static class and does not require a constructor. The methods of `Storage` class can be viewed in this [API References](../../reference/scapi/fw/dotnet/neo/Storage.md)

For instance, if you want to store the total supply of your token into storage:

```c#
// Key is totalSupply and value is 100000000
Storage.Put(Storage.CurrentContext, "totalSupply", 100000000);
```

Here `CurrentContext` Returns the current store context. After obtaining the store context, the object can be passed as an argument to other contracts (as a way of authorization), allowing other contracts to perform read/write operations on the persistent store of the current contract.

`Storage` work well for storing primitive values and while you can use an `StorageMap`  which can be used for storing structured data, this will store the entire container in a single key in smart contract storage.

```c#
//Get the totalSupply in the storageMap. The Map is used an entire container with key name "contract"
StorageMap contract = Storage.CurrentContext.CreateMap(nameof(contract));
return contract.Get("totalSupply").AsBigInteger();
```

## Data type

When using C# to develop smart contracts, you cannot use the full set of C# features due to the difference between NeoVM and Dotnet IL.

Because NeoVM is more compact, we can only compile limited C# / dotnet features into an AVM file.

NeoVM provides the following basic types：

- `Pointer`
- `Boolean`
- `Integer`
- `ByteString`
- `Buffer`
- `Array`
- `Struct`
- `Map`
- `InteropInterface`

The basic types of C# are:

- `Int8 int16 int32 int64 uint8 uint16 uint32 uint64`
- `float double`
- `Boolean`
- `Char String`

## Your first Neo contract

After analyzing the basic hello world contract, let us move to your first real-world smart contract. Here we provide a very simple DNS system which was written in C#. The main function of the DNS is store the domain for users. It contains all the points above except the events. We can investigate this smart contract to learn how to make a basic smart contract. The source code is here:

```c#
using Neo.SmartContract.Framework;
using Neo.SmartContract.Framework.Services.Neo;
using System.ComponentModel;

namespace Domain
{
    [Features(ContractFeatures.HasStorage)]
    public class Contract1 : SmartContract
    {

        [DisplayName("query")]
        public static byte[] Query(string domain)
        {
            return Storage.Get(Storage.CurrentContext, domain);
        }

        [DisplayName("register")]
        publilc static bool Register(string domain, byte[] owner)
        {
            // Check if the contract owner is the same as the one who invokes the contract
            if (!Runtime.CheckWitness(owner)) return false;
            byte[] value = Storage.Get(Storage.CurrentContext, domain);
            if (value != null) return false;
            Storage.Put(Storage.CurrentContext, domain, owner);
            return true;
        }

        [DisplayName("delete")]
        public static bool Delete(string domain)
        {
            // To do
        }
    }
}
```

Let's slice it and learn it step by step.

### Contract Features

```c#
[Features(ContractFeatures.HasStorage)]
```

Upon the contract class we add a contract feature, which enables the contract to access the storage.

Besides, you can declare more features:

```c#
[ManifestExtra("Author", "Neo")]
[ManifestExtra("Email", "dev@neo.org")]
[ManifestExtra("Description", "This is a contract example")]
[SupportedStandards("NEP-5", "NEP-10")]
[Features(ContractFeatures.HasStorage | ContractFeatures.Payable)]
public class Contract1 : SmartContract
{
    public static bool Main(string operation, object[] args)
    {
        // other code
    }
}
```

`ManifestExtra` represents the extra fields in the Manifest file, where you can add `Author`, `Email`,  `Description` and etc.

`SupportedStandards` represents the NEP standards the contract conform to, such as NEP-5, a token standard on Neo. 

You can also add other fields, such as:

```c#
[ManifestExtra("Name", "sample contract")]
[ManifestExtra("Version", "1.0.0")]
```

`Features` are contract features. There are four options for now: 

- `[Features(ContractFeatures.NoProperty)]`: no special feature added to the contract.
- `[Features(ContractFeatures.HasStorage)]`: enables the contract to access the storage.
- `[Features(ContractFeatures.Payable)]`: enables the contract to accept NEP-5 assets.
- `[Features(ContractFeatures.HasStorage | ContractFeatures.Payable)]`: enables both features described above.

### Entry function

Theoretically, smart contracts can have any entry points. Methods of the public static type in the contract can be used as an entry function to be invoked externally, for example:

```c#
using Neo.SmartContract.Framework;

namespace Neo.Compiler.MSIL.UnitTests.TestClasses
{
    class Contract_a : SmartContract.Framework.SmartContract
    {
        public static object First(string method, object[] args)
        {
            return 'a';
        }
        public static object Second(string method, object[] args)
        {
            return 'b';
        }
    }
}
```

The compiler marks the offset of `First` and `Second` in ABI. When invoking the contract, it assigns the value to initialPosition, finds and executes the matching method according to the offset recorded in the ABI.

### Trigger

A smart contract trigger is a mechanism that triggers the execution of smart contracts. There are three triggers introduced in the Neo smart contract，`Verification`,   `Application`, and `System`. However, for most smart contract development, you only need to implement the Verify method to provide the signature verification logic, without having to decide the trigger. 

#### Verification trigger

A Verification trigger is used to call the contract as a verification function, which can accept multiple parameters and should return a valid Boolean value, indicating the validity of the transaction or block.

```c#
public static bool Verify()
{
    return Runtime.CheckWitness(Owner);
}
```

### CheckWitness

In many, if not all cases, you will probably be wanting to validate whether the address invoking your contract code is really who they say they are.

The `Runtime.CheckWitness` method accepts a single parameter which represents the address that you would like to validate against the address used to invoke the contract code. In more deeper detail, it verifies that the transactions / block of the calling contract has validated the required script hashes.

Usually this method is used to check whether an specified address is the the contract caller,  and then the address can be used to do store change or something else.

Inside our `DNS smart contract`, the `Register` function is firstly check if the owner is the same as the one who invoke the contract. Here we use the `Runtime.CheckWitness` function. Then we try to fetch the domain owner first to see if the domain is already exists in the storage. If not, we can store our domain->owner pair using the `Storage.Put`method.

```c#
private static bool Register(string domain, byte[] owner)
{
    if (!Runtime.CheckWitness(owner)) return false;
    byte[] value = Storage.Get(Storage.CurrentContext, domain);
    if (value != null) return false;
    Storage.Put(Storage.CurrentContext, domain, owner);
    return true;
}
```

Similar to the Register method, the Delete function check the owner first and if it exists and it is the same as the one who invoke the contract, delete the pair using the `Storage.Delete`method. 

### Events

In Smart contract, events are a way  to communicate that something happened on the blockchain to your app front-end (or back-end), which can be 'listening' for certain events and take action when they happen. You might use this to update an external database, do analytics, or update a UI. In some specified contract standard,  it defined some events should be posted. It is not cover in this page, but is very useful for the other smart contracts. For instance, in the NEP-5 Token, the events `transfer` should be fired when user invoke the transfer function.

```c#
//Should be called when caller transfer nep-5 asset.
[DisplayName("Transfer")]
public static event Action<byte[], byte[], BigInteger> OnTransfer;
```

Transfer is the event name.

### Json serialization

In Neo3 smart contract, the Json serialization/deserialization feature is added:

```c#
using Neo.SmartContract.Framework.Services.Neo;

namespace Neo.Compiler.MSIL.TestClasses
{
    public class Contract_Json : SmartContract.Framework.SmartContract
    {
        public static string Serialize(object obj)
        {
            return Json.Serialize(obj);
        }

        public static object Deserialize(string json)
        {
            return Json.Deserialize(json);
        }
    }
}
```

### Pointer support

```c#
namespace Neo.Compiler.MSIL.TestClasses
{
    public class Contract_Pointers : SmartContract.Framework.SmartContract
    {
        public static object CreateFuncPointer()
        {
            return new Func<int>(MyMethod);
        }

        public static int MyMethod()
        {
            return 123;
        }
    }
}
```

In this code, you can get the pointer of  `MyMethod` by invoking `CreateFuncPointer`.
