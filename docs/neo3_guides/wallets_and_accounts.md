# Wallets and Accounts

The concept of wallets and accounts in Neo is as follows. An account entails a cryptographic key pair and the address that
corresponds to the keys. It has a balance of global assets and tokens and can interact with smart contracts. A wallet is
just a collection of accounts. For example, if one has assets distributed over multiple accounts, a wallet is a useful
abstraction when spending assets without caring from which account they are taken.

## Creating a Wallet

The easiest way to create a new wallet is by using one of the static creation methods.

```java
Wallet w = Wallet.createWallet();
```

This creates a wallet with a new account (new key pair). There are other versions of this method that allow us to
immediately encrypt the new private key or directly write the wallet to a file after creation.

There are multiple static creation methods. E.g. you can use `fromNEP6Wallet(...)` which reads the wallet information
from a NEP-6 wallet file.

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
Wallet w = Wallet.withAccounts(Account.createAccount())
        .name("MyWallet")
        .version(1);
```

## Creating an Account

You can use multiple static methods to create a new account. The following account holds a fresh key pair.

```java
Account a = Account.createAccount();
```

If you already have a key pair and its wif, you can use the following method.

```java
Account a = Account.fromWIF("L1WMhxazScMhUrdv34JqQb1HFSQmWeN2Kpc1R9JGKwL7CDNP21uR")
        .label("MyAccount");
```

Besides creating an account from a WIF you can provide a NEP-6 account object, a key pair or only an address. Beware
that in case of an address the account does not have any key pair with which it could sign transactions. Neow3j can
therefore not automatically sign transactions made with this account. In those cases, the signature has to be provided
manually.

You can also create a multi-sig account with the `createMultiSigAccount(...)` method. The account will hold the multi-sig
address and a NEP-6 contract object with the corresponding verification script. It will not hold the public keys nor
private keys of the involved accounts and can therefore not automatically sign transactions.

```java
List<BigInteger> publicKeys = Arrays.asList(
        ECKeyPair.createEcKeyPair().getPublicKey(),
        ECKeyPair.createEcKeyPair().getPublicKey(),
        ECKeyPair.createEcKeyPair().getPublicKey()
);

Account a2 = Account.fromMultiSigKeys(publicKeys, 2)
        .label("MyMultiSigAccount");
```

## Account Balances

To check the balances of an account, use the following method.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
Account a = Account.createAccount();

Map<ScriptHash, BigInteger> nep5Balances = a.getNep5Balances(neow3j);
```

This returns a map of Nep-5 contracts to the respective balance of the account `a`.
Checking the balances is left to the developer because that gives her the control of how often these RPCs are made.
