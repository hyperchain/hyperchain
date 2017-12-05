JSON-RPC API
============

1. JSON-RPC概述
---------------

JSON-RPC是一个无状态且轻量级的远程过程调用(RPC)协议。它允许运行在基于socket、http等诸多不同消息传输环境的同一进程中，其使用JSON作为数据格式。发送一个请求对象至服务端代表一个RPC调用,一个请求对象包含下列成员:

-  ``jsonrpc``: 指定JSON-RPC协议版本的字符串,如果是2.0版本,则必须准确写为 “2.0”。
-  ``method``: 表示所要调用方法名称的字符串。以RPC开头的方法名，用英文句号(U+002E or ASCII 46)连接的为预留给RPC内部的方法名及扩展名,且不能在其他地方使用。
-  ``params``: 调用方法所需要的结构化参数值,该成员参数可以被省略。
-  ``id``: 已建立客户端的唯一标识id，该值必须包含一个字符串、数值或NULL值。如果不包含该成员则被认定为是一次通知调用。该值一般不为NULL,若为数值则应为整数。

当发起一次rpc调用时，服务端都必须回复一个JSON对象作为响应,响应对象包含下列成员:

-  ``jsonrpc``: 指定JSON-RPC协议版本的字符串，如果是2.0版本，则必须准确写为“2.0”。
-  ``result``: 该成员在成功时必须包含，当调用方法失败时必须不包含该成员。服务端中的被调用方法决定了该成员的值。
-  ``error``: 该成员在失败时必须包含，当没有错误引起时,不包含该成员。若引起错误,则该成员对象将包含code和message两个属性。
-  ``id``: 该成员必须包含。该成员值必须与请求对象中的id成员值一致。若在检查请求对象id时错误(例如参数错误或无效请求)，则该值必须为空值（NULL）。

2. 接口设计
-----------

Hyperchain接口主要由六块接口组成： 
1. 交易服务，方法名前缀为 ``"tx"``;
2. 合约服务，方法名前缀为 ``"contract"``;
3. 区块服务，方法名前缀为 ``"block"``;
4. 消息订阅服务，方法名前缀为 ``"sub"``;
5. 节点服务，方法名前缀为 ``"node"``;
6. 证书服务，方法名前缀为 ``"cert"``;

接口设计基于JSON-RPC 2.0规范。所有HTTP请求均为POST请求，请求的参数包括：

-  ``jsonrpc``: 指定JSON-RPC协议版本的字符串,如果是2.0版本，则必须准确写为 “2.0”。
-  ``namespace``: 表示该条请求发送给哪个分区去处理。
-  ``method``: 表示所要调用方法名称的字符串，格式为：(服务前缀)_(方法名)。
-  ``params``: 调用方法所需要的结构化参数值，该成员参数可以被省略。
-  ``id``: 已建立客户端的唯一标识id，该值必须包含一个字符串、数值。

.. code:: bash

    # Request
    curl -X POST -d '{"jsonrpc":"2.0","method":"block_latestBlock","namespace":"global","params":[],"id":1}' localhost:8081

返回值格式为：

-  ``jsonrpc``: 指定JSON-RPC协议版本的字符串，如果是2.0版本，则必须准确写为 “2.0”。
-  ``namespace``: 表示该条请求所属分区。
-  ``code``: 状态码。若成功，则为0，其他状态码详见表5-1。
-  ``message``: 错误信息。若成功，则为“SUCCESS”，否则为错误详细信息。
-  ``result``: 被调用方法成功执行返回的结果。
-  ``id``: 该值应与请求对象中的id值保持一致。

.. code:: bash

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

比如，接口调用成功的话，返回的字段有：jsonrpc、namespace、id、code、message、result，且code值为0，message值为“SUCCESS”，用户可通过这两个字段的值来判断接口调用是否成功，若调用失败，则code为非0值，message为错误信息。code值的定义如下：

+--------+------------------------------------------------------------+
| code   | 含义                                                       |
+========+============================================================+
| 0      | 请求成功                                                   |
+--------+------------------------------------------------------------+
| -32700 | 服务端接收到无效的json。该错误发送于服务器尝试解析json文本 |
+--------+------------------------------------------------------------+
| -32600 | 无效的请求（比如非法的JSON格式）                           |
+--------+------------------------------------------------------------+
| -32601 | 方法不存在或者无效                                         |
+--------+------------------------------------------------------------+
| -32602 | 无效的方法参数                                             |
+--------+------------------------------------------------------------+
| -32603 | JSON-RPC内部错误                                           |
+--------+------------------------------------------------------------+
| -32000 | Hyperchain内部错误或者空指针或者节点未安装solidity环境     |
+--------+------------------------------------------------------------+
| -32001 | 查询的数据不存在                                           |
+--------+------------------------------------------------------------+
| -32002 | 余额不足                                                   |
+--------+------------------------------------------------------------+
| -32003 | 签名非法                                                   |
+--------+------------------------------------------------------------+
| -32004 | 合约部署出错                                               |
+--------+------------------------------------------------------------+
| -32005 | 合约调用出错                                               |
+--------+------------------------------------------------------------+
| -32006 | 系统繁忙                                                   |
+--------+------------------------------------------------------------+
| -32007 | 交易重复                                                   |
+--------+------------------------------------------------------------+
| -32008 | 合约操作权限不够                                           |
+--------+------------------------------------------------------------+
| -32009 | (合约)账户不存在                                           |
+--------+------------------------------------------------------------+
| -32010 | namespace不存在                                            |
+--------+------------------------------------------------------------+
| -32011 | 账本上无区块产生，查询最新区块的时候可能抛出该错误         |
+--------+------------------------------------------------------------+
| -32012 | 订阅不存在  (*预留状态码*)                                 |
+--------+------------------------------------------------------------+
| -32013 | 数据归档、快照相关错误                                     |
+--------+------------------------------------------------------------+
| -32098 | 请求未带cert或者错误cert导致认证失败                       |
+--------+------------------------------------------------------------+
| -32099 | 请求tcert失败                                              |
+--------+------------------------------------------------------------+

3. 接口概览
-----------

Transaction
~~~~~~~~~~~
- :ref:`tx_getTransactions`
- :ref:`tx_getDiscardTransactions`
- :ref:`tx_getTransactionByHash`
- :ref:`tx_getTransactionByBlockHashAndIndex`
- :ref:`tx_getTransactionByBlockNumberAndIndex`
- :ref:`tx_getTransactionsCount`
- :ref:`tx_getTxAvgTimeByBlockNumber`
- :ref:`tx_getTransactionReceipt`
- :ref:`tx_getBlockTransactionCountByHash`
- :ref:`tx_getBlockTransactionCountByNumber`
- :ref:`tx_getSignHash`
- :ref:`tx_getTransactionsByTime`
- :ref:`tx_getDiscardTransactionsByTime`
- :ref:`tx_getBatchTransactions`
- :ref:`tx_getBatchReceipt`

Contract
~~~~~~~~

- :ref:`contract_compileContract`
- :ref:`contract_deployContract`
- :ref:`contract_invokeContract`
- :ref:`contract_getCode`
- :ref:`contract_getContractCountByAddr`
- :ref:`contract_maintainContract`
- :ref:`contract_getStatus`
- :ref:`contract_getCreator`
- :ref:`contract_getCreateTime`
- :ref:`contract_getDeployedList`

Block
~~~~~

- :ref:`block_latestBlock`
- :ref:`block_getBlocks`
- :ref:`block_getBlockByHash`
- :ref:`block_getBlockByNumber`
- :ref:`block_getAvgGenerateTimeByBlockNumber`
- :ref:`block_getBlocksByTime`
- :ref:`block_getGenesisBlock`
- :ref:`block_getChainHeight`
- :ref:`block_getBatchBlocksByHash`
- :ref:`block_getBatchBlocksByNumber`

Subscription
~~~~~~~~~~~~

- :ref:`sub_newBlockSubscription`
- :ref:`sub_newEventSubscription`
- :ref:`sub_getLogs`
- :ref:`sub_newSystemStatusSubscription`
- :ref:`sub_getSubscriptionChanges`
- :ref:`sub_unSubscription`

Node
~~~~

- :ref:`node_getNodes`
- :ref:`node_getNodeHash`
- :ref:`node_deleteVP`
- :ref:`node_deleteNVP`

Certificate
~~~~~~~~~~~

- :ref:`cert_getTCert`

4. 接口描述
-----------

.. _tx_getTransactions:

tx_getTransactions
~~~~~~~~~~~~~~~~~~

查询指定区块区间的所有交易。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``from``: ``<blockNumber>`` - 起始区块号。
-  ``to``: ``<blockNumber>`` - 终止区块号。

``<blockNumber>``\ 可以是十进制整数或者进制字符串，可以是\ ``“latest”``\ 字符串表示最新的区块。\ ``from``\ 必须小于等于\ ``to``\ ，否则会返回error。

.. _validtransaction:

Returns
^^^^^^^

1. ``[<Transaction>]`` - Transaction对象字段如下：

-  ``version``: ``<string>`` - 平台版本号。
-  ``hash``: ``<string>`` - 32字节的十六进制字符串，交易哈希值。
-  ``blockNumber``: ``<string>`` - 十六进制，交易所在区块的高度。
-  ``blockHash``: ``<string>`` -
   32字节的十六进制字符串，交易所在区块的哈希。
-  ``txIndex``: ``<string>`` - 十六进制，交易在区块中的偏移量。
-  ``from``: ``<string>`` - 20字节的十六进制字符串，交易发送方的地址。
-  ``to``: ``<string>`` - 20字节的十六进制字符串，交易接收方的地址。
-  ``amount``: ``<string>`` - 转账金额。
-  ``timestamp``: ``<number>`` - 交易发生的unix时间戳（单位ns）。
-  ``nonce``: ``<number>`` - 16位随机数。
-  ``extra``: ``<string>`` - 交易的额外信息。
-  ``executeTime``: ``<string>`` - 交易的处理时间（单位ms）。
-  ``payload``: ``<string>`` -
   部署合约、调用合约、升级合约的时候才有这个值，可以通过这个值追溯到合约调用的方法以及调用传入的参数。

Example1: 正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2: 区块不存在
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _tx_getDiscardTransactions:

tx_getDiscardTransactions
~~~~~~~~~~~~~~~~~~~~~~~~~

查询所有非法交易。

Parameters
^^^^^^^^^^

无

.. _invalidtransaction:

Returns
^^^^^^^

1. ``[<Transaction>]`` - Transaction对象字段如下：

-  ``version``: ``<string>`` - 平台版本号。
-  ``hash``: ``<string>`` -
   32字节的十六进制字符串，交易哈希值。串，交易所在区块的哈希。
-  ``from``: ``<string>`` - 20字节的十六进制字符串，交易发送方的地址。
-  ``to``: ``<string>`` - 20字节的十六进制字符串，交易接收方的地址。
-  ``amount``: ``<string>`` - 转账金额。
-  ``timestamp``: ``<number>`` - 交易发生的unix时间戳（单位ns）。
-  ``nonce``: ``<number>`` - 16位随机数。
-  ``extra``: ``<string>`` - 交易的额外信息。
-  ``payload``: ``<string>`` -
   部署合约、调用合约、升级合约的时候才有这个值，可以通过这个值追溯到合约调用的方法以及调用传入的参数。
-  ``invalid``: ``<boolean>`` - 交易是否不合法。
-  ``invalidMsg``: ``<string>`` - 交易的不合法信息。

不合法的交易\ ``invalid``\ 值为true， ``invalidMsg``\ 可能为： 

-  **DEPLOY_CONTRACT_FAILED** - 合约部署失败； 
-  **INVOKE_CONTRACT_FAILED** - 合约方法调用失败； 
-  **SIGFAILED** - 签名非法； 
-  **OUTOFBALANCE** - 余额不足; 
-  **INVALID_PERMISSION** - 合约操作权限不够;

Example1：正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

    # Request
    curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getDiscardTransactions", "params": [], "id": 71}'

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

Example2：若没有非法交易
^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

    # Request
    curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getDiscardTransactions", "params": [], "id": 71}'

    # Response
    {
      "jsonrpc": "2.0",
      "namespace": "global",
      "id": 1,
      "code": -32001,
      "message": "Not found discard transactions "
    }

.. _tx_getTransactionByHash:

tx_getTransactionByHash
~~~~~~~~~~~~~~~~~~~~~~~

根据交易哈希查询交易详情。

Parameters
^^^^^^^^^^

1. ``<string>`` - 32字节的十六进制字符串，交易的哈希值。

Returns
^^^^^^^

1. ``<Transaction>`` - Transaction对象字段如下：

-  ``version``: ``<string>`` - 平台版本号。
-  ``hash``: ``<string>`` - 32字节的十六进制字符串，交易哈希值。
-  ``blockNumber``: ``<string>`` - 十六进制，交易所在区块的高度。
-  ``blockHash``: ``<string>`` -
   32字节的十六进制字符串，交易所在区块的哈希。
-  ``txIndex``: ``<string>`` - 十六进制，交易在区块中的偏移量。
-  ``from``: ``<string>`` - 20字节的十六进制字符串，交易发送方的地址。
-  ``to``: ``<string>`` - 20字节的十六进制字符串，交易接收方的地址。
-  ``amount``: ``<string>`` - 转账金额。
-  ``timestamp``: ``<number>`` - 交易发生的unix时间戳（单位ns）。
-  ``nonce``: ``<number>`` - 16位随机数。
-  ``extra``: ``<string>`` - 交易的额外信息。
-  ``executeTime``: ``<string>`` - 交易的处理时间（单位ms）。
-  ``payload``: ``<string>`` -
   部署合约、调用合约、升级合约的时候才有这个值，可以通过这个值追溯到合约调用的方法以及调用传入的参数。
-  ``invalid``: ``<boolean>`` - 交易是否不合法。
-  ``invalidMsg``: ``<string>`` - 交易的不合法信息。

不合法的交易\ ``invalid``\ 值为true， ``invalidMsg``\ 可能为： 

- **OUTOFBALANCE** - 余额不足，对应code是-32002； 
- **SIGFAILED** - 签名非法，对应code是-32003； 
- **DEPLOY_CONTRACT_FAILED** - 合约部署失败，对应code是-32004； 
- **INVOKE_CONTRACT_FAILED** - 合约方法调用失败，对应code是-32005； 
- **INVALID_PERMISSION** - 合约操作权限不够，对应code是-32008；

Example1：查询合法的交易
^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：查询非法的交易
^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example3：查询的交易不存在
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _tx_getTransactionByBlockHashAndIndex:

tx_getTransactionByBlockHashAndIndex
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

根据区块哈希和交易偏移量查询交易。

Parameters
^^^^^^^^^^

1. ``<string>`` - 32字节的十六进制字符串，区块的哈希值。
2. ``<number>`` - 交易在区块中的偏移量，可以是十进制整数或进制字符串。

Returns
^^^^^^^

1. ``<Transaction>`` - Transaction对象字段见 `合法交易 <#validtransaction>`__. 

Example1：正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：查询的区块不存在
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _tx_getTransactionByBlockNumberAndIndex:

tx_getTransactionByBlockNumberAndIndex
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

根据区块号和交易偏移量查询交易。

Parameters
^^^^^^^^^^

1. ``<blockNumber>`` - 区块号，可以是十进制整数、进制字符串或“latest”字符串表示最新的区块。
2. ``<number>`` - 交易在区块中的偏移量，可以是十进制整数或进制字符串。

Returns
^^^^^^^

1. ``<Transaction>`` - Transaction对象字段见 `合法交易 <#validtransaction>`__. 

Example1：正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：请求的区块不存在
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _tx_getTransactionsCount:

tx_getTransactionsCount
~~~~~~~~~~~~~~~~~~~~~~~

查询当前链上交易量。

Parameters
^^^^^^^^^^

无

Returns
^^^^^^^

1. ``<Object>``

-  ``count``: ``<string>`` - 交易数量，十六进制字符串表示。
-  ``timestamp``: ``<number>`` - 响应时间戳（单位ns）。

Example
^^^^^^^

.. code:: bash

    # Request
    curl -X POST --data '{"jsonrpc": "2.0", "namespace":"global", "method": "tx_getTransactionsCount", "params": [], "id": 71}'

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

.. _tx_getTxAvgTimeByBlockNumber:

tx_getTxAvgTimeByBlockNumber
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

根据指定的区块区间计算出每笔交易的平均处理时间。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``from``: ``<blockNumber>`` - 起始区块号。
-  ``to``: ``<blockNumber>`` - 终止区块号。

``blockNumber`` 可以是十进制整数或者进制字符串，可以是“latest”字符串表示最新的区块。 ``from`` 必须小于等于 ``to`` ，否则会返回error。如果 ``from`` 和 ``to`` 的值一样，则表示计算的是当前指定区块交易的平均处理时间。

Returns
^^^^^^^

1. ``<string>`` - 十六进制字符串，表示交易的平均处理时间（单位ms）。

Example
^^^^^^^

.. code:: bash

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

.. _tx_getTransactionReceipt:

tx_getTransactionReceipt
~~~~~~~~~~~~~~~~~~~~~~~~

根据交易哈希返回交易回执信息。

Parameters
^^^^^^^^^^

1. ``<string>`` - 32字节的十六进制字符串，交易的哈希值。

.. _receipt:

Returns
^^^^^^^

1. ``<Receipt>`` - Receipt对象字段如下：

-  ``version``: ``<string>`` - 平台版本号。
-  ``txHash``: ``<string>`` - 交易哈希。
-  ``vmType``: ``<string>`` - 该笔交易执行引擎类型，\ ``EVM``\ 或\ ``JVM``\ 。
-  ``contractAddress``: ``<string>`` - 合约地址。
-  ``gasUsed``: ``<number>`` - 该笔交易所耗费的gas。
-  ``ret``: ``<string>`` - 合约编译后的字节码或合约执行的结果。
-  ``log``: ``[<Log>]`` - Log对象数组，表示合约中的event log信息。Log对象如下：

   -  ``address``: ``<string>`` - 产生事件日志的合约地址。
   -  ``topics``: ``[<string>]`` - 一系列的topic，第一个topic是event的唯一标识。
   -  ``data``: ``<string>`` - 日志信息。
   -  ``blockNumber``: ``<number>`` - 所属区块的区块号。
   -  ``blockHash``: ``<string>`` - 所属区块的区块哈希。
   -  ``txHash``: ``<string>`` - 所属交易的交易哈希。
   -  ``txIndex``: ``<number>`` - 所属交易在当前区块交易列表中的偏移量。
   -  ``index``: ``<number>`` - 该日志在本条交易产生的所有日志中的偏移量。

如果该笔交易还没被确认，则返回的错误码为-32001的error，如果该笔交易处理过程中发生错误，则错误可能是：

- **OUTOFBALANCE** - 余额不足，对应code是-32002； 
- **SIGFAILED** - 签名非法，对应code是-32003； 
- **DEPLOY_CONTRACT_FAILED** - 合约部署失败，对应code是-32004； 
- **INVOKE_CONTRACT_FAILED** - 合约方法调用失败，对应code是-32005； 
- **INVALID_PERMISSION** - 合约操作权限不够，对应code是-32008；

Example1：交易未被确认
^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：合约部署出错
^^^^^^^^^^^^^^^^^^^^^^

在这个例子中我们使用以下合约来重现这个情况：

.. code:: bash

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

我们将该合约编译后返回的bin作为\ `contract_deployContract <#contract-deploycontract>`__\ 方法中参数\ ``payload``\ 的值，那么部署合约请求如下：

.. code:: bash

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

接着，根据返回的hash查找这条记录的receipt，会发现返回了合约部署失败的error：

.. code:: bash

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

Example3：合约方法调用出错
^^^^^^^^^^^^^^^^^^^^^^^^^^

在这个例子中我们使用以下合约来重现这个情况：

.. code:: bash

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

调用合约\ ``getLength()``\ 方法的请求如下：

.. code:: bash

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

接着，根据返回的hash查找这条记录的receipt，会发现返回了方法调用失败的error：

.. code:: bash

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

Example4：签名非法
^^^^^^^^^^^^^^^^^^

我们将Example3合约方法调用失败例子的参数稍微修改一下，把\ ``from``\ 的最后一个字母“c”改为“0”，那么调用合约请求如下：

.. code:: bash

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

接着，根据返回的hash查找这条记录的receipt，会发现返回了签名非法的error：

.. code:: bash

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

Example5
^^^^^^^^

.. code:: bash

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

.. _tx_getBlockTransactionCountByHash:

tx_getBlockTransactionCountByHash
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

根据区块哈希查询区块交易数量。

Parameters
^^^^^^^^^^

1. ``<string>`` - 32字节的十六进制字符串，区块的哈希值。

Returns
^^^^^^^

1. ``<string>`` - 十六进制字符串，交易数量。

Example1：正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：查询区块不存在
^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _tx_getBlockTransactionCountByNumber:

tx_getBlockTransactionCountByNumber
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

根据区块号查询区块交易数量。

Parameters
^^^^^^^^^^

1. ``<blcokNumber>`` - 区块号，可以是十进制整数、进制字符串或“latest”字符串来表示最新区块。

Returns
^^^^^^^

1. ``<string>`` - 十六进制字符串，交易数量。

Example1：正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：查询的区块不存在
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _tx_getSignHash:

tx_getSignHash
~~~~~~~~~~~~~~

获取用于签名算法的哈希。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``from``: ``<string>`` - 20字节的十六进制字符串，交易发送方的地址。
-  ``to``: ``<string>`` - [可选] 20字节的十六进制字符串，交易接收方的地址（普通账户或合约地址）。若是部署合约，则不需要这个参数。
-  ``nonce``: ``<number>`` - 16位随机数。
-  ``extra``: ``<string>`` - [可选] 交易的额外信息。
-  ``value``\ 或\ ``payload``: ``<string>`` -
   value表示转账金额，payload表示合约操作对应字节编码。
-  ``timestamp``: ``<number>`` - 交易发生时间戳（单位ns）。

.. Note:: 如果是部署合约的交易，则不要传to。若为普通转账，则传value，表示转账金额。若是部署合约、调用合约或升级合约的交易，则传payload，含义详见\ `部署合约 <#contract-deploycontract>`__\ 、\ `调用合约 <#contract-invokecontract>`__\ 或\ `升级合约 <#contract-maintaincontract>`__\ 的接口。

Returns
^^^^^^^

1. ``<string>`` - 十六进制字符串，签名哈希。

Example
^^^^^^^

.. code:: bash

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

.. _tx_getTransactionsByTime:

tx_getTransactionsByTime
~~~~~~~~~~~~~~~~~~~~~~~~

查询指定时间区间内的所有合法交易。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``startTime``: ``<number>`` - 起始时间戳（单位ns）。
-  ``endTime``: ``<number>`` - 结束时间戳（单位ns）。

Returns
^^^^^^^

1. ``[<Transaction>]`` - Transaction对象字段见 `合法交易 <#validtransaction>`__. 

Example1：正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：查询数据不存在
^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _tx_getDiscardTransactionsByTime:

tx_getDiscardTransactionsByTime
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

查询指定时间区间内的所有非法交易。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``startTime``: ``<number>`` - 起始时间戳（单位ns）。
-  ``endTime``: ``<number>`` - 结束时间戳（单位ns）。

Returns
^^^^^^^

1. ``[<Transaction>]`` - Transaction对象字段见 `非法交易 <#invalidtransaction>`__. 

Example1：正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _tx_getBatchTransactions:

tx_getBatchTransactions
~~~~~~~~~~~~~~~~~~~~~~~

根据交易哈希批量查询交易。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``hashs``: ``[<string>]`` - 交易哈希数组，哈希值为32字节的十六进制字符串。

Returns
^^^^^^^

1. ``[<Transaction>]`` - Transaction对象字段见 `合法交易 <#validtransaction>`__. 

Example
^^^^^^^

.. code:: bash

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

.. _tx_getBatchReceipt:

tx_getBatchReceipt
~~~~~~~~~~~~~~~~~~

根据交易哈希批量查询交易回执。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``hashs``: ``[<string>]`` - 交易哈希数组,，哈希值为32字节的十六进制字符串。

Returns
^^^^^^^

1. ``[<Receipt>]`` - Receipt对象字段见 `Receipt <#receipt>`__. 

Example
^^^^^^^

.. code:: bash

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

.. _contract_compileContract:

contract_compileContract
~~~~~~~~~~~~~~~~~~~~~~~~

编译智能合约。

Parameters
^^^^^^^^^^

1. ``<string>`` - 合约源码。

Returns
^^^^^^^

1. ``<Object>``

-  ``abi``: ``[<string>]`` - 合约源码对应的abi。
-  ``bin``: ``[<string>]`` - 合约编译而成的字节码。
-  ``types``: ``[<string>]`` - 对应合约的名称。

若源码中有多个合约，则bin为顶层合约的字节码。

Example
^^^^^^^

.. code:: bash

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

.. _contract_deployContract:

contract_deployContract
~~~~~~~~~~~~~~~~~~~~~~~

部署合约。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``from``: ``<string>`` - 20字节的十六进制字符串，交易发送方的地址。
-  ``nonce``: ``<number>`` - 16位随机数，该值必须为十进制整数。
-  ``extra``: ``<string>`` -[可选] 交易的额外信息。
-  ``timestamp``: ``<number>`` - 交易发生时间戳（单位ns）。
-  ``payload``: ``<string>`` - 合约编码。如果是\ **solidity**\ 合约，则该值为contract_complieContract方法返回的bin以及构造函数参数的拼接。如果是\ **java**\ 合约，该值为class文件和配置文件压缩后的字节流。
-  ``signature``: ``<string>`` - 交易签名。
-  ``type``: ``<string>`` - [可选] 指定合约执行引擎，默认为\ **“evm”**\ 。如果合约代码由java语言编写，则需要设置该值为\ **“jvm”**\ 。

.. Note:: 若合约构造函数需要传参，则payload为编译合约返回的bin与构造函数参数编码的字符串拼接。

Returns
^^^^^^^

1. ``<string>`` - 32字节的十六进制字符串，交易的哈希值。

Example
^^^^^^^

.. code:: bash

    # Request
    curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global",  "method":"contract_deployContract", "params":[{
        "from":"0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
        "nonce":5373500328656597,
        "payload":"0x60606040526000805463ffffffff1916815560ae908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d14605e578063d09de08a146084575b6002565b346002576000805460e060020a60243560043563ffffffff8416010181020463ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b34600257605c6000805460e060020a63ffffffff821660010181020463ffffffff1990911617905556",
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

.. _contract_invokeContract:

contract_invokeContract
~~~~~~~~~~~~~~~~~~~~~~~

调用合约。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``from``: ``<string>`` - 20字节的十六进制字符串，交易发送方的地址。
-  ``to``: ``<string>`` - 20字节的十六进制字符串，合约地址。
-  ``nonce``: ``<number>`` - 16位随机数，该值必须为十进制整数。
-  ``extra``: ``<string>`` -[可选] 交易的额外信息。
-  ``timestamp``: ``<number>`` - 交易发生时间戳（单位ns）。
-  ``payload``: ``<string>`` - 该值为方法名和方法参数经过编码后的input字节码。
-  ``signature``: ``<string>`` - 交易签名。
-  ``simulate``: ``<bool>`` - [可选] 默认为false。true表示交易不走共识，false表示走共识。
-  ``type``: ``<string>`` - [可选] 指定合约执行引擎，默认为\ **“evm”**\ 。如果合约代码由java语言编写，则需要设置该值为\ **“jvm”**\ 。

.. Note:: 说明：to合约地址需要在部署完合约以后，调用 \ `tx_getTransactionReceipt <#tx-gettransactionreceipt>`__\  方法来获取。

Returns
^^^^^^^

1. ``<string>`` - 32字节的十六进制字符串，交易的哈希值。

Example
^^^^^^^

.. code:: bash

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

.. _contract_getCode:

contract_getCode
~~~~~~~~~~~~~~~~

获取合约的字节编码。

Parameters
^^^^^^^^^^

1. ``<string>`` - 合约地址。

Returns
^^^^^^^

1. ``<string>`` - 十六进制字节码。

Example
^^^^^^^

.. code:: bash

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

.. _contract_getContractCountByAddr:

contract_getContractCountByAddr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

获取指定账户部署的合约量。

Parameters
^^^^^^^^^^

1. ``<string>`` - 20字节的十六进制字符串，账户地址。

Returns
^^^^^^^

1. ``<string>`` - 合约数量。

Example
^^^^^^^

.. code:: bash

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

.. _contract_maintainContract:

contract_maintainContract
~~~~~~~~~~~~~~~~~~~~~~~~~

升级合约、冻结合约、解冻合约。
只有合约的部署者才拥有升级合约、冻结合约、解冻合约的权限。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``from``: ``<string>`` - 20字节的十六进制字符串，交易发送方的地址。
-  ``to``: ``<string>`` - 20字节的十六进制字符串，合约地址。
-  ``nonce``: ``<number>`` - 16位随机数，该值必须为十进制整数。
-  ``extra``: ``<string>`` -[可选] 交易的额外信息。
-  ``timestamp``: ``<number>`` - 交易发生时间戳（单位ns）。
-  ``payload``: ``<string>`` - [可选] 编译后的新合约字节码。\ **升级合约才需要这个字段。**
-  ``signature``: ``<string>`` - 交易签名。
-  ``type``: ``<string>`` - 指定合约执行引擎，默认为\ **“evm”**\ 。如果合约代码由java语言编写，则需要设置该值为\ **“jvm”**\ 。
-  ``opcode``: 值为\ ``1``\ 表示\ **升级合约**\ ，值为\ ``2``\ 表示\ **冻结合约**\ 、值为\ ``3``\ 表示\ **解冻合约**\ 。

.. Note:: 说明：to合约地址需要在部署完合约以后，调用 \ `tx_getTransactionReceipt <#tx-gettransactionreceipt>`__\  方法来获取。

Returns
^^^^^^^

1. ``<string>`` - 32字节的十六进制字符串，交易的哈希值。

Example1：升级合约
^^^^^^^^^^^^^^^^^^

.. code:: bash

    # Request
    curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_maintainContract","params":[{
        "from": "17d806c92fa941b4b7a8ffffc58fa2f297a3bffc", 
        "to":"0x3a3cae27d1b9fa931458b5b2a5247c5d67c75d61", 
        "timestamp":1481767474717000000, 
        "nonce": 8054165127693853,
        "payload":"0x6fd7cc16000000000000000000000000000000000000000000000000000000000000007b00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000", 
        "signature":"0x19c0655d05b9c24f5567846528b81a25c48458a05f69f05cf8d6c46894b9f12a02af471031ba11f155e41adf42fca639b67fb7148ddec90e7628ec8af60c872c00", 
        "opcode": 1}],
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

Example2：冻结合约
^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example3：解冻合约
^^^^^^^^^^^^^^^^^^

.. code:: bash

    # Request
    curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global", "method": "contract_maintainContract","params":[{
        "from": "17d806c92fa941b4b7a8ffffc58fa2f297a3bffc", 
        "to":"0x3a3cae27d1b9fa931458b5b2a5247c5d67c75d61", 
        "timestamp":1481767474717000000, 
        "nonce": 8054165127693853,
        "signature":"0x19c0655d05b9c24f5567846528b81a25c48458a05f69f05cf8d6c46894b9f12a02af471031ba11f155e41adf42fca639b67fb7148ddec90e7628ec8af60c872c00", 
        "opcode": 3}],
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

.. _contract_getStatus:

contract_getStatus
~~~~~~~~~~~~~~~~~~

查询合约状态。

Parameters
^^^^^^^^^^

1. ``<string>`` - 20字节的十六进制字符串，合约地址。

Returns
^^^^^^^

1. ``<string>`` -
   合约状态。\ ``normal``\ 表示正常状态，\ ``frozen``\ 表示冻结状态，\ ``non-contract``\ 表示非合约，即为普通转账的交易。

Example
^^^^^^^

.. code:: bash

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

.. _contract_getCreator:

contract_getCreator
~~~~~~~~~~~~~~~~~~~

查询合约部署者。

Parameters
^^^^^^^^^^

1. ``<string>`` - 20字节的十六进制字符串，合约地址。

Returns
^^^^^^^

1. ``<string>`` - 20字节的十六进制字符串，合约部署者地址。

Example
^^^^^^^

.. code:: bash

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

.. _contract_getCreateTime:

contract_getCreateTime
~~~~~~~~~~~~~~~~~~~~~~

查询合约部署时间。

Parameters
^^^^^^^^^^

1. ``<string>`` - 20字节的十六进制字符串，合约地址。

Returns
^^^^^^^

1. ``<string>`` - 合约部署的日期时间。

Example
^^^^^^^

.. code:: bash

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

.. _contract_getDeployedList:

contract_getDeployedList
~~~~~~~~~~~~~~~~~~~~~~~~

查询已部署的合约地址列表。

Parameters
^^^^^^^^^^

1. ``<string>`` - 20字节的十六进制字符串，账户地址。

Returns
^^^^^^^

1. ``[<string>]`` - 已部署的所有合约地址。

Example
^^^^^^^

.. code:: bash

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

.. _block_latestBlock:

block_latestBlock
~~~~~~~~~~~~~~~~~

获取最新区块。

Parameters
^^^^^^^^^^

无

.. _block-object:

Returns
^^^^^^^

1. ``<Block>`` - Block对象字段如下：

-  ``version``: ``<string>`` - 平台版本号。
-  ``number``: ``<string>`` - 区块的高度。
-  ``hash``: ``<string>`` - 区块的哈希值，32字节的十六进制字符串。
-  ``parentHash``: ``<string>`` - 父区块哈希值，32字节的十六进制字符串。
-  ``writeTime``: ``<number>`` - 区块生成的unix时间戳。
-  ``avgTime``: ``<string>`` - 当前区块中，交易的平均处理时间（单位ms）。
-  ``txCounts``: ``<string>`` - 当前区块中打包的交易数量。
-  ``merkleRoot``: ``<string>`` - Merkle树的根哈希。
-  ``transactions``: ``[<Transaction>]`` - 区块中的交易列表。

Example1：正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：如果链上一个区块都没有
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _block_getBlocks:

block_getBlocks
~~~~~~~~~~~~~~~

查询指定区块区间的所有区块。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``from``: ``<blockNumber>`` - 起始区块号。
-  ``to``: ``<blockNumber>`` - 终止区块号。
-  ``isPlain``: ``<boolean>`` - [可选] 默认值为\ ``false``\ ，表示返回的区块包括区块内的交易信息，如果指定为true，表示返回的区块不包括区块内的交易。

``blockNumber`` 可以是十进制整数或者进制字符串，可以是“latest”字符串表示最新的区块。 ``from`` 必须小于等于 ``to`` ，否则会返回error。

Returns
^^^^^^^

1. ``[<Block>]`` - Block对象字段见 `Block <#block-object>`__.
   
Example1：返回的区块包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：返回的区块不包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _block_getBlockByHash:

block_getBlockByHash
~~~~~~~~~~~~~~~~~~~~

根据区块的哈希值查询区块详细信息。

Parameters
^^^^^^^^^^

1. ``<string>`` - 32字节的十六进制字符串，区块的哈希值。
2. ``<boolean>`` - 值为\ ``true``\ ，表示返回的区块不包括区块内的交易。值为\ ``false``\ 表示返回的区块包括区块内的交易信息。

Returns
^^^^^^^

1. ``<Block>`` - Block对象字段见 `Block <#block-object>`__.
  
Example1：返回的区块包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：返回的区块不包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _block_getBlockByNumber:

block_getBlockByNumber
~~~~~~~~~~~~~~~~~~~~~~

根据区块高度查询区块详细信息。

Parameters
^^^^^^^^^^

1. ``<blockNumber>`` - 区块号，可以是十进制整数、进制字符串或“latest”字符串来表示最新区块。
2. ``<boolean>`` - 值为\ ``true``\ ，表示返回的区块不包括区块内的交易。值为\ ``false``\ 表示返回的区块包括区块内的交易信息。

Returns
^^^^^^^

1. ``<Block>`` - Block对象字段见 `Block <#block-object>`__.
   
Example1：返回的区块包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：返回的区块不包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _block_getAvgGenerateTimeByBlockNumber:

block_getAvgGenerateTimeByBlockNumber
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

查询区块平均生成时间。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``from``: ``<blockNumber>`` - 起始区块号。
-  ``to``: ``<blockNumber>`` - 终止区块号。

``blockNumber`` 可以是十进制整数或者进制字符串，可以是“latest”字符串表示最新的区块。 ``from`` 必须小于等于 ``to`` ，否则会返回error。

Returns
^^^^^^^

1. ``<string>`` - 区块的平均生成时间（单位ms）。

Example
^^^^^^^

.. code:: bash

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

.. _block_getBlocksByTime:

block_getBlocksByTime
~~~~~~~~~~~~~~~~~~~~~

查询指定时间区间内的区块数量。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``startTime``: ``<number>`` - 起始unix时间戳（单位ns）。
-  ``endTime``: ``<number>`` - 结束unix时间戳（单位ns）。

Returns
^^^^^^^

1. ``<Object>``

-  ``sumOfBlocks``: ``<string>`` - 区块总数。
-  ``startBlock``: ``<string>`` - 起始区块号。
-  ``endBlock``: ``<string>`` - 结束区块号。

Example1: 正常的请求
^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：如果起始时间和终止时间均大于链上最新区块的写入时间
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _block_getGenesisBlock:

block_getGenesisBlock
~~~~~~~~~~~~~~~~~~~~~

查询创世区块号。

Parameters
^^^^^^^^^^

无

Returns
^^^^^^^

1. ``<string>`` - 区块号。

Example
^^^^^^^

.. code:: bash

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

.. _block_getChainHeight:

block_getChainHeight
~~~~~~~~~~~~~~~~~~~~

查询最新区块号。

Parameters
^^^^^^^^^^

无

Returns
^^^^^^^

1. ``<string>`` - 区块号。

Example
^^^^^^^

.. code:: bash

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

.. _block_getBatchBlocksByHash:

block_getBatchBlocksByHash
~~~~~~~~~~~~~~~~~~~~~~~~~~

根据区块哈希列表批量查询区块详细信息。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``hashes``: ``[<string>]`` -
   要查询的区块哈希数组，哈希值为32字节的十六进制字符串。
-  ``isPlain``: ``<boolean>`` -
   值为\ ``true``\ ，表示返回的区块不包括区块内的交易。值为\ ``false``\ 表示返回的区块包括区块内的交易信息。

Returns
^^^^^^^

1. ``[<Block>]`` - Block对象数组，Block对象字段见 `Block <#block-object>`__.

Example1：返回的区块包含交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：返回的区块不包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _block_getBatchBlocksByNumber:

block_getBatchBlocksByNumber
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

根据区块号列表批量查询区块详细信息。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``numbers``: ``[<blockNumber>]`` - 要查询的区块号数组，区块号可以是十进制整数或者进制字符串，也可以是“latest”字符串表示最新的区块。
-  ``isPlain``: ``<boolean>`` - 值为\ ``true``\ ，表示返回的区块不包括区块内的交易。值为\ ``false``\ 表示返回的区块包括区块内的交易信息。

Returns
^^^^^^^

1. ``[<Block>]`` - Block对象数组，Block对象字段见 `Block <#block-object>`__.

Example1：返回的区块包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

Example2：返回的区块不包括交易信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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

.. _sub_newBlockSubscription:

sub_newBlockSubscription
~~~~~~~~~~~~~~~~~~~~~~~~

订阅新区块事件并且创建一个过滤器用于通知客户端，当有一个新区块产生的时候，该区块信息会缓存在过滤器中。

Parameters
^^^^^^^^^^

1. ``<boolean>`` - 是否返回完整数据；值为\ ``true``\ 表示返回完整 `Block对象 <#block-object>`__\ ；值为\ ``false``\ 表示只返回区块哈希。

Returns
^^^^^^^

1. ``<string>`` - 订阅标号。

Example
^^^^^^^

.. code:: bash

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

.. _sub_newEventSubscription:

sub_newEventSubscription
~~~~~~~~~~~~~~~~~~~~~~~~

订阅虚拟机事件并且创建一个过滤器用于通知客户端，当虚拟机事件被触发的时候，该事件日志会缓存在过滤器中。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``fromBlock``: ``<number>`` - [可选] 十进制整数，表示起始区块号；若为空则默认没有限制。起始区块号大于或等于当前最新区块号。
-  ``toBlock``: ``<number>`` - [可选] 十进制整数，表示终止区块号；若为空则默认没有限制。终止区块号是大于起始区块号的未来某一个区块号。
-  ``addresses``: ``[<string>]`` - [可选] 表示监听指定地址的合约产生的事件；若为空则表示监听所有合约产生的事件。
-  ``topics``: ``[<string>][<string>]`` - [可选] 二维字符串数组，表示事件的话题，用于事件的内容过滤。若为空表示没有过滤条件。\ ``topics``\ 可能有以下组合：

   -  ``[A, B] = A && B``
   -  ``[A, [B, C]] = A && (B || C)``
   -  ``[null, A, B] = ANYTHING && A && B`` ``null`` 表示通配符。

Returns
^^^^^^^

1. ``<string>`` - 订阅标号。

Example
^^^^^^^

.. code:: bash

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

.. _sub_getLogs:

sub_getLogs
~~~~~~~~~~~

获取符合条件的虚拟机事件。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``fromBlock``: ``<number>`` - [可选] 十进制整数，表示起始区块号；若为空则默认为0。起始区块号不能小于当前创世区块号。
-  ``toBlock``: ``<number>`` - [可选] 十进制整数，表示终止区块号；若为空则默认没有限制。终止区块号不能大于当前最新区块号。
-  ``addresses``: ``[<string>]`` - [可选] 一维数组，表示监听指定地址的合约产生的事件；若为空则表示监听所有合约产生的事件。
-  ``topics``: ``[<string>][<string>]`` - [可选] 二维字符串数组，表示事件的话题，用于事件的内容过滤。\ ``topics``\ 可能有以下组合：

   -  ``[A, B] = A && B``
   -  ``[A, [B, C]] = A && (B || C)``
   -  ``[null, A, B] = ANYTHING && A && B`` ``null`` 表示通配符。

Returns
^^^^^^^

1. ``[<Log>]`` - 事件信息，Log对象字段如下：

   -  ``address``: ``<string>`` - 20字节的十六进制字符串，产生事件的合约地址。
   -  ``topics``: ``[<string>]`` - 一系列的topic。
   -  ``data``: ``<string>`` - 数据段。
   -  ``blockNumber``: ``<number>`` - 十进制整数，所属区块号。
   -  ``blockHash``: ``<string>`` - 所属区块哈希。
   -  ``txHash``: ``<string>`` - 所属交易哈希。
   -  ``txIndex``: ``<number>`` - 十进制整数，所属交易在当前区块交易列表中的偏移量。
   -  ``index``: ``<number>`` - 十进制整数，该日志在本条交易产生的所有日志中的偏移量。

Example
^^^^^^^

.. code:: bash

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

.. _sub_newSystemStatusSubscription:

sub_newSystemStatusSubscription
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

订阅系统状态事件。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``modules``: ``[<string>]`` - [可选] 一维数组，表示要订阅哪些模块的状态信息，若为空，则表示订阅所有模块。比如：\ **p2p**\ 、\ **consensus**\ 、\ **executor**\ 等。
-  ``modules_exclude``: ``[<string>]`` - [可选] 一维数组，表示排除哪些模块的状态信息，若为空，则表示不排除。
-  ``subtypes``: ``[<string>]`` - [可选] 一维数组，表示要订阅模块下面的哪一类状态信息，若为空，则表示订阅所有类型。比如：\ **viewchange**\ 等。
-  ``subtypes_exclude``: ``[<string>]`` - [可选] 一维数组，表示要排除模块下面的哪一类状态信息，若为空，则表示不排除。
-  ``error_codes``: ``[<number>]`` - [可选] 一维数组，元素为十进制整数，表示要订阅指定的具体哪一条状态信息，若为空，则表示订阅所有状态信息。
-  ``error_codes_exclude``: ``[<number>]`` - [可选] 一维数组，元素为十进制整数，表示要排除指定的具体哪一条状态信息，若为空，则表示不排除。

Returns
^^^^^^^

1. ``<string>`` - 订阅标号。

Example
^^^^^^^

.. code:: bash

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

.. _sub_getSubscriptionChanges:

sub_getSubscriptionChanges
~~~~~~~~~~~~~~~~~~~~~~~~~~

获取所订阅的事件。通过轮询这个方法，可以实时获取订阅的事件。

Parameters
^^^^^^^^^^

1. ``<string>`` - 订阅标号。

Returns
^^^^^^^

1. ``<Array>`` - 一维数组，所订阅的事件返回的内容。

Example
^^^^^^^

.. code:: bash

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

.. _sub_unSubscription:

sub_unSubscription
~~~~~~~~~~~~~~~~~~

取消订阅。

Parameters
^^^^^^^^^^

1. ``<string>`` - 订阅标号。

Returns
^^^^^^^

1. ``<boolean>`` - 值为\ ``true``\ 表示取消订阅指定事件成功，否则失败。

Example
^^^^^^^

.. code:: bash

    # Request
    curl -X POST --data '{"jsonrpc":"2.0", "namespace":"global", "method":"sub_unsubscription","params":[“0x7e533eb0647ecbe473ae610ebdd1bba6”], "id":1}'

    # Response
    {
      "jsonrpc": "2.0",
      "namespace":"global",
      "id": 1,
      "code": 0,
      "message": "SUCCESS",
      "result":true,
    }

.. _node_getNodes:

node_getNodes
~~~~~~~~~~~~~

获取节点信息。

Parameters
^^^^^^^^^^

无

Returns
^^^^^^^

1. ``[<PeerInfo>]`` - PeerInfo对象字段如下

-  ``id``: ``<number>`` - 该节点id。
-  ``ip``: ``<string>`` - 该节点IP地址。
-  ``port``: ``<number>`` - 该节点的grpc端口号。
-  ``namespace``: ``<string>`` - 该节点所在分区。
-  ``hash``: ``<string>`` - 该节点哈希值。
-  ``hostname``: ``<string>`` - 节点主机名。
-  ``isPrimary``: ``<bool>`` - 表示该节点是否为主节点。
-  ``isvp``: ``<bool>`` - 表示该节点是否为VP节点。
-  ``status``: ``<number>`` - 表示该节点的状态，值为\ ``0``\ 表示节点处于Alive状态，值为\ ``1``\ 表示节点处于Pending状态，值为\ ``2``\ 表示节点处于Stop状态。
-  ``delay``: ``<number>`` - 表示该节点与本节点的延迟时间（单位ns），若为0，则为本节点。

Example
^^^^^^^

.. code:: bash

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

.. _node_getNodeHash:

node_getNodeHash
~~~~~~~~~~~~~~~~

获取当前节点哈希值（向哪个节点发送请求即获取那个节点的哈希值）。

Parameters
^^^^^^^^^^

无

Returns
^^^^^^^

1. ``<string>`` - 节点哈希值。

Example
^^^^^^^

.. code:: bash

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

.. _node_deleteVP:

node_deleteVP
~~~~~~~~~~~~~

删除VP节点。

说明：假设当前有5个VP节点，如果要删除第3个节点，则需要向其他4个节点都发送删除节点的请求，3号节点才能成功删除。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``nodehash``: ``<string>`` - 要删除的VP节点的哈希值。

Returns
^^^^^^^

1. ``<string>`` - 请求发送成功的mesage。

Example
^^^^^^^

.. code:: bash

    # Request
    curl -X POST --data ' {"jsonrpc":"2.0", "namespace":"global", "method":"node_deleteVP","params":[{"nodehash":"c605d50c3ed56902ec31492ed43b238b36526df5d2fd6153c1858051b6635f6e"}],"id":1}'

    # Response
    {
      "jsonrpc": "2.0",
      "namespace":"global",
      "id": 1,
      "code": 0,
      "message": "SUCCESS",
      "result": "successful request to delete vp node"
    }

.. _node_deleteNVP:

node_deleteNVP
~~~~~~~~~~~~~~

VP节点断开与NVP节点的连接。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``nodehash``: ``<string>`` - 要删除的NVP节点的哈希值。

Returns
^^^^^^^

1. ``<string>`` - 请求发送成功的mesage。

Example
^^^^^^^

.. code:: bash

    # Request
    curl -X POST --data ' {"jsonrpc":"2.0","namespace":"global", "method":"node_deleteNVP","params":[{"nodehash":"c605d50c3ed56902ec31492ed43b238b36526df5d2fd6153c1858051b6635f6e"}],"id":1}'

    # Response
    {
      "jsonrpc": "2.0",
      "namespace":"global",
      "id": 1,
      "code": 0,
      "message": "SUCCESS",
      "result": "successful request to delete nvp node"
    }

.. _cert_getTCert:

cert_getTCert
~~~~~~~~~~~~~

获取节点颁发给用户的tcert证书。

Parameters
^^^^^^^^^^

1. ``<Object>``

-  ``pubkey``: ``<string>`` - 十六进制表示的pem格式公钥。

Returns
^^^^^^^

1. ``<Object>``

-  ``tcert``: ``<string>`` - tcert证书。

Example：获取tcert失败
^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

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
