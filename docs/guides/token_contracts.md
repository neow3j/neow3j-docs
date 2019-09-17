# NEP-5 Token Contracts

Currently, neow3j does not provide a separate abstraction for interaction with NEP-5 token contracts. We can use the RPC method `invoke` and the `ContractInvocation` class instead. All token information (e.g. total supply, token name, and decimals) can be retrieved via RPC, and the write-actions (deploy and transfer) can be executed with `ContractInvocation`.


## Deploying the Token

Once a new token contract has been deployed on the blockchain you will probably want to call the `deploy` method on it to initialize the token. This can be done as follows.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account acct = Account.createAccount();
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

Note that even though the `deploy` method does not take any extra arguments, it is necessary to pass the empty array parameter to the invocation.

The account that is used to call the `deploy` method, needs to have the authority to do so. In the example above we assume that any account can deploy the token.


## Getting Token Information

To fetch information about a token, it is not necessary to execute a contract invocation with `ContractInvocation`. It is enough, and more concise, to use the `invoke` RPC method. 

For example, if you want to know the token balance of some address you can do the following.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
ScriptHash scriptHash = new ScriptHash("3cd311b1c6cd8963e55cd8285740977c722b7b61");
List<ContractParameter> params = Arrays.asList(
        ContractParameter.string("balanceOf"),
        ContractParameter.array(ContractParameter.string("ATrzHaicmhRj15C3Vv6e6gLfLqhSD2PtTr")));

InvocationResult result = neow3j.invoke(scriptHash.toString(), params)
        .send().getInvocationResult();
BigInteger balance = result.getStack().get(0).asByteArray().getAsNumber();
System.out.println("Balance: " + balance);
```

Or if you need to know the number of decimals that the token has.

```java
params = Arrays.asList(
        ContractParameter.string("decimals"),
        ContractParameter.array());

result = neow3j.invoke(scriptHash.toString(), params)
        .send().getInvocationResult();
BigInteger decimals = result.getStack().get(0).asInteger().getValue();
System.out.println("Decimals: " + decimals);
```

Note that the resulting value is not always of the same type. The balance was returned as a byte array and the number of decimals as an integer. You need to know the return type to be able to correctly interpret it. You could also check the `StackItem` for its type with `StackItem.getType()` and then cast it to the specific stack item type (see the cast methods on the `StackItem` class). 


## Transferring Tokens

To transfer tokens from one account to another you will again need to use the `ContractInvocation` class. The following example shows the necessary setup.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://seed7.ngd.network:10332"));
Account acct = Account.createAccount();
ScriptHash scriptHash = new ScriptHash("3cd311b1c6cd8963e55cd8285740977c722b7b61");
List<ContractParameter> params = Arrays.asList(
        ContractParameter.string("transfer"),
        ContractParameter.array(
                ContractParameter.string(acct.getAddress()),
                ContractParameter.string("AK2nJJpJr6o664CWJKi1QRXjqeic2zRp8y"),
                ContractParameter.integer(10)));

ContractInvocation invoc = new ContractInvocation.Builder(neow3j)
        .contractScriptHash(scriptHash)
        .parameters(params)
        .account(acct)
        .build()
        .sign()
        .invoke();
```

Note that, in a correctly implemented token contract, the account used to initiate the transfer must be the owner of the transferred tokens. I.e. the from-address must be equal to the address of the account used in the transfer.

