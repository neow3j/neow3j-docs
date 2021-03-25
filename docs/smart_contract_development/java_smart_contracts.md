# Java Smart Contracts

This section discusses the possibilities and restrictions one has when implementing a smart
contract in Java. Multiple Java smart contract examples can be found in the neow3j-examples
[repository](https://github.com/neow3j/neow3j-examples-java/tree/master/neo3-examples/src/main/java/io/neow3j/examples/contract_development/contracts)
repository.

Even though we are coding in Java we have to keep in mind that we are not coding for the Java
virtual machine (JVM) but for the Neo virtual machine (neo-vm). Blockchain programmers need to be
more careful with computational resources because every step in the resulting neo-vm script costs
GAS. Therefore, it is a bad idea to make use of classes like `java.util.List` or `java.util.Map`,
even though we might be very familiar with those data structures. You will see that with the current
version of the compiler you are not even allowed to construct such classes. We have to get along
with more basic constructs (integers, Strings, byte arrays) and classes specifically supporting Neo
smart contract development (provided by the devpack).

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
the return value of a method call as shown in the following examples.

```java
static final int initialSupply = 200_000_000;
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
and convert them to byte arrays, script hashes, or integers. The following example uses it to set
the owner script hash using the owner account's address.

```java
static final byte[] owner = StringLiteralHelper.addressToScriptHash(
    "NZNos2WqTbu5oCgyfss9kUJgBXJqhuYAaj");
```

## Contract Methods

Contract methods are methods on the contract class. They all need to be static, since the contract
class is never instantiated.
Methods that are `public` will show up in the contract's manifest and can be called from the
outside, i.e. by a direct invocation or by other contracts. Every other access modifier will make
the a method not visible to the outside. 

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
object with `Blockchain.getTransaction(byte[] txId)` all its properties are available to the
contract class through public instance variables on the object. Similarly you can create and
instantiate custom classes that have instance methods and variables that are directly accessible to
the contract class.

Java objects are handled as arrays inside the neo-vm. The array holds variables of the object in the
order they appear in the class definition. I.e., in the following example the `lowNote` variable has
index 0 and `highNote` has index 1.

```java
public class Bongo {

    public String lowNote;
    public String highNote;

    public Bongo(String lowNote, String highNote) {
        this.lowNote = lowNote;
        this.highNote = highNote;
    }
}
```

Thus, when invoking a contract method that takes an object as a parameter, you need to use a
parameter of type `Array`.

Assume the above `Bongo` class is a method parameter. Using the neow3j SDK, you would have to
construct the following parameter representing a `Bongo` instance.

```java
ContractParameter.array(
    ContractParameter.string("C2"), 
    ContractParameter.string("C5")));
```

The same applies when the object is used as a return type. In other words, expect a return value of
type `Array` that will hold the object's variables in the order they appear in the class.

> Don't let yourself be confused by the contradicting information you will find in the contract
> manifest. When using objects as parameters or return types the manifest will set the type to `Any`
> instead of `Array`. This is the currently valid convention. It discerns objects from actual
> arrays, e.g. an `int[]`.

Storing and loading objects to and from a contract's storage is possible by using the devpack's 
`Binary` class. Use `Binary.serialize(Object obj)` before writting the object to storage and
`Binary.deserialize(byte[] bytes)` after fetching the object from storage.


## Integers

You can use all of Java's integer types in a smart contract, including their wrapper classes.
That is, `byte`, `short`, `char`, `int`, `long` and `Boolean`, `Byte`, `Short`, `Character`,
`Integer`, `Long`. All of these types are treated equally because the neo-vm knows only one number
type. It doens't have any size restrictions. This means that an `int` is not restricted to 32 bit as
we know it from regular Java applications. You can use `int` without worrying about size
restrictions. In fact we recommend using `int` everywhere and ignoring the types `short` and `long`.
Think of `int` as `BigInteger` - that is actually its representation in the neo-vm.

Note that when converting an integer type to a smaller one, the number does
not get truncated. E.g., casting an `int` to `byte` does not truncate the number from 32 to 8 bit.
The number will keep the same value on the neo-vm. Thus, do not use such casts in expectation that 
the upper few bytes to get truncated. The same applies to the wrapper type methods like
`Integer.byteValue()`.

For the specific case of `int` to `byte` conversion, two helper methods `Helper.asByte(int value)` 
and `Helper.asSignedByte(int value)` are available. Both should only be used if you know that the
argument is in the range of an unsigned or signed byte, respectively. The methods do not truncate
the argument.


## Strings

Intuitively you can use Java's `String` type for strings. But be aware that on the neo-vm a `String`
is not represented as an object, but rather as a UTF8-encoded byte array. This means that you cannot
make use of the `String` methods like `contains()` or `indexOf()`. The only exception is the
`length()` method. It works as expected. Support for other methods might be added in the
future. The `io.neow3j.devpack.Helper` and `io.neow3j.devpack.StringLiteralHelper` already offer a
couple of String related helper methods.

If you want to compare two strings use the `==` operator. Because `String` is represented as a byte
array, it makes sense to compare two strings with `==` instead of an object-related `equals()`.

String concatenation with the `+` is supported but it cannot be mixed with other types. For example,
`"hello" + "world"` is allowed, but `"hello" + 5` is not. This is because such mixed expressions
perfrom calls to the `toString()` method of the involved types, which doesn't make sense if a type
is not represented as an object on the neo-vm. 


## Exceptions

Exceptions and try-catch blocks are supported and can be used as in normal Java
applications. Though, only the `java.lang.Exception` class can be used to throw exceptions. Either
with or without a string argument. Because only one type of exception is available you cannot have
multiple catch clauses per try clause, nor multiple different exceptions handled by one catch block.


## Unsupported Java Features

The following constructs of the Java language are not supported in Neo smart contract development.

- Floating point numbers like `float` and `double` 
- Inheritance
- Enums
- Interfaces
- Abstract classes