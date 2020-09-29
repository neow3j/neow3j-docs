# Deployment

## Using a Neo Node

If you have control over a Neo node, you can use that node directly to deploy smart contracts. The
information on how to do so, can be found in the [Neo
documentation](https://docs.neo.org/v3/docs/en-us/sc/deploy/deploy.html).  
But to get the NEF and manifest produced by the neow3j compiler onto the node, they must first be
written to a file. The following snippet shows how this could be done.

```java
CompilationResult res = new Compiler().compileClass("fully.qualified.name.SmartContract");
String nefFilePath = "/path/to/nef/file/test.nef";
String manifestFilePath = "/path/to/manifest/file/test.manifest.json";
try (FileOutputStream s = new FileOutputStream(nefFilePath)) {
    s.write(res.getNef().toArray());
}
try (FileOutputStream s = new FileOutputStream(manifestFilePath)) {
    ObjectMapperFactory.getObjectMapper().writeValue(s, res.getManifest());
}
```

## Using neow3j

Neow3j offers a `SmartContract` class that can be instantiated with a NEF and manifest file for
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

File nefFile = new File("/path/to/ExampleContract.nef");
File manifestFile = new File("/path/to/ExampleContract.manifest.json");
SmartContract sc = new SmartContract(nefFile, manifestFile, neow);

Account a = Account.fromWIF( "L1WMhxazScMhUrdv34JqQb1HFSQmWeN2Kpc1R9JGKwL7CDNP21uR")
Wallet w = Wallet.withAccounts(a);
NeoSendRawTransaction response = sc.deploy()
        .withWallet(w)
        .withSigner(Signer.calledByEntry(a.getScriptHash()))
        .build()
        .sign()
        .send();
```

Depending on the network you are connecting to, you need to make sure that you use the same magic
number in neow3j as the network is using. You can set the magic number globally on the `NeoConfig`
object.

```java
NeoConfig.setMagicNumber(new byte[]{0x01, 0x03, 0x00, 0x0}); // Magic number 769, little-endian
```
