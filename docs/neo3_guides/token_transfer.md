# Transferring Tokens

> **Consider that there is no testnet for Neo 3 yet!** To use neow3j versions `3.+`, you need a local node of Neo 3 running.
> You can find one [here](http://github.com/axlabs/neo3-privatenet-docker).

On the Neo blockchain the [NEP-5 Token Standard](http://github.com/neo-project/proposals/blob/master/nep-5.mediawiki)
is used for everything concerning fungible tokens. In the following the transfer interaction with a NEP-5 smart contract is illustrated.

To invoke any contract and to transfer tokens, respectively, you will need a connection to an RPC node via a `Neow3j` instance.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:40332"));
```

## Structure

In the following graph the structure concerning token transfers with neow3j is illustrated. To deploy, invoke or just
retrieve information about any contract's state on the blockchain, the class `SmartContract` can be used (read more
about this in the [next Section](neo3_guides/contract_invocation.md)).
In the subtype `Token` the common methods used in token contracts like `getDecimals()`, `getSymbol()` or `getTotalSupply()`
are collected.
In the subtype `Nep5Token` the NEP-5 standard is implemented from which the native tokens `NeoToken` and `GasToken` are
derived, that contain their individual additional methods, e.g. `registerValidator`, `getRegisteredValidators` or `vote`
(for more, see [here](https://docs.neo.org/v3/docs/en-us/reference/scapi/api/neo.html)).
The `NFToken` represents a wrapper for token contracts that comply with the currently not finalized NEP-11 standard for
non-fungible tokens.

```
             Build invocation script     ->      Specify Tx Signers, etc.     ->   Tx ready to sign and send
                ---------------
               | SmartContract |
                ---------------
                       |
                    -------
                   | Token |
                    -------                        --------------------                -------------
                   /        \             ->      | TransactionBuilder |      ->      | Transaction |
           -----------      ---------              --------------------                -------------
          | Nep5Token |    | NFToken |
           -----------      ---------
            /       \
   ----------       ----------
  | NeoToken |     | GasToken |
   ----------       ----------
```

These classes provide convenient methods to build a script and transaction for contract invocation.
The methods return a `TransactionBuilder` that can be used to further configure the transaction,
e.g., specify signers or an additional network fee. The transaction can then be signed and built,
such that the returned `Transaction` can be sent. Or an unsigned `Transaction` can be created for
later signing (e.g. when using a multi-sig account).

## NEP-5 Tokens

### Transfer from the default Account

In the simplest scenario, you have a `Wallet` that contains an `Account` with a public and a private key. E.g. a newly
created one, as in the example below, or one from a wallet that you loaded from a NEP-6 file. If you instantiated
the wallet from a NEP-6 file, make sure that the account you want to use has been decrypted either with
`Account.decryptPrivateKey(...)` or `Wallet.decryptAllAccounts(...)`. This is necessary for automatically creating the
transaction signature.

```java
Account account1 = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
Wallet wallet = Wallet.withAccounts(account1);

ScriptHash to = ScriptHash.fromAddress("NWcx4EfYdfqn5jNjDz8AHE6hWtWdUGDdmy");
BigDecimal amount = new BigDecimal("15");

NeoSendRawTransaction response = NeoToken(neow3j)
        .transferFromDefaultAccount(wallet, to, amount)
        .wallet(wallet)
        .signers(Signer.calledByEntry(account1.getScriptHash()))
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
the other accounts in the wallet can be used to cover this amount. The method adds the necessary
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

In this example, if the balance of account1 is not high enough to cover the amount, account2 is used to cover the remaining amount,
then if the amount is still not covered, account3 is used. E.g. you want to send 15 Neo and account1, 2 and 3 all hold 10 Neo each.
The above sent transaction would contain a transfer of 10 Neo from account1 and a transfer of 5 Neo from account2 or account3.

> **Note:** In the method `transfer()` the full wallet can be used. The order of the used accounts is not explicitly defined.
> If you want to force a specific order, see the next section.

### Transfer from a specific Accounts

If you don't want to use your full wallet or want to use your accounts in a specific order to cover an amount of a transfer,
you can use the method `transferFromSpecificAccounts()`. This method allows you to pass only the accounts that should
be used in the given order. The first passed account is used first, if it cannot cover the amount, the second account
is used for the remaining amount, et cetera.
The method adds the necessary signers and the wallet to the `TransactionBuilder`, so it is not
necessary to configure that manually.

```java
NeoSendRawTransaction response = NeoToken(neow3j)
        .transferFromSpecificAccounts(wallet, to, amount, account3.getScriptHash(), account2.getScriptHash())
        .wallet(wallet)
        .signers(Signer.calledByEntry(account3.getScriptHash()), Signer.calledByEntry(account2.getScriptHash()))
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
(see Section [Wallets and Accounts](neo3_guides/wallets_and_accounts.md#creating-an-account)).

Then create the transaction by using the method `getUnsignedTransaction()` in the `TransactionBuilder` and follow the steps
in [this Section](neo3_guides/contract_invocation.md#signing-a-transaction-with-a-multi-sig-account) to sign the transaction.

## Non-fungible Tokens (NEP-11)

neow3j provides a wrapper class `NFToken` that can interact with token contracts that support the [NEP-11](https://github.com/neo-project/proposals/pull/41) standard.

> The NEP-11 standard is not yet final, so there might still be changes to it.

As shown above in the structure, the `NFToken` is a subtype of the class `Token`, so it utilizes the methods `getName()`, `getSymbol()`, `getTotalSupply()` and `getDecimals()`.

For the following methods of the wrapper, it is important to understand that the standard supports non-divisible as well as divisible non-fungible tokens. This means that for divisible tokens, a token can have multiple owners. Each owner owns a fraction of that token and each of these fractions are owned completely by that owner. You can use the method `balanceOf()` to get the owned fraction of a token. The method `ownersOf()` and `tokensOf()` return an enumerator of all the owners of a token and an enumerator of all tokens that an account holds (non-divisible or fractions).

There are two transfer methods, one for non-divisible and one for divisible tokens - `transfer()` and `transferFraction()`. E.g. to transfer `0.2` fractions of a divisible token with `tokenID` `1` and 2 decimals, you can use the following code example that produces the transfer script and creates a `TransactionBuilder` that then can be signed and sent:

```java
NFToken nft = new NFToken("ebc856327332bcffb7587a28ef8d144df6be8537", neow3j);
ScriptHash to = new ScriptHash("6367377798df20bb04737d34fa2dda19a283dbb5");
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
