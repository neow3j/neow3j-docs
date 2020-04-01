# Key Pairs and NEO Addresses

In neow3j the concept of a cryptographic "key pair" is represented by the `ECKeyPair` class. This class holds the respective public and private keys as integers. The curve used for all of neow3j's elliptic curve cryptography (ECC) is _secp256r1_, which, of course, is congruent with NEO.

Neow3j provides many convenient methods to perform operations related to key pairs and NEO addresses.

You can find running code examples related to key pairs and addresses [here](https://github.com/neow3j/neow3j-examples-java/tree/master/src/main/java/io/neow3j/examples/keys).


## Create a new Key Pair

To simply create a new key you can perform the following.

```java
ECKeyPair ecKeyPair = Keys.createEcKeyPair();
```


## Get the WIF from a Key Pair

WIF is the acronym for [Wallet Import Format](https://en.bitcoin.it/wiki/Wallet_import_format). A WIF is a private key encoded in such a way to make it more practical to copy it in string representation.

Based on an `ECKeyPair` instance, you can do the following.

```java
String wif = ecKeyPair.exportAsWIF();
```

## Create a Key Pair from WIF

If you have the WIF of a private key, you can perform the following to recover the `ECKeyPair` instance from it. The corresponding public key of the key pair is derived from the private key.

```java
ECKeyPair ecKeyPair = ECKeyPair.create(WIF.getPrivateKeyFromWIF("Kx9xMQVipBYAAjSxYEoZVatdVQfhYHbMFWSYPinSgAVd1d4Qgbpf"));
System.out.println("Private Key (Hex): " + Numeric.toHexStringNoPrefix(ecKeyPair.getPrivateKey()));
System.out.println("Public Key (Hex): " + Numeric.toHexStringNoPrefix(ecKeyPair.getPublicKey()));
```


## Get the NEO Address from a Key Pair

Once you have the `ECKeyPair` instance, you can derive the corresponding NEO address from it. Check out the NEO [documentation](https://docs.neo.org/developerguide/en/articles/wallets.html#3_address) to see how an address is derived from the public key.

```java
String neoAddress = Keys.getAddress(ecKeyPair);
```


## Create a multi-sig NEO Address

The whole concept of a multi-signature address, or in short "multi-sig", is to create an address that is controlled by several parties. The multi-sig address is created from multiple key pairs. It is called multi-sig because, for making a valid transaction from a multi-sig address, multiple signatures are be needed. Each signature is created by one of the parties that holds one of the key pairs.

Thus, it's possible in neow3j to create an address from multiple `ECKeyPair` instances. When creating that address you need to define the minimum number of signatures needed when using that address in a transaction. For example, you can create a NEO address that requires at least 3 signatures out of 5 possible:

```java
String multiSigAddress = Keys.getMultiSigAddress(
                              3,
                              ecKeyPair1.getPublicKey(),
                              ecKeyPair2.getPublicKey(),
                              ecKeyPair3.getPublicKey(),
                              ecKeyPair4.getPublicKey(),
                              ecKeyPair5.getPublicKey()
);
```

Note that the order in which the keys are added here is important. When later creating signatures for a transaction with this address, the order must be the same.
