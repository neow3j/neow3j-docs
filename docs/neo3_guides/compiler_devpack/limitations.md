# Limitations

Because the NeoVM and the Java virtual machine differ in capabilitites, the neow3j compiler cannot
support the full feature set of the Java language. Therefore, when you write a smart contract in
Java be aware of the following limitations. This list is not exhaustive because the compiler is
still under development.

## Number Types

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

## Strings

Java `String` is supported, but only with limited features. Strings are represented as UTF8-encoded
byte arrays in the NeoVM. The neow3j devpack offers sevaral methods that work with Strings but the
methods of `String` intself are not yet supported. E.g. `s.concat()` or `s.indexOf()` are not
available in smart contracts.

If you want to compare two strings use the `==` operator. The neow3j compiler currently does not
support the `equals()` method but uses the `==` operator instead for equality checks. The compiled
NeoVM code will not perform a reference comparison but an actual object comparison when using `==`.

## Classes, objects, methods, and variables

A Java-based smart contract consists of one class. Because the NeoVM does not support the concept of
objects the usage of classes is restricted to their static methods, with some exceptions. This might
change in the future if we can introduce a reasonable compromise to translate Java objects into
NeoVM structs.  
Thus, all methods in your smart contract class have to be static.
Class variables are also restricted to static ones. Moverover, static constructors are not
supported. You can initialize static class variables right where they are defined. The
initialization can be more complex (e.g., include method calls) than just a simple constant value
assignment.

Constructing new objects with the `new` keyword is unsupported. Exceptions are the primitive type
wrapper classes (e.g., `Byte`) and arrays. This restriction implies that you cannot use common
objects like `java.lang.List` or `java.lang.Map`. Use the related classes from the neow3j devpack
instead, e.g., `io.neow3j.devpack.framework.Enumerator`.

## Arrays

The NeoVM does not support array initialization with non-constant length. Thus the following example
will not work.

```java
public static byte[] doSomething() {
    byte[] b2 = new byte[b1.length];
    return b2;
}
```

## Inheritance

Inheritance is not supported.

## Enums

Enums are not supported.

## Interfaces and abstract classes

Interfaces and abstract classes are not supported.
