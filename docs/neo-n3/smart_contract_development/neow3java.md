# NeowJava

This section discusses the possibilities and restrictions that exist when implementing a smart contract in Java. Even
though we are coding in Java the produced byte code is not meant for the JVM but for the NeoVM. Therefore, only a subset
of Java can be used and we call that subset NeowJava.

Blockchain programmers need to be careful with computational resources because every step in their code costs GAS.
You should avoid using Java's standard library and any library not explicitely implemented for smart contract purposes,
because such libraries might contain unsupported code or code that is very costly for execution on a blockchain.

## Types

The NeoVM works with types called stack items (because the NeoVM is a stack machine). These stack
items don't all have a matching type provided by Java and its standard library. Thus, the devpack adds new types that
map to the corresponding stack item on the NeoVM. The following table shows which Java types map to which NeoVM stack
items. 

| Java type                      | NeoVM stack item | Description  |
|--------------------------------|------------------|--------------|
| `int`/`Integer`                | Integer          | Java integers have a corresponding integer stack item on the NeoVM. The difference to normal Java integers is the range. On the NeoVM the integer range is not restricted to [2<sup>-31</sup>, 2<sup>31</sup>-1]. We discuss integers in more depth below. |   
| `boolean`/`Boolean`            | Boolean          | Java's `boolean` maps to the Boolean stack item on the NeoVM. But, the NeoVM also uses Integer stack items in the range [0,1] for boolean values. Don't worry if you get an Integer stack item as a return value even if your contract method returns a `boolean`. 
| `io.neow3j.devpack.ByteString` | ByteString       | A NeoVM ByteString is an immutable byte array. This is a type introduced by the devpack because there exists no corresponding Java type for the ByteString stack item. |
| `java.lang.String`             | ByteString       | Java `String`s map to UTF-8 ByteStrings on the NeoVM. Thus, converting from `String` to `ByteString`, e.g., via `new ByteString(String s)` does not add any costs to a contract. |
| `byte[]`/`Byte[]`              | Buffer           | Java's byte arrays map to a stack item called Buffer on the NeoVM. The difference to `ByteString` is that they are mutable. |
| `io.neow3j.devpack.Map`        | Map              | The NeoVM has a dedicated type for maps for which the devpack provides a corresponding `Map` type. Note that it is not possible to use `java.utils.Map` instead. | 
| `io.neow3j.devpack.List`       | Array            | The NeoVM has a dedicated type for arrays of other types. The devpack provides a corresponding type with the `List` class. Note, that it's not possible to use `java.utils.List` instead. Instead of using `List` you can also use array definitions as usual in Java, e.g., `String[]` can be used in place of `List<String>`. |
| `io.neow3j.devpack.Hash160`    | ByteString       | `Hash160` is useful to ensure correct handling of contract and account hashes. On the NeoVM they map to a ByteString. Conversion from `Hash160` to `ByteString` does. |
| `io.neow3j.devpack.Hash256`    | ByteString       | `Hash256` was added to the devpack to ensure correct handling of block and transaction hashes. On the NeoVM they are represented by a ByteString. Therefore, the conversion to `ByteString` does not actually add an instruction on the NeoVM. |
| `io.neow3j.devpack.ECPoint`    | ByteString       | `ECPoint` was added to the devpack to ensure correct handling of elliptic curve points. On the NeoVM they are represented by a ByteString. Therefore, the conversion to `ByteString` does not actually add an instruction on the NeoVM. |


### Byte Arrays

The NeoVM has two stack items for containing byte arrays. The ByteString is an immutable byte array while Buffer is
mutable. We recommend to use the devpack's `ByteString` by default and only use `byte[]` (which maps to the Buffer
stack item) if you need mutability. For example, for method parameters `ByteString` is the right choice, making it clear
that the method will not have any side effects on the argument. You can convert `ByteString` to `byte[]` with the
`toByteArray()` method and the other way around by using the `ByteString(byte[] buffer)` constructor.

### Integers

You can use all Java integer types, including their wrapper classes. That is, `byte`, `short`, `char`, `int`,
`long` and `Byte`, `Short`, `Character`, `Integer`, `Long`. Because the NeoVM knows only one integer stack item, the 
neow3j compiler treats all these types qually. Non of them have any size restrictions. Think of all integer types as
BigIntegers. This means that Java's definition of an `int` having a size of 32 bit does not hold true for smart
contracts. You can use `int` without worrying about size restrictions. We recommend using `int`/`Integer` everywhere and
ignoring `short`, `char`, and `long`. `Byte` and `byte` are useful for indicating small values, like constants,
and they appear in `byte[]`/`Byte[]`.

Note that when casting an `int` variable to a `byte`, the underlying value is not truncated from 32 to 8 bit. Merely the
superficial Java type canges from `int` to `byte`. The same applies if you use wrapper type methods like
`Integer.byteValue()`. The devpack provides two helper methods `Helper.asByte(int value)` and 
`Helper.asSignedByte(int value)` that you can to convert `int` to `byte` if you know that the integer value fits into a 
byte. The methods do not truncate the argument, but throw an exception if the value doesn't fit into a byte.

### Strings

Intuitively you can use Java's `String` type for strings. But be aware that on the neo-vm a `String` is not represented
as an object, but as a UTF8-encoded ByteString stack item. This means that you cannot make use of `String` instance
methods like `contains()` or `indexOf()`. The exception is the `length()` method, which works as expected. Checkout the
`io.neow3j.devpack.Helper` and `io.neow3j.devpack.StringLiteralHelper` clases which offer of String related helper
methods.

Neow3j supports string concatenation with the `+` operator but mixing in other types is not. For example, 
`"hello" + "world"` works, but `"hello" + 5` does not. 

### Objects

Instances of all other devpack classes not mentioned in the above table, and also the classes you define yourself, are
represented as Arrays on the NeoVM. The NeoVM array contains the object's field variables in the order they appear in
the class definition. I.e., in the following example the `lowNote` variable has index 0 and `highNote` has index 1 in
the array that represents an instance of this class on the NeoVM.

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

The devpack itself defines classes that can be instantiated. For example, calling `Storage.getStorageContext()`
returns a `StorageContrext` instance. On that object you can call instance methods like `createMap(...)`.  Another
example is the `io.neow3j.devpack.neo.Transaction` class. After retrieving a `Transaction` object with
`LedgerContract.getTransaction(Hash256 txId)` all its properties are available to the contract class through public
instance variables on the object. Of course, we could add getter and setter methods to the classes instead of accessing
the members directly, but that incurs a higher GAS fee when executing the contract, because of the extra method call.


## Contract Class

NeowJava smart contracts consist of one main contract class and possibly other classes containing functionality used in
the main class. We refer to this main class as the *contract class* and to the others as *auxiliary classes*. Only
methods of the contract class are accessible and show up in the contract manifest if they are public.

Everything on the contract class is static. The concept of creating an instance of it and deploying it on the blockchain
does not apply here. The contract's state is not saved in its class variables but accessed through a storage API. So
rather think of the contract class as being the managing entity handling incoming invocations, contract storage and
event emission but not actually holding state itself. This is different, for example, to Solidity where the class
variables hold the contract state. Auxiliary classes can be purely static classes too, basically serving as a collection
of functions which the contract class makes use of. But, they can also be classes in an object-oriented sense that get
instantiated in the contract class and provide structure and functionality to contract's data.

When compiling a smart contract, the contract class is the input to the compilation. The compiler will also look for and
compile any auxiliary classes used in the contract class.

### Contract Methods

All methods on the contract class need to be static. If you mark a method with the `public` modifier it show up in the
contract's manifest and is callable from the outside. Any other access modifier will make the a method invisible to
the outside, i.e., it's not necessary to use `private`. Of course, you can use the `private` modifier to make it
explicit and to prohibit any auxiliary class to call it. Making methods in auxiliary contract public will not put them
into the contract manifest. If these classes are in the same package as your contract class it's enough to use no
access modifier. If they are in another package they need to be public as with normal Java.

### Contract Variables

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