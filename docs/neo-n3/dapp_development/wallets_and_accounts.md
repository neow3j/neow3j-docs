# Wallets and Accounts

Accounts and wallets are important concepts in blockchain. A Neo account in its basic form is made up of an EC key pair.
From its public key an address is derived, which is used to identify the account. Besides this simple form there are
multi-signature (multi-sig) accounts which are made up of multiple public keys that participate in the account. 
Neow3j uses the same `Account` class to represent both single- and multi-sig accounts.

A wallet is a collection of one or more accounts.

## Wallets

The easiest way to create a new wallet is by using one of the static creation methods.

```java
Wallet w = Wallet.create();
```

This creates a wallet with a new account (with a new key pair). There are other versions of this method that allow us to
immediately encrypt the new private key or directly write the wallet to a file after creation.

If you have a NEP-6 wallet file exported from some other wallet software, you can use
`fromNEP6Wallet(...)` which reads the wallet information from the NEP-6 file.

```java
String absoluteFileName = "/path/to/your/NEP6.wallet";
Wallet w = Wallet.fromNEP6Wallet(absoluteFileName)
        .name("NewName");
```

> **Note:** When reading the wallet from a NEP-6 wallet file, the private keys of the contained accounts will be
> encrypted until you call `decryptAllAccounts(String password)` on the wallet. An encrypted account cannot be used for
> signing transactions.

If you already have `Account` objects you can create a wallet with the method `withAccounts(...)`. Furthermore, you can
manually set a name, version and [scrypt](https://en.wikipedia.org/wiki/Scrypt) parameters for the wallet. If nothing is
set, default values are used.

```java
Wallet w = Wallet.withAccounts(Account.create())
        .name("MyWallet")
        .version("1.0");
```

> **Note:** If you want to extract a wallet instance, e.g. as a data transfer object make sure to create a
> `NEP6Wallet` instance and use this instance. To do so, you will have to encrypt the wallet first and then call
> `toNEP6Wallet`. This automatically initiates `NEP6Account` instances for all accounts in the wallet.

## Accounts

Accounts in neow3j can be created with or without EC key material. If the private key of the account is available, the
account can be used for transaction signing.
The following methods create an account with the private key available. 

- `create()`
- `new Account(ECkeyPair exKeyPair)`
- `fromNEP6Account(NEP6Account nep6Acct)` - Requires decryption of the private key with `decryptPrivateKey(...)`. 
- `fromWIF(String wif)`

The following methods create an account without a private key:
  
 - `fromAddress(String address)`
 - `fromScriptHash(Hash160 scriptHash)`
 - `fromPublicKey(ECPublicKey publicKey)`
 - `fromVerificationScript(VerificationScript script)`

The label of an account is by default set to its address. After creating an account you can set it's label if needed.

```java
Account a = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda")
        .label("MyAccount");
```

> **Note:** When you want to extract a single account instance, e.g. as a data transfer object (dto) make sure to create a
> `NEP6Account` instance and use this instance. To do so, you will have to encrypt the account first and then call
> `toNEP6Account`. This conversion is automatically made for each account when creating a `NEP6Wallet` from a wallet instance.

## Multi-sig Accounts

Multi-signature accounts can be created with the following methods:

- `fromVerificationScript(VerificationScript script)`
- `fromNEP6Account(NEP6Account nep6Acct)`
- `createMultiSigAccount(List<ECPublicKey> publicKeys, int signingThreshold)`
- `createMultiSigAccount(String address, int signingThreshold, int nrOfParticipants)`

The frist two methods can be used for single- and multi-sig accounts. The verification script - which is available in
the NEP6Account's script - holds all information needed about the multi-sig account. This includes the signing
threshold, the number of participants, and the involved public keys.  
In the latter two methods there is now verificaiton script available. Therfore, one or both of signing threshold and
number of participants has to be specified explicitely in order that the multi-sig account can be used as a signer in
transactions. The `TransactionBuilder` needs the signing threshold and number of participants to determine the network
fee that has to be paid for signing with the account.

Multi-sig accounts do not hold EC key pairs. That would defeat the purpose of multi-sig accounts, because
their key material should be spread over multiple entities.

In the following example a new mutli-sig account is created from three public keys. It's signing threshold is two, i.e.,
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
