# NEP-5 Token Contracts

Neow3j provides a separate abstraction for interaction with NEP-5 token contracts. The `Nep5` class handles the
functions that are provided in a NEP-5 token contract. All token information (total supply, token name, decimals and
symbol), as well as the balance of an address, can be retrieved via RPC. The `Nep5` class can further execute transfers,
whereas the deployment of the contract needs to be executed directly with the `ContractInvocation` class.

## Deploying the Token

Once a new token contract has been deployed on the blockchain, you will probably want to call the `deploy` method on it
to initialize the token. This can be done as follows.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account acct = Account.create();
ScriptHash scriptHash = new ScriptHash("3cd311b1c6cd8963e55cd8285740977c722b7b61");

List<ContractParameter> params = Arrays.asList(
        ContractParameter.string("deploy"),
        ContractParameter.array());

ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(scriptHash)
        .parameters(params)
        .account(acct)
        .build()
        .sign()
        .invoke();
```

Note that even though the `deploy` method does not take any extra arguments, it is necessary to pass the empty array
parameter to the invocation.

The account that is used to call the `deploy` method, needs to have the authority to do so. In the example above we
assume that any account can deploy the token.

## Getting Token Information

To fetch information about a token, you can use the provided methods in the `Nep5` class. The naming of these methods
is identical to the naming in the NEP-5 standard (name, totalSupply, symbol, decimals, balanceOf).

For example, if you need to know the number of decimals that the token has, you can do the following.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
ScriptHash contractScriptHash = new ScriptHash("3cd311b1c6cd8963e55cd8285740977c722b7b61");

Nep5 nep5 = new nep5.Builder(neow3j)
        .fromContract(contractScriptHash)
        .build();

BigInteger decimals = nep5.decimals();
System.out.println("Decimals: " + decimals);
```

Or if you want to know the token balance of some address.

```java
ScriptHash accountScriptHash = new ScriptHash("ATrzHaicmhRj15C3Vv6e6gLfLqhSD2PtTr");

BigInteger balance = nep5.balanceOf(accountScriptHash);
System.out.println("Balance: " + balance);
```

Note that the methods `name` and `symbol` return a String, whereas the methods `totalSupply`, `decimals` and
`balanceOf` return a BigInteger.

## Transferring Tokens

To transfer tokens from one account to another, you can use the `transfer` method in the `Nep5` class.
The following example shows how to setup a transfer of tokens.

Let us assume that you are holding the private key of the `fromAccount` and want to transfer 10 tokens to the address
`AThahtT85bNZj9VKPVBet7SF1aXJ7jw3RY`.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account from = Account.fromWIF("KxDgvEKzgSBPPfuVfw67oPQBSjidEiqTHURKSDL1R7yGaGYAeYnr");
ScriptHash to = ScriptHash.fromAddress("AThahtT85bNZj9VKPVBet7SF1aXJ7jw3RY");

Nep5 nep5 = new nep5.Builder(neow3j)
        .fromContract(contractScriptHash)
        .build();

nep5.transfer(from, to, "10");
```

The transfer method can take the amount to be transferred as a BigInteger, as well as a String for better
developer experience.

Further, note that in a correctly implemented token contract the account used to initiate the transfer must be the owner
of the transferred tokens.
