API文档
^^^^^^^^^^^^

JSON-RPC是一个无状态且轻量级的远程过程调用(RPC)协议。它允许运行在基于socket、http等诸多不同消息传输环境的同一进程中，使用JSON作为数据格式。

发送一个请求对象至服务端代表一个RPC调用，一个请求对象包含下列成员: 

jsonrpc
 指定JSON-RPC协议版本的字符，如果是2.0版本,则必须准确写为 “2.0”。

method
 表示所要调用方法名称的字符串，以RPC开头的方法名,用英文句号(U+002E or ASCII 46)连接的为预留给RPC内部的方法名及扩展名,且不能在其他地方使用。

params
 调用方法所需要的结构化参数值，该成员参数可以被省略。

id
 已建立客户端的唯一标识id，该值必须包含一个字符串、数值或NULL值。如果不包含该成员则被认定为是一次通知调用。该值一般不为NULL,若为数值则应为整数。

当发起一次rpc调用时，服务端都必须回复一个JSON对象作为响应，响应对象包含下列成员: 

jsonrpc
 指定JSON-RPC协议版本的字符串，如果是2.0版本，则必须准确写为“2.0”。
method
 该成员在成功时必须包含，当调用方法失败时必须不包含该成员。服务端中的被调用方法决定了该成员的值。
error
 该成员在失败时必须包含，当没有错误引起时，不包含该成员，若引起错误,则该成员对象将包含code和message两个属性。
id
 该成员必须包含，该成员值必须与请求对象中的id成员值一致。若在检查请求对象id时错误(例如参数错误或无效请求)，则该值必须为空值（NULL）。

API设计
-------------

接口类别
>>>>>>>>>>>>

平台接口按照提供的功能服务不同，主要分为：

区块服务(block)、节点服务(node) 、合约服务(contract) 、MQ服务(sub)、数据归档服务(archive) 、文件共享服务(fm)、链级权限服务(config、account) 、接口权限服务(auth)、simulate合约服务(simulate)。

请求参数
>>>>>>>>>>>>>

接口设计遵循JSON-RPC 2.0规范，所有HTTP请求均为POST请求，请求参数包括：

jsonrpc
 指定JSON-RPC协议版本的字符串，如果是2.0版本,则必须准确写为 “2.0”。

namespace
 表示该条请求发送给哪个分区去处理。

method
 表示所要调用方法名称的字符串，格式为：(服务前缀)_(方法名)。

params
 调用方法所需要的结构化参数值，该成员参数可以被省略。

id
 已建立客户端的唯一标识id，该值必须包含一个字符串、数值。

::

 //  Request
  
 curl -X POST  -d '{
 "jsonrpc":"2.0",
 "method":"block_latestBlock",
 "namespace":"global",
 "params":[],
 "id":1}'  
 localhost:8081

返回参数
>>>>>>>>>>>>

返回参数包括:

`jsonrpc`: 指定JSON-RPC协议版本的字符串,如果是2.0版本,则必须准确写为 “2.0”。

`namespace`: 表示该条请求所属分区。

`code`: 状态码，若成功，则为0，其他状态码详见下表“code错误码”。

`message`: 错误信息，若成功，则为“SUCCESS”，否则为错误详细信息。

`result`: 请求返回的结果，下文接口描述的表格中返回值即表示这个字段返回的值。

`id`: 已建立客户端的唯一标识id,该值必须包含一个字符串、数值。

::

 //  Result  
 {
   "jsonrpc":  "2.0",
   "namespace":  "global",
   "id": 1,
 	 "code":  0,
	 "message":  "SUCCESS",
   "result": {
     "version":  "1.0",
     "number":  "0x3",
     "hash":  "0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
     "parentHash":  "0x2b709670922de0dda68926f96cffbe48c980c4325d416dab62b4be27fd73cee9",
     "writeTime":  1481778653997475900,
     "avgTime":  "0x2",
     "txcounts":  "0x1",
     "merkleRoot":  "0xc6fb0054aa90f3bfc78fe79cc459f7c7f268af7eef23bd4d8fc85204cb00ab6c",
      "transactions": [
       {
         "version": "1.0",
         "hash":  "0xf57a6443d08cda4a3dfb8083804b6334d17d7af51c94a5f98ed67179b59169ae",
         "blockNumber": "0x3",
	 			 "blockHash":"0x00acc3e13d8124fe799d55d7d2af06223148dc7bbc723718bb1a88fead34c914",
         "txIndex": "0x0",
         "from":  "0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
         "to":  "0xaeccd2fd1118334402c5de1cb014a9c192c498df",
         "amount":  "0x0",
         "timestamp": 1481778652973000000,
         "nonce":  3573634504790373,
         "executeTime": "0x2",
         "payload":  "0x81053a70000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000001c8"
       }
     ]
   }
 }

例如，若接口调用成功，返回字段包括：jsonrpc、id、code、message、result，且code值为0，message值为“SUCCESS”，用户可通过这两个字段的值来判断接口调用是否成功；若调用失败，则code为非0值，message为错误信息。

code错误码
>>>>>>>>>>>>>>>

-32700
 服务端接收到无效的json。该错误发送于服务器尝试解析json文本

-32600
 无效的请求（比如非法的JSON格式）

-32601
 方法不存在或者无效

-32602
 无效的方法参数

-32603
 JSON-RPC内部错误

-32000
 Hyperchain内部错误或者空指针或者节点未安装solidity环境

-32001
 查询的数据不存在

-32002
 余额不足

-32003
 签名非法

-32004
 合约部署出错

-32005
 合约调用出错

-32006
 系统繁忙

-32007
 交易重复

-32008
 合约操作权限不够

-32009 
 (合约)账户不存在

-32010
 namespace不存在

-32011
 账本上无区块产生，查询最新区块的时候可能抛出该错误

-32012
 订阅不存在

-32013
 数据归档、快照相关错误


**下文将给出几个错误示例。**

**Example1: 请求无效**

::

 // Request 
 curl -X GET localhost:8081 
 // Result  
 {
    "jsonrpc": "2.0",
    "code": -32600,
    "message": "EOF"
 }

**Example2: 交易重复**

::

  // Request
 curl  localhost:8081 --data  '{
  "jsonrpc":"2.0",
  "namespace":"global",
  "method":"contract_deployContract",
  "params":[{
   "from":"0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd",
   "nonce":1204154551977848,
   "payload":"0x60606040526000805463ffffffff19168155609e908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d146056578063d09de08a14607c575b6002565b346002576000805463ffffffff8116600435016024350163ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b3460025760546000805463ffffffff19811663ffffffff90911660010117905556",
   "signature":"97501a6f16b00916298f93fb865d296074e2bcbbee87d276c3d968ebf5b4bdab5ed01b679d24dc918180840414d27fbfe3c21fc5f3534b866024f6995c330e0b00",
   "timestamp":1486995057580182971
  }], 
	"id":"1"}'
  
  // Result
 {
   "jsonrpc": "2.0", 
   "namespace":"global",
   "id": 1,
   "code": -32007,
   "message": "repeated tx"
 }

**Example3: 方法不存在**

::

 //  Request
 curl localhost:8081 --data  '{
 "jsonrpc":"2.0",
 "namespace":"global",
 "method":"tx_test",
 "params":["0x0e0758305cde33c53f8c2b852e75bc9b670c14c547dd785d93cb48f661a2b36a  "],
 "id":1
 }'

 // Result
 {
   "jsonrpc": "2.0",
	 "namespace":"global"
   "id": 1,
   "code":  -32601,
   "message": "The method  tx_test does not exist/is not available"
  }

交易服务
-------------

1. tx_sendTransaction：发送交易
2. tx_getTransactionByHash：根据交易哈希值查询交易详情
3. tx_getTransactionByBlockHashAndIndex：根据区块哈希值查询交易详情
4. tx_getTransactionByBlockNumberAndIndex：根据区块号查询交易详情
5. tx_getTransactionsCount：查询链上所有交易量
6. tx_getTxAvgTimeByBlockNumber：查询指定区块中交易平均处理时间
7. tx_getTransactionReceipt：查询指定交易回执信息
8. tx_getBlockTransactionCountByHash：根据区块哈希查询区块交易数量
9. tx_getBlockTransactionCountByNumber：根据区块号查询区块交易数量
10. tx_getTransactionsVersion：查询平台当前的交易版本号
11. tx_getTransactionsWithLimit：查询指定区块区间的交易（可用于分页）
12. tx_getTransactionsByTimeWithLimit：查询指定时间区间内的交易（可用于分页）
13. tx_getTransactionsByExtraID：查询extraId为指定值的交易
14. tx_getTransactionsByFilter：查询符合过滤条件的交易

接口详情
>>>>>>>>>>>>>

1. tx_sendTransaction(发送交易接口)
:::::::::::::::::::::::::::::::::::::::::

**参数：**

- from：<string> 账户地址
- to：<string> 账户地址
- value：<number>转账值
- signature：<string> 交易签名
- timestamp：<number> 交易发生的unix时间戳(单位ns)
- simulate：<bool> [可选] true表示交易不走共识，false表示走共识，默认为false
- nonce：<number> 16位的随机数，该值必须为十进制整数
- type：<string> [可选] 指定合约执行引擎，默认为evm，如果合约代码由java语言编写，则需要设置该值为hvm
- extra：<string> [可选] 额外信息
- extraIdInt64：<int64> [可选] int64类型的extraId数组
- extraIdString：<string> [可选] string类型的extraId数组

**返回值：**

- transactionHash：<string> 交易的哈希值，32字节的十六进制字符串

发送交易的时候用户可以自定义extraId字段的值，该值作为这笔交易的可索引信息，数组表示一笔交易可以有多个索引信息

由于平台交易结构的extraId的值支持int64数组、string数组或者包含int64和string的数组，因此本接口针对这两种类型分别定义了 **extraIdInt64** 和 **extraIdString** 两个参数，分别用来接收不同数据类型的索引值。允许接收到的发送交易请求参数里不包含这两个字段、只包含extraIdInt64字段、只包含extraIdString字段或者同时包含这两个字段。如果同时包含这两个字段，则最终生成的交易结构的ExtraId值为这两个数组的拼接， **数组拼接顺序为int64数组后面拼接string数组**。

2. tx_getTransactionsByHash(根据哈希查询交易接口)
:::::::::::::::::::::::::::::::::::::::::::::::::::::::

**参数：**

- transactionHash：<string> 交易的哈希值,32字节的十六进制字符串	

**返回值：**

- <TransactionResult> 交易详情

TransactionResult对象结构如下::

 {
    version: <string> 平台版本号
    hash: <string> 交易的哈希值,32字节的十六进制字符串
    blockNumber: <number> 交易所在的区块高度
    blockHash: <string> 交易所在区块哈希值
    txIndex: <number> 交易在区块中的交易列表的位置
    from: <string> 交易发送方的地址,20字节的十六进制字符串
    to: <string> 交易接收方的地址,20字节的十六进制字符
    cName: <string> 交易接收方的名称，合约的命名
    amount: <number> 交易量
    timestamp: <number> 交易发生的unix时间戳(单位 ns)
    nonce: <number> 16位随机数
    extra: <string> 交易的额外信息
    executeTime: <string> 交易的处理时间(单位ms)
    payload: <string> 部署合约与调用合约的时候才有这个值，可以通过这个值追溯到合约调用的方法以及调用传入的参数
    signature: <string> 交易签名
    invalid: <boolean> 交易是否不合法
    invalidMsg: <string> 交易的不合法信息
    blockTimestamp:<number> 交易打包时间
    blockWriteTime:<number> 交易写块时间
  }

不合法的交易invalid值为true， invalidMsg可能为：

- 合约部署失败DEPLOY_CONTRACT_FAILED
- 合约方法调用失败INVOKE_CONTRACT_FAILED
- 签名非法SIGFAILED
- 余额不足OUTOFBALANCE
- 合约操作权限不够INVALID_PERMISSION

3. tx_getTransactionsByBlockHash(根据区块哈希查询交易接口)
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**RPC方法：**

tx_getTransactionByBlockHashAndIndex

**参数：**

- blockHash: <string> 区块的哈希值，32字节的十六进制字符串
- index: <number> 区块中的位置	

**返回值：**

- <TransactionResult> 交易详情

index可以是十进制整数或者进制字符串。

**TransactionResult** 对象结构请参见getTransactionsByHash接口。

4. tx_getTransactionsByBlockNumber(根据区块号查询交易接口)
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**RPC方法：**

- tx_getTransactionByBlockNumberAndIndex

**参数：**

- blockNumber: <blockNumber> 区块的高度
- index: <number> 区块中的位置

**返回值：**

- <TransactionResult> 交易详情

blockNumber可以是十进制整数或者进制字符串，可以是“latest”字符串表示最新的区块；index可以是十进制整数或者进制将字符串。

**TransactionResult** 对象结构请参见getTransactionsByHash接口。

5. tx_getTransactionsCount(查询链上所有交易量接口)
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**RPC方法：**

- tx_getTransactionsCount

**参数：**

- 无

**返回值：**

- count: <number> 交易数量
- timestamp: <number> 响应时间戳(单位ns)

6. tx_getTxAvgTime(根据指定的区块区间计算出每笔交易的平均处理时间接口)
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**RPC方法：**

- tx_getTxAvgTimeByBlockNumber

**参数：**

- from: <blockNumber> 起始区块号
- to: <blockNumber> 终止区块号

**返回值：**

- time: <number> 交易的平均处理时间（单位ms）

blockNumber可以是十进制整数或者进制字符串，可以是“latest”字符串表示最新的区块。

from必须小于等于to，否则会返回error。如果 from 和 to 的值一样，则表示计算的是当前指定区块交易的平均处理时间。

7. tx_getTransactionReceipt(根据交易hash返回交易回执信息接口)
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**RPC方法：**

- tx_getTransactionReceipt

**参数：**

- transactionHash: <string> 交易hash

**返回值：**

- <Receipt> 交易回执

Receipt对象结构如下::

 {
    version: <string> 版本号
    txHash: <string> 交易哈希
    vmType: <string> 该笔交易执行引擎类型，EVM或HVM、BVM
    contractAddress: <string> 合约地址
    contractName: <string> 合约命名
    ret: <string> 合约执行的结果
    log: <[Log]> 合约中的event log信息
 }

Log对象结构如下::

 {
    address: <string> 产生事件日志的合约地址   
    topics: <[string]> 一系列的topic，第一个topic是event的唯一标识
    data: <string> 日志信息
    blockNumber: <int> 所属区块的区块号
    blockHash: <string> 所属区块的区块哈希
    txHash: <string> 所属交易的交易哈希
    txIndex: <int> 所属交易在当前区块交易列表中的偏移量
    index: <int> 该日志在本条交易产生的所有日志中的偏移量
 }

若调用这个方法得到错误码-32001，则表明这笔交易还没被确认，若返回了error，则可能是：

- 余额不足OUTOFBALANCE，对应code是-32002;
- 签名非法SIGFAILED，对应code是-32003；
- 合约部署失败DEPLOY_CONTRACT_FAILED，对应code是-32004；
- 合约方法调用失败INVOKE_CONTRACT_FAILED，对应code是-32005；
- 合约操作权限不够INVALID_PERMISSION，对应code是-32008;

8. tx_getBlockTransactionCountByHash(根据区块hash返回区块交易数量接口)
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**RPC方法：**

- tx_getBlockTransactionCountByHash

**参数：**

- blockHash: <string> 区块hash

**返回值：**

- count: <number> 交易数量

9. tx_getBlockTransactionCountByNumber(根据区块number返回区块交易数量接口)
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**RPC方法：**

- tx_getBlockTransactionCountByNumber

**参数：**

- blockNumber: <string> 区块号

**返回值：**

- count: <number> 交易数量

10. tx_getTransactionsWithLimit[查询指定区块区间的交易(可分页)]
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**注意：该接口最多返回5000条交易，返回值最大为8Mb。**

**RPC方法：**

- tx_getTransactionsWithLimit

**参数：**

- from: <blockNumber> 起始区块号
- to: <blockNumber> 终止区块号
- metadata: <Metadata> [可选] 指定本次查询请求的元信息，定义见下文

**返回值：**

- hasmore: <bool> 表示是否有更多符合查询条件[from, to]的交易未被返回
- data: [<TransactionResult>] 交易详情数组

blockNumber可以是十进制整数或者进制字符串，可以是“latest”字符串表示最新的区块。from必须小于等于to，否则会返回error。

Metadata 对象结构如下::

 {
    pagesize: <int32> [可选] 表示本次查询返回多少条交易。如果未指定，则pagesize默认值为5000，如果超过5000，则使用节点默认值5000。如果返回值的条数未达到pagesize时大小就超过了8Mb，则不再继续往下查询，直接返回，返回结果里hasmore置为true。
    bookmark: <Bookmark> [可选] 表示本次查询的书签位置，即起始位置，返回的结果里不包含用户指定的书签所对应的交易。如果未指定且backward为false，则默认从最新区块开始向前遍历，如果未指定且backward为true，则默认从创世区块开始向后遍历。   
    backward: <bool> [可选] 表示本次查询的方向，false表示以起始位置为起点从高区块往低区块遍历，true表示以起始位置为起点从低区块往高区块遍历，默认查询方向为false。
 }

Bookmark对象结构如下::

 {
    blkNum: <uint64> 交易所在区块号
    txIndex: <int64> 交易索引号，即交易在区块内的位置
 }

客户端可以利用该接口实现区块链的“分页查询”，根据返回结果里的hasmore来判断是否要继续查询剩下的数据。下面对该接口的参数做进一步说明：

- 如果查询条件未指定metadata，则 metadata.backward 默认为 false、书签位置默认为最新区块最后一条交易，从书签位置开始往前遍历，limit默认为5000条、8Mb。
- 如果查询条件里未指定metadata.bookmark，若 metadata.backward 为 false，则默认书签位置为最新区块的最后一条交易，若metadata.backward 为 true，则默认书签位置为第一个区块的第一条交易。
- 如果查询条件里指定的书签位置metadata.bookmark 位于区块区间 [1, latest] 里，则我们需要根据metadata.backward 的值来调整遍历的区块区间。如果 metadata.backward 为false，则区块区间调整为[1, metadata.bookmark.blkNum]；如果 metadata.backward 为true，则区块区间调整为[metadata.bookmark.blkNum, latest]。
- 当backward 为 false 的时候，如果指定的书签位置在区块1之前，则接口返回error。当backward为 true 的时候，如果指定的书签位置在最新区块之后，则接口返回error。

TransactionResult对象结构请参见getTransactionsByHash接口

11. tx_getTransactionsByTimeWithLimit[查询指定时间区间内的所有交易（可用于分页）]
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

**注意：该接口最多返回5000条交易，返回值最大为8Mb。**

**RPC方法：**

- tx_getTransactionsByTimeWithLimit	

**参数：**

- startTime: <number> 起始unix时间戳(单位ns)
- endTime: <number> 结束unix时间戳(单位ns)
- metadata: <Metadata> [可选] 指定本次查询请求的元信息，定义见下文
- filter: <Filter> [可选] 指定本次查询过滤条件，包括交易发送方地址、交易接收方地址和交易extraId,用于筛选符合时间条件的交易里又符合过滤条件的交易	hasmore: <bool> 表示是否有更多符合查询条件[from, to]的交易未被返回
- data: <TransactionResult> 交易详情数组

Metadata 对象结构如下::

 {
    pagesize: <int32> [可选] 表示本次查询返回多少条交易。如果未指定，则pagesize默认值为5000，如果超过5000，则使用节点默认值5000。如果返回值的条数未达到pagesize时大小就超过了8Mb，则不再继续往下查询，直接返回，返回结果里hasmore置为true。
    bookmark: <Bookmark> [可选] 表示本次查询的书签位置，即起始位置，返回的结果里不包含用户指定的书签所对应的交易。如果未指定且backward为false，则默认从最新区块开始向前遍历，如果未指定且backward为true，则默认从创世区块开始向后遍历。   
    backward: <bool> [可选] 表示本次查询的方向，false表示以起始位置为起点从高区块往低区块遍历，true表示以起始位置为起点从低区块往高区块遍历，默认查询方向为false。
 }

Bookmark对象结构如下::

 {
    blkNum: <uint64> 交易所在区块号
    txIndex: <int64> 交易索引号，即交易在区块内的位置
 }

Filter对象结构如下::

 {
    extraId: <array> [必选] 指定交易extraId的值
    txTo: <string> [可选] 指定交易接收方的地址
    txName:<string> [可选] 执行交易接收方的名字
    extraId: <array> [可选] 指定交易extraId的值
 }

客户端可以利用该接口实现区块链的“分页查询”，根据返回结果里的hasmore来判断是否要继续查询剩下的数据。下面对该接口的参数做进一步说明：

- 如果查询条件未指定metadata，则 metadata.backward 默认为 false、书签位置默认为最新区块最后一条交易，从书签位置开始往前遍历，limit默认为5000条、8Mb。
- 如果查询条件里未指定metadata.bookmark，若 metadata.backward 为 false，则默认书签位置为最新区块的最后一条交易，若metadata.backward 为 true，则默认书签位置为第一个区块的第一条交易。
- 如果查询条件里指定的书签位置metadata.bookmark 位于区块区间 [1, latest] 里，则我们需要根据metadata.backward 的值来调整遍历的区块区间。如果 metadata.backward 为false，则区块区间调整为[1, metadata.bookmark.blkNum]；如果 metadata.backward 为true，则区块区间调整为[metadata.bookmark.blkNum, latest]。
- 当backward 为 false 的时候，如果指定的书签位置在区块1之前，则接口返回error。当backward为 true 的时候，如果指定的书签位置在最新区块之后，则接口返回error。

TransactionResult对象结构请参见getTransactionsByHash接口

12. tx_getTransactionsByExtraID[查询指定extraId为指定值的所有交易（可分页）]
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

根据extraId 的值和交易接收方地址（可选）来查询符合条件的交易。该接口最多返回5000条交易。为了提供更加丰富的查询，该接口支持针对extraId的精确查询和模糊查询，详情见下表的mode入参。

说明：只有当接收请求的节点开启数据索引（或者说开启索引数据库）功能时，该接口才可用。该接口必须要指定 extraId 的值，交易接收方地址为可选项。

**RPC方法：**

- tx_getTransactionsByExtraID

**参数：**

- detail: <bool> [可选] 表示是否返回交易详情，默认为false;若为true，则返回的结果为交易详情[<TransactionResult>]，否则为交易摘要[<TransactionSummary>]
- mode: <int> [可选] 表示本次查询请求的查询模式，目前有0、1、2三个值可选，默认为0，0 表示按序精确查询模式，即筛选出的的交易 extraId 数组的数值和顺序都与查询条件完全一致；1 表示非按序精确查询模式，即筛选出的交易 extraId 数组包含查询条件里指定的全部数值，顺序无要求；2 表示非按序匹配查询模式，即筛选出的交易 extraId 数组包含部分或全部查询条件指定的值，且顺序无要求
- metadata: <Metadata> [可选] 指定本次查询的起始位置、查询方向以及返回的条数，若未指定，则默认从最新区块开始向前查询，默认返回条数上限是5000条
- filter: <Filter> [必选] 指定本次查询过滤条件，包括交易extraId和交易接收方地址

**返回值：**
- hasmore: <bool> 表示是否有更多符合查询条件的交易未被返回
- data: [<TransactionResult>] 交易详情数组或[<TransactionSummary>] 交易摘要数组

Metadata 对象结构如下::

 {
    pagesize: <int32> [可选] 表示本次查询返回多少条交易。如果未指定，则pagesize默认值为5000，如果超过5000，则使用节点默认值5000。如果符合条件的交易数量实际上超过pagesize，则返回结果里hasmore为true。   
    bookmark: <Bookmark> [可选] 表示本次查询的书签位置，即起始位置，返回的结果里不包含用户指定的书签所对应的交易。如果未指定且backward为false，则默认从最新区块开始向前遍历，如果未指定且backward为true，则默认从创世区块开始向后遍历。   
    backward: <bool> [可选] 表示本次查询的方向，false表示以起始位置为起点从高区块往低区块遍历，true表示以起始位置为起点从低区块往高区块遍历，默认查询方向为false。
 }

Bookmark对象结构如下::

 {
    blkNum: <uint64> 交易所在区块号
    txIndex: <int64> 交易索引号，即交易在区块内的位置
 }

Filter对象结构如下::

 {
    extraId: <array> [必选] 指定交易extraId的值
    txTo: <string> [可选] 指定交易接收方的地址
    txName:<string> [可选] 执行交易接收方的名字
 }

客户端可以利用该接口实现区块链的“分页查询”，根据返回结果里的hasmore来判断是否要继续查询剩下的数据。下面对该接口的参数做进一步说明：

- 如果查询条件未指定metadata，则 metadata.backward 默认为 false、书签位置默认为最新区块最后一条交易，从书签位置开始往前遍历，limit默认为5000条。
- 如果查询条件里未指定metadata.bookmark，若 metadata.backward 为 false，则默认书签位置为最新区块的最后一条交易，若metadata.backward 为 true，则默认书签位置为第一个区块的第一条交易。
- 如果查询条件里指定的书签位置metadata.bookmark 位于区块区间 [1, latest] 里，则我们需要根据metadata.backward 的值来调整遍历的区块区间。如果 metadata.backward 为false，则区块区间调整为[1, metadata.bookmark.blkNum]，如果 metadata.backward 为true；则区块区间调整为[metadata.bookmark.blkNum, latest]。
- 当backward 为 false 的时候，如果指定的书签位置在区块1之前，则接口返回error。当backward为 true 的时候，如果指定的书签位置在最新区块之后，则接口返回error。

TransactionSummary对象结构如下::

 {
    txHash: <string> 交易哈希值
    blkNum: <uint64> 交易所在区块号
    txIndex: <int64> 交易在区块里的索引值
    from: <string> 交易发送方地址
    to: <string> 交易接收方地址
    extraId:<any> 交易的extraId值
 }

TransactionResult对象结构请参见getTransactionsByHash接口。

13. tx_getTransactionsByFilter[查询符合过滤条件的所有交易（可分页）]
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

根据过滤条件来得到符合查询条件的交易。为了提供更加丰富的查询，该接口支持多条件与查询和多条件或查询，详情见下表的mode入参。

**注意：只有当接收请求的节点开启数据索引（或者说开启索引数据库）功能时，该接口才可用。**

**RPC方法：**

- tx_getTransactionsByFilter

**参数：**

- detail: <bool> [可选] 表示是否返回交易详情，默认为false；若为true，则返回的结果为交易详情[ <TransactionResult>]，否则为交易摘要[<TransactionSummary>]
- mode: <int> [可选] 表示本次查询请求的查询模式，目前有 0、1 两个值可选，默认为0，0 表示多条件与查询模式，即交易对应字段的值与查询条件里所有指定的字段值都完全一致；1 表示多条件或询模式，即交易对应字段的值至少有一个等于查询条件里指定的字段值
- metadata: <Metadata> [可选] 指定本次查询的起始位置、查询方向以及返回的条数。若未指定，则默认从最新区块开始向前查询，默认返回条数上限是5000条
- filter: <Filter> [必选] 指定本次查询过滤条件

**返回值：**

- hasmore: <bool> 表示是否有更多符合查询条件的交易未被返回
- data: [<TransactionResult>] 交易详情数组或者[<TransactionSummary>] 交易摘要数组

Metadata对象结构如下::

 {
    pagesize: <int32> [可选] 表示本次查询返回多少条交易。如果未指定，则pagesize默认值为5000，如果超过5000，则使用节点默认值5000。如果符合条件的交易数量实际上超过pagesize，则返回结果里hasmore为true  
    bookmark: <Bookmark> [可选] 表示本次查询的书签位置，即起始位置，返回的结果里不包含用户指定的书签所对应的交易。如果未指定且backward为false，则默认从最新区块开始向前遍历，如果未指定且backward为true，则默认从创世区块开始向后遍历  
    backward: <bool> [可选] 表示本次查询的方向，false表示以起始位置为起点从高区块往低区块遍历，true表示以起始位置为起点从低区块往高区块遍历，默认查询方向为false
 }

Bookmark对象结构如下::

 {
    blkNum: <uint64> 交易所在区块  
    txIndex: <int64> 交易索引号，即交易在区块内的位置
 }

Filter对象结构如下::

 {
    txHash: <string> [可选] 指定交易的哈希值
    blkNumber: <uint64> [可选] 指定交易所在的区块号
    txIndex: <int64> [可选] 指定交易在区块内的索引位置
    txFrom: <string> [可选] 指定交易发送方的地址
    txTo: <string> [可选] 指定交易接收方的地
    txName:<string> [可选]执行交易接收方的名字
    extraId: <array> [可选] 指定交易extraId的值
 }

客户端可以利用该接口实现区块链的“分页查询”，根据返回结果里的hasmore来判断是否要继续查询剩下的数据。下面对该接口的参数做进一步说明：

- 如果查询条件未指定metadata，则 metadata.backward 默认为 false、书签位置默认为最新区块最后一条交易，从书签位置开始往前遍历，limit默认为5000条。
- 如果查询条件里未指定metadata.bookmark，若 metadata.backward 为 false，则默认书签位置为最新区块的最后一条交易，若metadata.backward 为 true，则默认书签位置为第一个区块的第一条交易。
- 如果查询条件里指定的书签位置metadata.bookmark 位于区块区间 [1, latest] 里，则我们需要根据metadata.backward 的值来调整遍历的区块区间。如果 metadata.backward 为false，则区块区间调整为[1, metadata.bookmark.blkNum]；如果 metadata.backward 为true，则区块区间调整为[metadata.bookmark.blkNum, latest]。
- 当backward 为 false 的时候，如果指定的书签位置在区块1之前，则接口返回error。当backward为 true 的时候，如果指定的书签位置在最新区块之后，则接口返回error。

TransactionSummary对象结构如下::

 {
    txHash: <string> 交易哈希值
    blkNum: <uint64> 交易所在区块号
    txIndex: <int64> 交易在区块里的索引值
    from: <string> 交易发送方地址
    to: <string> 交易接收方地址
    extraId:<any> 交易的extraId值
 }

TransactionResult对象结构请参见getTransactionsByHash接口。

区块服务
-------------

接口概览
>>>>>>>>>>>>>>>>


