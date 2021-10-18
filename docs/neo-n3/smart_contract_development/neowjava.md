# NeowJava

This section discusses the possibilities and restrictions that exist when implementing a smart contract in Java. Even
though we are coding in Java the produced byte code is not meant for the JVM but for the NeoVM. Therefore, only a subset
of Java can be used and we call that subset NeowJava.

Blockchain programmers need to be careful with computational resources because every step in their code costs GAS.
You should **avoid using Java's standard library** and any library not explicitely implemented for smart contract
purposes, because such libraries might contain unsupported code or code that is costly for execution on a blockchain.

## Types

The NeoVM works with types called stack items (because the NeoVM is a stack machine). These stack
items don't all have a matching type provided by Java and its standard library. Thus, the devpack adds new types that
map to the corresponding stack item on the NeoVM. The following table shows which Java types map to which NeoVM stack
items. 

| Java type                      | Stack Item | Description |
|--------------------------------|------------|-------------|
| `int`/`Integer`                | Integer    | Java integers have a corresponding integer stack item on the NeoVM. The difference to normal Java integers is the range. On the NeoVM the integer range is not restricted to [2<sup>-31</sup>, 2<sup>31</sup>-1]. We discuss integers in more depth below. |   
| `boolean`/`Boolean`            | Boolean    | Java's `boolean` maps to the Boolean stack item on the NeoVM. But, the NeoVM also uses Integer stack items in the range [0,1] for boolean values. Don't worry if you get an Integer stack item as a return value even if your contract method returns a `boolean`. |
| `io.neow3j.devpack.ByteString` | ByteString | A NeoVM ByteString is an immutable byte array. This is a type introduced by the devpack because there exists no corresponding Java type for the ByteString stack item. |
| `java.lang.String`             | ByteString | Java `String`s map to UTF-8 ByteStrings on the NeoVM. Thus, converting from `String` to `ByteString`, e.g., via `new ByteString(String s)` does not add any costs to a contract. |
| `io.neow3j.devpack.Hash160`    | ByteString | `Hash160` is useful to ensure correct handling of contract and account hashes. On the NeoVM they map to ByteString. |
| `io.neow3j.devpack.Hash256`    | ByteString | `Hash256` is useful to ensure correct handling of block and transaction hashes. On the NeoVM they map to ByteString. |
| `io.neow3j.devpack.ECPoint`    | ByteString | `ECPoint` is useful to ensure correct handling of elliptic curve points. On the NeoVM they map to ByteString. |
| `byte[]`/`Byte[]`              | Buffer     | Java's byte arrays map to a stack item called Buffer on the NeoVM. The difference to `ByteString` is that they are mutable. |
| `io.neow3j.devpack.Map`        | Map        | The NeoVM has a dedicated stack item for maps for which the devpack provides a corresponding `Map` type. Note that it is not possible to use `java.utils.Map` instead. | 
| `io.neow3j.devpack.List`       | Array      | The NeoVM has a dedicated stack item for arrays (other than byte array) for which the devpack provides a corresponding `List` type. Note, that it's not possible to use `java.utils.List` instead. |
| Arrays like `int[]`            | Array      | Instead of using `List` you can also use arrays as usual in Java, e.g., `String[]` can be used in place of `List<String>`. |
| `io.neow3j.devpack.Iterator.Struct` | Struct | The NeoVM's Struct stack item is similar to Array. The devpack uses it only for key-value pairs when iterating over contract storage entries. |
| Custom Objects                 | Array      | All other classes, or rather instances of these classes, are represented as Arrays on the NeoVM. A more detailed explanation is given below. |

Java uses the categories of primitive and complex types. If we carry this concept over to NeowJava we have the following
primitive types: `int` (and all other number types), `boolean`, `byte[]`, `ByteString`, `String`, `Hash160`, `Hash256`,
and `ECPoint`. So, even though some of these types are complex types in the Java world, on the NeoVM they are primitive
types.  
The complex types are `List`, arrays, `Map`, `Iterator.Struct`, and custom objects.

### Byte Arrays

The NeoVM has two stack items for containing byte arrays. The ByteString is an immutable byte array while Buffer is
mutable. We recommend to use the devpack's `ByteString` by default and only use `byte[]` (which maps to the Buffer
stack item) if you need mutability. For example, for method parameters `ByteString` is the right choice, making it clear
that the method will not have any side effects on the argument. You can convert `ByteString` to `byte[]` with the
`toByteArray()` method and the other way around by using the `ByteString(byte[] buffer)` constructor.

### Integers

You can use all Java integer types, including their wrapper classes. That is, `byte`, `short`, `char`, `int`,
`long` and `Byte`, `Short`, `Character`, `Integer`, `Long`. Because the NeoVM knows only one integer stack item, the 
neow3j compiler treats all these types equally. None of them have any size restrictions. Think of all integer types as
BigIntegers. This means that Java's definition of an `int` having a size of 32 bit does not hold true for smart
contracts. You can use `int` without worrying about size restrictions. We recommend using `int`/`Integer` everywhere and
ignoring `short`, `char`, and `long`. `Byte` and `byte` are useful for indicating small values, like constants,
and they appear in `byte[]`/`Byte[]`.

With the above said, the question arises: "How do I initialize large integers?". Java will deny your attempts of
initializing an `int` with values outside of the range [2<sup>-31</sup>, 2<sup>31</sup>-1]. To circumvent this you can
use the `StringLiteralHelper` and it's method `stringToInt(String number)` that allows you to initialize any integer
value.

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

###  Multi-dimensional Arrays

Arrays with multiple dimensions are supported. To create a multi-dimensional array without initialization, it is required
to specify the first (and only the first) dimension size. You can, for example, create the following two-dimensional array
and later set the value at index `1` to an array that consists of values of type `String`.

```java
String[][] arr = new String[2][];
arr[1] = new String[]{"hello", " ", "world", "!"};
```

The above multi-dimensional array can also be initialized directly when creating it.

```java
String[][] arr = new String[][]{null, new String[]{"hello", " ", "world", "!"}};
```


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

### Contract Variables (Static Variables)

All variables on the contract class have to be static. Note that these variables do not represent the contract's state.
Their values are not written into the contract's storage. The NeoVM initializes them in every invocation of the
contract meaning that you cannot use them to store state. Changes to static variables are lost as soon as an invocations
finishes. 

You can apply the `final` keyword to a static variable if you want to prohibit changes to its value. But, be aware that
static, final variables that are initialised with a constant value (i.e., not derived from a method call) will be inlined
wherever they are used. For example, the following variable will not be reused throughout the rest of the contract but
its value will be copied to all the script locations where it is used.
```java
static final int initialSupply = 200_000_000;
```
This is fine with small values but might become GAS-intensive with very large values, like long strings, that are used
often in the smart contract. It bloats the resulting script and incurs higher GAS costs when invoking the contracts
methods that use the variable multiple times.

In classes that are not the main contract class, the only static variables that are currently supported are final static
variables that are initialized with a constant value, as `initialSupply` above. Other static variables will lead to a
compielr error. 

You can initialize contract variables with constant values (e.g., string literals) but also with a return value of a
method call as shown in the following examples. As mentioned above, the variables that are dynamically initialised, are
not inlined when marking them with `final`.

```java
static int initialSupply = 200_000_000;
static String totalSupplyKey = "totalSupply";
static StorageContext sc = Storage.getStorageContext();
static StorageMap assetMap = sc.createMap(assetPrefix);
```

Neow3j also supports the static initializer clause as shown below. But, the instance initializer, i.e., the same
clause without the `static` keyword is not supported.

```java
static final String assetPrefix;
static final StorageContext sc;

static {
    assetPrefix = "asset";
    sc = Storage.getStorageContext();
}
```

For convenience in variable initialization, the devpack offers the `StringLiteralHelper`.  As its name suggests, it
operates on string literals. It offers methods that take string literals and convert them to byte arrays, script hashes,
or integers. The following example uses it to set the owner script hash using the owner account's address.

```java
static final Hash160 owner = StringLiteralHelper.addressToScriptHash("NZNos2WqTbu5oCgyfss9kUJgBXJqhuYAaj");
```

## Exceptions

Neow3j supports exceptions and try-catch blocks, but restricts you to using the `java.lang.Exception` class. You can
either use the constructor with a string argument (`new Exception(String message)`) or the one without any arguments
(`new Exception()`).  No other exception types are permitted. This restriction implies that you cannot have multiple
catch clauses that each handle a different exception type. 

## Assertions

Assertions in NeowJava are handled like exceptions, they are catchable and a message can be passed. You can pass a
message by adding a colon with a string or a method that returns a string. In the following example, if the witness
check fails, an exception is thrown with the message `No authorization.`.

```java
assert Runtime.checkWitness(owner) : "No authorization.";
```

>**Note:** The NeoVM supports an Opcode `ASSERT`, however, it does not (yet) allow to pass a message, hence the above
>workaround.

## Type Comparison

NeowJava has a few caveats when it comes to type comparison. As you know, in Java, the `==` operator compares primitive
data types by value and complex type by reference. In NeowJava this is similar but with some exceptions. 
The types that compare by value are: 

- `int` (and all other number types)
- `boolean`
- `String`
- `ByteString`
- `Hash160`
- `Hash256` 
- `ECPoint`
- `Iterator.Struct` 

If we take the example of `String`, we see that in NeowJava using `==` or `equals(String s)` does the same (comparison
by value), while in Java `==` is a reference comparison.

The types that compare by reference are:
- `byte[]`
- `List`
- Arrays (e.g., `int[]`) 
- `Map`
- Custom objects. 

If you want to compare does types by value you need to write extra methods to do so, i.e., implement `equals(...)`.

## Instance of

The neow3j compiler supports the `instanceof` keyword for only for the following types:

- `int` (and all other number types)
- `boolean`
- `byte[]`
- `java.lang.String`
- `io.neow3j.devpack.ByteString`
- `io.neow3j.devpack.Map`
- `io.neow3j.devpack.List`
- `io.neow3j.devpack.InteropInterface`
- `io.neow3j.devpack.Iterator.Struct`

This is because only these types have a corresponding stack item on the NeoVM for which a type check can be made. The
NeoVM will represent all other classes that you create as Array stack items without any type information.  Thus,
checking the type with `instanceof` is not possible. The neow3j compiler will throw an error if you try to put any other
type than what's on the list above on the right side of `instanceof`.

## Inheritance

Every Java class that doesn't explicitely extend another class is a subclass of `Object` and has access to its methods
even without overwriting them. However, NeowJava prohibits usage of `Object` methods like `toString` or `equals`
if they are not explicitely overridden. 

Suport for inheritance with the `extends` keyword is in development. It currently only works properly with Contract
Interfaces as described [here](neo-n3/smart_contract_development/devpack.md#smart-contract-interfaces).

## Unsupported Features

First of all, the neow3j compiler is based on Java 8. Thus, any Java features added in higher versions are not
supported. Other features that Java 8 includes but the neow3j compiler does not support are:

- Floating point numbers, i.e., `float`/`Float` and `double`/`Double`. Floating point numbers are not supported by the
  NeoVM and can, thus, not be used in NeowJava smart contracts. Everything happens with integers.

- Enums

- Lambda Expressions

- Interfaces