# dApp Development

In our definition, dApp development includes the creation of systems that interact and are based on a blockchain. Although smart contracts are also a component of a dApp, we distinguish between contract development and dApp development because neow3j's libraries cleanly separate these two activities. In this section, we introduce the neow3j SDK, which focuses on dApp development.

The neow3j SDK aims to offer a high-level abstraction layer that frees developers from handling the technical intricacies of the Neo blockchain. Simultaneously, it provides the capability for more detailed configuration in scenarios requiring greater control.

For instance, you can utilize the `NeoToken` class to easily create a NEO transfer with a single method call, or build a transaction from the ground up using the `TransactionBuilder`.

The neow3j SDK is divided into two modules: `io.neow3j:core` and `io.neow3j:contract`. These modules separate lower-level core functionalities, such as signing, from more abstract functionalities, such as building contract-specific transactions. The `core` module is a dependency of the `contract` module. Therefore, if you add `contract` as a dependency to your project, you will automatically include the `core` module as well.

**`io.neow3j:core`** contains:

- **Basic definitions and enums**: This includes foundational definitions and enumerations used within the neow3j SDK to establish core concepts and data structures.

- **Utility methods**: Notable classes here handle tasks such as manipulating hex strings efficiently.

- **Cryptographic methods**: This involves functionalities related to key pairs (`ECKeyPair` class) and cryptographic signatures (`Sign` class).

- **Wallet and account management**: The SDK facilitates tasks like managing key pairs, reading from, and writing to NEP-6 wallet files.

- **Interaction with the Neo blockchain**: This component enables communication with a Neo node via JSON-RPC, facilitating interaction and data exchange with the blockchain.

- **Transaction and ScriptBuilder classes**: These classes are essential for constructing various types of contract invocations, forming the core functionality for interacting with and executing actions on the Neo blockchain.

**`io.neow3j:contract`** contains:

- **Functionality related to smart contracts**: This includes features and utilities specifically designed for working with smart contracts, such as deployment, invocation, and state management.

- **Classes for easy interaction with native contracts**: These classes provide streamlined methods for interacting with built-in or native contracts on the Neo blockchain.

- **Classes for standard contract interaction**: This encompasses classes tailored for interacting with contracts that adhere to specific standards, such as the NEP-17 token standard. These classes simplify operations like token transfers and balance inquiries.

- **Interfaces for token contracts**: The SDK likely includes interfaces for fungible and non-fungible token contracts, enabling developers to interact with these contracts in a standardized manner. Additionally, support for building invocation scripts for transactions involving native contracts would be part of this functionality.

These sections demonstrate the neow3j SDK's capabilities by highlighting common concepts and activities integral to decentralized application (dApp) development on the Neo blockchain. The SDK aims to streamline and simplify the process of interacting with various types of contracts and blockchain components, empowering developers to focus on application logic and user experience.
