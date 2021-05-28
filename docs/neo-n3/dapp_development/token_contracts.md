# Token Contracts

There are two token standards on Neo, one for fungible and one for non-fungible tokens. The neow3j SDK covers the
interaction with such token contracts in the classes `FungibleToken` and `NonFungibleToken`.

The following graph shows how the classes around token contracts are structured. They are sub-types of the
`SmartContract` base class and add all token contract specific methods ontop.
In the subtype `Token` the common methods used in token contracts like `getDecimals()`, `getSymbol()` or
`getTotalSupply()` are collected.  In the subtype `FungibleToken` the NEP-17 standard is implemented. Two specific
instances of NEP-17 contracts are the native contracts `NeoToken` and `GasToken`. They contain their additional methods
that are specific to them, e.g., `registerValidator`, `getRegisteredValidators` or `vote`.  The `NonFungibleToken`
represents a wrapper for token contracts that comply with the NEP-11 standard for non-fungible tokens. The
`NeoNameService` contract is a specific example for such contracts.

```
             Build invocation script                 ->      Specify Tx Signers, etc.     ->   Tx ready to sign and send
                 ---------------
                | SmartContract |
                 ---------------
                        |
                     -------
                    | Token |
                     -------                                  --------------------                -------------
                    /        \                       ->      | TransactionBuilder |      ->      | Transaction |
        ---------------     ------------------                --------------------                -------------
       | FungibleToken |   | NonFungibleToken |
        ---------------     ------------------
          /         \                 \
   ----------      ----------         ----------------
  | NeoToken |    | GasToken |       | NeoNameService |
   ----------      ----------         ----------------
```

All of these classes provide methods to build scripts and a transactions for invoking the repspective contract. The
methods return a `TransactionBuilder` that can be used to further configure the transaction, e.g., specify signers or an
additional network fee. The transaction can then be signed and built, and the returned `Transaction` sent to a Neo node.
Or an unsigned `Transaction` can be created for later signing.


## Fungible Token Contracts (NEP-17)

### Transfer from the default Account

In the simplest scenario, you have a `Wallet` that contains an `Account` with a public and a private key. E.g. a newly
created one, as in the example below, or one from a wallet that you loaded from a NEP-6 file. If you instantiated
the wallet from a NEP-6 file, make sure that the account you want to use has been decrypted either with
`Account.decryptPrivateKey(...)` or `Wallet.decryptAllAccounts(...)`. This is necessary for automatically creating the
transaction signature.

```java
Account account1 = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
Wallet wallet = Wallet.withAccounts(account1);

Hash160 to = Hash160.fromAddress("NWcx4EfYdfqn5jNjDz8AHE6hWtWdUGDdmy");
BigDecimal amount = new BigDecimal("15");

NeoSendRawTransaction response = NeoToken(neow3j)
        .transferFromDefaultAccount(wallet, to, amount)
        .signers(Signer.calledByEntry(account1.getScriptHash()))
        .wallet(wallet)
        .additionalNetworkFee(1L)
        .sign()
        .send();
```

In this example, the method `transfer()` builds an invocation script and returns a `TransactionBuilder`. There, the wallet, signers
and an additional network fee are specified. With the method `sign()` the transaction is build, signed and a `Transaction` is returned
that is then sent to the Neo network.

### Transfer using all Accounts in the Wallet

In the previous example, the wallet only holds one account. However, if your wallet contains
multiple accounts, and your default account may not hold enough token for a transfer you want to
make, you can use the method `transfer()` which uses your whole wallet to cover a tranfer if
necessary. That means, that if the default account in the wallet cannot cover the specified amount,
the other accounts in the wallet are used to cover this amount. The method adds the necessary
signers and the wallet to the `TransactionBuilder`, so it is not necessary to configure that
manually.

```java
Account account2 = Account.fromWIF("KwjpUzqHThukHZqw5zu4QLGJXessUxwcG3GinhJeBmqj4uKM4K5z");
Account account3 = Account.fromWIF("KyHFg26DHTUWZtmUVTRqDHg8uVvZi9dr5zV3tQ22JZUjvWVCFvtw");
wallet.addAccounts(account2, account3);

NeoSendRawTransaction response = NeoToken(neow3j)
        .transfer(wallet, to, amount)
        .additionalNetworkFee(1L)
        .sign()
        .send();
```

In this example, if the balance of account1 is not high enough to cover the amount, account2 or account3 is used to cover the remaining
amount (the order is not guaranteed!), then if the amount is still not covered, the remaining account is used. E.g. you want to send 15 Neo
and account1, 2 and 3 all hold 10 Neo each. The above sent transaction would contain a transfer of 10 Neo from account1 and a transfer of
5 Neo from account2 or account3.

> **Note:** In the method `transfer()` the full wallet can be used. The order of the used accounts is not explicitly defined.
> If you want to force a specific order, see the next section.

### Transfer from specific Accounts

If you don't want to use your full wallet or want to use your accounts in a specific order to cover an amount of a
transfer, you can use the method `transferFromSpecificAccounts()`. This method allows you to pass only the accounts that
should be used in the given order. The first passed account is used first, if it cannot cover the amount, the second
account is used for the remaining amount, et cetera.  The method adds the necessary signers and the wallet to the
`TransactionBuilder`, so it is not necessary to configure that manually.

```java
NeoSendRawTransaction response = NeoToken(neow3j)
        .transferFromSpecificAccounts(wallet, to, amount, account3.getScriptHash(), account2.getScriptHash())
        .signers(Signer.calledByEntry(account3.getScriptHash()), Signer.calledByEntry(account2.getScriptHash()))
        .wallet(wallet)
        .additionalNetworkFee(1L)
        .sign()
        .send();
```

In this example, if the balance of account3 is not high enough to cover the amount, account2 is used to cover the remaining amount.
E.g. you want to send 15 Neo and account3 and account2 both hold 10 Neo each. The above sent transaction would contain a transfer
of 10 Neo from account3 and a transfer of 5 Neo from account2.

### Transfer from a multi-sig Account

Multi-sig accounts are usually not controlled by one single entity. Meaning the private keys of the involved key pairs
are not all available to sign a transaction locally. So this scenario is different in the signing step.

The account used in the transfer must be constructed from the public keys involved in the multi-sig account
(see Section [Wallets and Accounts](dapp_development/wallets_and_accounts.md#creating-an-account)).

Then create the transaction by using the method `getUnsignedTransaction()` in the `TransactionBuilder` and follow the steps
in [this Section](dapp_development/contract_invocation.md#signing-a-transaction-with-a-multi-sig-account) to sign the transaction.


## Non-fungible Token Contracts (NEP-11)

> **Note:** This section is currently out of date with the current version of neow3j.

neow3j provides a wrapper class `NonFungibleToken` that can interact with token contracts that support the
[NEP-11](https://github.com/neo-project/proposals/pull/41) standard.

For the following methods of the wrapper, it is important to understand that the standard supports non-divisible as well
as divisible non-fungible tokens. This means that for divisible tokens, a token can have multiple owners. Each owner
owns a fraction of that token and each of these fractions are owned completely by that owner. You can use the method
`balanceOf()` to get the owned fraction of a token. The method `ownersOf()` and `tokensOf()` return an enumerator of all
the owners of a token and an enumerator of all tokens that an account holds (non-divisible or fractions).

There are two transfer methods, one for non-divisible and one for divisible tokens - `transfer()` and
`transferFraction()`. E.g. to transfer `0.2` fractions of a divisible token with `tokenID` `1` and 2 decimals, you can
use the following code example that produces the transfer script and creates a `TransactionBuilder` that then can be
signed and sent:

```java
NFToken nft = new NFToken("ebc856327332bcffb7587a28ef8d144df6be8537", neow3j);
Hash160 to = new Hash160("6367377798df20bb04737d34fa2dda19a283dbb5");
TransactionBuilder builder = nft.transferFraction(wallet, account1.getScriptHash(), to, new byte[]{1});
```

Further, you can get the properties of each token with the method `properties()`.

```java
NFTokenProperties properties = nft.properties(new byte[]{1});
String name = properties.getName();
String description = properties.getDescription();
String image = properties.getImage();
String tokenURI = properties.getTokenURI();
```