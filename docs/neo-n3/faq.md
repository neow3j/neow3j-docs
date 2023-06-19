# Frequently Asked Questions

This page contains questions that we've been asked in the past and might arise during your development process.

### General Questions

##### Why do I need docker installed for running the test framework?

The test framework runs a neo-node instance inside of a docker container. This allows you to test your smart contract
and invocations with a running Neo N3 blockchain.

##### How can I make use of Neo's fine granularity for transaction signatures?

For this, we suggest you to read through
[this Medium article](https://neospcc.medium.com/thou-shalt-check-their-witnesses-485d2bf8375d) about the mechanisms
around signers, scopes and witnesses.

##### What is the difference between Witness Rules and the `@Permission`s on a smart contract 

Witness rules (but also the `allowContracts` and `allowGroups` witness scopes) allow you to control the usage of your
signature on a per-transaction basis. It tells the NeoVM in which contracts it is allowed to use the provided
signature. If your invocation leads to a call to a contract not mentioned in your rules, then the called contract cannot
make changes on your behalf. On the other hand, the `@Permission` feature on a smart contract restricts the calls that 
the contract can make once deployed. I.e., it doesn't care about signatures but can be used to prohibit calling
arbitrary contracts from within a contract. These restrictions are applied to all invocations of that contract. Both
concepts are used to reduce the attack surface on a transaction. Note, that permissions only apply when it's an contract
call that changes state or emits notifications. Reads from one contract to another cannot be restricted with this
feature. 

### SDK - dApp Development

##### What RPC is used to relay a transaction that I will pay for, and which ones are just for reading?

The RPC `sendrawtransaction` is used to relay transactions that have been signed and will persist on-chain (i.e., it
costs GAS). The RPCs `invokefunction` and `invokescript` are read-only and can be used to "mock" invocations to figure
out what would happen if this function or script was invoked/executed in a signed transaction.

##### How does the node validate a witness check when I only use read invocations?

When invoking methodds with `invokefunction` or `invokescript` RPCs, calls to `checkWitness` return true as long as the
correct signers (with the correct scope) are attached in the request.

### Devpack - Smart Contract Development

No FAQs yet for the devpack.

**Please reach out to us in our [Discord Server](https://discord.gg/RBukhnEeke) if you think there are questions and
answers that we should add on this page.**
