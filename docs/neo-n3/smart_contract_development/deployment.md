# Deployment

You can deploy smart contracts with different tools from the Neo ecosystem, for example, with the Neo Blockchain
Toolkit. Here, we show how to deploy with neow3j. In our smart contract projects, we like to use a separate source set
to hold deployment related code. Adding such a source set is described in the
[Setup](neo-n3/smart_contract_development/setup_and_compilation.md) guide.

The [boilerplate](https://github.com/neow3j/neow3j-boilerplate) repository shows how this could look like.

The central class for deployment is `ContractManagement` that was introduced
[here](neo-n3/dapp_development/smart_contracts.md#ContractManagement) and is part of neow3j SDK. Thus, you need the
module `io.neow3j:contract`. See the [Quickstart](README.md#quickstart) section on how to import it. 
Before you calling the `ContractManagement` contract, we need to compile our contract code. 

```java
CompilationUnit res = new Compiler().compile(HelloWorldSmartContract.class.getCanonicalName(), substitutions);
```

Then we can pass the resulting NEF and manifest to `ContractManagement` by calling its `deploy` method. 

```java
AccountSigner signer = AccountSigner.none(deploymentAccount);
Hash160 owner = deploymentAccount.getScriptHash();
TransactionBuilder builder = new ContractManagement(neow3j)
        .deploy(res.getNefFile(), res.getManifest(), hash160(owner))
        .signers(signer);
Hash256 txHash = builder.sign().send().getSendRawTransaction().getHash();
Await.waitUntilTransactionIsExecuted(txHash, neow3j);
```

Note that we're also passing `hash160(owner)` to the `ContractManagement`'s `deploy` method as a parameter. That
parameter will be passed to the `_deploy` method of your contract, so you can use it to configure your contract at time
of deployment. The `ContractManagement` contract will deploy your contract and then call the `_deploy` method on it.
You can add this method to your contract by annotating a method with `@OnDeployment` (see
[here](neo-n3/smart_contract_development/devpack.md#_deploy)). Here's an example for such a method.

```java
@OnDeployment
public static void deploy(Object data, boolean update) {
        if (!update) {
                Storage.put(ctx, OWNER_KEY, (Hash160) data);
        }
}
```

Above we passed `hash160(owner)` as a deploy parameter, but, if you want to pass multiple parameters use
`ContractParameter.array(...)` and add all parameters to it.

Finally, To get the newly deployed contract's hash you can execute the code below. The hash depends on the sender of the
deploy transaction, the NEF checksum and the contract's name. The hash does not change even if you update the contract
in the future.

```java
Hash160 contractHash = SmartContract.getContractHash(
        a.getScriptHash(), 
        nefFile.getCheckSumAsInteger(), 
        manifest.getName());
```