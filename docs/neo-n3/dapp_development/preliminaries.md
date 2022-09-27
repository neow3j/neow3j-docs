# Preliminaries

The following sections introduce general concepts and types that you will meet throughout the neow3j SDK.
## Hashes

Hashes appear a lot in the blockchain world and Neo is not an exception. There are two kinds of hashes in Neo. One
is 256 bit long and produced by applying the SHA-256 (member of the SHA-2 cryptographic hash functions) twice to some
input data. It is used for transaction and block hashes. The other is 160 bit long and produced by first applying
SHA-256 and then RIPEMD-160 to some input data. It is used for identifying accounts and contracts in the form of script
hashes, i.e., hashes of the scripts underlying those accounts and contracts.  

The neow3j SDK uses the types `io.neow3j.contract.Hash256` and `io.neow3j.contract.Hash160` for those two hash types,
respectively. You will see that the SDK's API almost always requires you to work with these two types instead of a
simple string or byte array. Note, that the hashes stored by these types are stored in big-endian order but can be
retrieved in little-endian order via the `toLittleEndianArray()` method. Endianness might be an issue when inspecting
results from contract invocations, because the Neo node might return a hash in little-endian order, which you have to be
aware of when constructing a `Hash160` or `Hash256`.


## NeoVM Stack Items

The NeoVM is the virtual machine on which all smart contract code is executed. When invoking a contract on the 
Neo blockchain, the result of that invocation is a list of items that are left on the NeoVM's stack at the end of the
 invocation. These items are called stack items and are represented by the `StackItem` class in neow3j SDK. Stack items
can be one of several defined types that exist on the NeoVM and they have to be mapped into Java types when the SDK
receives them from a Neo node. This mapping is not a simple one-to-one mapping but leaves space for more interpretation.
For example, a stack item containing an integer can be interpreted as a boolean value if it is in the range of 0 and 1.
Or a byte array can be interpreted as an integer value.

The `StackItem` class in neow3j lets you choose to which Java type you want to "cast" a stack item. Any stack item that
is returned by a Neo node will be wrapped into the `StackItem` class. Then, if you know that an invocation is returning
a boolean value you can call `stackItem.getBoolean()` and you will receive true or false even if the stack item is an
integer of value 0 or 1 behind the scenes. If the stack item is not castable to the type you desire, an exception will
be thrown. E.g., if you call `stackItem.getMap()` on an integer stack item, it will not give you a map-interpretation of
that integer but simply throw an exception. Thus, you still need to know what type of stack item an invocation will return.
