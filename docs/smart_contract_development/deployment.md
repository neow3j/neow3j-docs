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

File nefFile = new File("/path/to/ExampleContract.nef");
File manifestFile = new File("/path/to/ExampleContract.manifest.json");
SmartContract sc = new SmartContract(nefFile, manifestFile, neow);

Account a = Account.fromWIF( "L3kCZj6QbFPwbsVhxnB8nUERDy4mhCSrWJew4u5Qh5QmGMfnCTda")
Wallet w = Wallet.withAccounts(a);
NeoSendRawTransaction response = sc.deploy()
        .wallet(w)
        .signers(Signer.calledByEntry(a.getScriptHash()))
        .sign()
        .send();
```

Depending on the network you are connecting to, you need to make sure that you use the same magic
number in neow3j as the network is using. You can set the magic number globally on the `NeoConfig`
object.

```java
NeoConfig.setMagicNumber(new byte[]{0x01, 0x03, 0x00, 0x0}); // Magic number 769, little-endian
```
