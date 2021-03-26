# Java Smart Contracts

This section discusses the possibilities and restrictions that exist when implementing a smart
contract in Java. 
Even though we are coding in Java we have to keep in mind that we are not coding for the Java
virtual machine (JVM) but for the Neo virtual machine (neo-vm). Blockchain programmers need to be
more careful with computational resources because every step in the resulting neo-vm script costs
GAS. Therefore, it is a bad idea to make use of classes like `java.util.List` or `java.util.Map`,
because they can imply very costly operations. Instead we have to rely on Neo-specific types that
are provided in the devpack. 

The following sections discuss the most common java concepts and how they can be used for smart
contract development.

## Contract Class

Java smart contracts consist of one or more classes each containing some of the contract's
functionality. Even though they can be multiple classes, there can be only one one main contract
class (simply called contract class), while the other classes can hold logic and are used by the
contract class. Only the methods of the contract class are accessible from the outside once the
contract is deployed.

The contract class can only have static methods and variables. Thus, you should not think of a
contract as an instantiated object that holds the contract's state in its variables. Think of the
contract class being the managing entity handling incoming invocations, contract storage and event
emission. This is different, for example, to Solidity where the contract's state is managed in its
variables. Although the contract class is static, it can still instantiate and make use of objects.

## Contract Methods

All methods on the contract class need to be static, since the contract class is never instantiated.
Methods that are `public` will show up in the contract's manifest and are, therefore, callable from
the outside. Any other access modifier will make the a method invisible to the outside, so it makes
sense to use the `private` modifier on those methods for readability.

## Contract Variables

Similar to the methods, the contract class can only hold static variables. Again, variables of
the contract class do not map to the contract's storage. These variables are initialized every
time the contract is invoked, meaning that they cannot be used to store state. Changes to static
variables are lost as soon as an invocations finishes. Storing state to the contract's storage is
discussed later.
The static contract variables don't necessarily have to be final. They can still be used to remember a
value while an invocation moves through the contract code. Though, if a contract variable is ment
to be a constant it makes sense to mark it with the `final` keyword. This helps to enforce your
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

For convenience in variable initialization, the devpack offers the `StringLiteralHelper`.
As its name suggests, it operates on string literals. It offers methods that take string literals
and convert them to byte arrays, script hashes, or integers. The following example uses it to set
the owner script hash using the owner account's address.

```java
static final Hash160 owner = StringLiteralHelper.addressToScriptHash(
    "NZNos2WqTbu5oCgyfss9kUJgBXJqhuYAaj");
```


## Objects

Even though a smart contract class is never used as an object, you can still make use of objects
inside of the contract class. Though, it is not advised to carelessly use any type of the Java
standard or beyond. An example issue that would arise, is that the initialization of static class
variables of such classes is not supported by the compiler. Therefore, using such a class will
sooner or later lead to errors in the neo-vm script. Instead, only make use of Neo-specific classes
that are available in the devpack, custom classes created by yourself, and several basic types (e.g.
`String` or `Integer`).

The devpack makes use of objects itself. For example, calling `Storage.getStorageContext()` returns
a `StorageContrext` instance. On that object you can call instance methods like `createMap(...)`.
Another example is the `io.neow3j.devpack.neo.Transaction` class. After retrieving a `Transaction`
object with `LedgerContract.getTransaction(Hash256 txId)` all its properties are available to the
contract class through public instance variables on the object. Similarly you can create and
instantiate custom classes that have instance methods and variables that are directly accessible to
the contract class.

Java objects are handled as arrays inside the neo-vm. The array holds the object's variables in the
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

Of course, you could add getter and setter methods to the class, but using those methods would incur a higher GAS fee
when executing the contract, because they imply an extra method call, which you can get rid of by accessing the public
variables directly.

Storing and loading objects to and from a contract's storage is possible by using the `StdLib`
native contract. Use `StdLib.serialize(Object obj)` before writting the object to storage and
`StdLib.deserialize(byte[] bytes)` after fetching the object from storage.


## Integers

You can use all of Java's integer types, including their wrapper classes. That is, `byte`, `short`, `char`, `int`,
`long` and `Boolean`, `Byte`, `Short`, `Character`, `Integer`, `Long`. All of these types are treated equally because
the neo-vm knows only one number type. It doens't have any size restrictions. This means that Java's definition of an
`int` having a size of 32 bit does not hold true for smart contracts. You can use `int` without worrying about size
restrictions. In fact we recommend using `int` everywhere and ignoring the types `short` and `long`. Think of `int` as
`BigInteger`, which is actually its representation in the neo-vm.

Note that when converting an integer type to a smaller one, the number does not get truncated. E.g., casting an `int` to
`byte` does not truncate the number from 32 to 8 bit. The number will keep the same value on the neo-vm. Thus, do not
use such casts in expectation that the upper few bytes will get truncated. The same applies to the wrapper type methods
like `Integer.byteValue()`. For the specific case of `int` to `byte` conversion, two helper methods 
`Helper.asByte(int value)` and `Helper.asSignedByte(int value)` are available. Both should only be used if you know that
the argument is in the range of an unsigned or signed byte, respectively. The methods do not truncate the argument.


## Strings

Intuitively you can use Java's `String` type for strings. But be aware that on the neo-vm a `String`
is not represented as an object, but rather as a UTF8-encoded byte array. This means that you cannot
make use of the `String` methods like `contains()` or `indexOf()`. The only exception is the
`length()` method. It works as expected. Support for other methods might be added in the
future. The `io.neow3j.devpack.Helper` and `io.neow3j.devpack.StringLiteralHelper` already offer a
couple of String related helper methods.

String concatenation with the `+` is supported but it cannot be mixed with other types. For example,
`"hello" + "world"` is allowed, but `"hello" + 5` is not. This is because such mixed expressions
perfrom calls to the `toString()` method of the involved types, which doesn't make sense if a type
is not represented as an object on the neo-vm. 


## Exceptions

Exceptions and try-catch blocks are supported and can be used as in normal Java applications. Though, only the
`java.lang.Exception` class can be used to throw exceptions. Either with or without one string argument. Because only one
type of exception is available you cannot have multiple catch clauses per try clause, nor multiple different exceptions
handled by one catch block.

## Equality

If you want to compare two variables you should generally use the `==` operator. On simple types like integers or
strings, this will first perform a reference comparison and, if that fails, do a value comparison. For arrays and
objects it only does a reference comparison. I.e. if you need a comparison by value, you need to write an equals method.

You should avoid using the `String.equals()` method because that will compile the whole `equals()` method into neo-vm
code which is redundant, since using the `==` operator will already do a value comparison.


## Unsupported Java Features

- Floating point numbers: `float` and `double` are not supported by the neo-vm. Everything happens with integers.

- Enum: Java's enums are not supported yet.

- Class field variables: As discussed before, static field variables are only supported on the main contract class. They
  are ignored on other classes.