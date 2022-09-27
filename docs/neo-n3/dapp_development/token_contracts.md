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
`NeoNameService` contract is a specific example for such a contract.

```
             Build invocation script                 ->      Specify Tx Signers, etc.    ->   Tx ready to sign and send
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

All of these classes provide methods to build scripts and a transactions for contract invocation. The methods that
change state return a `TransactionBuilder` that can be used to further configure the transaction, e.g., specify signers
or an additional network fee. The transaction can then be signed and built, and the returned `Transaction` sent to a Neo
node.

## Fungible Token Contracts (NEP-17)

The most prominent method on [NEP-17](https://github.com/neo-project/proposals/blob/master/nep-17.mediawiki) token
contracts is the `transfer` method. There are a couple of overloads for this method.  In its basic form `transfer` takes
an `Account` as the for the sender. That account is added as a signer to the transaction builder with witness scope
`calledByEntry` . If the account contains a private key, you can use `sign()` to automatically sign the resulting
transaction.

```java
Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
Hash160 to = Hash160.fromAddress("NWcx4EfYdfqn5jNjDz8AHE6hWtWdUGDdmy");

NeoSendRawTransaction response = new NeoToken(neow3j)
        .transfer(account, to, new BigInteger("15"))
        .sign()
        .send();
```

Another `transfer` method allows you to pass a `Hash160` instead of `Account` for the sender. In this version, no signer
is attached and it is up to you to add the correct signer. This is useful when the sender is a contract as in the
example below.

```java
Hash160 contractHash = new Hash160("0xacce6fd80d44e1796aa0c2c625e9e4e0ce39efc0")
Hash160 to = Hash160.fromAddress("NWcx4EfYdfqn5jNjDz8AHE6hWtWdUGDdmy");
// Owner of the contract. Required in for verifying the withdraw from the contract.
Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");

NeoSendRawTransaction response = new NeoToken(neow3j)
        .transfer(contractHash, to, new BigInteger("15"))
        .signers(AccountSigner.calledByEntry(account),
                ContractSigner.calledByEntry(contractHash))
        .sign()
        .send();
```

In both variations of the `transfer` method, there is another overload that allows you to pass a contract parameter.
That parameter is forwarded to the `onNep17Payment` method if the receiver is a smart contract and has implemented that
method. If you want to pass multiple parameters you need to do it inside a `ContractParameter.array(...)`.

## Non-fungible Token Contracts (NEP-11)

The wrapper class `NonFungibleToken` provides support for the
[NEP-11](https://github.com/neo-project/proposals/blob/master/nep-11.mediawiki) standard. Take a look at the standard
for detailed information about non-fungible tokens on the Neo blockchain.

It is important to understand that the standard supports non-divisible as well as divisible non-fungible tokens.
For divisible tokens, a token can have multiple owners. Each owner owns a fraction of that token.
The provided wrapper class supports both divisible and non-divisible tokens, while methods that are only intended
for divisible tokens are not allowed to be used for non-divisible tokens, vice-versa.

Both divisible and non-divisible token contracts implement the method `balanceOf(owner)`. It simply returns the number
of tokens owned by the given address. On divisible tokens this includes fractional amounts of tokens. To get the
fractional amount owned of a specific token, you can use the method `balanceOf(owner, tokenId)` that is specifically
implemented by divisible tokens.

Further, NFTs provide the methods `tokens`, `tokensOf` and `ownerOf`. The method `ownerOf` returns the single owner of a
non-divisible token. With divisible tokens, this method returns an iterator with all owners that own a share of this token.

Then, there are multiple overloads for the `transfer` methods, some for non-divisible and some for divisible tokens that
are both. For divisible tokens the method has an additional parameter for the fractional token amount to transfer. 
Similar to the overloads on the `FungibleToken` class (see previous section), `NonFungibleToken` also provides further
overloads for the sender address to be either an `Account` or `Hash160`, and for adding a contract parameter that is
passed to the `onNep11Payment` method if the receiver is a smart contract.

Here's an example for sending 200 fractions of the token with ID 1. The fractional amount has to be in accordance with
the number of decimals the token can have. You can fetch the number of decimals via the `getDecimals()` method or
calculate the amount via the `toFractions(...)` method.

```java
Account account = Account.fromWIF("L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda");
Hash160 to = Hash160.fromAddress("NWcx4EfYdfqn5jNjDz8AHE6hWtWdUGDdmy");

NonFungibleToken nft = new NonFungibleToken(new Hash160("ebc856327332bcffb7587a28ef8d144df6be8537"), neow3j);
TransactionBuilder builder = nft.transfer(account, to, new BigInteger("200"), new byte[]{1});
```

The NEP-11 has an optional method `properties` that returns the properties of a token. You can get a tokens properties as shown below.

```java
NFTokenState properties = nft.properties(new byte[]{1});
String name = properties.getName();
String description = properties.getDescription();
String image = properties.getImage();
String tokenURI = properties.getTokenURI();
```
