# Key Pairs and NEO Addresses

`neow3j` has the concept of a "key pair" represented by the `ECKeyPair` class. Such class stores the respective public and private keys.

It's possible to either generate a brand new key pair -- which will give a new NEO address -- or, yet, load an existing key pair.

The `neow3j` library provides several classes that acts as convenience wrappers to perform operations related to key pais and NEO addresses.

You can find running examples related to key pairs and addresses, [here](https://github.com/neow3j/neow3j-examples/tree/master/src/main/java/io/neow3j/examples/keys).

## Create a new Key Pair

You can perform the following:

```java
ECKeyPair ecKeyPair = Keys.createEcKeyPair();
```

## Get the WIF from a Key Pair

WIF is the acronym for [Wallet Import Format](https://en.bitcoin.it/wiki/Wallet_import_format).

Based on the a `ECKeyPair` instance, you can:

```java
String wif = ecKeyPair.exportAsWIF();
System.out.println("WIF: " + wif);
```

## Create a Key Pair from WIF

If you have the WIF of a private key, you can perform the following to get the `ECKeyPair` instance:

```java
ECKeyPair ecKeyPair = ECKeyPair.create(WIF.getPrivateKeyFromWIF("Kx9xMQVipBYAAjSxYEoZVatdVQfhYHbMFWSYPinSgAVd1d4Qgbpf"));
System.out.println("Private Key (Hex): " + Numeric.toHexStringNoPrefix(ecKeyPair.getPrivateKey()));
System.out.println("Public Key (Hex): " + Numeric.toHexStringNoPrefix(ecKeyPair.getPublicKey()));
```

## Get the NEO Address from a Key Pair

Once you have the `ECKeyPair` instance, it's possible to get the corresponding NEO address:

```java
String neoAddress = Keys.getAddress(ecKeyPair);
System.out.println("NEO Address: " + neoAddress);
```

## Create a multisig NEO Address

The whole concept of a multi-signature address, or in short "multisig", is to create an address that is controlled by several parties. The parties, in this context, are key pairs holders.

Thus, it's possible in `neow3j` to create an address from multiple `ECKeyPair` instances -- which, in the NEO blockchain, would require the signature of all `ECKeyPair` instances to unlock transactions.

For example, you can create a NEO address that requires at least 3 signatures out of 5 possible:

```java
String multiSigAddress = Keys.getMultiSigAddress(
          						3,
                				ecKeyPair1.getPublicKey(),
                				ecKeyPair2.getPublicKey(),
                                ecKeyPair3.getPublicKey(),
                                ecKeyPair4.getPublicKey(),
                                ecKeyPair5.getPublicKey()
);
System.out.println("NEO MultiSig Address: " + multiSigAddress);
```


