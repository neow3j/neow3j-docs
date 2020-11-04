# Java Smart Contracts

This chapter discusses the possibilities and restrictions one has when implementing a smart
contraact in Java. We often reference the `BongoCatToken` contract in code snippets. The full
contract can be found at the end of this page or in the
[neow3j-examples](https://github.com/neow3j/neow3j-examples-java/blob/master/neo3-examples/src/main/java/io/neow3j/examples/contract_development/contracts/BongoCatToken.java)
repository.

Even though we are coding in Java we have to keep in mind that we are not coding for the Java
virtual machine (JVM) but for the Neo virtual machine (neo-vm). Blockchain programmers need to be
more careful with computational resources because every step in the resulting neo-vm script costs
GAS. Therefore, it is a bad idea to make use of classes like `java.util.List` or `java.util.Map`,
even though we might be very familiar with those data structures. You will see that in the current
version of the compiler you are not even allowed to construct such classes. We have to make do with
more basic constructs (integers, Strings, byte arrays) and classes specifically supporting Neo smart
contract development (provided by the devpack).


## Contract Class

Java smart contracts consist of one or more classes each containing some of the contract's
functionality. The contract has one main contract class, but there can be other classes
that hold logic and are used by the contract class. Only the methods of the contract class are
accessible from the outside once the contract is deployed. 

The contract class can only have static methods and variables. Thus, you should not think of a
contract as an instantiated object that holds the contract's state in its variables. Think of the
contract class being the managing entity handling incoming invocations, contract storage and event
emission. This is different, for example, to Solidity where the contract's state is managed in its
variables. Although the contract class is static, it can still instantiate and make use of objects.


## Contract Variables

A Java smart contract can only hold static field variables. We call them contract variables. These
variables are initialized every time the contract is invoked meaning that they cannot be used to
store state. Changes to contract variables are lost as soon as the invocations finishes. If you need
to store state that persists over multiple invocations, use the contract's storage, which will be
discussed later. This doesn't imply that all contract variables have to be final. They can still be
used to remember a value while an invocation moves through the contract code. If a contract variable
is ment to be a constant it makes sense to mark it with the `final` keyword. This helps enforce your
intentions but is not strictly required by the compiler.

Contract variables can be initialized with constant values (e.g., string literals) but also with
the return value of a method call as shown at the example of the `BongoCatToken`.

```java
static final int initialSupply = 200_000_000;
static final String assetPrefix = "asset";
static final String totalSupplyKey = "totalSupply";
static final StorageContext sc = Storage.getStorageContext();
static final StorageMap assetMap = sc.createMap(assetPrefix);
```

It is also possible to use a static initializer clause. Note, that the instance initializer, i.e. the
same clause without the `static` keyword, is not supported.

```java
static final String assetPrefix;
static final StorageContext sc;
static {
    assetPrefix = "asset";
    sc = Storage.getStorageContext();
}
```

For convenience in variable initialization the devpack offers the `StringLiteralHelper`.
As its name suggests, it operates on strings literals. It offers methods that take string literals
and convert them to byte arrays, script hashes, or integers. The `BongoCatToken` uses it to set the
owner script hash, as a byte array, using the owner account's address.

```java
static final byte[] owner = StringLiteralHelper.addressToScriptHash(
    "AJunErzotcQTNWP2qktA7LgkXZVdHea97H");
```


## Contract Methods

Contract methods are methods on the contract class. They all need to be static, since the contract
class is never instantiated.
Methods that are `public` will show up in the contract's manifest and can be called from the
outside, i.e. by a direct invocation or by other contracts. Every other access modifier will make
the a method not visible to the outside. At the example of the `BongoCatToken` the method
`balanceOf` is visible but the method `addToBalance` is not.
The name of a method in the contract manifest is exactly the name given in the Java code. 
<!-- TODO: Mention the different handling of the methods `_deploy` and `_verify` once supported by neow3j-->


## Objects

Even though a smart contract class is never used as an object, you can still make use of objects
inside of the contract class. Instantiation of simple classes is supported. With simple we mean
classes that do not extend from another class except `Object`. The neow3j compiler will throw an
exception if you try to instantiate a class that uses inheritance. 

The devpack makes use of objects itself. For example, calling `Storage.getStorageContext()` returns
a `StorageContrext` instance. On that object you can call an instance method like `createMap(...)`.
Another example is the `io.neow3j.devpack.neo.Transaction` class. After retrieving a `Transaction`
object with `Blockchain.getTransaction(byte[])` all its properties are available to the contract
class through public instance variables on the object.
Similarly you can create and instantiate custom classes that have instance methods and variables
that are directly accessible to the contract class.

Objects are handled as arrays or structs inside the neo-vm. Thus, if you use an object as a 
parameter for a contract method, use the `Array` `ContractParameterType` when calling the
method from the outside. In the array the instance variables of the object appear in the order
they are defined in the class.
I.e., in the following example the `lowNote` has index 0 and `highNote` has index 1 in the array.
The same applies when the object is used as a return type, in other words, wxpect an `Array` return 
type holding the instance variables of the object.

```
public class Bongo {

    public String lowNote;
    public String highNote;

    public Bongo(String lowNote, String highNote) {
        this.lowNote = lowNote;
        this.highNote = highNote;
    }
}
```

Storing and loading objects to and from a contract's storage is possible by using the devpack's 
`Binary` class. Use `Binary.serialize(Object obj)` before writting the object to storage and
`Binary.deserialize(byte[] bytes)` after fetching the object from storage.

## Storage



## Integers

All of Java's integer types are supported in their primitive form and their wrapper classes.
That is: `boolean`, `byte`, `short`, `char`, `int`, `long` and `Boolean`, `Byte`, `Short`,
`Character`, `Integer`, `Long`.

The NeoVM knows only one integer type to which all of the above are mapped. It is represented by a
`BigInteger` in the VM implementation. Though, Java's `BigIteger` is not yet supported but is a
planned feature.

Note that when converting an integer type to a smaller one, e.g., `int` to `byte`, the number does
not get truncated in the NeoVM script. Thus, do not use such casts in expectation that the upper few
bytes to get truncated. The same applies to the wrapper type methods like `Integer.byteValue()`.

Floating point types (`float` and `double`) are not supported.


## Complete BongoCatToken Contract

<!-- TODO insert BongoCatToken -->
