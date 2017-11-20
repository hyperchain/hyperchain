# JSON-RPC API

## JSON-RPC Overview

JSON-RPC is a stateless, light-weight remote procedure call(RPC) protocol. It is transport agnostic in that the concepts can be used within the same process, over sockets, over http, or in many various message passing environments. It uses [JSON](http://www.json.org/) ([RFC 4627](http://www.ietf.org/rfc/rfc4627.txt)) as data format. A rpc call is represented by sending a Request object to a Server. The Request object has the following members:

**jsonrpc**

  A String specifying the version of the JSON-RPC protocol. MUST be exactly "2.0".

**method**

  A String containing the name of the method to be invoked. Method names that begin with the word rpc followed by a period character (U+002E or ASCII 46) are reserved for rpc-internal methods and extensions and MUST NOT be used for anything else.

**params**

  A Structured value that holds the parameter values to be used during the invocation of the method. This member MAY be omitted.

**id**

  An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null and Numbers SHOULD NOT contain fractional parts.

When a rpc call is made, the Server MUST reply with a Response object. The Response object has the following members:

**jsonrpc**

  A String specifying the version of the JSON-RPC protocol. MUST be exactly "2.0".

**result**

  This member is REQUIRED on success. This member MUST NOT exist if there was an error invoking the method. The value of this member is determined by the method invoked on the Server.

**error**

  This member is REQUIRED on error. This member MUST NOT exist if there was no error triggered during invocation. The value for this member MUST be an Object including `code` field and `message` field.

**id**

  This member is REQUIRED. It MUST be the same as the value of the id member in the Request Object. If there was an error in detecting the id in the Request object (e.g. Parse error/Invalid Request), it MUST be Null.

## Hyperchain JSON-RPC API Design

Hyperchain JSON-RPC API consists of seven services: 

1. Transaction service，the method name prefix is "**tx**".

2. Contract service，the method name prefix is "**contract**".

3. Block service，the method name prefix is "**block**".

4. Archive service，the method name prefix is "**archive**".

5. Event subscription service，the method name prefix is "**sub**".

6. Node service，the method name prefix is "**node**".

7. Cert service，the method name prefix is "**cert**".

Hyperchain JSON-RPC API design bases on JSON-RPC 2.0 specification, all the requests are POST. The Request object has the following members:

- **`jsonrpc`**: A String specifying the version of the JSON-RPC protocol. MUST be exactly "2.0".
- **`namespace`**: A String specifying namespace name. This request will be sent to this namespace to handle.
- **`method`**: A String containing the name of the method to be invoked. The format is: [service prefix]_[method name].
- **`params`**: A Structured value that holds the parameter values to be used during the invocation of the method. This member MAY be omitted.
- **`id`**: An identifier established by the Client that MUST contain a String, Number.

```bash
# Request
curl -X POST -d '{"jsonrpc":"2.0","method":"block_latestBlock","namespace":"global","params":[],"id":1}' localhost:8081

```

The Response object has the following members:

- **`jsonrpc`**: A String specifying the version of the JSON-RPC protocol. MUST be exactly "2.0".
- **`namespace`**: A String specifying which namespace this response is sent from。
- **`code`**: Status code. If successful, the value is 0，others status code see 表5-1。
- **`message`**: Status message. If successful, the value is SUCCESS, otherwise, the value is error detail message.
- **`result`**: The data returned by the method invoked on the Server. This member doen't exist if there was an error invoking the method.
- **`id`**: The same as the value of the id member in the Request Object.

```bash
# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.4",
    "number": "0x3",
    "hash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
    "parentHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
    "writeTime": 1481778653997475900,
    "avgTime": "0x2",
    "txcounts": "0x1",
    "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c",
    "transactions": [
      {
        "version": "1.4",
        "hash": "0xf57a6443d08cda4a3dfb8083804b6334d17d7af51c94a5f98ed67179b59169ae",
        "blockNumber": "0x3",
        "blockHash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
        "txIndex": "0x0",
        "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
        "to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
        "amount": "0x0",
        "timestamp": 1481778652973000000,
        "nonce": 3573634504790373,
        "extra": "",
        "executeTime": "0x2",
        "payload": "0x81053a70000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000001c8"
      }
    ]
  }
}
```


If the method call succeeds, the members returned including: `jsonrpc`, `namespace`, `id`, `code`, `message`,  `result`. And the `code` value is 0, `message` value is SUCCESS,  so the client can determine the success of the method call through the values of these two fields. 

If the method call failed, the members returned including: `jsonrpc`, `namespace`, `id`, `code`, `message`. And the `code` is non-zero value, `message` value is error message. Status code definition: 

|  code  |               implication                |
| :----: | :--------------------------------------: |
|   0    |           Request successfully           |
| -32700 | Parse error, invalid JSON was received by the server. <br/>An error occurred on the server while parsing the JSON text. |
| -32600 | Invalid request,  the JSON sent is not a valid Request object. |
| -32601 | The method does not exist / is not available. |
| -32602 |       Invalid method parameter(s).       |
| -32603 |         Internal JSON-RPC error.         |
| -32000 | Internal Hyperchain error or the node does not install solidity environment. |
| -32001 |  The data for the query does not exist.  |
| -32002 |      The account is out of balance.      |
| -32003 |      Invalid transaction signature.      |
| -32004 |         Contract deploy failed.          |
| -32005 |         Contract invoke failed.          |
| -32006 |           System is too busy.            |
| -32007 | duplicate transaction was received by the server. |
| -32008 | Not enough permission to operate contract. |
| -32009 |  The (contract) account does not exist.  |
| -32010 |      The namespace does not exist.       |
| -32011 | No block generated on the chain. It may return When querying latest block. |
| -32012 | The event client subscribed does not exist. (*Reserved code*) |
| -32013 | It returns when making snapshot or data archive happen unexpected error. |
| -32098 | Invalid TCert or missing TCert when sending request to node. |
| -32099 |           Failed to get TCert.           |



## JSON-RPC Methods

#### Transaction

- [tx_getTransactions](#tx_getTransactions)
- [tx_getDiscardTransactions](#tx_getDiscardTransactions)
- [tx_getTransactionByHash](#tx_getTransactionByHash)
- [tx_getTransactionByBlockHashAndIndex](#tx_getTransactionByBlockHashAndIndex)
- [tx_getTransactionByBlockNumberAndIndex](#tx_getTransactionByBlockNumberAndIndex)
- [tx_getTransactionsCount](#tx_getTransactionsCount)
- [tx_getTxAvgTimeByBlockNumber](#tx_getTxAvgTimeByBlockNumber)
- [tx_getTransactionReceipt](#tx_getTransactionReceipt)
- [tx_getBlockTransactionCountByHash](#tx_getBlockTransactionCountByHash)
- [tx_getBlockTransactionCountByNumber](#tx_getBlockTransactionCountByNumber)
- [tx_getSignHash](#tx_getSignHash)
- [tx_getTransactionsByTime](#tx_getTransactionsByTime)
- [tx_getDiscardTransactionsByTime](#tx_getDiscardTransactionsByTime)
- [tx_getBatchTransactions](#tx_getBatchTransactions)
- [tx_getBatchReceipt](#tx_getBatchReceipt)

#### Contract

- [contract_compileContract](#contract_compileContract)
- [contract_deployContract](#contract_deployContract)
- [contract_invokeContract](#contract_invokeContract)
- [contract_getCode](#contract_getCode)
- [contract_getContractCountByAddr](#contract_getContractCountByAddr)
- [contarct_maintainContract](#contarct_maintainContract)
- [contract_getStatus](#contract_getStatus)
- [contract_getCreator](#contract_getCreator)
- [contract_getCreateTime](#contract_getCreateTime)
- [contract_getDeployedList](#contract_getDeployedList)

#### Block

- [block_latestBlock](#block_latestBlock)
- [block_getBlocks](#block_getBlocks)
- [block_getBlockByHash](#block_getBlockByHash)
- [block_getBlockByNumber](#block_getBlockByNumber)
- [block_getAvgGenerateTimeByBlockNumber](#block_getAvgGenerateTimeByBlockNumber)
- [block_getBlocksByTime](#block_getBlocksByTime)
- [block_getGenesisBlock](#block_getGenesisBlock)
- [block_getChainHeight](#block_getChainHeight)
- [block_getBatchBlocksByHash](#block_getBatchBlocksByHash)
- [block_getBatchBlocksByNumber](#block_getBatchBlocksByHash)

#### Subscription

- [sub_newBlockSubscription](#sub_newBlockSubscription)
- [sub_newEventSubscription](#sub_newEventSubscription)
- [sub_getLogs](#sub_getLogs)
- [sub_newSystemStatusSubscription](#sub_newSystemStatusSubscription)
- [sub_newArchiveSubscription](#sub_newArchiveSubscription)
- [sub_getSubscriptionChanges](#sub_getSubscriptionChanges)
- [sub_unSubscription](#sub_unSubscription)

#### Node

- [node_getNodes](#node_getNodes)
- [node_getNodeHash](#node_getNodeHash)
- [node_deleteVP](#node_deleteVP)
- [node_deleteNVP](#node_deleteNVP)

#### Certificate

- [cert_getTCert](#cert_getTCert)



## JSON-RPC API Reference

### Transaction

#### <a name="tx_getTransactions">tx_getTransactions</a>

Returns a list of transactions in blocks from start block number to end block number.

##### Parameters

1. `<Object>`
- `from`: `<blockNumber>` - Start block number.
- `to`: `<blockNumber>` - End block number.

 `from` must be less than or equal `to`, otherwise returns error.

<a name="blockNumber"></a>

Type `<blockNumber>` can be:

- Decimal integer.
- Hex string. 
- The string `"latest"` for the latest block.

##### Returns<a name="validTransaction"></a>

1. `[<Transaction>]` - the valid Transaction object has the following members:
- `version`: `<string>` - Platform version number.
- `hash`: `<string>`, 32 Bytes - Hash of the transaction.
- `blockNumber`: `<string>` - Block number where this transaction was in.
- `blockHash`: `<string>`, 32 Bytes - Hash of the block where this transaction was in.
- `txIndex`: `<string>` - Transaction index in the block.
- `from`: `<string>`, 20 Bytes - Address of the sender.
- `to`: `<string>`, 20 Bytes - Address of the receiver.
- `amount`: `<string>` - Transfer amount.
- `timestamp`: `<number>` - The unix timestamp for when the transaction was generated.
- `nonce`: `<number>` - 16-bit random number.
- `extra`: `<string>` - Extra information of this transaction.
- `executeTime`: `<string>` - The time it takes to execute this transaction (ms).
- `payload`: `<string>` - The data send along with deploying contract, invoking contract and upgrading contract.

##### Example1: Nomal request

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getTransactions", "params": [{"from": 1, "to": 2}], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "result": [
    {
      "version": "1.0",
      "hash": "0x88d5b325dc9042ff92a9fa26ed8c943719bb049ac7022abd09bb85da36f531e4",
      "blockNumber": "0x2",
      "blockHash": "0xc6418753c28ad6d744cb4bbe689521696ba65ad010ce24056b6f8def9fc5cdd5",
      "txIndex": "0x0",
      "from": "0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd",
      "to": "0x0000000000000000000000000000000000000000",
      "amount": "0x0",
      "timestamp": 1486994814684628715,
      "nonce": 7948317390228704,
	  "extra": "",
      "executeTime": "0x2",
      "payload": "0x60606040526000805463ffffffff19168155609e908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d146056578063d09de08a14607c575b6002565b346002576000805463ffffffff8116600435016024350163ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b3460025760546000805463ffffffff19811663ffffffff90911660010117905556"
    },
    {
      "version": "1.0",
      "hash": "0xf7149a8349f1853d8d713a15935e5059e6f55c2827f0c88f8414dd0402d6760b",
      "blockNumber": "0x1",
      "blockHash": "0x4bab3f9297e737eb197d666a2f08219f94460ace08a8e1ecad87e6e52183bcd5",
      "txIndex": "0x0",
      "from": "0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd",
      "to": "0x0000000000000000000000000000000000000000",
      "amount": "0x0",
      "timestamp": 1486994799163184948,
      "nonce": 2099818402815731,
      "extra": "",
      "executeTime": "0x7",
      "payload": "0x60606040526000805463ffffffff19168155609e908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d146056578063d09de08a14607c575b6002565b346002576000805463ffffffff8116600435016024350163ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b3460025760546000805463ffffffff19811663ffffffff90911660010117905556"
    }
  ]
}
```

##### Example2: Block does not exist

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getTransactions", "params": [{"from": 1, "to": 2}], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 71,
  "code": -32602,
  "message": "block number 1 is out of range, and now latest block number is 0"
}
```

#### <a name="tx_getDiscardTransactions">tx_getDiscardTransactions</a>

Returns all the invalid transactions.

##### Parameters

none

##### Returns<a name="invalidTransaction"></a>

1. `[<Transaction>]` - the invalid Transaction object has the following members:
- `version`: `<string>` - Platform version number.
- `hash`: `<string>`, 32 Bytes - Hash of the transaction.
- `from`: `<string>`, 20 Bytes - Address of the sender.
- `to`: `<string>`, 20 Bytes - Address of the receiver.
- `amount`: `<string>` - Transfer amount.
- `timestamp`: `<number>` - The unix timestamp for when the transaction was generated.
- `nonce`: `<number>` - 16-bit random number.
- `extra`: `<string>` - Extra information of this transaction.
- `payload`: `<string>` - The data send along with deploying contract, invoking contract and upgrading contract.
- `invalid`: `<boolean>` - The flag of invalid transaction.
- `invalidMsg`: `<string>` - Invalid message of this transaction.

For invalid transaction, `invalid` value is true, and `invalidMsg` has following situations:  

- **DEPLOY_CONTRACT_FAILED** - Contract deploy failed;
- **INVOKE_CONTRACT_FAILED** - Contract invoke failed；
- **SIGFAILED** - The transaction signature is invalid；
- **OUTOFBALANCE** - Transfer of account is out of balance;
- **INVALID_PERMISSION** - Not enough permission to operate this contract;

##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_ getDiscardTransactions", "params": [], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "",
      "hash": "0x100ff931204d149f88c0778f6e7b8d4b11ba3c8c720f0cc3e204b46999954ed4",
      "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
      "to": "0x0000000000000000000000000000000000000000",
      "amount": "0x0",
      "timestamp": 1482405417011000000,
      "nonce": 6848885244669098,
"extra": "",
      "payload": "0x60606040526002600055600256",
      "invalid": true,
      "invalidMsg": "DEPLOY_CONTRACT_FAILED"
}
  ]
}
```

##### Example2: There is no invalid transactions

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_ getDiscardTransactions", "params": [], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": -32001,
  "message": "Not found discard transactions "
}
```

#### <a name= "tx_getTransactionByHash">tx_getTransactionByHash</a>

Returns the information about a transaction by transaction hash.

##### Parameters

1. `<string>`, 32 Bytes - Hash of a transaction.

##### Returns

1. `[<Transaction>]` - the Transaction object has the following members:
- `version`: `<string>` - Platform version number.
- `hash`: `<string>`, 32 Bytes - Hash of the transaction.
- `blockNumber`: `<string>` - Block number where this transaction was in.
- `blockHash`: `<string>`, 32 Bytes - Hash of the block where this transaction was in.
- `txIndex`: `<string>` - Transaction index in the block.
- `from`: `<string>`, 20 Bytes - Address of the sender.
- `to`: `<string>`, 20 Bytes - Address of the receiver.
- `amount`: `<string>` - Transfer amount.
- `timestamp`: `<number>` - The unix timestamp for when the transaction was generated.
- `nonce`: `<number>` - 16-bit random number.
- `extra`: `<string>` - Extra information of this transaction.
- `executeTime`: `<string>` - The time it takes to execute this transaction (ms).
- `payload`: `<string>` - The data send along with deploying contract, invoking contract and upgrading contract.
- `invalid`: `<boolean>` - The flag of invalid transaction.
- `invalidMsg`: `<string>` - Invalid message of this transaction.

For invalid transaction, `invalid` value is true, and `invalidMsg` has following situations:  

- **DEPLOY_CONTRACT_FAILED** - Contract deploy failed;
- **INVOKE_CONTRACT_FAILED** - Contract invoke failed；
- **SIGFAILED** - The transaction signature is invalid；
- **OUTOFBALANCE** - Transfer of account is out of balance;
- **INVALID_PERMISSION** - Not enough permission to operate this contract;

##### Example1: Query valid transaction

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method":"tx_getTransactionByHash", "params":["0xe652e25e617c5f193b240c0d8ff1941a8cfb1d15434eb3830892b7a8389730aa"], "id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.0",
    "hash": "0xe652e25e617c5f193b240c0d8ff1941a8cfb1d15434eb3830892b7a8389730aa",
    "blockNumber": "0x4",
    "blockHash": "0x6ea0c80c1532c273c124511e364fc0a9225e0d129e53249f8e26752ee7d7d989",
    "txIndex": "0x0",
    "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
    "to": "0x0000000000000000000000000000000000000000",
    "amount": "0x0",
    "timestamp": 1482405601747000000,
    "nonce": 6788653222523786,
    "extra": "",
    "executeTime": "0x2",
    "payload": "0x606060405260008055602d8060146000396000f3606060405260e060020a6000350463be1c766b8114601c575b6002565b34600257600060026000908155600256"
  }
}
```

##### Example2: Query invalid transaction

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0","method":"tx_getTransactionByHash","params":["0x1f6dc4c744ce5e8a39e6a19f19dc27c99d7efd8e38061e80550bf5e7ab1060e1"],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.3",
    "hash": "0x1f6dc4c744ce5e8a39e6a19f19dc27c99d7efd8e38061e80550bf5e7ab1060e1",
    "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
    "to": "0x0000000000000000000000000000000000000000",
    "amount": "0x0",
    "timestamp": 1509448178302000000,
    "nonce": 1166705097783423,
    "extra": "",
    "payload": "0x6060604052600080553415601257600080fd5b5b6002600090815580fd5b5b5b60918061002d6000396000f300606060405263ffffffff7c0100000000000000000000000000000000000000000000000000000000600035041663be1c766b8114603c575b600080fd5b3415604657600080fd5b604c605e565b60405190815260200160405180910390f35b6000545b905600a165627a7a723058201e24ee668219357c2daa85cc0d2b3b31c192431783315ea37ed69c9e80a100e90029",
    "invalid": true,
    "invalidMsg": "DEPLOY_CONTRACT_FAILED"
  }
}
```

##### Example3: The transaction requested does not exist

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method":"tx_getTransactionByHash", "params":[" 0x0e707231fd779779ce25a06f51aec60faed8bf6907e6d74fb11a3fd585831a7e"], "id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": -32001,
  "message": "Not found transaction 0x0e707231fd779779ce25a06f51aec60faed8bf6907e6d74fb11a3fd585831a7e"
}
```

#### <a name="tx_getTransactionByBlockHashAndIndex">tx_getTransactionByBlockHashAndIndex</a>

Returns information about a transaction by block hash and transaction index position.

##### Parameters

1. `<string>`, 32 Bytes - Hash of a block.
2. `<number>` - Transaction index position. This value can be decimal integer or hex string.

##### Returns

1. `<Transaction>` - the members of Transaction object see [Valid Transaction](#validTransaction).

##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getTransactionsByBlockHashAndIndex","params": ["0xd198976fa8b4ca2de6b1b137552b84dc08b7cdcbebbf9388add88f4710fd2cf9", 0], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.0",
    "hash": "0xe81d39df11779c7f83e6073cc659c7ee85708c135b6557d318e765b9f938c02f",
    "blockNumber": "0x2",
    "blockHash": "0xd198976fa8b4ca2de6b1b137552b84dc08b7cdcbebbf9388add88f4710fd2cf9",
    "txIndex": "0x0",
    "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
    "to": "0x3a3cae27d1b9fa931458b5b2a5247c5d67c75d61",
    "amount": "0x0",
    "timestamp": 1481767474717000000,
    "nonce": 8054165127693853,
    "extra": "",
    "executeTime": "0x2",
    "payload": "0x6fd7cc16000000000000000000000000000000000000000000000000000000000000007b00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
  }
}
```

##### Example2: The block requested does not exist

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getTransactionsByBlockHashAndIndex","params": ["0xd198976fa8b4ca2de6b1b137552b84dc08b7cdcbebbf9388add88f4710fd2cf9", 0], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 71,
  "code": -32001,
  "message": "Not found block 0xd198976fa8b4ca2de6b1b137552b84dc08b7cdcbebbf9388add88f4710fd2cf9"
}
```

#### <a name="tx_getTransactionByBlockNumberAndIndex">tx_getTransactionByBlockNumberAndIndex</a>

Returns information about a transaction by block number and transaction index position.

##### Parameters

1. `<blockNumber>` - Block number. See type [Block Number](#blockNumber).
2. `<number>` - Transaction index position. This value can be decimal integer or hex string.

##### Returns

1. `<Transaction>` - the members of Transaction object see [Valid Transaction](#validTransaction).

##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method":   "tx_getTransactionByBlockNumberAndIndex", "params": [2,0], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.0",
    "hash": "0xe81d39df11779c7f83e6073cc659c7ee85708c135b6557d318e765b9f938c02f",
    "blockNumber": "0x2",
    "blockHash": "0xd198976fa8b4ca2de6b1b137552b84dc08b7cdcbebbf9388add88f4710fd2cf9",
    "txIndex": "0x0",
    "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
    "to": "0x3a3cae27d1b9fa931458b5b2a5247c5d67c75d61",
    "amount": "0x0",
    "timestamp": 1481767474717000000,
    "nonce": 8054165127693853,
    "extra": "",
    "executeTime": "0x2",
    "payload": "0x6fd7cc16000000000000000000000000000000000000000000000000000000000000007b00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
  }
}
```

##### Example2: The block requested does not exist

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method":   "tx_getTransactionsByBlockNumberAndIndex", "params": [2,0], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 71,
  "code": -32602,
  "message": "block number 2 is out of range, and now latest block number is 0"
}
```

#### <a name="tx_getTransactionsCount">tx_getTransactionsCount</a>

Returns the number of transactions on the chain.

##### Parameters

none

##### Returns

1. `<Object>`
- `count`: `<string>` - The number of transactions.
- `timestamp`: `<number>` - The unix timestamp for response (ns).

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_ getTransactionsCount", "params": [], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 71,
  "code": 0,
  "message": "SUCCESS":,
  "result": {
    "count": "0x9",
    "timestamp": 1480069870678091862
   }
}
```

#### <a name="tx_getTxAvgTimeByBlockNumber">tx_getTxAvgTimeByBlockNumber</a>

Returns the average execution time of all transactions in the given block number.

##### Parameters

1. `<Object>`
- `from`: `<blockNumber>` - Start block number. See type [Block Number](#blockNumber).
- `to`: `<blockNumber>` - End block number. See type [Block Number](#blockNumber).

 `from` must be less than or equal `to`, otherwise returns error.

##### Returns

1. `<string>` -  the average execution time of all transactions (ms).

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getTxAvgTimeByBlockNumber", "params": [{"from":10, "to":19}], "id": 71}'

# Response
{
  "id":71,
  "jsonrpc": "2.0",
  "namespace":"global",
  "code": 0,
  "message": "SUCCESS",
  "result": "0xa9"
}
```

#### <a name="tx_getTransactionReceipt">tx_getTransactionReceipt</a>

Returns the receipt of a transaction by transaction hash.

##### Parameters

1. `<string>`, 32 Bytes - Hash of a transaction.

##### Returns<a name="receipt"></a>

1. `<Receipt>` - the Receipt object has the following members:
- `version`: `<string>` - Platform version number.
- `txHash`: `<string>`, 32 Bytes - Hash of the transaction.
- `vmType`: `<string>` - The execution engine type used by this transaction execution, this value is `EVM` OR `JVM`.
- `contractAddress`: `<string>`, 20 Bytes - The contract address created, if the transaction was a contract creation, otherwise `0x0000000000000000000000000000000000000000`.
- `ret`: `<string>` - Contract compling code OR contract execution results.
- `log`: `[<Log>]` - Array of Log. The Log object has following members:
  - `address`: `<string>`, 20 Bytes - Contract address by which this event log is generated.
  - `topics`: `[<string>]` - Array of 32 Bytes string. Topics are order-dependent. Each topic can also be an array of string with "or" options. The first topic is the unique identity of the event.
  - `data`: `<string>` - Log message or data.
  - `blockNumber`: `<number>` - Block number where this transaction was in.
  - `blockHash`: `<string>`, 32 Bytes - Hash of the block where this transaction was in.
  - `txIndex`: `<number>` - Transaction index position in the block.
  - `index`: `<number>` - Event log index position in all logs generated in this transaction.

If the transaction requested has not been confirmed, the error code returned is **-32001**. If an error occurs during the transaction processing, the error may be:  

- **OUTOFBALANCE** - Transfer of account is out of balance, code is **-32002**;
- **SIGFAILED** - The transaction signature is invalid, code is **-32003**；
- **DEPLOY_CONTRACT_FAILED** - Contract deploy failed, code is **-32004**;
- **INVOKE_CONTRACT_FAILED** - Contract invoke failed, code is **-32005**；
- **INVALID_PERMISSION** - Not enough permission to operate this contract, code is **-32008**;

##### Example1: The transaction has not been confirmed

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method":"tx_getTransactionReceipt","params":["0x0e0758305cde33c53f8c2b852e75bc9b670c14c547dd785d93cb48f661a2b36a "],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": -32001,
  "message": "Not found receipt by 0x0e0758305cde33c53f8c2b852e75bc9b670c14c547dd785d93cb48f661a2b36a"
}
```

##### Example2: Deploying contract failed

For this example, we use the following contract to recreate the situation:

```bash
contract TestContractor{
    int length = 0;
    
    modifier justForTest(){
        length = 2;
        throw;
        _;
    }
    function TestContractor()justForTest{
    }
    
    function getLength() returns(int){
        return length;
    }
}
```

We use contract compiled code as the value of `payload` in [contract_deployContract](#contract_deployContract) , then the deploying contract request is as follows:

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method":"contract_deployContract", "params":[{
"from":"17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
"nonce":7021040367249265,
"payload":"0x60606040526000600055346000575b60026000556000565b5b5b603f806100266000396000f3606060405260e060020a6000350463be1c766b8114601c575b6000565b3460005760266038565b60408051918252519081900360200190f35b6000545b9056",
"timestamp":1487042279126000000,
"signature":"0xfc1cb1986dd4ee4a5f8d8238e2f7bac1866aad235d587eb641d76270bf686418310ab7d42dc0f2575aa858a88ae7732cd617281eedb38636e843ff3b49b8f8ab01"}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "0x33aef7e6bad2ae27c23a8ab44f56aef87042f1f0b02e1b0ee5e8a304705292a6"
}
```

Next, trying to get information about receipt of this transaction by transaction hash, an error will be returned: 

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method":"tx_getTransactionReceipt","params":["0x33aef7e6bad2ae27c23a8ab44f56aef87042f1f0b02e1b0ee5e8a304705292a6"],"id":1}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":-32004,
  "message":"DEPLOY_CONTRACT_FAILED"
}
```

##### Example3: Invoking contract failed

For this example, we use the following contract to recreate the situation:

```bash
contract TestContractor{
    int length = 0;
    
    modifier justForTest(){
        length = 2;
        throw;
        _;
    }
    function TestContractor(){
    }
    
    function getLength()justForTest returns(int){
        return length;
    }
}
```

We invoke the method `getLength()`,  then the invoking contract request is as follows:

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_invokeContract", "params": [{
"from": "17d806c92fa941b4b7a8ffffc58fa2f297a3bffc", 
"to":"0xaeccd2fd1118334402c5de1cb014a9c192c498df", 
"timestamp":1487042517534000000, 
"nonce":2472630987523856,
"payload":"0xbe1c766b", 
"signature":"0x8c56f025610dd9cb3f4ac346d35978639a536505527b7593d87f3b45c35328637280995ed32f6a6809069da915740b363c1b357cf31a7eb83e05dde0afc4937300"}],"id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result":"0x5233d18f46e9c1ed49dbdeb4273c1c1e0eb176efcedf6edb6d9fa59d33d02fee "
}
```

Next, trying to get information about receipt of this transaction by transaction hash, an error will be returned: 

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method":"tx_getTransactionReceipt","params":["0x5233d18f46e9c1ed49dbdeb4273c1c1e0eb176efcedf6edb6d9fa59d33d02fee"],"id":1}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":-32005,
  "message":"INVOKE_CONTRACT_FAILED"
}
```

##### Example4: Invalid transaction signature

In this example, we use Example3 request example, but change the last letter "c" of param `from` to "0" in order to simulate the situation of illegal signature, then the invoking contract request is as follows:

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_invokeContract", "params": [{
"from": "17d806c92fa941b4b7a8ffffc58fa2f297a3bff0", 
"to":"0xaeccd2fd1118334402c5de1cb014a9c192c498df", 
"timestamp":1481872888621000000, 
"nonce":9206467481004664,
"payload":"0xbe1c766b", 
"signature":"0x57dfa7f2c2d8c762c9c0e5ef7b1c4dda84b584f36799ab751891c8dc553862145f64d1991441c9460481af4e4231db393744ad3cfd37c8cce74c873002745aa401"}],"id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "id": "SUCCESS",
  "result":"0x621d09cd9d5e9027d9b82c5e1fd911ac31297775dbb0c4dab6c6fcd64310fe23"
}
```

Next, trying to get information about receipt of this transaction by transaction hash, an error will be returned: 

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method":"tx_getTransactionReceipt","params":["0x621d09cd9d5e9027d9b82c5e1fd911ac31297775dbb0c4dab6c6fcd64310fe23"],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": -32003,
  "message": " SIGFAILED "
}
```

##### Example5

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getTransactionReceipt", "params":["0x70376053e11bc753b8cc778e2fbb662718671712e1744980ba1110dd1118c059"], "id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.3",
    "txHash": "0x70376053e11bc753b8cc778e2fbb662718671712e1744980ba1110dd1118c059",
    "vmType": "EVM",
    "contractAddress": "0x0000000000000000000000000000000000000000",
    "ret": "0x0",
    "log": [
      {
        "address": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
	"topics": [
                    "0x24abdb5865df5079dcc5ac590ff6f01d5c16edbc5fab4e195d9febd1114503da"
        ],
	"data": "0000000000000000000000000000000000000000000000000000000000000064",
        "blockNumber": 2,
        "blockHash": "0x0c14a89b9611f7f268f26d4ce552de966bebba4aab6aaea988022f3b6817f61b",
        "txHash": "0x70376053e11bc753b8cc778e2fbb662718671712e1744980ba1110dd1118c059",
        "txIndex": 0,
	"index": 0
      }
    ]
  }
}
```

#### <a name="tx_getBlockTransactionCountByHash">tx_getBlockTransactionCountByHash</a>

Returns the number of transactions in a block from a block matching the given block hash.

##### Parameters

1. `<string>`, 32 Bytes - Hash of a block.

##### Returns

1. `<string>` - The number of transactions in a block.

##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method":"tx_getBlockTransactionCountByHash", "params":  ["0x7a87bd1fb51a86763e9791eab1d5ecca7f004bea1cfcc426113b4625d267f699"], "id": 71}'

# Response
{
  "id":71,
  "jsonrpc": "2.0",
  "namespace":"global",
  "code": 0,
  "message": "SUCCESS",
  "result": "0xaf5"
}
```

##### Example2: The block requested does not exist

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method":"tx_getBlockTransactionCountByHash", "params":  ["0x7a87bd1fb51a86763e9791eab1d5ecca7f004bea1cfcc426113b4625d267f699"], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 71,
  "code": -32001,
  "message":"Not found block 0x7a87bd1fb51a86763e9791eab1d5ecca7f004bea1cfcc426113b4625d267f699"
}
```

#### <a name="tx_getBlockTransactionCountByNumber">tx_getBlockTransactionCountByNumber</a>

Returns the number of transactions in a block from a block matching the given block number.

##### Parameters

1. `<blcokNumber>` - Block number. See type [Block Number](#blockNumber).

##### Returns

1. `<string>` - The number of transactions in a block.

##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method":" tx_getBlockTransactionCountByNumber", "params":  ["0x2"], "id": 71}'

# Response
{
  "id":71,
  "jsonrpc": "2.0",
  "namespace":"global",
  "code": 0,
  "message": "SUCCESS",
  "result": "0xaf5"
}
```

##### Example2: The block requested does not exist

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method":" tx_getBlockTransactionCountByNumber", "params":  ["0x2"], "id": 71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 71,
  "code": -32602,
  "message": "block number 0x2 is out of range, and now latest block number is 0"
}
```

#### <a name="tx_getSignHash">tx_getSignHash</a>

Returns transaction content hash string used to sign a transaction by client.

##### Parameters

1. `<Object>`
- `from`: `<string>`, 20 Bytes - Address of the sender.
- `to`: `<string>`, 20 Bytes - [optional] Address of the receiver(account address OR contract address). If it's a contract deployment transaction, needn't specify this member.
- `nonce`: `<number>` - 16-bit random number.
- `extra`: `<string>` - [optional] Extra information of this transaction.
- `value`: `<string>` - [optional] Transfer amount. Value can not be empty if it's a normal transfer transaction, otherwise, you needn't specify this member.
- `payload`: `<string>` - [optional] Payload can not be empty if it's a contract deployment transaction(See [contract_deployContract](#contract_deployContract)), a contract invoke transaction(See [contract_invokeContract](#contract_invokeContract)) or a contract upgrade transaction(See [contract_maintainContract](#contract_maintainContract)), otherwise, you needn't specify this member.
- `timestamp`: `<number>` - The unix timestamp for when the transaction was generated.

##### Returns

1. `<string>` - A hash string.

##### Example

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method":"tx_getSignHash", "params":[{
	"from":"0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
	"nonce":5373500328656597,
	"payload":"0x60606040526000805463ffffffff1916815560ae908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d14605e578063d09de08a146084575b6002565b346002576000805460e060020a60243560043563ffffffff8416010181020463ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b34600257605c6000805460e060020a63ffffffff821660010181020463ffffffff1990911617905556",
	"timestamp":1487771157166000000 }],"id":"1"}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "0x2e6a644a4ca6a9daba4444995dc0dda039208e642df11db35438d18e7c3b13c3"
}
```

#### <a name="tx_getTransactionsByTime">tx_getTransactionsByTime</a>

Returns a list of valid transactions generated at specific time periods.

##### Parameters

1. `<Object>`
- `startTime`: `<number>` - The start unix timestamp.
- `endTime`: `<number>` - The end unix timestamp.

##### Returns

1. `[<Transaction>]` - the members of Transaction object see [Valid Transaction](#validTransaction).

##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"tx_getTransactionsByTime","params":[{"startTime":1, "endTime":1581776001230590326}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [{
    "version": "1.0",
    "hash": "0xbd441c7234e3b83a05c89ed5d548c3d1877306975e271a08e7354d74e45431bc",
    "blockNumber": "0x1",
    "blockHash": "0xa6a4b2df16c7bdeb578aa7de7b05f9b54d96202bdc8414196741842834156ebd",
    "txIndex": "0x0",
    "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
    "to": "0x0000000000000000000000000000000000000000",
    "amount": "0x0",
    "timestamp": 1481767468349000000,
    "nonce": 1775845467490815,
    "extra": "",
    "executeTime": "0x2",
    "payload": "0x606060405234610000575b6101e1806100186000396000f3606060405260e060020a60003504636fd7cc16811461002957806381053a7014610082575b610000565b346100005760408051606081810190925261006091600491606491839060039083908390808284375093955061018f945050505050565b6040518082606080838184600060046018f15090500191505060405180910390f35b346100005761010a600480803590602001908201803590602001908080602002602001604051908101604052809392919081815260200183836020028082843750506040805187358901803560208181028481018201909552818452989a9989019892975090820195509350839250850190849080828437509496506101bc95505050505050565b6040518080602001806020018381038352858181518152602001915080519060200190602002808383829060006004602084601f0104600302600f01f1509050018381038252848181518152602001915080519060200190602002808383829060006004602084601f0104600302600f01f15090500194505050505060405180910390f35b6060604051908101604052806003905b600081526020019060019003908161019f5750829150505b919050565b60408051602081810183526000918290528251908101909252905281815b925092905056"
  }]
}
```

##### Example2: There is no data

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0","method":"tx_getTransactionsByTime","params":[{"startTime":1681776001230590326, "endTime":1681776001230590326}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": []
}
```

#### <a name="tx_getDiscardTransactionsByTime">tx_getDiscardTransactionsByTime</a>

Returns a list of invalid transactions generated at specific time periods.

##### Parameters

1. `<Object>`
- `startTime`: `<number>` - The start unix timestamp.
- `endTime`: `<number>` - The end unix timestamp.

##### Returns

1. `[<Transaction>]` - the members of invalid Transaction object see [Invalid Transaction](#invalidTransaction).


##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":" tx_getDiscardTransactionsByTime","params":[{"startTime":1, "endTime":1581776001230590326}],"id":1}'

# Result
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.3",
      "hash": "0x4e468969d94b92622e385246779d05981ef43869b17c8afedc7e6b5b138ae807",
      "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
      "to": "0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd",
      "amount": "0x1",
      "timestamp": 1501586411342000000,
      "nonce": 4563214039387098,
      "extra": "",
      "payload": "0x0",
      "invalid": true,
      "invalidMsg": "OUTOFBALANCE"
    }
  ]
}
```

#### <a name="tx_getBatchTransactions">tx_getBatchTransactions</a>

Returns a list of transactions by a list of specific transaction hash.

##### Parameters

1. `<Object>`
- `hashs`: `[<string>]` - Array of 32 Bytes string, a list of transaction hash.

##### Returns

1. `[<Transaction>]` -  Array of Transaction object, the members of Transaction object see [Valid Transaction](#validTransaction).

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0","method":"tx_getBatchTransactions","params":[{
	"hashes":["0x22321358931c577ceaa2088d914758148dc6c1b6096a0b3f565d130f03ca75e4","0x7aebde51531bb29d3ba620f91f6e1556a1e8b50913e590f31d4fe4a2436c0602"]
}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.3",
      "hash": "0x22321358931c577ceaa2088d914758148dc6c1b6096a0b3f565d130f03ca75e4",
      "blockNumber": "0x2",
      "blockHash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
      "txIndex": "0x0",
      "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
      "to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
      "amount": "0x0",
      "timestamp": 1509440823410000000,
      "nonce": 8291834415403909,
      "extra": "",
      "executeTime": "0x6",
      "payload": "0x0a9ae69d"
    },
    {
      "version": "1.3",
      "hash": "0x7aebde51531bb29d3ba620f91f6e1556a1e8b50913e590f31d4fe4a2436c0602",
      "blockNumber": "0x1",
      "blockHash": "0x4cd9f393aabb2df51c09e66925c4513e23f0dbbb9e94d0351c1c3ec7539144a0",
      "txIndex": "0x0",
      "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
      "to": "0x0000000000000000000000000000000000000000",
      "amount": "0x0",
      "timestamp": 1509440820498000000,
      "nonce": 5098902950712745,
      "extra": "",
      "executeTime": "0x11",
      "payload": "0x6060604052341561000f57600080fd5b60405160408061016083398101604052808051919060200180519150505b6000805467ffffffff000000001963ffffffff19821663ffffffff600393840b8701840b81169190911791821664010000000092839004840b860190930b16021790555b50505b60de806100826000396000f300606060405263ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416630a9ae69d811460465780638466c3e614606f575b600080fd5b3415605057600080fd5b60566098565b604051600391820b90910b815260200160405180910390f35b3415607957600080fd5b605660a9565b604051600391820b90910b815260200160405180910390f35b600054640100000000900460030b81565b60005460030b815600a165627a7a7230582073eeeb74bb45b3055f1abe89f428d164ef7425bf57a999d219cbaefb6e3c0080002900000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000005"
    }
  ]
}
```

#### <a name="tx_getBatchReceipt">tx_getBatchReceipt</a>

Returns a list of receipt of transactions by a list of specific transaction hash.

##### Parameters

1. `<Object>`
- `hashs`: `[<string>]` - Array of 32 Bytes string, a list of transaction hash.

##### Returns

1. `[<Receipt>]` - Array of Receipt object, the Receipt object see type [Receipt](#receipt).

##### Example

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0","method":"tx_getBatchReceipt","params":[{
	"hashes":["0x22321358931c577ceaa2088d914758148dc6c1b6096a0b3f565d130f03ca75e4","0x7aebde51531bb29d3ba620f91f6e1556a1e8b50913e590f31d4fe4a2436c0602"]
}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.3",
      "txHash": "0x22321358931c577ceaa2088d914758148dc6c1b6096a0b3f565d130f03ca75e4",
      "vmType": "EVM",
      "contractAddress": "0x0000000000000000000000000000000000000000",
      "ret": "0x0000000000000000000000000000000000000000000000000000000000000005",
      "log": []
    },
    {
      "version": "1.3",
      "txHash": "0x7aebde51531bb29d3ba620f91f6e1556a1e8b50913e590f31d4fe4a2436c0602",
      "vmType": "EVM",
      "contractAddress": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
      "ret": "0x606060405263ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416630a9ae69d811460465780638466c3e614606f575b600080fd5b3415605057600080fd5b60566098565b604051600391820b90910b815260200160405180910390f35b3415607957600080fd5b605660a9565b604051600391820b90910b815260200160405180910390f35b600054640100000000900460030b81565b60005460030b815600a165627a7a7230582073eeeb74bb45b3055f1abe89f428d164ef7425bf57a999d219cbaefb6e3c00800029",
      "log": []
    }
  ]
}
```

### Contract

#### <a name="contract_compileContract">contract_compileContract</a>

Compiles contract and returns compiled solidity code and abi definition.

##### Parameters

1. `<string>` - The source code.

##### Returns

1. `<Object>`
- `abi`: `[<string>]` - The contract abi definition.
- `bin`: `[<string>]` - The compiled solidity code.
- `types`: `[<string>]` - The contract name.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"contract_compileContract", "params":["contract Accumulator{    uint32 sum = 0;   function increment(){         sum = sum + 1;     }      function getSum() returns(uint32){         return sum;     }   function add(uint32 num1,uint32 num2) {         sum = sum+num1+num2;     } }"],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "abi": [
	      "[{\"constant\":false,\"inputs\":[{\"name\":\"num1\",\"type\":\"uint32\"},{\"name\":\"num2\",\"type\":\"uint32\"}],\"name\":\"add\",\"outputs\":[],\"payable\":false,\"type\":\"function\"},{\"constant\":false,\"inputs\":[],\"name\":\"getSum\",\"outputs\":[{\"name\":\"\",\"type\":\"uint32\"}],\"payable\":false,\"type\":\"function\"},{\"constant\":false,\"inputs\":[],\"name\":\"increment\",\"outputs\":[],\"payable\":false,\"type\":\"function\"}]"
	      ],
    "bin": [
	      "0x60606040526000805463ffffffff1916815560ae908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d14605e578063d09de08a146084575b6002565b346002576000805460e060020a60243560043563ffffffff8416010181020463ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b34600257605c6000805460e060020a63ffffffff821660010181020463ffffffff1990911617905556"
	      ],
    "types": [
	      "Accumulator"
	      ]
  }
}
```

#### <a name="contract_deployContract">contract_deployContract</a>

Returns a transaction hash after deploying contract.

##### Parameters

1. `<Object>`
- `from`: `<string>`, 20 Bytes - Address of the creator.
- `nonce`: `<number>` - 16-bit ramdom value.
- `extra`: `<string>` - [optional] Extra information of this transaction.
- `timestamp`: `<number>` - The unix timestamp for when the transaction was generated.
- `payload`: `<string>` - For **solidity** contract, if the constructor of contract need parameters, `payload` value is the complied solidity code and encoded parameters. For **java** contract, `payload` value is  the byte stream after the class file and the configuration file are compressed.
- `signature`: `<string>` - The signature of transaction.
- `type`: `<string>` - [optional, default `"EVM"`] The execution engine type used by this transaction execution, this value is `EVM` OR `JVM`.

##### Returns

1. `<string>`, 32 Bytes - Hash of the transaction.

##### Example

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global",  "method":"contract_deployContract", "params":[{
"from":"0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
"nonce":5373500328656597,	"payload":"0x60606040526000805463ffffffff1916815560ae908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d14605e578063d09de08a146084575b6002565b346002576000805460e060020a60243560043563ffffffff8416010181020463ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b34600257605c6000805460e060020a63ffffffff821660010181020463ffffffff1990911617905556",
"signature":"0x388ad7cb71b1281eb5a0746fa8fe6fda006bd28571cbe69947ff0115ff8f3cd00bdf2f45748e0068e49803428999280dc69a71cc95a2305bd2abf813574bcea900",
"timestamp":1487771157166000000
}],"id":"1"}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "0x406f89cb205e136411fd7f5befbf8383bbfdec5f6e8bcfe50b16dcff037d1d8a"
}
```

#### <a name="contract_invokeContract">contract_invokeContract</a>

Returns a transaction hash after invoking contract.

##### Parameters

1. `<Object>`
- `from`: `<string>`, 20 Bytes - Address of the account invoked the contract.
- `to`: `<string>`, 20 Bytes - Address of the contract. You can get contract address through [tx_getTransactionReceipt](#tx_getTransactionReceipt) after deploying contract.
- `nonce`: `<number>` - 16-bit random value.
- `extra`: `<string>` - [optional] Extra information of this transaction.
- `timestamp`: `<number>` - The unix timestamp for when the transaction was generated.
- `payload`: `<string>` - The hash of the invoked method signature and encoded parameters.
- `signature`: `<string>` - The signature of transaction.
- `simulate`: `<boolean>` - [optional, default `false`] Determines whether the transaction requires consensus or not, if true, no consensus.
- `type`: `<string>` - [optional, default `"EVM"`] The execution engine type used by this transaction execution, this value is `EVM` OR `JVM`.

##### Returns

1. `<string>`, 32 Bytes - Hash of the transaction.

##### Example

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_invokeContract", "params":[{
"from":"0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
"nonce":5019420501875693,
"payload":"0x3ad14af300000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000002",
"signature":"0xde467ec4c0bd9033bdc3b6faa43a8d3c5dcf393ed9f34ec1c1310b0859a0ecba15c5be4480a9ad2aaaea8416324cb54e769856775dd5407f9fd64f0467331c9301",
"simulate":false,
"timestamp":1487773188814000000,
"to":"0x313bbf563991dc4c1be9d98a058a26108adfcf81"
}],"id":"1"}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":0,
  "message":"SUCCESS",
  "result":"0xd7a07fbc8ea43ace5c36c14b375ea1e1bc216366b09a6a3b08ed098995c08fde"
}
```

#### <a name="contract_getCode">contract_getCode</a>

Returns the compiled contract code by the given contract address.

##### Parameters

1. `<string>` - The address of contract.

##### Returns

1. `<string>` - contract code.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"contract_getCode","params": ["0xaeccd2fd1118334402c5de1cb014a9c192c498df"],"id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "0x606060405236156100565760e060020a600035046301000dd7811461005b5780638e739461146100e55780638f24d79614610107578063ae9f75e314610191578063b30cd67c1461021e578063e01da11e14610289575b610000565b346100005761006e6004356024356102e0565b604051808315158152602001806020018281038252838181518152602001915080519060200190808383829060006004602084601f0104600302600f01f150905090810190601f1680156100d65780820380516001836020036101000a031916815260200191505b50935050505060405180910390f35b34610000576100f56004356103f1565b60408051918252519081900360200190f35b346100005761006e600435602435610409565b604051808315158152602001806020018281038252838181518152602001915080519060200190808383829060006004602084601f0104600302600f01f150905090810190601f1680156100d65780820380516001836020036101000a031916815260200191505b50935050505060405180910390f35b346100005761006e6004356024356044356104b7565b604051808315158152602001806020018281038252838181518152602001915080519060200190808383829060006004602084601f0104600302600f01f150905090810190601f1680156100d65780820380516001836020036101000a031916815260200191505b50935050505060405180910390f35b346100005761026e6004808035906020019082018035906020019080806020026020016040519081016040528093929190818152602001838360200280828437509496506105cf95505050505050565b60408051921515835260208301919091528051918290030190f35b3461000057610296610683565b60405180806020018281038252838181518152602001915080519060200190602002808383829060006004602084601f0104600302600f01f1509050019250505060405180910390f35b6040805160208181018352600080835285815290819052918220541561033e57505060408051808201909152601781527f75736572206973206578697374656420616c726561647900000000000000000060208201526000906103ea565b60018054806001018281815481835581811511610380576000838152602090206103809181019083015b8082111561037c5760008155600101610368565b5090565b5b505050916000526020600020900160005b508590555050506000828152602081815260409182902084815560019081018490558251808401909352601083527f6e6577207573657220737563636573730000000000000000000000000000000091830191909152905b9250929050565b6000818152602081905260409020600101545b919050565b60408051602081810183526000808352858152908190529182208054151561046a5760408051808201909152601181527f75736572206973206e6f7420657869737400000000000000000000000000000060208201526000935091506104af565b600180820180548601905560408051808201909152601381527f7365742062616c616e6365207375636365737300000000000000000000000000602082015290935091505b509250929050565b6040805160208181018352600080835286815290819052828120858252928120835491939115806104e757508054155b1561052b5760408051808201909152601181527f75736572206973206e6f7420657869737400000000000000000000000000000060208201526000945092506105c5565b84826001015410156105765760408051808201909152601481527f616d6f756e74206973206e6f7420656e6f75676800000000000000000000000060208201526000945092506105c5565b60018083018054879003905581810180548701905560408051808201909152601081527f7472616e73666572207375636365737300000000000000000000000000000000602082015290945092505b5050935093915050565b8051600180548282556000828152928392917fb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf69081019190602087018215610633579160200282015b82811115610633578251825591602001919060010190610618565b5b506106549291505b8082111561037c5760008155600101610368565b5090565b505060017f7365742055736572496473207375636365737300000000000000000000000000915091505b915091565b6040805160208181018352600082526001805484518184028101840190955280855292939290918301828280156106d957602002820191906000526020600020905b8154815260200190600101908083116106c5575b505050505090505b9056"
} 
```

#### <a name="contract_getContractCountByAddr">contract_getContractCountByAddr</a>

Returns the number of contract that has been deployed by given account address.

##### Parameters

1. `<string>`, 20 Bytes - The address of account.

##### Returns

1. `<string>` - The number of contract.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "contract_getContractCountByAddr", "params": ["0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b"], "id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "0x3"
} 
```

#### <a name="contarct_maintainContract">contarct_maintainContract</a>

Upgrade contract, freeze contract and unfreeze contract.

*Note:* Only contract deployers have the authority to upgrade contract, freeze contract and unfreeze contract.

##### Parameters

1. `<Object>`
- `from`: `<string>`, 20 Bytes - Address of the account.
- `to`: `<string>`, 20 Bytes - Address of the contract. You can get contract address through [tx_getTransactionReceipt](#tx_getTransactionReceipt) after deploying contract.
- `nonce`: `<number>` - 16-bit random value.
- `extra`: `<string>` - [optional] Extra information of this transaction.
- `timestamp`: `<number>` - The unix timestamp for when the transaction was generated.
- `payload`: `<string>` - [optional] *Only upgrade contract operation need to specify this member.* Represents the new contract compiled code.
- `signature`: `<string>` - The signature of transaction.
- `type`: `<string>` - [optional, default `"EVM"`] The execution engine type used by this transaction execution, this value is `EVM` OR `JVM`.
- `opcode`: This value may be:
  - `1`: Upgrade contract.
  - `2`: Freeze contract.
  - `3`: Unfreeze contract.

##### Returns

1. `<string>`, 32 Bytes - Hash of the transaction.

##### Example1: Upgrade contract

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_maintainContract","params":[{
"from": "17d806c92fa941b4b7a8ffffc58fa2f297a3bffc", 
"to":"0x3a3cae27d1b9fa931458b5b2a5247c5d67c75d61", 
"timestamp":1481767474717000000, 
"nonce": 8054165127693853,
"payload":"0x6fd7cc16000000000000000000000000000000000000000000000000000000000000007b00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000", 
"signature":"0x19c0655d05b9c24f5567846528b81a25c48458a05f69f05cf8d6c46894b9f12a02af471031ba11f155e41adf42fca639b67fb7148ddec90e7628ec8af60c872c00", 
"opcode": 1}],"id": 1}'


# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":0,
  "message":"SUCCESS",
  "result":"0xd7a07fbc8ea43ace5c36c14b375ea1e1bc216366b09a6a3b08ed098995c08fde"
}
```

##### Example2: Freeze contract

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_maintainContract","params":[{
	"from": "17d806c92fa941b4b7a8ffffc58fa2f297a3bffc", 
	"to":"0x3a3cae27d1b9fa931458b5b2a5247c5d67c75d61", 
	"timestamp":1481767474717000000, 
	"nonce": 8054165127693853,
	"signature":"0x19c0655d05b9c24f5567846528b81a25c48458a05f69f05cf8d6c46894b9f12a02af471031ba11f155e41adf42fca639b67fb7148ddec90e7628ec8af60c872c00", 
	"opcode": 2}],
"id": 1}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":0,
  "message":"SUCCESS",
  "result":"0xd7a07fbc8ea43ace5c36c14b375ea1e1bc216366b09a6a3b08ed098995c08fde"
}
```

##### Example3: Unfreeze contract

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_maintainContract","params":[{
"from": "17d806c92fa941b4b7a8ffffc58fa2f297a3bffc", 
"to":"0x3a3cae27d1b9fa931458b5b2a5247c5d67c75d61", 
"timestamp":1481767474717000000, 
"nonce": 8054165127693853,
"signature":"0x19c0655d05b9c24f5567846528b81a25c48458a05f69f05cf8d6c46894b9f12a02af471031ba11f155e41adf42fca639b67fb7148ddec90e7628ec8af60c872c00", 
"opcode": 3}],"id": 1}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":0,
  "message":"SUCCESS",
  "result":"0xd7a07fbc8ea43ace5c36c14b375ea1e1bc216366b09a6a3b08ed098995c08fde"
}
```

#### <a name="contract_getStatus">contract_getStatus</a>

Returns status of a contract by contract address.

##### Parameters

1. `<string>`, 20 Bytes - The address of contract.

##### Returns

1. `<string>` - Status of the contract, this value may be:
   - `normal`: Normal status.
   - `frozen`: The contract has been frozen.
   - `non-contract`: The given address is not a contract address. It may be a normal account address.

##### Example

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_getStatus","params": ["0xbbe2b6412ccf633222374de8958f2acc76cda9c9"],"id": 1}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":0,
  "message":"SUCCESS",
  "result":" normal"
}
```

#### <a name="contract_getCreator">contract_getCreator</a>

Returns the address of contract creator.

##### Parameters

1. `<string>`, 20 Bytes - The address of contract.

##### Returns

1. `<string>`, 20 Bytes - The address of contract creator.

##### Example

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_getCreator","params": ["0xbbe2b6412ccf633222374de8958f2acc76cda9c9"],"id": 1}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":0,
  "message":"SUCCESS",
  "result":" 0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd "
}
```

#### <a name="contract_getCreateTime">contract_getCreateTime</a>

Returns the date and time a contract was created.

##### Parameters

1. `<string>`, 20 Bytes - The address of contract.

##### Returns

1. `<string>` - The date and time of contract created.

##### Example

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_getCreateTime","params": ["0xbbe2b6412ccf633222374de8958f2acc76cda9c9"],"id": 1}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":0,
  "message":"SUCCESS",
  "result":"2017-04-07 12:37:06.152111325 +0800 CST"
}
```

#### <a name="contract_getDeployedList">contract_getDeployedList</a>

Returns a list of deployed contract address by account address.

##### Parameters

1. `<string>`, 20 Bytes - The address of contract creator.

##### Returns

1. `[<string>]` - a list of deployed contract address.

##### Example

```bash
# Request
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_getDeployedList","params": ["0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd"],"id": 1}'

# Response
{
  "jsonrpc":"2.0",
  "namespace":"global",
  "id":1,
  "code":0,
  "message":"SUCCESS",
  "result":["0xbbe2b6412ccf633222374de8958f2acc76cda9c9"]
}
```

### Block

#### <a name="block_latestBlock">block_latestBlock</a>

Returns information about the latest block.

##### Parameters

none

##### Returns<a name="block"></a>

1. `<Block>` - The Block object has the following members: 
- `version`: `<string>` - Platform version number.
- `number`: `<string>` - The block number.
- `hash`: `<string>`, 32 Bytes - Hash of the block.
- `parentHash`: `<string>`, 32 Bytes - Hash of the parent block.
- `writeTime`: `<number>` - The unix timestamp for when the block was written.
- `avgTime`: `<string>` - The average time it takes to execute transactions in the block (ms).
- `txCounts`: `<string>` - The number of transactions in the block.
- `merkleRoot`: `<string>` - Merkle tree root hash.
- `transactions`: `[<Transaction>]` - The list of transactions in the block. The Transaction object see  [Valid Transaction](#validTransaction).

##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":" block_latestBlock","params":[],"id":71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.0",
    "number": "0x3",
    "hash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
    "parentHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
    "writeTime": 1481778653997475900,
    "avgTime": "0x2",
    "txcounts": "0x1",
    "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c",
    "transactions": [
      {
        "version": "1.0",
	"hash": "0xf57a6443d08cda4a3dfb8083804b6334d17d7af51c94a5f98ed67179b59169ae",
	"blockNumber": "0x3",
	"blockHash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
	"txIndex": "0x0",
	"from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
	"to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
	"amount": "0x0",
	"timestamp": 1481778652973000000,
	"nonce": 3573634504790373,
	"extra": "",
	"executeTime": "0x2",
	"payload": "0x81053a70000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000001c8"
      }
    ]
  }
}
```

##### Example2: There is no block on the chain

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":" block_latestBlock","params":[],"id":71}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": -32602,
  "message": "There is no block generated!"
}
```

#### <a name="block_getBlocks">block_getBlocks</a>

Returns a list of blocks from start block number to end block number.

##### Parameters

1. `<Object>`
- `from`: `<blockNumber>` - Start block number. See type [Block Number](#blockNumber).
- `to`: `<blockNumber>` - End block number. See type [Block Number](#blockNumber).
- `isPlain`: `<boolean>` - [optional, default `false`] If `true` it returns block excluding transactions, if `false` it returns block including transactions.

 `from` must be less than or equal `to`, otherwise returns error.

##### Returns

1. `[<Block>]` - array of Block, the members of Block object see [Block](#block).

##### Example1: Returns block including transactions

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "block_getBlocks", "params": [{"from":2,"to":3}], "id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.0",
      "number": "0x3",
      "hash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
      "parentHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
      "writeTime": 1481778653997475900,
      "avgTime": "0x2",
      "txcounts": "0x1",
      "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c",
      "transactions": [
        {
	  "version": "1.0",
	  "hash": "0xf57a6443d08cda4a3dfb8083804b6334d17d7af51c94a5f98ed67179b59169ae",
	  "blockNumber": "0x3",
	  "blockHash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
	  "txIndex": "0x0",
	  "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
	  "to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
	  "amount": "0x0",
	  "timestamp": 1481778652973000000,
	  "nonce": 3573634504790373,
	  "extra": "",
	  "executeTime": "0x2",
	  "payload": "0x81053a70000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000001c8"
	}
      ]
    },
    {
      "version": "1.0",
      "number": "0x2",
      "hash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
      "parentHash": "0xe287c62aae77462aa772bd68da9f1a1ba21a0d044e2cc47f742409c20643e50c",
      "writeTime": 1481778642328872960,
      "avgTime": "0x2",
      "txcounts": "0x1",
      "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c",
      "transactions": [
        {
	  "version": "1.0",
	  "hash": "0x07d606a25d1eab009f5374950383e9c0697599e6c35999337b969ba356800168",
	  "blockNumber": "0x2",
	  "blockHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
	  "txIndex": "0x0",
	  "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
	  "to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
	  "amount": "0x0",
	  "timestamp": 1481778641306000000,
	  "nonce": 1628827117185765,
	  "extra": "",
	  "executeTime": "0x2",
	  "payload": "0x6fd7cc16000000000000000000000000000000000000000000000000000000000000303a00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
	}
      ]
    }
  ]
}
```

##### Example2: Returns block excluding transactions

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "block_getBlocks", "params": [{"from":2,"to":3,"isPlain":true}], "id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.0",
      "number": "0x3",
      "hash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
      "parentHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
      "writeTime": 1481778653997475900,
      "avgTime": "0x2",
      "txcounts": "0x1",
      "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c"
    },
    {
      "version": "1.0",
      "number": "0x2",
      "hash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
      "parentHash": "0xe287c62aae77462aa772bd68da9f1a1ba21a0d044e2cc47f742409c20643e50c",
      "writeTime": 1481778642328872960,
      "avgTime": "0x2",
      "txcounts": "0x1",
      "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c"
    }
  ]
}
```

#### <a name="block_getBlockByHash">block_getBlockByHash</a>

Returns information about a block by hash.

##### Parameters

1. `<string>`, 32 Bytes - Hash of a block.
2. `<boolean>` - If `true` it returns block excluding transactions, if `false` it returns block including transactions.

##### Returns

1. `<Block>` - the members of Block object see [Block](#block).

##### Example1: Returns block including transactions

```bash
# Request
curl -X POST –data  '{"jsonrpc":"2.0","namespace":"global","method":"block_getBlockByHash","params":["0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914", false],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.0"
    "number": "0x3",
    "hash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
    "parentHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
    "writeTime": 1481778653997475900,
    "avgTime": "0x2",
    "txcounts": "0x1",
    "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c",
    "transactions": [
      {
        "version": "1.0",
	"hash": "0xf57a6443d08cda4a3dfb8083804b6334d17d7af51c94a5f98ed67179b59169ae",
	"blockNumber": "0x3",
	"blockHash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
	"txIndex": "0x0",
	"from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
	"to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
	"amount": "0x0",
	"timestamp": 1481778652973000000,
	"nonce": 3573634504790373,
	"extra": "",
	"executeTime": "0x2",
	"payload": "0x81053a70000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000001c8"
      }
    ]
  }
}
```

##### Example2: Returns block excluding transactions

```bash
# Request
curl -X POST –data  '{"jsonrpc":"2.0","namespace":"global","method":"block_getBlockByHash","params":["0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914", true],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.0",
    "number": "0x3",
    "hash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
    "parentHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
    "writeTime": 1481778653997475900,
    "avgTime": "0x2",
    "txcounts": "0x1",
    "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c"
  }
}
```

##### <a name="block_getBlockByNumber">block_getBlockByNumber</a>

Returns information about a block by number.

##### Parameters

1. `<blockNumber>` - The block number. See type [Block Number](#blockNumber).
2. `<boolean>` - If `true` it returns block excluding transactions, if `false` it returns block including transactions.

##### Returns

1. `<Block>` - the members of Block object see [Block](#block).

##### Example1: Returns block including transactions

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "block_getBlockByNumber", "params": ["0x3", false], "id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.0",
    "number": "0x3",
    "hash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
    "parentHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
    "writeTime": 1481778653997475900,
    "avgTime": "0x2",
    "txcounts": "0x1",
    "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c",
    "transactions": [
      {
        "version": "1.0",
	"hash": "0xf57a6443d08cda4a3dfb8083804b6334d17d7af51c94a5f98ed67179b59169ae",
	"blockNumber": "0x3",
	"blockHash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
	"txIndex": "0x0",
	"from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
	"to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
	"amount": "0x0",
	"timestamp": 1481778652973000000,
	"nonce": 3573634504790373,
	"extra": "",
	"executeTime": "0x2",
	"payload": "0x81053a70000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000001c8"
      }
    ]
  }
}
```

##### Example1: Returns block excluding transactions

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "block_getBlockByNumber", "params": ["0x3", true], "id": 1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "version": "1.0",
    "number": "0x3",
    "hash": "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
    "parentHash": "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
    "writeTime": 1481778653997475900,
    "avgTime": "0x2",
    "txcounts": "0x1",
    "merkleRoot": "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c"
  }
}
```

#### <a name="block_getAvgGenerateTimeByBlockNumber">block_getAvgGenerateTimeByBlockNumber</a>

Returns the average generation time of all blocks for the given block number.

##### Parameters

1. `<Object>`
- `from`: `<blockNumber>` - Start block number. See type [Block Number](#blockNumber).
- `to`: `<blockNumber>` - End block number. See type [Block Number](#blockNumber).

##### Returns

1. `<string>` - the average generation time of all blocks (ms).

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":" block_getAvgGenerateTimeByBlockNumber","params": [{"from": 10, "to": 19}],"id":71}'

# Response
{
  "id":71,
  "jsonrpc": "2.0",
  "namespace":"global",
  "code": 0,
  "message": "SUCCESS",
  "result": "0x32"
}
```

#### <a name="block_getBlocksByTime">block_getBlocksByTime</a>

Returns the number of blocks generated at specific time periods.

##### Parameters

1. `<Object>`
- `startTime`: `<number>` - The start unix timestamp.
- `endTime`: `<number>` - The end unix timestamp.

##### Returns

1. `<Object>`
- `sumOfBlocks`: `<string>` - The number of blocks.
- `startBlock`: `<string>` - The start block number.
- `endBlock`: `<string>` - The end block number.

##### Example1: Normal request

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"block_getBlocksByTime","params":[{"startTime":1481778635567920177, "endTime":1481778653997475900}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "sumOfBlocks": "0x3",
    "startBlock": "0x1",
    "endBlock": "0x3"
  }
}
```

##### Example2: Start unix timestamp and end unix timestamp are both more than written unix timestamp of the latest block.

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"block_getBlocksByTime","params":[{"startTime":1481778635567920177, "endTime":1481778653997475900}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": {
    "sumOfBlocks": "0x0",
    "startBlock": null,
    "endBlock": null
  }
}
```

#### <a name="block_getGenesisBlock">block_getGenesisBlock</a>

Returns current genesis block number.

##### Parameters

none

##### Returns

1. `<string>` - The genesis block number.

##### Example

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0","method":"block_getGenesisBlock","params":[],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "0x8"
}
```

#### <a name="block_getChainHeight">block_getChainHeight</a>

Returns the current chain height.

##### Parameters

none

##### Returns

1. `<string>` - The latest block number.

##### Example

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0","method":"block_getChainHeight","params":[],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "0x11"
}
```

#### <a name="block_getBatchBlocksByHash">block_getBatchBlocksByHash</a>

Returns a list of blocks by a list of specific block hash.

##### Parameters

1. `<Object>`
- `hashes`: `[<string>]` - Array of block hash.
- `isPlain`: `<boolean>` - If `true` it returns block excluding transactions, if `false` it returns block including transactions.

##### Returns

1. `[<Block>]` - Array of Block object, the members of Block object see [Block](#block).

##### Example1: Returns block including transactions

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0","method":"block_getBatchBlocksByHash","params":[{
	"hashes":["0x810c92919fba632471b543905d8b4f8567c4fac27e5929d2eca8558c68cb7cf0","0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b"]
}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.3",
      "number": "0x3",
      "hash": "0x810c92919fba632471b543905d8b4f8567c4fac27e5929d2eca8558c68cb7cf0",
      "parentHash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
      "writeTime": 1509448178829111592,
      "avgTime": "0x0",
      "txcounts": "0x0",
      "merkleRoot": "0x97b0d9473478886f5b0aee123d5652b15d4ae3ab41cc487cda9d8885cb003481"
    },
    {
      "version": "1.3",
      "number": "0x2",
      "hash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
      "parentHash": "0x4cd9f393aabb2df51c09e66925c4513e23f0dbbb9e94d0351c1c3ec7539144a0",
      "writeTime": 1509440823930976319,
      "avgTime": "0x6",
      "txcounts": "0x1",
      "merkleRoot": "0x97b0d9473478886f5b0aee123d5652b15d4ae3ab41cc487cda9d8885cb003481",
      "transactions": [
        {
	  "version": "1.3",
	  "hash": "0x22321358931c577ceaa2088d914758148dc6c1b6096a0b3f565d130f03ca75e4",
          "blockNumber": "0x2",
          "blockHash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
          "txIndex": "0x0",
          "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
          "to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
          "amount": "0x0",
          "timestamp": 1509440823410000000,
          "nonce": 8291834415403909,
          "extra": "",
          "executeTime": "0x6",
          "payload": "0x0a9ae69d"
        }
      ]
    }
  ]
}
```

##### Example2: Returns block excluding transactions

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0","method":"block_getBatchBlocksByHash","params":[{
	"hashes":["0x810c92919fba632471b543905d8b4f8567c4fac27e5929d2eca8558c68cb7cf0","0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b"],
	"isPlain": true
}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.3",
      "number": "0x3",
      "hash": "0x810c92919fba632471b543905d8b4f8567c4fac27e5929d2eca8558c68cb7cf0",
      "parentHash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
      "writeTime": 1509448178829111592,
      "avgTime": "0x0",
      "txcounts": "0x0",
      "merkleRoot": "0x97b0d9473478886f5b0aee123d5652b15d4ae3ab41cc487cda9d8885cb003481"
    },
    {
      "version": "1.3",
      "number": "0x2",
      "hash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
      "parentHash": "0x4cd9f393aabb2df51c09e66925c4513e23f0dbbb9e94d0351c1c3ec7539144a0",
      "writeTime": 1509440823930976319,
      "avgTime": "0x6",
      "txcounts": "0x1",
      "merkleRoot": "0x97b0d9473478886f5b0aee123d5652b15d4ae3ab41cc487cda9d8885cb003481"
    }
  ]
}
```

#### <a name="block_getBatchBlocksByNumber">block_getBatchBlocksByNumber</a>

Returns a list of blocks by a list of specific block number.

##### Parameters

1. `<Object>`
- `numbers`: `[<blockNumber>]` - Array of block number. See type [Block Number](#blockNumber).
- `isPlain`: `<boolean>` - If `true` it returns block excluding transactions, if `false` it returns block including transactions.

##### Returns

1. `[<Block>]` -  Array of Block object, the members of Block object see [Block](#block).

##### Example1: Returns block including transactions

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0","method":"block_getBatchBlocksByNumber","params":[{
	"numbers": ["1","2"]
}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.3",
      "number": "0x1",
      "hash": "0x4cd9f393aabb2df51c09e66925c4513e23f0dbbb9e94d0351c1c3ec7539144a0",
      "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
      "writeTime": 1509440821032039312,
      "avgTime": "0x11",
      "txcounts": "0x1",
      "merkleRoot": "0x97b0d9473478886f5b0aee123d5652b15d4ae3ab41cc487cda9d8885cb003481",
      "transactions": [
        {
          "version": "1.3",
          "hash": "0x7aebde51531bb29d3ba620f91f6e1556a1e8b50913e590f31d4fe4a2436c0602",
          "blockNumber": "0x1",
          "blockHash": "0x4cd9f393aabb2df51c09e66925c4513e23f0dbbb9e94d0351c1c3ec7539144a0",
          "txIndex": "0x0",
          "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
          "to": "0x0000000000000000000000000000000000000000",
          "amount": "0x0",
          "timestamp": 1509440820498000000,
          "nonce": 5098902950712745,
          "extra": "",
          "executeTime": "0x11",
          "payload": "0x6060604052341561000f57600080fd5b60405160408061016083398101604052808051919060200180519150505b6000805467ffffffff000000001963ffffffff19821663ffffffff600393840b8701840b81169190911791821664010000000092839004840b860190930b16021790555b50505b60de806100826000396000f300606060405263ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416630a9ae69d811460465780638466c3e614606f575b600080fd5b3415605057600080fd5b60566098565b604051600391820b90910b815260200160405180910390f35b3415607957600080fd5b605660a9565b604051600391820b90910b815260200160405180910390f35b600054640100000000900460030b81565b60005460030b815600a165627a7a7230582073eeeb74bb45b3055f1abe89f428d164ef7425bf57a999d219cbaefb6e3c0080002900000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000005"
         }
      ]
    },
    {
      "version": "1.3",
      "number": "0x2",
      "hash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
      "parentHash": "0x4cd9f393aabb2df51c09e66925c4513e23f0dbbb9e94d0351c1c3ec7539144a0",
      "writeTime": 1509440823930976319,
      "avgTime": "0x6",
      "txcounts": "0x1",
      "merkleRoot": "0x97b0d9473478886f5b0aee123d5652b15d4ae3ab41cc487cda9d8885cb003481",
      "transactions": [
        {
          "version": "1.3",
          "hash": "0x22321358931c577ceaa2088d914758148dc6c1b6096a0b3f565d130f03ca75e4",
          "blockNumber": "0x2",
          "blockHash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
          "txIndex": "0x0",
          "from": "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
          "to": "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
          "amount": "0x0",
          "timestamp": 1509440823410000000,
          "nonce": 8291834415403909,
          "extra": "",
	  "executeTime": "0x6",
	  "payload": "0x0a9ae69d"
        }
      ]
    }
  ]
}
```

##### Example2: Returns block excluding transactions

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0","method":"block_getBatchBlocksByNumber","params":[{
	"numbers": ["1","2"],
	"isPlain": true
}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "version": "1.3",
      "number": "0x1",
      "hash": "0x4cd9f393aabb2df51c09e66925c4513e23f0dbbb9e94d0351c1c3ec7539144a0",
      "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
      "writeTime": 1509440821032039312,
      "avgTime": "0x11",
      "txcounts": "0x1",
      "merkleRoot": "0x97b0d9473478886f5b0aee123d5652b15d4ae3ab41cc487cda9d8885cb003481"
    },
    {
      "version": "1.3",
      "number": "0x2",
      "hash": "0x9c41efcc50ec6af6e3d14e1669f37bd1fc0cfe5836af6ab1e43ced98653c938b",
      "parentHash": "0x4cd9f393aabb2df51c09e66925c4513e23f0dbbb9e94d0351c1c3ec7539144a0",
      "writeTime": 1509440823930976319,
      "avgTime": "0x6",
      "txcounts": "0x1",
      "merkleRoot": "0x97b0d9473478886f5b0aee123d5652b15d4ae3ab41cc487cda9d8885cb003481"
    }
  ]
}
```

### Subscription

#### <a name="sub_newBlockSubscription">sub_newBlockSubscription</a>

To subscribe  a new block event and create filter to notify client. The information of new block will be cached in filter when a new block is generated.

##### Parameters

1. `<boolean>` - If `true`,  the filter will return the full object [Block](#block) , if `false` it will return block hash. 

##### Returns

1. `<string>` - Subscription ID.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"sub_newBlockSubscription","params":[false],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result":"0x7e533eb0647ecbe473ae610ebdd1bba6"
}
```

#### <a name="sub_newEventSubscription">sub_newEventSubscription</a>

To subscribe a new VM event and create filter to notify client. The VM event log will be cached when a VM event is triggered.

##### Parameters

1. `<Object>`

- `fromBlock`: `<number>` - [optional, default no limit] Start block number. This block number should be more than or equal to current latest block number.
- `toBlock`: `<number>` - [optional, default no limit] End block number. This block number is the future block number that is more than the start block number.
- `addresses`: `[<string>]` - [optional, default no limit] Array of 20 Bytes string, indicates that listen to event generated by specified address of contract.
- `topics`: `[<string>][<string>]` - [optional, default no limit] two-dimensional array of string, topics which the incoming message's topics should match. You can use the following combinations:
  - `[A, B] = A && B`
  - `[A, [B, C]] = A && (B || C)`
  - `[null, A, B] = ANYTHING && A && B` `null` works as a wildcard

##### Returns

1. `<string>` - Subscription ID.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"sub_newEventSubscription","params":[{
	"fromBlock":100,  
	"addresses": ["000f1a7a08ccc48e5d30f80850cf1cf283aa3abd"]
}],
"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result":"0x7e533eb0647ecbe473ae610ebdd1bba6"
}
```

#### <a name="sub_getLogs">sub_getLogs</a>

Returns the eligible VM event log.

##### Parameters

1. `<Object>`

- `fromBlock`：`<number>` - [optional, default `0`] Start block number. This block number shouldn't be less than the genesis block number.
- `toBlock`：`<number>` - [optional, default `the latest block number`] End block number. This block number shouldn't be more than current latest block number.
- `addresses`：`[<string>]` - [optional, default no limit]  Array of 20 Bytes string, indicates that listen to event generated by specified address of contract.
- `topics`：`[<string>][<string>]` - [optional, default no limit] two-dimensional array of string, topics which the incoming message's topics should match. You can use the following combinations:
  - `[A, B] = A && B`
  - `[A, [B, C]] = A && (B || C)`
  - `[null, A, B] = ANYTHING && A && B` `null` works as a wildcard

##### Returns

1. `[<Log>]` -  the Log object has the following members:
   - `address`: `<string>`, 20 Bytes - Contract address by which this event log is generated.
   - `topics`: `[<string>]` - Array of 32 Bytes string. Topics are order-dependent. Each topic can also be an array of string with "or" options. The first topic is the unique identity of the event.
   - `data`: `<string>` - Log message or data.
   - `blockNumber`: `<number>` - Block number where this transaction was in.
   - `blockHash`: `<string>`, 32 Bytes - Hash of the block where this transaction was in.
   - `txHash`: `<string>`, 32 Bytes - Hash of the transaction where this log was in.
   - `txIndex`: `<number>` - Transaction index position in the block.
   - `index`: `<number>` - Event log index position in all logs generated in this transaction.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"sub_getLogs","params":[{
	"addresses": ["0x313bbf563991dc4c1be9d98a058a26108adfcf81"]
}],
"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result":[
    {
      "address":"0x313bbf563991dc4c1be9d98a058a26108adfcf81",
      "topics":["0x24abdb5865df5079dcc5ac590ff6f01d5c16edbc5fab4e195d9febd1114503da"],
      "data":"0000000000000000000000000000000000000000000000000000000000000064",
      "blockNumber":4,
      "blockHash":"0xee93a66e170f2b20689cc05df27e290613da411c42a7bdfa951481c08fdefb16",
      "txHash":"0xa676673a23f33a95a1a5960849ad780c5048dff76df961e9f78329b201670ae2",
      "txIndex":0,
      "index":0
    }
  ]
}
```

#### <a name="sub_newSystemStatusSubscription">sub_newSystemStatusSubscription</a>

To subscribe a new system status event and create filter to notify client. The information of system status will be cached when a new system status event is thrown.

##### Parameters

1. `<Object>`

- `modules`: `[<string>]` - [optional, default no limit] Array of string, modules of system status which the client wants to subscribe. For example, **p2p**, **consensus**, **executor** and so on.
- `modules_exclude`: `[<string>]` - [optional, default no limit] Array of string, modules of system status which the client doesn't want to subscribe.
- `subtypes`: `[<string>]` - [optional, default no limit] Array of string, which type of system status information under the module the client wants to subscribe. For example, **viewchange**.
- `subtypes_exclude`: `[<string>]` - [optional, default no limit] Array of string, which type of system status information under the module the client doesn't want to subscribe.
- `error_codes`: `[<number>]` - [optional, default no limit] Array of number, specific status information under the module the client wants to subscribe.
- `error_codes_exclude`: `[<number>]` - [optional, default no limit] Array of number, specific status information under the module the client doesn't want to subscribe. 

##### Returns

1. `<string>` - Subscription ID.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"sub_ newSystemStatusSubscription","params":[{
	"modules":["executor", "consensus"],  
	"subtypes": ["viewchange"], 
	"error_codes_exclude": [-1, -2]
}],
"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result":"0x7e533eb0647ecbe473ae610ebdd1bba6"
}
```

#### <a name="sub_newArchiveSubscription">sub_newArchiveSubscription</a>

To subscribe a data archiving event and create filter to notify client. The information of archiving will be cached when a new data archiving event is triggered.

##### Parameters

none

##### Returns

1. `<string>` - Subscription ID.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"sub_ newArchiveSubscription","params":[], "id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result":"0x7e533eb0647ecbe473ae610ebdd1bba6"
}
```

#### <a name="sub_getSubscriptionChanges">sub_getSubscriptionChanges</a>

Polling method for filters. Returns new messages since the last call of this method.

##### Parameters

1. `<string>` - Subscription ID.

##### Returns

1. `<Array>` - Array of messages received since last poll.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"sub_getSubscriptionChanges","params":[“0x7e533eb0647ecbe473ae610ebdd1bba6”], "id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result":{}
}
```

#### <a name="sub_unSubscription">sub_unSubscription</a>

To unsubscribe a subscription with given id. This method should be called when watch is no longer needed.

##### Parameters

1. `<string>` - Subscription ID.

##### Returns

1. `<boolean>` - `true` if the subscription was successfully unsubscribe, otherwise `false`.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"sub_ unsubscription","params":[“0x7e533eb0647ecbe473ae610ebdd1bba6”], "id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result":true,
}
```

### Node

#### <a name="node_getNodes">node_getNodes</a>

Returns information of all nodes.

##### Parameters

none

##### Returns

1. `[<PeerInfo>]` - Array of PeerInfo object, the PeerInfo object has following members:
- `id`: `<number>` - The node ID. 
- `ip`: `<string>` - IP address of the node.
- `port`: `<number>` - GRPC port of the node.
- `namespace`: `<string>` - Which namespace the node is in.
- `hash`: `<string>` - Hash of the node.
- `hostname`: `<string>` - Hostname of the node.
- `isPrimary`: `<bool>` - If `true` represents the node is a primary node, otherwise not.
- `isvp`: `<bool>` - If `true` represents the node is a VP node, otherwise not.
- `status`: `<number>` - Status of the node.  This value may be:
  - `0`: Alive status.
  - `1`: Pending status. 
  - `2`: Stop status.
- `delay`: `<number>` - Represents the delay time(ns) between this node and requested node. If this value is `0`, it represents this PeerInfo is the information of local node.

##### Example

```bash
# Request
curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "node_getNodes", "params": [],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": [
    {
      "id": 1,
      "ip": "127.0.0.1",
      "port": "50011",
      "namespace": "global",
      "hash": "fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a",
      "hostname": "node1",
      "isPrimary": true,
      "isvp": true,
      "status": 0,
      "delay": 0
    },
    {
      "id": 2,
      "ip": "127.0.0.1",
      "port": "50012",
      "namespace": "global",
      "hash": "c82a71a88c58540c62fc119e78306e7fdbe114d9b840c47ab564767cb1c706e2",
      "hostname": "node2",
      "isPrimary": false,
      "isvp": true,
      "status": 0,
      "delay": 347529
    },
    {
      "id": 3,
      "ip": "127.0.0.1",
      "port": "50013",
      "namespace": "global",
      "hash": "0c89dc7d8bdf45d1fed89fdbac27463d9f144875d3d73795f64f35dc204480fd",
      "hostname": "node3",
      "isPrimary": false,
      "isvp": true,
      "status": 0,
      "delay": 369554
    },
    {
      "id": 4,
      "ip": "127.0.0.1",
      "port": "50014",
      "namespace": "global",
      "hash": "34d299742260716bab353995fe98727004b5c27bde52489f61de093176e82088",
      "hostname": "node4",
      "isPrimary": false,
      "isvp": true,
      "status": 0,
      "delay": 430356
    }
  ]
}
```

#### <a name="node_getNodeHash">node_getNodeHash</a>

Return hash of the requested node.

##### Parameters

none

##### Returns

1. `<string>` - hash of the node.

##### Example

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0", "namespace":"global", "method":"node_getNodeHash","params":[],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "c605d50c3ed56902ec31492ed43b238b36526df5d2fd6153c1858051b6635f6e"
}
```

#### <a name="node_deleteVP">node_deleteVP</a>

To disconnect a connected VP peer.

##### Parameters

1. `<Object>`
- `nodehash`: `<string>` - Hash of the VP peer to disconnect.

##### Returns

1. `<string>` - A message indicates whether the request is sent successfully or not.

##### Example

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0", "namespace":"global", "method":"node_deleteVP","params":[{"nodehash":"c605d50c3ed56902ec31492ed43b238b36526df5d2fd6153c1858051b6635f6e"}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "successful request to delete vp node, hash c82a71a88c58540c62fc119e78306e7fdbe114d9b840c47ab564767cb1c706e2"
}
```

#### <a name="node_deleteNVP">node_deleteNVP</a>

VP node disconnects to connected NVP peer by hash of NVP peer.

##### Parameters

1. `<Object>`
- `nodehash`: `<string>` - Hash of NVP peer to disconnect.

##### Returns

1. `<string>` - A message indicates whether the request is sent successfully or not.

##### Example

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0","namespace":"global", "method":"node_deleteNVP","params":[{"nodehash":"c605d50c3ed56902ec31492ed43b238b36526df5d2fd6153c1858051b6635f6e"}],"id":1}'

# Response
{
  "jsonrpc": "2.0",
  "namespace":"global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "successful request to delete nvp node, hash 050ba6f3a19a4aca46adf51f0d46cf822b0b97f50014207a2b8d8535f5da7aa8"
}
```

### Certificate

#### <a name="cert_getTCert">cert_getTCert</a>

Returns the tcert certificate issued by the node to the client.

##### Parameters

1. `<Object>`
- `pubkey`: `<string>` - Public key(pem format).

##### Returns

1. `<Object>`
- `tcert`: `<string>` - tcert certificate.

##### Example1: Getting tcert certificate failed

```bash
# Request
curl -X POST --data ' {"jsonrpc":"2.0", "namespace":"global", "method":"cert_getTCert","params":[{
"pubkey":"2d2d2d2d2d424547494e204543445341205055424c4943204b45592d2d2d2d2d0a424a4b73413554414d2b5763446c79357250515a2b32595264574a664f446a62393658476a426b59367373352b346a67424f636834394e3064447744633877610a362b46434954436b7a584d4139436d392b436e68722b633d0a2d2d2d2d2d454e44204543445341205055424c4943204b45592d2d2d2d2d"}],"id":"1"}'

# Response
{
   "jsonrpc": "2.0",
   "namespace":"global",
   "id": "1",
   "code": -32099,
   "message": "signed tcert failed"
}
```
