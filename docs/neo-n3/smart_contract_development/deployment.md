# Deployment

## Using neow3j

The neow3j SDK offers a `SmartContract` class that can be instantiated with a NEF and manifest file for
deployment purposes. To use this feature add the `io.neow3j:contract` package to your project as
described in the [Gettig Started](overview/getting_started.md?id=sdk) section.

The following snippet shows how the `SmartContract` class can be used for deployment.  
The `Neow3j` object is needed to send the deployment transaction to a Neo node of your choice.
An account with enough GAS to cover the deployment fees is required. In the example such an account
is recovered from a WIF. The `Wallet` created from the account is handed to the deployment builder.
Without any other configurations the added account is automatically used to cover the fees and sign
the transaction.

```java
Neow3j neow = Neow3j.build(new HttpService("http://localhost:40332"));
Account a = Account.fromWIF( "L24Qst64zASL2aLEKdJtRLnbnTbqpcRNWkWJ3yhDh2CLUtLdwYK2")
Wallet w = Wallet.withAccounts(a);

File contractNefFile = Paths.get("build", "neow3j", "ContractName" + ".nef").toFile();
File contractManifestFile = Paths.get("build", "neow3j", "ContractName" + ".manifest.json").toFile();

NefFile nefFile = NefFile.readFromFile(contractNefFile);
ContractManifest manifest;
try (FileInputStream s = new FileInputStream(contractManifestFile)) {
    manifest = ObjectMapperFactory.getObjectMapper().readValue(s, ContractManifest.class);
}

NeoSendRawTransaction response = new ContractManagement(neow)
        .deploy(nefFile, manifest)
        .signers(Signer.global(a.getScriptHash()))
        .wallet(w)
        .sign()
        .send();
```

To get the newly deployed contract's hash you can do the following:

```
Hash160 contractHash = SmartContract.getContractHash(
        a.getScriptHash(), 
        nefFile.getCheckSumAsInteger(), 
        manifest.getName());
```

As you can see, the hash depends on the sender of the deploy transaction, the NEF checksum and the contract's name. 