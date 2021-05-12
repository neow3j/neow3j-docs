# Wallets and Accounts

Accounts and wallets are important concepts in blockchain. A Neo account is made up of an EC key pair. From its public
key an address is derived, which is used to identify the account. A wallet is a collection of one or multiple accounts
and can be used as an abstraction if one has multiple accounts and doesn't care which account is used in a transaction.

## Creating a Wallet

The easiest way to create a new wallet is by using one of the static creation methods.

```java
Wallet w = Wallet.create();
```

This creates a wallet with a new account (new key pair). There are other versions of this method that allow us to
immediately encrypt the new private key or directly write the wallet to a file after creation.

If you have a NEP-6 wallet file exported from some other wallet software, you can use
`fromNEP6Wallet(...)` which reads the wallet information from the NEP-6 file.

```java
String absoluteFileName = "/path/to/your/NEP6.wallet";
Wallet w = Wallet.fromNEP6Wallet(absoluteFileName)
        .name("NewName");
```

> **Note**: When reading the wallet from a NEP-6 wallet file the private keys of the contained accounts will be
> encrypted until you call `decryptAllAccounts(String password)` on the wallet. An encrypted account cannot be used for
> signing transactions.

If you don't want to load from a NEP-6 wallet file, then create an `Account` and use the static method `withAccounts(...)`
to initialize a wallet. Further, you can manually set a name, version and scrypt parameters for the wallet. If nothing is
set, default values are used.

```java
Wallet w = Wallet.withAccounts(Account.create())
        .name("MyWallet")
        .version("1.0");
```

## Creating an Account

You can use multiple static methods to create a new account. The following account holds a fresh key pair.

```java
Account a = Account.create();
```

If you already have a key pair and its WIF, you can use the following method.

```java
Account a = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda")
        .label("MyAccount");
```

Besides creating an account from a WIF you can provide a NEP-6 account object, a key pair or only an address. Beware
that in case of an address the account does not have any key pair with which it could sign transactions. Neow3j can
therefore not automatically sign transactions made with this account. In those cases, the signature has to be provided
manually.

You can also create a multi-sig account with the `Account.createMultiSigAccount(...)` method.  The account object will
hold the multi-sig address and the corresponding verification script. It will not hold the key pairs of the involved
accounts.  For automatic signing of transactions issued from a multi-sig account, all the accounts involved in the
mulsi-sig address need to be in the wallet.

```java
List<ECPublicKey> publicKeys = Arrays.asList(
        ECKeyPair.createEcKeyPair().getPublicKey(),
        ECKeyPair.createEcKeyPair().getPublicKey(),
        ECKeyPair.createEcKeyPair().getPublicKey()
);

Account a2 = Account.createMultiSigAccount(publicKeys, 2)
        .label("MyMultiSigAccount");
```

## Account Balances

To check the NEP-17 balances of an account, use the following method.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
Map<Hash160, BigInteger> nep5Balances = a.getNep17Balances(neow3j);
```

This returns a map containing all NEP-17 contract balances of the account.
