# NeowJava

This section discusses the possibilities and restrictions that exist when implementing a smart contract in Java. Even though we are coding in Java the produced byte code is not meant for the JVM but for the NeoVM. Therefore, only a subset of Java can be used and we call that subset NeowJava.

Blockchain programmers need to be careful with computational resources because every step in their code costs GAS. You should **avoid using Java's standard library** and any library not explicitly implemented for smart contract purposes, because such libraries might contain unsupported code or code that is costly for execution on a blockchain.

## Types

The NeoVM works with types called stack items - the NeoVM is a stack machine. The neow3j devpack maps Java types to those stack item types. The following table shows the devpack's most important Java types and their correspdongin NeoVM stack items.

| Java type                      | Stack Item Type | Description |
|--------------------------------|-----------------|-------------|
| `int`/`Integer`                | Integer         | Java integers have a corresponding integer stack item on the NeoVM. The difference to normal Java integers is the range. On the NeoVM the integer range is not restricted to [2<sup>-31</sup>, 2<sup>31</sup>-1]. We discuss integers in more depth below. |   
| `boolean`/`Boolean`            | Boolean         | Java's `boolean` maps to the Boolean stack item on the NeoVM. But, the NeoVM also uses Integer stack items in the range [0,1] for boolean values. Don't worry if you get an Integer stack item as a return value even if your contract method returns a `boolean`. |
| `byte[]`/`Byte[]`              | Buffer          | Java's byte array maps to a stack item called Buffer on the NeoVM. This is a mutable byte aray.
| `io.neow3j.devpack.ByteString` | ByteString      | This is an immutable byte array. The NeoVM ByteString type does not have a corresponding type in Java, thus, `ByteString` was introduced by the devpack. |
| `java.lang.String`             | ByteString      | The Java `String` is represented as a UTF-8 encoded byte array on the NeoVM. The stack item type is ByteString. |
| `io.neow3j.devpack.Map`        | Map             | The NeoVM has a dedicated stack item type for maps. The devpack provides a corresponding `Map` type. Note, that it is not possible to use `java.utils.Map` instead. | 
| `io.neow3j.devpack.List`       | Array           | The NeoVM has a dedicated stack item type for arrays that are not byte arrays. The devpack provides a corresponding `List` type . Note, that it's not possible to use `java.utils.List` instead. |
| Arrays like `int[]`            | Array           | Instead of using `List` you can also use array types, e.g., `String[]`. They are also represented with the Array stack item type. |
| Custom Objects                 | Array           | All other classes, or rather instances of these classes, are represented as Arrays on the NeoVM. A more detailed explanation is given below. |

### Byte Arrays

The NeoVM has two stack items for containing byte arrays. The ByteString is an immutable byte array while Buffer is mutable. We recommend to use the devpack's `ByteString` by default and only use `byte[]` (which maps to the Buffer stack item) if you need mutability. For example, for method parameters `ByteString` is the right choice, making it clear that the method will not have any side effects on the argument. You can convert `ByteString` to `byte[]` with the `toByteArray()` method and the other way around by using the `ByteString(byte[] buffer)` constructor.

### Integers

You can use all Java integer types, including their wrapper classes. That is, `byte`, `short`, `char`, `int`, `long` and `Byte`, `Short`, `Character`, `Integer`, `Long`. Because the NeoVM knows only one integer stack item, the neow3j compiler treats all these types equally. All of them have the same size restrictions. Think of all integer types as BigIntegers. This means that Java's definition of an `int` having a size of 32 bit does not hold true for Neo smart contracts. The limit is much higher, more precisely, an integer on the NeoVM can take up to 32 bytes. We recommend using `int`/`Integer` everywhere and ignoring `short`, `char`, and `long`. `Byte` and `byte` are useful for indicating small values, like constants.

With the above said, the question arises: "How do I initialize large integers?". Java will deny your attempts of initializing an `int` with values outside of the range [2<sup>-31</sup>, 2<sup>31</sup>-1]. To circumvent this you can use the `StringLiteralHelper` and it's method `stringToInt(String number)` that allows you to initialize any integer value.

Note that when casting an `int` variable to a `byte`, the underlying value is not truncated from 32 to 8 bit. Merely the superficial Java type canges from `int` to `byte`. The same applies if you use wrapper type methods like `Integer.byteValue()`. The devpack provides two helper methods `Helper.asByte(int value)` and `Helper.asSignedByte(int value)` that you can to convert `int` to `byte` if you know that the integer value fits into a byte. The methods do not truncate the argument, but throw an exception if the value doesn't fit into a byte.

### Strings

You can use Java's `String` type for strings of text. But be aware that on the NeoVM a `String` is not represented as an object, but as a UTF8-encoded ByteString stack item. This means that you cannot make use of `String` instance methods like `contains()` or `indexOf()`. The exception is the `length()` method, which works but will give you the byte length of the string not the character count. Instead use `io.neow3j.devpack.contracts.StdLib.strLen()` to get the character count of a `String`.

Neow3j supports string concatenation with the `+` operator. But, mixing in other types is not supported. For example,  `"hello" + "world"` works, but `"hello" + 5` does not.

Checkout the `io.neow3j.devpack.Helper` and `io.neow3j.devpack.StringLiteralHelper` classes which offer String related helper methods.

### Arrays

You can use arrays as usual. You will only face some restrictions with multi-dimensional arrays. When initialising a multi-dimensional array you cannot specify the length of all other dimensions but the first one. Thus, the expression `new String[10][4]` will fail, but `new String[10][]` will compile. To set the second dimensions you could cycle through the first dimension and initialize each slot as follows.

```java
String[][] arr = new String[10][];
for (int i = 0; i < arr.length; i++) {
    arr[i] = new String[4];
}
```

Direct instantiatetion with specific values also works.

```java
String[][] arr = new String[][]{null, new String[]{"hello", " ", "world", "!"}};
```

In any case, the compiler will tell you if what you try works or not. You can use `io.neow3j.devpack.List` instead of Java arrays. It provides more convenience than plain arrays. It is represented as the same stack item on the NeoVM as an array and is, therefore, not more expensive.

### Objects

Instances of all other devpack classes not mentioned in the above table, and also the classes you define yourself, are represented as Arrays on the NeoVM. The NeoVM array contains the object's field variables in the order they appear in the class definition. I.e., in the following example the `lowNote` variable has index 0 and `highNote` has index 1 in the array that represents an instance of this class on the NeoVM.

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

The devpack itself defines classes that can be instantiated. For example, calling `Storage.getStorageContext()` returns a `StorageContrext` instance. On that object you can call instance methods like `createMap(...)`. Another example is the `io.neow3j.devpack.neo.Transaction` class. After retrieving a `Transaction` object with `LedgerContract.getTransaction(Hash256 txId)` all its properties are available to the contract class through public instance variables on the object. Of course, we could add getter and setter methods to the classes instead of accessing the members directly, but that incurs a higher GAS fee when executing the contract, because of the extra method call.

## Contract Class

NeowJava smart contracts consist of one main contract class and possibly other classes containing functionality used in the main class. We refer to this main class as the *contract class* and to the others as *auxiliary classes*. Only methods of the contract class are accessible and show up in the contract manifest if they are public.

Everything on the contract class is static. The concept of creating an instance of it and deploying it on the blockchain does not apply here. The contract's state is not saved in its class variables but accessed through a storage API. So rather think of the contract class as being the managing entity handling incoming invocations, contract storage and event emission but not actually holding state itself. This is different, for example, to Solidity where the class variables hold the contract state. Auxiliary classes can be purely static classes too, basically serving as a collection of functions which the contract class makes use of. But, they can also be classes in an object-oriented sense that get instantiated in the contract class and provide structure and functionality to contract's data.

When compiling a smart contract, the contract class is the input to the compilation. The compiler will also look for and compile any auxiliary classes used in the contract class.

### Contract Methods

All methods on the contract class need to be static. If you mark a method with the `public` modifier it will show up in the contract's manifest and is callable from the outside. Any other access modifier will make the method invisible to the outside, i.e., it's not necessary to use `private`. Of course, you can use the `private` modifier to make it explicit and to prohibit any auxiliary class to call it. Making methods in an auxiliary class public will not put them into the contract manifest. If these classes are in the same package as your contract class it's enough to use no access modifier. If they are in another package they need to be public as with normal Java.

### Contract Variables (Static Variables)

All variables on the contract class have to be static. Note that these variables do not represent the contract's state. Their values are not written into the contract's storage. The NeoVM initializes them in every invocation of the contract meaning that you cannot use them to store state. Changes to static variables are lost as soon as an invocations finishes.

You can apply the `final` keyword to a static variable if you want to prohibit changes to its value. But, be aware that static, final variables that are initialised with a constant value (i.e., not derived from a method call) will be inlined wherever they are used. For example, the following variable will not be reused throughout the rest of the contract but its value will be copied to all the script locations where it is used.

```java
static final int initialSupply = 200_000_000;
```

This is fine with small values but might become GAS-intensive with very large values, like long strings, that are used often in the smart contract. It bloats the resulting script and incurs higher GAS costs when invoking the contracts methods that use the variable multiple times.

In classes that are not the main contract class, the only static variables that are currently supported are final static variables that are initialized with a constant value, as `initialSupply` above. Other static variables will lead to a compiler error. 

You can initialize contract variables with constant values (e.g., string literals) but also with a return value of a method call as shown in the following examples. As mentioned above, the variables that are dynamically initialised, are not inlined when marking them with `final`.

```java
static int initialSupply = 200_000_000;
static String totalSupplyKey = "totalSupply";
static StorageContext sc = Storage.getStorageContext();
static StorageMap assetMap = new StorageMap(sc, "assets");
```

Neow3j also supports the static initializer clause as shown below. But, the instance initializer, i.e., the same clause without the `static` keyword is not supported.

```java
static final String assetPrefix;
static final StorageContext sc;

static {
    assetPrefix = "asset";
    sc = Storage.getStorageContext();
}
```

For convenience in variable initialization, the devpack offers the `StringLiteralHelper`.  As its name suggests, it operates on string literals. It offers methods that take string literals and convert them to byte arrays, script hashes, or integers. The following example uses it to set the owner script hash using the owner account's address.

```java
static final Hash160 owner = StringLiteralHelper.addressToScriptHash("NZNos2WqTbu5oCgyfss9kUJgBXJqhuYAaj");
```

## Exceptions

Neow3j supports exceptions and try-catch blocks, but restricts you to using the `java.lang.Exception` class. You can either use the constructor with a string argument (`new Exception(String message)`) or the one without any arguments (`new Exception()`). In case no message is provided, a default message `"error"` is passed.

There is a special case for assertions described [below](#assertions), otherwise, no other exception types are permitted. This restriction implies that you cannot have multiple catch clauses that each handle a different exception type.

If you want to get the exception message in case of a caught exception, you can use the method `getMessage()` as in the following example:

```java
try {
    throw new Exception("Not allowed.");
} catch (Exception e) {
    e.getMessage(); // equals "Not allowed."
}
```

## Assertions

Neow3j supports Java's `assert` statement. It is converted to the `ASSERT` opcode on the Neo VM, which is not catchable. If an assertion fails, the Neo VM faults and the transaction reverts. The opcode on the Neo VM does not consume a message. Hence, passing a message to Java's `assert` is not allowed, and will result in a `CompilerException` at compile time.

```java
assert getRemainingBalance() >= 0;
```

## Type Comparison

As you know, in Java, the `==` operator compares primitive data types by value and complex type by reference. In NeowJava behaves differently. When using `==` the following rules apply.

- The types that compare by value are: `int`, `Integer`, `boolean`, `Boolean`, `String`, `ByteString`, `Hash160`, `Hash256`, `ECPoint`, `Iterator.Struct`.  

- The types that compare by reference are:`byte[]`, `List`, `Map`, arrays (e.g., `int[]`), custom objects. 

If a devpack class provides an `equals` method, you can be assured it is a comparison by value. Take `String` as an example. Using `==` to compare two strings in NeowJava will result in a comparison by value. Using `equals` does the same.

## Type Checking

The neow3j compiler supports the `instanceof` keyword for the following types: `int`, `Integer`,`boolean`, `Boolean`, `byte[]`, `String`, `ByteString`, `Hash160`, `Hash256`, `ECPoint`, `Map`, `List`, `Notification`, `InteropInterface`, `Iterator.Struct`, arrays.

Custom objects are not supported with the `instanceof` operation. This is because the NeoVM doesn't carry the type information with custom objects on the stack. They are represented as array stack items without type information. The neow3j compiler will throw an error if you use `instanceof` with an unsupported type.

## Structs

For a collection of specific types you can create a class with field variables of different types. On the NeoVM this will refer to a struct stack item. In order to use a class as a struct, you can use the annotation `io.neow3j.devpack.annotations.Struct`. You can then set up a constructor that sets its values. Have a look at the following example:

```java
@Struct
public static class ExampleStruct {
    public int id;
    public Hash160 owner;
    public Map<String, Integer> customValues;

    ExampleStruct(int id, Hash160 owner, Map<String, Integer> customValues) {
        this.id = id;
        this.owner = owner;
        this.customValues = customValues;
    }
}
```

## Inheritance

Every Java class that doesn't explicitly extend another class is a subclass of `Object` and has access to its methods even without overwriting them. However, NeowJava prohibits usage of `Object` methods like `toString` or `equals` if they are not explicitly overridden.

Neow3j supports the inheritance of struct classes, i.e., classes annotated with `@Struct`. You can use the keyword `extends` to inherit field variables from another struct. In the following example, by instantiating a `Car`, the created struct on the NeoVM will hold both variables `brand` and `model`.

```java
class Vehicle {
    public String brand;

    Vehicle(String brand) {
        this.brand = brand;
    }
}

class Car extends Vehicle {
    public String model;

    Car(String brand, String model) {
        super(brand);
        this.model = model;
    }
```

Further support for inheritance with the `extends` keyword is in development. It currently only with structs and Contract Interfaces as described [here](neo-n3/smart_contract_development/devpack.md#smart-contract-interfaces).

## Unsupported Features

First of all, the neow3j compiler is based on Java 8. Thus, any Java features added in higher versions are not supported. Other features that Java 8 includes but the neow3j compiler does not support are:

- Floating point numbers, i.e., `float`/`Float` and `double`/`Double`. Floating point numbers are not supported by the
    NeoVM and can, thus, not be used in NeowJava smart contracts. Everything happens with integers.

- Enums

- Lambda Expressions

- Interfaces
