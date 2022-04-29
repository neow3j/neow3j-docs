# Deployment

You can deploy smart contracts with different tools from the Neo ecosystem, for example, with the Neo Blockchain
Toolkit. Here, we show how to deploy with neow3j. 

The central class for deployment is `ContractManagement` that was introduced
[here](neo-n3/dapp_development/smart_contracts.md#ContractManagement) and is part of neow3j SDK. Thus, you need the
module `io.neow3j:contract`. See the [Quickstart](README.md#quickstart) section on how to import it. Below is an example
on how to use `ContractManagement` for deployment.


```java
Neow3j neow = Neow3j.build(new HttpService("http://localhost:40332"));
Account account = Account.fromWIF( "L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2")

File contractNefFile = Paths.get("build", "neow3j", "ContractName" + ".nef").toFile();
File contractManifestFile = Paths.get("build", "neow3j", "ContractName" + ".manifest.json").toFile();

NefFile nefFile = NefFile.readFromFile(contractNefFile);
ContractManifest manifest;
try (FileInputStream s = new FileInputStream(contractManifestFile)) {
    manifest = ObjectMapperFactory.getObjectMapper().readValue(s, ContractManifest.class);
}

NeoSendRawTransaction response = new ContractManagement(neow3j)
        .deploy(nefFile, manifest)
        .signers(AccountSigner.calledByEntry(account)
                .setAllowedContracts(ContractManagement.SCRIPT_HASH))
        .sign()
        .send();
```

The `Neow3j` object is needed to send the deployment transaction to a Neo node of your choice. Next, an account and
wallet with enough GAS to cover the deployment fees is setup. Then, we load the contract's NEF and manifest files from
disk, i.e., this expects that the contract was already compiled. Finally, the `deploy` method of the
`ContractManagement` contract is called.

The ContractManagement contract will deploy your contract and then call the special `_deploy` method on it, if
available. You can add this method to your contract by annotating a method with `@OnDeployment` (see
[here](neo-n3/smart_contract_development/devpack.md#_deploy)).
You can pass parameters to the `_deploy` method by passing them to the `deploy` method of the ContractManagement
contract. Here's an example.

```java
ContractParameter param = ContractParameter.hash160(alice);
NeoSendRawTransaction response = contractMgmt.deploy(nef, manifest, param)
        .signers(AccountSigner.none(account)) // only needed to pay fees
        .sign()
        .send();
```

After the contract is deployed the `param` will be passed as the `data` object to your contract's `_deploy` method, as in the example of a deploy method below.

```java
    @OnDeployment
    public static void deploy(Object data, boolean update) throws Exception {...}
```

If you want to pass multiple parameters use `ContractParameter.array(...)` and add all parameters to it.

To get the newly deployed contract's hash you can execute the code below. The hash depends on the sender of the deploy
transaction, the NEF checksum and the contract's name. The hash does not change even if you update the contract in the
future.

```java
Hash160 contractHash = SmartContract.getContractHash(
        a.getScriptHash(), 
        nefFile.getCheckSumAsInteger(), 
        manifest.getName());
```
