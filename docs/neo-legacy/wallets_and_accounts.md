# Wallets and Accounts

The concept of wallets and accounts is as follows. An account entails a cryptographic key pair and the address that
corresponds to the keys. It has a balance of global assets and tokens and can interact with smart contracts. A wallet is
just a collection of accounts. For example, if one has assets distributed over multiple accounts, a wallet is a useful
abstraction when spending does assets without caring from which account they are taken.

## Creating a wallet

The easiest way to create a new wallet is by using one of the static creation methods.

```java
Wallet w = Wallet.createGenericWallet();
```

This creates a wallet with a new account (new key pair). There are other versions of this method that allow us to
immediately encrypt the new private key or directly write the wallet to a file after creation.

Besides the static creation methods, the `Wallet` class applies the builder pattern for creating new instances. You can
instantiate the builder by yourself or start the building process with one of the static `fromNEP6Wallet(...)` methods
that read the wallet information from a NEP-6 wallet file but still let you make changes before instantiating the wallet
completely.

```java
String absoluteFileName = "/path/to/your/NEP6.wallet";
Wallet w = Wallet.fromNEP6Wallet(absoluteFileName)
        .name("NewName")
        .build();
```

> **Note**: When reading the wallet from a NEP-6 wallet file the private keys of the contained accounts will be
> encrypted until you call `decryptAllAccounts(String password)` on the wallet. An encrypted account cannot be used for
> signing transactions.

If you don't want to load from a NEP-6 wallet file, then use the builder directly. Every attribute that you don't set on
the builder is populated by a default value. Except for the accounts. If you don't specify one or more accounts the
wallet will not contain any.

```java
Wallet w = new Wallet.Builder()
        .name("MyWallet")
        .version(1)
        .account(Account.create())
        .build();
```

## Creating an account

The easiest way to create a new account is by using the static creation method that bypasses the builder pattern. This
account holds a fresh key pair.

```java
Account a = Account.create();
```

The other static creation methods return a builder for further configuration. You are forced to use one of these and
cannot instantiate the builder directly. This makes sure that only valid accounts can be created. All attributes that
are not explicitly set on the builder are set implicitly to default values. An account is by default not locked and not
the default account.

```java
Account a = Account.fromWIF("KxDgvEKzgSBPPfuVfw67oPQBSjidEiqTHURKSDL1R7yGaGYAeYnr")
        .isDefault(true)
        .label("MyAccount")
        .build();
```

Besides creating an account from a WIF you can provide a NEP-6 account object, a key pair or only an address. Beware
that in case of an address the account does not have any key pair with which it could sign transactions. Neow3j can
therefore not automatically sign transactions made with this account. In those cases, the signature has to be provided
manually.

You can also create a multi-sig account with the `fromMultiSigKeys(...)` method. The account will hold the multi-sig
address and a NEP-6 contract object with the corresponding verification script. It will not hold the public keys nor
private keys of the involved accounts and can therefore not automatically sign transactions.

```java
List<BigInteger> publicKeys = Arrays.asList(
        ECKeyPair.createEcKeyPair().getPublicKey(),
        ECKeyPair.createEcKeyPair().getPublicKey(),
        ECKeyPair.createEcKeyPair().getPublicKey()
);

Account a2 = Account.fromMultiSigKeys(publicKeys, 2)
        .label("MyMultiSigAccount")
        .build();
```

## Account Balances

An important part of an account is its asset and token balances. Be aware that an account's balances are not constantly
synced with the blockchain state. When you use an account for example in a transaction that requires assets or tokens,
you need to make sure that the local balance information is up to date before making the transaction. This can be done
with the methods `updateAssetBalances(Neow3j neow3j)` and `updateTokenBalances(Neow3j neow3j)`, which execute an RPC to
retrieve the balance information.

Updating the balances is left to the developer because that gives her the control of how often these RPCs are made.
