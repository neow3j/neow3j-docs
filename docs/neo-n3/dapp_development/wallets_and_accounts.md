# Wallets and Accounts

Accounts and wallets are important concepts in blockchain. A Neo account in its basic form consists of an EC key pair. From this EC key pair, a `script hash` and an `address` can be derived, which are used to identify the account.

In addition to this simple form, there are also multi-signature (multi-sig) accounts that consist of multiple public keys participating in the account. Neow3j uses the same `Account` class to represent both single- and multi-sig accounts.

A wallet is a collection of one or more accounts.

## Wallets

The easiest way to create a new wallet is by using one of the static creation methods.

```java
Wallet w = Wallet.create();
```

This creates a wallet with a new account (with a new key pair). There are other versions of this method that allow you to immediately encrypt the new private key or directly write the wallet to a file after creation.

[NEP-6](https://github.com/neo-project/proposals/blob/master/nep-6.mediawiki) is the wallet standard for Neo. If you have a NEP-6 wallet file exported from some other wallet software, you can use `fromNEP6Wallet(...)` to read the wallet information from the NEP-6 file.

```java
String absoluteFileName = "/path/to/your/NEP6.wallet";
Wallet w = Wallet.fromNEP6Wallet(absoluteFileName)
        .name("NewName");
```

> **Note:** When reading the wallet from a NEP-6 wallet file, the private keys of the contained accounts will be encrypted until you call `decryptAllAccounts(String password)` on the wallet. An encrypted account cannot be used for signing transactions.

If you already have `Account` objects, you can create a wallet with the static method `withAccounts(...)`. Furthermore, you can manually set a name, version, and [scrypt](https://en.wikipedia.org/wiki/Scrypt) parameters for the wallet. If nothing is set, default values are used.

```java
Wallet w = Wallet.withAccounts(Account.create())
        .name("MyWallet")
        .version("1.0");
```

> **Note:** If you want to extract a wallet instance, for example as a data transfer object (DTO), make sure to create a `NEP6Wallet` instance and use this instance. To do so, you will need to encrypt the wallet first and then call `toNEP6Wallet`. This automatically initializes `NEP6Account` instances for all accounts in the wallet.

## Accounts

Accounts in neow3j can be created with or without EC key material. If the private key of the account is available, the account can be used for signing arbitrary data, which includes transactions or other raw data.

The following methods create an account with the private key available:

- `create()`
- `new Account(ECKeyPair ecKeyPair)`
- `fromNEP6Account(NEP6Account nep6Account)` - Requires decryption of the private key with `decryptPrivateKey(...)`.
- `fromWIF(String wif)`

It is also possible to create an account without a private key. This applies, for example, to a multi-sig account where only the public keys or its derived verification script are known. You can create an account without a private key with the following static methods:

- `fromAddress(String address)`
- `fromScriptHash(Hash160 scriptHash)`
- `fromPublicKey(ECPublicKey publicKey)`
- `fromVerificationScript(VerificationScript script)`

> **Note:** If you are working with Neo addresses or `watch-only` accounts (i.e., without a private key), you can use the `Hash160` class for that. However, if you are working with wallets and using NEP-6 wallet files, you can use the `Account` class for that.

An account can hold a label that is set to its address by default. If needed, you can change it to a custom label using the `label(String)` method.

```java
Account a = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda")
        .label("MyAccount");
```

> **Note:** When you want to extract a single account instance, for example as a data transfer object (DTO), make sure to create a `NEP6Account` instance and use this instance. To do so, you will need to encrypt the account first and then call `toNEP6Account`. This conversion is automatically performed for each account when creating a `NEP6Wallet` from a wallet instance.

## Multi-sig Accounts

Multi-signature accounts can be created with the following methods:

- `fromVerificationScript(VerificationScript script)`
- `fromNEP6Account(NEP6Account nep6Account)`
- `createMultiSigAccount(List<ECPublicKey> publicKeys, int signingThreshold)`
- `createMultiSigAccount(String address, int signingThreshold, int numberOfParticipants)`

The first two methods can be used for single- and multi-sig accounts. The verification script, available in the NEP6Account's script, contains all information needed about the multi-sig account. This includes the signing threshold, the number of participants, and the involved public keys.

In the latter two methods, no verification script is available. Therefore, one or both of the signing threshold and number of participants must be specified explicitly so that the multi-sig account can be used as a signer in transactions. The `TransactionBuilder` requires the signing threshold and number of participants to determine the network fee that must be paid for signing with the account.

Multi-sig accounts do not hold EC key pairs. That would defeat the purpose of multi-sigaccounts, because
their key material should be spread over multiple entities.

In the following example a new mutli-sig account is created from three public keys. Its signing threshold is two, i.e.,
for a transaction signed by this account to be successful, at least two of the three participants have to sign.

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
Map<Hash160, BigInteger> nep17Balances = a.getNep17Balances(neow3j);
```

This returns a map containing all NEP-17 token balances of the account.
