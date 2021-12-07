# Testing

Thoroughly testing a smart contract requires deploying it on a running Neo instance and invoking its methods possibly
with a specific chain setup. Neow3j offers a test library that allows convenient smart contract testing on top of JUnit.
This library lives in the `io.neow3j:devpack-test` module.

After setting up a smart contract project with Gradle as described in the
[Setup](neo-n3/smart_contract_development/setup_and_compilation.md) guide you are also setup for writing contract tests.

Checkout the [boilerplate repository](https://github.com/neow3j/neow3j-boilerplate) for a simple example of a contract test class.

## Test Configuration

A contract test must be annotated with `@ContractTest`. Besides adding JUnit-related functionality it also allows you to
configure the test. Here's an example of how this might look:

```java
@ContractTest(contracts = HelloWorldSmartContractTest.class, blockTime = 1)
public class HelloWorldSmartContractTest {
```

**Contracts**

Most importantly, you need to specify the contracts under test via the `contracts` property. The specified contracts will
be automatically compiled and deployed before all tests in this test class.

**Block Time**

Smart contract tests run on an underlyig Neo blockchain implementation that produces blocks in a fixed time interval. You
can change that interval with the `blockTime` property. 

**Batch File**

Some Neo blockchain implementations, like neo-express, have a **batch** functionality that allows you to execute a
series of commands that alter the blockchain state before running it. You can, thereby, initialize the blockchain to a
desired state before running the tests. Place the batch file in the *resources* directory (i.e., *src/test/resources*)
and set the name of the file in the `batchFile` property. The batch file is applied once before all tests.

Checkout the official neo-express
[documentation](https://github.com/neo-project/neo-express/blob/master/docs/command-reference.md#neoxp-batch) on how to
write a batch file for neo-express.

**Checkpoint File**

Some Neo blockchain implementations, like neo-express, have a **checkpoint** functionality that allows you to load a
blockchain state - a checkpoint - that was previously exported from another blockchain instance. You can use this
functionality in tests by applying such a checkpoint before tests are run. The blockchain will then start running
starting at the state of the checkpoint.  Place the checkpoint file in the *resources* directory (i.e.,
*src/test/resources*) and set the name of the file in the `checkpoint` property. The checkpoint file is applied once
before all tests. If a batch file is also configured, the checkpoint is applied first.

Checkout the official neo-express
[documentation](https://github.com/neo-project/neo-express/blob/master/docs/command-reference.md#neoxp-checkpoint) on
how to generate a checkpoint file for neo-express. 

**Chain Configuration**

The underlying blockchain instance can be configured via a configuration file. E.g., neo-express is configured via
a .neo-express configuration file that specify, among others, the key material for the consensus node, preset accounts
and blockchain parameters. The devpack-test library contains a default neo-express configuration file with one consensus
node and an account that controls the consensus node's private key. You can use a custom configuration by adding a
configuration file to the *resources* directory (i.e., *src/test/resources*) and set the name of the file in the
`neoxpConfig` property. The default configuration will be overwritten. As a template for the file you can use the
library's default neo-express config file
[here](https://github.com/neow3j/neow3j/blob/master-3.x/test-tools/src/main/resources/default.neo-express).

Checkout the official neo-express
[documentation](https://github.com/neo-project/neo-express/blob/master/docs/settings.md) on what settings can be used in
a neo-express config file.

## Test Extension

Next to the `@ContractTest` annotation you have to add the `ContractTestExtension` to your test class. It is a JUnit 5
test extension that hooks into the before and after stages of the test class. It configures and starts the test
blockchain, applies batche and checkpoint files, compiles and deploys your contract, and tears everything down after all
tests were run.

```java
    @RegisterExtension
    private static ContractTestExtension ext = new ContractTestExtension();
```

The `ContractTestExtension` has a second constructor that takes an instance of `TestBlockchain`. This provides
flexbility for the underlying Neo blockchain implementation used for the tests. Currently only one implementation of
`TestBlockchain` exists. It uses neo-express in a docker container. In the future other implementations could be added,
e.g., for running tests directly on the testnet but with limited capabilities compared to a neo-express instance.

The `ContractTestExtension` provides several methods to control the blockchain instance. You can halt, resume, and
fast-forward block production, create new accounts or retrieve the genesis accounts with its associated private keys.
Furthermore, it provides access to a `Neow3j` configured to do RPC method calls to the underlying blockchain and access
to the `SmartContract` objects representing the deployed contracts. You could retrieve both in a setup method before
all tests and set them as static variables on your test class.

```java
    @BeforeAll
    public static void setUp() {
        neow3j = ext.getNeow3j();
        contract = ext.getDeployedContract(HelloWorldSmartContract.class);
    }
```

## Deployment Configuration

The `devpack-test` allows you to configure the deployment of you smart contracts under test by adding a static
configuration method for each contract. The methods must be annotated with `@DeployConfig` and the annotation's value
must be set with the class of the contract this configuration is meant for. The method must have a void return type and
take at least an argument of type `DeployConfiguration` and can additionally take an argument of type `DeployContext`.
Configuration happens on the `DeployConfiguration` object. Currently, it allows you to set the deployment parameter and
mappings for placeholder string substitution (described
[here](neo-n3/smart_contract_development/devpack.md#placeholder-substitution)).

```java
    @DeployConfig(ExampleContract.class)
    public static void config2(DeployConfiguration config, DeployContext ctx) {
        SmartContract sc = ctx.getDeployedContract(AnotherContract.class);
        config.setDeployParam(ContractParameter.hash160(sc.getScriptHash()));
        config.setSubstitution("<owner_address>", "NXXazKH39yNFWWZF5MJ8tEN98VYHwzn7g3");
        config.setSubstitution("<contract_hash>", "ef4073a0f2b305a38ec4050e4d3d28bc40ea63f5");
    }

```

The example shows, that you can access other contracts under test if they are preceeding in the order of deployment.
This allows you to grab the contract hash and use it as a deployment parameter in another contract.

Note, that you can set only one deployment parameter. That is according to the standardized `_deploy` method in smart
contracts. It takes only one deployment parameter. Thus, if you want to pass multiple parameters, pack them into an
array parameter.

Also note, if you use a `setUp` method annotated with `@BeforeAll`, beware that deployment configuration methods are
invoked before the `setUp` method. Thus, things that you set up there, will not be available yet in the configuration
methods. The same is true for access to the `ContractTestExtension` object. It is not yet in a consistent state when the
configuration methods are called, i.e., don't use the `ContractTestExtension` object in the config methods.