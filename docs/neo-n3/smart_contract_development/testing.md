# Testing

Thoroughly testing a smart contract requires deploying it on a running Neo instance and invoking its methods possibly
with a specific chain setup. Neow3j offers a test library that allows convenient smart contract testing on top of JUnit.
This library lives in the `io.neow3j:devpack-test` module separate from the `devpack`.

After setting up a smart contract project with Gradle as described in the
[Setup](neo-n3/smart_conract_development/setup_and_compilation.md) guide you are ready to write contract tests.

The [boilerplate repository](https://github.com/neow3j/neow3j-boilerplate) contains an example test, which we use here to explain the basics of a smart contract test. 

## Test Configuration

A contract test must be annotated with `@ContractTest`. Besides adding some JUnit related functionality it also allows you to configure the test. Here's an example of how this might look:

```java
@ContractTest(blockTime = 1, contractClass = HelloWorldSmartContract.class)
public class HelloWorldSmartContractTest {
```

**Contract Class**

Most importantly you need to specify the contract under test via the `contractClass` property. The specified contract will be automatically compiled and deployed before all tests (similar to a method annotated with `@BeforeAll`). 

**Block Time**

Smart contract tests run on a neo-express instance that, by default, produces blocks in the interval of 1 second. You
can change that interval with the `blockTime` property. 

**Batch File**

Neo-Express has a **batch** functionality that allows you to execute a series of commands that alter the blockchain
state before running it. You can, thereby, initialize the blockchain to a desired state before running the tests. Place
the batch file in the *resources* directory (i.e., *src/test/resources*) and set the name of the file in the `batchFile`
property. The batch file is applied once before all tests.

Checkout the official neo-express
[documentation](https://github.com/neo-project/neo-express/blob/master/docs/command-reference.md#neoxp-batch) on how to
write a batch file. 

**Checkpoint File**

Neo-Express has a **checkpoint** functionality that allows you to load a blockchain state - a checkpoint - that was
previously exported from another neo-express instance. You can use this functionality in tests by applying such a
checkpoint before tests are run. The blockchain will then start running starting at the state of the checkpoint.
Place the checkpoint file in the *resources* directory (i.e., *src/test/resources*) and set the name of the file in the
`checkpoint` property. The checkpoint file is applied once before all tests. If a batch file is also configured, the
checkpoint is applied first.

Checkout the official neo-express
[documentation](https://github.com/neo-project/neo-express/blob/master/docs/command-reference.md#neoxp-checkpoint) on
how to generate a checkpoint file. 

**Neo-Express Configuration**

Neo-Express instances are configured via .neo-express configuration files that specify things like the key material for
the consensus node, pre-setup accounts and blockchain parameters. The devpack-test library contains a default
configuration with one consensus node and an account with the name "Alice" that controls the consensus node's private
key. You can use a custom configuration by adding a .neo-express file to the *resources* directory (i.e.,
*src/test/resources*) and set the name of the file in the `neoxpConfig` property. The default configuration will be overwritten.
As a template for the file you can use the library's default config file [here](<!-- TODO Insert link -->).

Checkout the official neo-express
[documentation](https://github.com/neo-project/neo-express/blob/master/docs/settings.md) on what settings can be used in
the config file.

## Test Extenstion

Next to the `@ContractTest` annotation you have to add the `ContractTestExtension` to your test class. It is a JUnit 5
test extension that hooks into the before and after stages of the test class. It starts a test container, runs the
neo-express instance, applies batches and checkpoints, compiles and deploys your contract, and tears everything down
after all tests were run.

```java
    @RegisterExtension
    private static ContractTestExtension ext = new ContractTestExtension();
```

Furthermore, it provides several methods to control the neo-express instance. You can stop and restart neo-express,
create new accounts, make assert transfers and retrieve accounts by name.  E.g., `ext.getAccount("Alice")` will check if
the account with name "Alice" exists on the neo-express instance, fetch its key material, and return an `Account` object
that can be used, e.g., for signing transactions.

## Test Parameters

When annotating your test with the `@ContractTest` annotation, you can make use of a test constructor and automatic
parameter injection. The available parameters are `Neow3jExpress`, `SmartContract` and `NeoExpressTestContainer`. Add a
constructor to your test and add does classes as parameters to make JUnit inject them into your test. You don't need to
add all of them. Add only the ones you actually need in the tests.

```java
    private Neow3jExpress neow3j;
    private SmartContract smartContract;
    private NeoExpressTestContainer container;

    public HelloWorldSmartContractTest(Neow3jExpress neow3j, SmartContract contract, NeoExpressTestContainer container) {
        this.neow3j = neow3j;
        this.smartContract = contract;
        this.container = container;
    }
```

**SmartContract**

The most important parameter is the `SmartContract` instance that will allow you to make calls to the deployed smart
contract. You can use it as described in the SDK's [section](neo-n3/dapp_development/smart_contracts.md) on smart
contracts. 
Here's an example:

```java
    @Test
    public void invokeBongoCat() throws IOException {
        NeoInvokeFunction result = smartContract.callInvokeFunction(BONGO_CAT);
        assertEquals(result.getInvocationResult().getStack().get(0).getString(), "neowwwwwwwwww");
    }
}
```

Beware that `smartContract.callInvokeFunction(...)` does not change the blockchain state while `smartContrat.invokeFunction(...)` does.

If your contract is a token contract, e.g., a non-fungible token, you could use an instance of `FungibleToken` or
`NonFungibleToken` as described [here](neo-n3/dapp_development/token_contracts.md). This gives you a bit more
convenience. Just get the script hash of the contract from the `SmartContract` object and create a token contract
instance with it.

**Neow3jExpress**

The `Neow3jExpress` parameter provides you access to all RPC methods of the neo-express node. Some functions of
`Neow3jExpress` (and its base class `Neow3j`) are explained [here](neo-n3/dapp_development/interacting_with_a_node.md).
One method that is often used in tests is `getApplicationLog`. Once you issued a transaction with
`SmartContract.invokeFunction(...)`, you can use the application logs to inspect the transactions result on the running
blockchain. Here's an example:

```java
Hash256 txHash = contract.invokeFunction("method")
        .signers(AccountSigner.calledByEntry(account))
        .sign().send().getSendRawTransaction().getHash();
Await.waitUntilTransactionIsExecuted(txHash, neow3j);
NeoApplicationLog log = neow3j.getApplicationLog(txHash).send().getApplicationLog();
assertEquals(log.getExecutions().get(0).getStack().get(0).getInteger().intValue(), 1);
```

**NeoExpressTestContainer**

The `NeoExpressTestContainer` is available for cases in which you need more fine-grained control over the test container
on which the tests are run.