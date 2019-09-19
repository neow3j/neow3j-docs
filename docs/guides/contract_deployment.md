# Deploying Smart Contracts

Smart contract deployments are handled by the `ContractDeployment` class. Its builder makes it easy to configure all attributes needed for deployment.

A contract deployment costs GAS (see NEO [documentation](https://docs.neo.org/docs/en-us/sc/fees.html) on fees). The basic system fee for a deployment is 100 GAS. If the contract requires storage and dynamic calls then 400 GAS and 500 GAS are required respectively. Therefore, make sure that the account you use in the deployment has the appropriate UTXOs to cover the system fee. The system fee is calculated and added to the deployment transaction automatically by neow3j. The only fee that you can attach manually is a network fee, which could become necessary depending on the size of the deployment transaction.

Neow3j does not include a smart contract compiler. You have to provide the compiled AVM file.

To begin with we need a connection to an RPC node and the account that will sign the deployment and pay for the system fee. In this example, the account is created from its WIF and then its asset balances are updated so that neow3j can correctly determine the inputs from the account's UTXOs.

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:30333"));
Account acct = Account.fromWIF("KxDgvEKzgSBPPfuVfw67oPQBSjidEiqTHURKSDL1R7yGaGYAeYnr").build();
acct.updateAssetBalances(neow3j);

new ContractDeployment.Builder(neow3j)
      .account(acct)
```

Then, we'll configure which AVM file needs to be loaded and which of the following properties the contract needs to have:
- if it should be possible to make payments to the contract
- if the contract requires storage
- if the contract requires dynamic invokes

These are all separate configuration options on the builder. Mentioning them in the configuration sets them to true and not mentioning them sets them to false. In the example they are all set.

```java
      .loadAVMFile(this.getClass().getResource("/contracts/ns.avm").getFile())
      .isPayable()
      .needsStorage()
      .needsDynamicInvoke()   
```

Next up are the input parameters and return type of the contract's entry point (main method). If the contract adheres to the NEP-3 standard, the parameters are a string and an array parameter, and the return type is a byte array.

```java
      .parameters(ContractParameterType.STRING, ContractParameterType.ARRAY)
      .returnType(ContractParameterType.BYTE_ARRAY)
```

What remains are the solely descriptive properties of the contract.

```java
      .name("name service")
      .version("1")
      .author("author name")
      .email("email@address.com")
      .description("description")
```

The whole deployment code looks as follows. 

```java
Neow3j neow3j = Neow3j.build(new HttpService("http://localhost:30333"));
Account acct = Account.fromWIF("KxDgvEKzgSBPPfuVfw67oPQBSjidEiqTHURKSDL1R7yGaGYAeYnr").build();
acct.updateAssetBalances(neow3j);

Contract contract = new ContractDeployment.Builder(neow3j)
      .account(acct)
      .loadAVMFile(this.getClass().getResource("/contracts/ns.avm").getFile())
      .needsStorage()
      .needsDynamicInvoke()
      .isPayable()
      .parameters(ContractParameterType.STRING, ContractParameterType.ARRAY)
      .returnType(ContractParameterType.BYTE_ARRAY)
      .name("ico")
      .version("1")
      .author("author name")
      .email("email@address.com")
      .description("description")
      .build()
      .sign()
      .deploy();
```

Depending on the size of your AVM file, the deployment transaction will have a size that requires a network fee on top of the system fee (See this NEO [blog post](https://neo.org/blog/details/4148) describing the network fee structure). In that case, the deployment will fail with an `IllegalStateException` and log the necessary network fee. This check is made already when building the `ContractDeployment` object and not only when calling `deploy()`. It is advisable to add a bit more than the exact stated amount because the transaction size will again change when adding the network fee to it.

You could use the `Contract` object resulting from the deployment to do invocations right away. Though, it will take some time until the deployment is propagated through the blockchain network fully. For usage with `ContractInvocation`, grab the script hash of the contract with `Contract.getContractScriptHash()`.