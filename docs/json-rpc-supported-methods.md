# NEO JSON-RPC Support

The following NEO JSON-RPC methods are implemented in neow3j:

### Version 2.10.*

- [ ] `claimgas <address>`: Claims GAS in the wallet. **Needs to open the wallet**.
- [x] `dumpprivkey <address>`: Exports the private key of the specified address. **Needs to open the wallet**.
- [x] `getaccountstate <address>`: Checks account asset information according to account address.
- [x] `getassetstate <asset_id>`: Queries asset information according to the specified asset number.
- [x] `getbalance <asset_id>`: Returns the balance of the corresponding asset in the wallet according to the 
- [x] `getbestblockhash`: Gets the hash of the tallest block in the main chain.
- [x] `getblock <hash> [verbose=0]`: Returns the corresponding block information according to the specified hash value.
- [x] `getblock	<index> [verbose=0]`: Returns the corresponding block information according to the specified index.
- [x] `getblockcount`: Gets the number of blocks in the main chain.
- [x] `getblockhash <index>`: Returns the hash value of the corresponding block based on the specified index.
- [x] `getblockheader <hash> [verbose=0]`: Returns the corresponding block header information according to the specified script hash.
- [x] `getblocksysfee <index>`: Returns the system fees before the block according to the specified index.
- [x] `getclaimable <address>`: Returns claimable GAS information of the specified address.
- [x] `getconnectioncount`: Gets the current number of connections for the node.
- [x] `getcontractstate <script_hash>`: Returns information about the contract based on the specified script hash.
- [ ] `getmetricblocktimestamp <blockNumbers> <endHeight>`: Returns timestamps of the specified block and its previous n blocks.
- [x] `getnep5balances <address>`: Returns the balance of all NEP-5 assets in the specified address.
- [ ] `getnep5transfers <address>`: Returns all the NEP-5 transaction information occurred in the specified address.
- [x] `getnewaddress`: Creates a new address. **Needs to open the wallet**.
- [x] `getrawmempool`: Gets a list of unconfirmed transactions in memory.
- [x] `getrawtransaction <txid> [verbose=0]`: Returns the corresponding transaction information based on the specified hash value.
- [ ] `getunclaimed <address>`: Returns unclaimed GAS amount of the specified address.
- [ ] `getunclaimedgas`: Gets the amount of unclaimed GAS in the wallet. **Needs to open the wallet**.
- [x] `getunspents <address>`: Returns information of the unspent UTXO assets at the specified address.
- [x] `getstorage <script_hash> <key>`: Returns the stored value based on the contract script hash and key.
- [ ] `gettransactionheight <txid>`: Returns the block index in which the transaction is found.
- [x] `gettxout <txid> <n>`: Returns the corresponding transaction output (change) information based on the specified hash and index.
- [x] `getpeers`: Gets a list of nodes that are currently connected/disconnected by this node.
- [x] `getversion`: Gets version information of this node.
- [x] `getvalidators`: Gets NEO consensus nodes information.
- [x] `getwalletheight`: Gets the current wallet index height. **Needs to open the wallet**.
- [ ] `importprivkey <key>`: Imports the private key to the wallet. **Needs to open the wallet**.
- [x] `invoke <script_hash> <params>`: Invokes a smart contract at specified script hash with the given parameters.
- [x] `invokefunction <script_hash> <operation> <params>`: Invokes a smart contract at specified script hash, passing in an operation and its params.
- [x] `invokescript <script>`: Runs a script through the virtual machine and returns the results.
- [x] `listaddress`: Lists all the addresses in the current wallet.	**Needs to open the wallet**.
- [ ] `listplugins`: Returns a list of plugins loaded by the node.
- [x] `sendrawtransaction <hex>`: Broadcast a transaction over the network. See the network protocol documentation.
- [ ] `sendfrom <asset_id> <address> <value> [fee=0]`: Transfers from the specified address to the destination address. **Needs to open the wallet**.
- [x] `sendtoaddress <asset_id> <address> <value> [fee=0] [change_address]`: Transfer to specified address. **Needs to open the wallet**.
- [x] `sendmany <outputs_array> [fee=0] [change_address]`: Bulk transfer order. **Needs to open the wallet**.
- [x] `submitblock <hex>`: Submit new blocks. **Needs to be a consensus node**.
- [x] `validateaddress <address>`: Verify that the address is a correct NEO address.

### Version 2.9.*

- [x] `dumpprivkey <address>`: Exports the private key of the specified address. **Needs to open the wallet**.
- [x] `getaccountstate <address>`: Checks account asset information according to account address.
- [x] `getassetstate <asset_id>`: Queries asset information according to the specified asset number.
- [x] `getbalance <asset_id>`: Returns the balance of the corresponding asset in the wallet according to the specified asset number. **Need to open the wallet**.
- [x] `getbestblockhash`: Gets the hash of the tallest block in the main chain.
- [x] `getblock <hash> [verbose=0]`: Returns the corresponding block information according to the specified hash value.
- [x] `getblock	<index> [verbose=0]`: Returns the corresponding block information according to the specified index.
- [x] `getblockcount`: Gets the number of blocks in the main chain.
- [x] `getblockhash <index>`: Returns the hash value of the corresponding block based on the specified index.
- [x] `getblockheader <hash> [verbose=0]`: Returns the corresponding block header information according to the specified script hash.
- [x] `getblocksysfee <index>`: Returns the system fees before the block according to the specified index.
- [x] `getconnectioncount`: Gets the current number of connections for the node.
- [x] `getcontractstate <script_hash>`: Returns information about the contract based on the specified script hash.
- [x] `getnewaddress`: Creates a new address. **Needs to open the wallet**.
- [x] `getrawmempool`: Gets a list of unconfirmed transactions in memory.
- [x] `getrawtransaction <txid> [verbose=0]`: Returns the corresponding transaction information based on the specified hash value.
- [x] `getstorage <script_hash> <key>`: Returns the stored value based on the contract script hash and key.
- [x] `gettxout <txid> <n>`: Returns the corresponding transaction output (change) information based on the specified hash and index.
- [x] `getpeers`: Gets a list of nodes that are currently connected/disconnected by this node.
- [x] `getversion`: Gets version information of this node.
- [x] `getvalidators`: Gets NEO consensus nodes information.
- [x] `getwalletheight`: Gets the current wallet index height. **Needs to open the wallet**.
- [x] `invoke <script_hash> <params>`: Invokes a smart contract at specified script hash with the given parameters.
- [x] `invokefunction <script_hash> <operation> <params>`: Invokes a smart contract at specified script hash, passing in an operation and its params.
- [x] `invokescript <script>`: Runs a script through the virtual machine and returns the results.
- [x] `listaddress`: Lists all the addresses in the current wallet.	**Needs to open the wallet**.
- [x] `sendrawtransaction <hex>`: Broadcast a transaction over the network. See the network protocol documentation.
- [x] `sendtoaddress <asset_id> <address> <value> [fee=0] [change_address]`: Transfer to specified address. **Needs to open the wallet**.
- [x] `sendmany <outputs_array> [fee=0] [change_address]`: Bulk transfer order. **Needs to open the wallet**.
- [x] `submitblock <hex>`: Submit new blocks. **Needs to be a consensus node**.
- [x] `validateaddress <address>`: Verify that the address is a correct NEO address.