## Iterators

**TL;DR:** You can use `SmartContract.callFunctionReturningIterator(...)` to invoke a method that returns an iterator and traverse through the iterator using the resulting `Iterator` object and its utility methods.

A smart contract method can return an iterator to simplify reading through storage with many entries. When calling such a method using an RPC (e.g., `invokefunction`), the node you're connected to opens a session and returns a `sessionId` and an `iteratorId`. You can then use these values with the RPC `traverseIterator`, specifying the number `n` of entries you want to iterate through with each call. Initially, this RPC returns the first `n` entries of the iterator. If there are more entries, subsequent calls to the same RPC return the next `n` entries of the iterator, and so on.

**Note 1:** You cannot traverse an iterator that is returned as the result of a write transaction.

**Note 2:** Some nodes (especially public nodes) have disabled sessions due to DoS concerns. If this is the case, neow3j provides a custom utility method to unwrap the iterator on the NeoVM level and return an array of entries with the method `SmartContract.callFunctionAndUnwrapIterator(...)`.
