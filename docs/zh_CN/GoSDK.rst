.. _GoSDK:

gosdk 使用手册
^^^^^^^^^^^^^^^^^^

第一章 前言
=============

此SDK文档面向Hyperchain和flato的区块链平台的应用开发者，提供Hyperchain和flato GoSDK的使用指南。


`Gosdk rpc 接口文档 <https://upload.filoop.com/RTD-Hyperchain%2FGosdk%20rpc%20%E6%8E%A5%E5%8F%A3%E6%96%87%E6%A1%A3.pdf>`_

`Gosdk grpc 接口文档 <https://upload.filoop.com/RTD-Hyperchain%2FGosdk%20grpc%20%E6%8E%A5%E5%8F%A3%E6%96%87%E6%A1%A3.pdf>`_

1.1 配置
------------------

1.1.1 目录结构
>>>>>>>>>>>>>>>>

所有配置文件、证书都应放在一个文件夹下，默认配置文件夹名为 `conf` ，`hpc.toml` 在配置文件夹下根目录，所有配置为文件路径都应相对于配置文件夹存放，例如我们以默认文件夹名 `conf` 为例::

  conf
  ├── certs
  │   ├── sdkcert.cert
  │   ├── sdkcert.priv
  │   ├── unique.priv
  │   └── unique.pub
  │   ├── tls
  │   │   ├── tls_peer.cert
  │   │   ├── tls_peer.priv
  │   │   └── tlsca.ca
  └── hpc.toml



certs目录下的证书需要对应于Flato或Hyperchain节点，在Flato或Hyperchain部署文档的 **证书配置** 一章节中，我们可以获得 **证书套件** ，证书套件目录图如下图所示（以flato为例）。其中sdkcerts中的四个文件对应于上述conf/certs目录下的四个文件，需要向几号节点发送交易，则选用几号节点的证书进行替换。

.. image:: ../../images/GoSDK1.png
    :align: center
    :alt: 证书套件目录


1.1.2 配置文件
>>>>>>>>>>>>>>>>

我们通过读取hpc.toml文件来加载配置::


  title = "GoSDK configuratoin file"

  namespace = "global"

  #发送重新连接请求间隔(/ms)
  reConnectTime = 10000

  [jsonRPC]
      # local environment
      nodes = ["localhost","localhost","localhost","localhost"]

      # JsonRpc connect port
      # local ports
    ports = ["8081", "8082", "8083", "8084"]

  [webSocket]
      # webSocket connect port
      ports = ["10001", "10002", "10003", "10004"]
  [grpc]
      ports = ["11001", "11002", "11003", "11004"]

  [polling]
      #重发次数
      resendTime = 10
      #第一次轮训时间间隔 unit /ms
      firstPollingInterval = 100
      #发送一次,第一次轮训的次数
      firstPollingTimes = 10
      #第二次轮训时间间隔 unit /ms
      secondPollingInterval = 1000
      #发送一次,第二次轮训的次数
      secondPollingTimes = 10

  [privacy]
      #send Tcert during the request or not
      sendTcert = true
      #if sendTcert is true , you should add follow path.
      #the paths followed are relative to conf root path
      sdkcertPath = "certs/sdkcert.cert"
      sdkcertPrivPath = "certs/sdkcert.priv"
  #    sdkcertPath = "certs/sdkcert_cfca.cert"
  #    sdkcertPrivPath = "certs/sdkcert_cfca.priv"
      uniquePubPath = "certs/unique.pub"
      uniquePrivPath = "certs/unique.priv"
      cfca = true

  [security]
      #Use Https
      https = false
      #If https is true, you shoule add follow properties
      #the paths followed are relative to conf root path
      tlsca = "certs/tls/tlsca.ca"
      tlspeerCert = "certs/tls/tls_peer.cert"
      tlspeerPriv = "certs/tls/tls_peer.priv"

  [log]
      #设置日志输出门槛
      #"CRITICAL","ERROR","WARNING","NOTICE","INFO","DEBUG",
      log_level = "INFO"
      #存放日志文件夹
      log_dir = "../logs"

  [transport]
      # MaxIdleConns controls the maximum number of idle (keep-alive)
    # connections across all hosts. Zero means no limit.
      maxIdleConns = 0
    # MaxIdleConnsPerHost, if non-zero, controls the maximum idle
    # (keep-alive) connections to keep per-host. If zero,
    # DefaultMaxIdleConnsPerHost is used.
    maxIdleConnsPerHost = 10
    # MaxRecvMsgSize, set max recv msg size for grpc /b
    maxRecvMsgSize = 51200
    # MaxSendMsgSize, set max sendMsg size for grpc /b
    maxSendMsgSize = 51200
    #DailTimeout dail time out
    dailTimeout = 5
    # MaxLifetime(unused), set the connections max life time, if zero, use default value, /s
    maxLifetime = 0
    # MaxStreamLifeTime, set the stream max life time, if zero, no limit, /s
    maxStreamLifeTime = 5

  [inspector]
      enable = true
      defaultAccount = "keystore/0xfc546753921c1d1bc2d444c5186a73ab5802a0b4"
      accountType = "ecdsa"

  [tx]
      # if it is use for hyperchain, please use 1.0 to replace default
      # if use for flato, please use 2.0 to replace default
      version = "2.5"


1.1.3 配置项释义
>>>>>>>>>>>>>>>>>>>>>

.. list-table::
 :widths: 25 25 25 25
 :align: center
 :header-rows: 1

 * - 一级
   - 二级
   - 释义
   - 默认值
 * - namespace
   -
   - 为初始化时传入的namespace
   - “global”
 * - jsonRP
   - nodes
   - 表示平台各节点的IP
   - ["localhost","localhost","localhost","localhost"]
 * - jsonRP
   - ports
   - 各节点rpc服务的端口
   - ["8081", "8082", "8083", "8084"]
 * - webSocket
   - ports
   - 为事件订阅服务的端口
   - ["10001", "10002", "10003", "10004"]
 * - grpc
   - ports
   - 各节点grpc服务端口
   - ["11001", "11002", "11003", "11004"]
 * - polling
   - resendTime
   - 重发次数
   - 10
 * - polling
   - firstPollingInterval
   - 第一次轮询的间隔，单位 ms
   - 100
 * - polling
   - firstPollingTimes
   - 第一次轮询的次数
   - 10
 * - polling
   - secondPollingInterval
   - 第二次轮询的间隔，单位 ms
   - 1000
 * - polling
   - secondPollingTimes
   - 第二次轮询的次数
   - 10
 * - privacy
   - sendTcert
   - 是否开启Tcert认证， 布尔值
   - true                            |
 * - privacy
   - sdkcertPath
   - 开启Tcert认证时签名用的公钥路径
   - "certs/sdkcert.cert"
 * - privacy
   - sdkcertPrivPath
   - 开启Tcert认证时签名用的私钥路径
   - "certs/sdkcert.priv"
 * - privacy
   - uniquePubPath
   - 表示请求tcert时所使用的请求体公钥路径
   - "certs/unique.pub"
 * - privacy
   - uniquePrivPath
   - 表示请求tcert时所使用的请求体私钥路径
   - "certs/unique.priv"
 * - privacy
   - cfca
   - 是否开启ca证书， 布尔值
   - true
 * - log
   - log_level
   - 日志等级， 分为"CRITICAL","ERROR","WARNING","NOTICE","INFO","DEBUG"
   - "INFO"
 * - log
   - log_dir
   - 存放日志文件的目录
   - "../logs"
 * - transport
   - maxIdleConns
   - 所有节点一共最大空闲活跃rpc连接数，如果为0，表示不设限制
   - 0
 * - transport
   - maxIdleConnsPerHost
   - 每个节点最大空闲活跃rpc连接数， 如果为0，表示使用系统默认配置
   - 10
 * - transport
   - maxRecvMsgSize
   - grpc双向流从节点接受的信息最大size，此处配置必须小于等于节点配置的值，单位是比特
   - 51200
 * - transport
   - maxSendMsgSize
   - 通过grpc双向流发送到节点的信息最大size，此处配置必须小于等于节点配置的值，单位是比特
   - 51200
 * - transport
   - dailTimeout
   - 建立grpc连接的超时时间，单位s
   - 5
 * - transport
   - maxLifetime
   - 判定接收不到grpc心跳多久后表示连接已失效，单位s， 如果为0，表示使用默认值
   - 0
 * - transport
   - maxStreamLifeTime
   - grpc双向流的存活时间，此处配置必须小于等于节点配置的值，0表示默认（即依靠节点配置的时间），单位秒
   - 5
 * - tx
   - version
   - sdk所对应的平台版本号
   - 2.5



- `namespace` 为初始化时传入的namespace；

- [jspnRPC]模块：`nodes` 表示平台各节点的IP；`ports` 也就是平台发送消息的端口。

- webSocket模块：`ports` 为事件订阅连接的端口，他们分别对应着node中配置的IP。

- grpc 模块: `ports` 为grpc连接的端口

- polling模块：`resendTime` 参数表示重发次数； `pollingInterval` 表示轮训去获取交易的时间间隔（分为第一次和第二次），单位为毫秒； `pollingTimes` 表示发送一次交易的轮训次数（分为第一次和第二次）。

- privacy模块： `sendTcert` 表示开启Tcert，请配合平台一起使用； `sdkcertPath` 和 `sdkcertPrivPath` 表示请求tcert时需要用来签名的公私钥对； `uniquePubPath` 和 `uniquePrivPath` 表示请求tcert时所使用的请求体公私钥对； `cfca` 表示开启cfca证书。 **需要注意证书路径均相对于** `hpc.toml` **文件** 。

- security模块： `https` 开启后则请求都会使用https来发送，`tlsca` 表示客户端需要验证的服务器ca，`tlspeerCert` 表示服务器需要验证的SDK的ca的公钥， `tlspeerPriv` 则为对应的ca公钥的私钥。

- log模块： `log_level` 表示输出的日志级别。 `log_dir` 表示存放日志的文件夹路径。

- 【transport】模块： `maxRecvMsgSize` ： 通过grpc双向流从节点接受的信息最大size，此处配置必须小于等于节点配置的值，单位是比特

- txVersion 需要和平台版本对应

- maxRecvMsgSize： 通过grpc双向流从节点接受的信息最大size，此处配置必须小于等于节点配置的值，单位是比特

- maxSendMsgSize： 通过grpc双向流发送到节点的信息最大size，此处配置必须小于等于节点配置的值，单位是比特

- dailTimeout： grpc连接创建超时时间

- maxStreamLifeTime： grpc双向流的存活时间，此处配置必须小于等于节点配置的值，0表示默认（即依靠节点配置的时间），单位秒


第二章 账户
==================



账户分为普通账户和DID账户，而根据加密方式可分为国密和非国密类型。目前根据加密方式不同一共有19种类型账户

非国密账户有：

.. list-table::
 :widths: 50 50
 :align: center
 :header-rows: 1

 * - 参数名
   - 编码
 * - ECKDF2
   - 0x01(暂不支持)
 * - ECDES
   - 0x02
 * - ECRAW
   - 0x03
 * - ECAES
   - 0x04
 * - EC3DES
   - 0x05
 * - ED25519DES
   - 0x21
 * - ED25519RAW
   - 0x22
 * - ED25519AES
   - 0x23
 * - ED255193DES
   - 0x24
 * - ECKDF2R1
   - 0x011(暂不支持)
 * - ECDESR1
   - 0x021
 * - ECRAWR1
   - 0x031
 * - ECAESR1
   - 0x041
 * - EC3DESR1
   - 0x051



国密账户有：

.. list-table::
 :widths: 50 50
 :align: center
 :header-rows: 1

 * - 参数名
   - 编码
 * - SMSM4
   - 0x11
 * - SMDES
   -  0x12
 * - SMRAW
   - 0x13
 * - SMAES
   - 0x14
 * - SM3DES
   - 0x15


2.1 普通账户
----------------

2.1.1 创建账户
>>>>>>>>>>>>>>>>>>>>>

**2.1.1.1 通过账户类型创建账户**

方法签名：

::

  `func NewAccountJson(acType, password string) (string, error)`


.. list-table::
 :widths: 50 50
 :align: center
 :header-rows: 1

 * - 参数
   - 释义
 * - 参数1 （acType）
   - 加密方式
 * - 参数2 （password）
   - 账户密码
 * - 返回值1
   - 账户JSON串
 * - 返回值2
   - error， 详情见第七章


应用实例:

 ::

  ac, err := account.NewAccountJson(account.ECDES, "123")


**2.1.1.2 创建国密账户**

方法签名：

 ::

  `func NewAccountSm2(password string) (string, error)`

说明： 创建sm2国密算法的accountJSON

.. list-table::
 :widths: 50 50
 :align: center
 :header-rows: 1

 * - 参数
   - 释义
 * - 参数 password
   - 账户密码，不为空则采用SMDES方式加密，为空则采用SMRAW方式加密（即不加密）
 * - 返回值1
   - 账户JSON串
 * - 返回值2
   - error， 详情见第七章



应用实例:

 ::

  ac, err := account.NewAccountSm2("123")


**2.1.1.3  通过pfx格式的证书创建账户**

方法签名:
::

   `NewAccountJsonFromPfx(password string, pfx []byte) (string, error)`


-  参数1 : 账户密码
-  参数2 : pfx格式证书
-  返回值1 : 账户JSON串
-  返回值2 : error， 详情见第七章

应用实例:

 ::

  temp, err := ioutil.ReadFile("idcert.pfx")
  if err != nil {
    return
  }
  ac, err := account.NewAccountJsonFromPfx("123", temp)
  if err != nil {
    return
  }




**2.1.1.4 创建ed25519类型账户**

方法签名：

::

  `func NewAccountED25519(password string) (string, error)`


- 参数1 （password） : 账户密码，若为空，则采用ED25519RAW方式加密, 否则采用ED25519DES方式加密
- 返回值1     : 账户JSON串
- 返回值2     : error， 详情见第七章

应用实例:

 ::

  ac, err := account.NewAccountED25519("123")




**2.1.1.5 创建ecdsa账户**

方法签名：

 ::

  ` func NewAccount(password string) (string, error)`


- 参数1 （password） : 账户密码，若为空，则采用ECRAW方式加密, 否则采用ECDES方式加密
- 返回值           :
- 返回值1          : 账户JSON串
- 返回值2          :| error， 详情见第七章

应用实例:

 ::

  ac, err := account.NewAccount("123")




**2.1.1.6 创建R1账户**

方法签名：

 ::

   `func NewAccountR1(password string) (string, error)`


- 参数1 （password） : 账户密码，若为空，则采用ECRAWR1方式加密, 否则采用ECDESR1方式加密
- 返回值1           : 账户JSON串
- 返回值2           : error， 详情见第七章

应用实例::

  ac, err := account.NewAccountR1("123")




### 2.1.2 生成key

方法签名：

::

  `func GenKeyFromAccountJson(accountJson, password string) (key interface{}, err error)`

- 参数1 （accountJson） : 账户json串
- 参数2 （password）    : 账户密码
- 返回值               :
- 返回值1              : 账户key
- 返回值2              :| error， 详情见第七章

应用实例::

  accountJson := `{"address":"0x4037f0011a9a5db6e426d702b14861e65b4f2a46","algo":"0x12","version":"4.0","publicKey":"0x040e15655c836ce58be6b5b3da93bd257d817809ec5a621b01d1024e1b6de7b38d0f12309accb6cd1a91faadffeabda0f8d2b835606daf01022e0b26ea0b9a3125","privateKey":"9de41e04445dc3024c3713a9cf043d55f6d8509fbfe6b3ebb108b89d83669328feb959b7d4642fcb"}`
  password := "12345678"
  key, err := account.GenKeyFromAccountJson(accountJson, password)


2.2 did账户
---------------

did账户与普通账户在国密场景一致，仅需要对非国密情况进行区分，由此带来的变化如下

2.2.1 创建账户
>>>>>>>>>>>>>>>>>>>

**2.2.1.1 通过ecdsa创建账户**

方法签名:

::

   `func NewAccountDID(password string) (string, error)`

- 参数1 （password） : 账户密码，若为空，则采用ECRAW方式加密, 否则采用ECDES方式加密
- 返回值            :
- 返回值1           : 账户JSON串
- 返回值2           : error， 详情见第七章

应用实例::

  ac, err := account.NewAccountDID("123")

**2.2.1.2 通过加密类型创建账户**

方法签名：

::

  `func NewDIDAccountJson(acType, password string) (string, error)`

- 参数1 （acType）   : 加密方式
- 参数2 （password） : 账户密码
- 返回值            :
- 返回值1           : 账户JSON串
- 返回值2           : error， 详情见第七章

应用实例::

  ac, err := account.NewDIDAccountJson(account.ECDES, "123")

**2.2.1.2 通过key生成DID账户**

方法签名：

 ::

  func NewDIDAccount(key Key, chainID, suffix string) (didKey *DIDKey)


- 参数1  : 账户key
- 参数2  : 链唯一的chainId
- 参数3  : DID账户的后缀，用于构成did账户的地址
- 返回值1 : DID账户

应用实例::

  password := "hyper"
  accountJson, _ = account.NewAccountSm2(password)
  chainId := "chainId_01"
  //生成普通账户key
  key, _ := account.GenKeyFromAccountJson(accountJson, password)
  suffix := "didAddr_suffix"
  didKey := account.NewDIDAccount(key.(account.Key), chainId, suffix)

2.2.2 生成key
>>>>>>>>>>>>>>>>>>>

**2.2.2.1 通过账户json生成key**

方法签名：

 `func GenDIDKeyFromAccountJson(accountJson, password string) (key interface{}, err error)`

- 参数1 （accountJson） : 账户json串
- 参数2 （password）    : 账户密码
- 返回值1              : 账户key
- 返回值2              : error， 详情见第七章

应用实例

 ::

  accountJson := `{"address":"0x4037f0011a9a5db6e426d702b14861e65b4f2a46","algo":"0x12","version":"4.0","publicKey":"0x040e15655c836ce58be6b5b3da93bd257d817809ec5a621b01d1024e1b6de7b38d0f12309accb6cd1a91faadffeabda0f8d2b835606daf01022e0b26ea0b9a3125","privateKey":"9de41e04445dc3024c3713a9cf043d55f6d8509fbfe6b3ebb108b89d83669328feb959b7d4642fcb"}`
  password := "12345678"
  key, err := account.GenDIDKeyFromAccountJson(accountJson, password)






第三章 交易体
====================

3.1 交易体结构
-----------------

交易是与hyperchain交互的重要形式，在GoSDK中，交易由 **Transaction** 结构体代表。GoSDK中交易体用户可配置参数共有如下（以下字段名首字母大写即为该字段的设值函数）：

【from】：交易发起方账户地址。 **必填** 。

【to】：交易接收方地址，合约调用时为合约地址。默认 **0x0** 。

【value】：转账金额。默认为**0**。

【payload】：交易荷载，在部署、调用、更新合约时使用。默认为 **空字符串** 。

【opcode】：操作码，在合约管理时使用， **1** 代表 *更新合约* ， **2** 代表 *冻结合约* ， **3** 代表 *解冻合约* 。默认为 **0** 。

【extra】：存证信息，可以在交易中附带一些信息。默认为 **空字符串** 。

【simulate】：配置当前交易是否为模拟交易。若为true，则交易数据不会存入链上数据库。默认为 **false** 。

【vmType】：当前交易针对的虚拟机，目前支持配置EVM、HVM、KVSQL、BVM、TRANSFER、FVM六种。默认为 **EVM** 。

【GasPrice】： gasPrice 默认为0，初始化 **rpc** 会从节点获取

【GasLimit】： gasLimit 默认 **1000000000**

【Participant】: participant 聚合签名相关

 ::

  // TransactionInfo is packaged result of TransactionInfo
  type TransactionInfo struct {
    Version        string
    Hash           string
    BlockNumber    uint64
    BlockHash      string
    TxIndex        uint64
    From           string
    To             string
    CName          string
    Amount         uint64
    Timestamp      uint64
    Nonce          uint64
    ExecuteTime    int64
    Payload        string
    Extra          string
    Invalid        bool
    InvalidMsg     string
    BlockTimestamp int64
    BlockWriteTime int64
      ExtraIdString       []interface{}
    Op                  int
    VmType              string
    GasPrice            int64
    GasLimit            int64
    ExpirationTimestamp int64
    Participant         Participant
  }

  type Participant struct {
    Initiator   []byte   `json:"initiator"`
    Withholding [][]byte `json:"withholding"`
  }

  // Oracle is packaged result of oracle event
  type Oracle struct {
    Url             string
    Method          string
    Header          map[string]string
    Body            string
    TxHash          string
    BizId           string
    CallBackAddress string
    CallBackMethod  string
    Uuid            string
    ContractAddress string
  }


  // TxReceipt is packaged result of transaction receipt
  type TxReceipt struct {
    TxHash          string
    PrivTxHash      string
    ContractAddress string
    ContractName    string
    Ret             string
    Log             []TxLog
    VMType          string
    Version         string
    Valid           bool
    ErrorMsg        string
    GasUsed         int64
    Oracles         Oracle
  }


3.2 交易体构造
---------------------

方法签名：

::

  `func NewTransaction(from string) *Transaction`

- 参数1  : 交易发起方地址
- 返回值1 : 交易体

应用实例

 ::

  trans := rpc.NewTransaction(address)


3.2.1 设置nonce
>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Nonce(nonce int64) *Transaction
  func (t *Transaction) SetNonce(nonce int64)


应用实例

 ::

  trans.Nonce(1)
  // or
  trans.SetNonce(1)


3.2.2 设置交易体时间戳
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名：

 ::

  func (t *Transaction) Timestamp(timestamp int64) *Transaction


应用实例

 ::

  trans.Timestamp(time.Now().Unix())



3.2.3 设置是否为Simulate交易
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Simulate(simulate bool) *Transaction


应用实例

 ::

  trans.Simulate(true)


3.2.4  设置交易对应虚拟机类型
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名：

 ::

  func (t *Transaction) VMType(vmType VMType) *Transaction


- 参数1（vmType） : 虚拟机类型，EVM, HVM, BVM, TRANSFER, KVSQL
- 返回值1        : 交易体实例

应用实例

 ::

  trans = trans.VMType(rpc.KVSQL)




3.2.5 转账交易构造
>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Transfer(to string, value int64) *Transaction


- 参数1（to）     : 收款方地址
- 参数2 （value） : 转账数
- 返回值1        : 交易体实例

应用实例

 ::

  trans.Transfer("bfa5bd992e3eb123c8b86ebe892099d4e9efb783", int64(0))




3.2.6 合约管理交易构造
>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Maintain(op int64, to, payload string) *Transaction


- 参数1（op）       : 操作码， 1 合约升级， 2 冻结合约， 3 解冻合约
- 参数2 （to）      : 操作的合约地址
- 参数3 （payload） : 合约bin， 可为空字符串
- 返回值1          : 交易体实例

应用实例

 ::

  contractAddress := "0x421a1fb06bd9c9fae9b8cdaf8a662cf3c41ffa10" // 合约部署后地址
  trans = trans.Maintain(1, contractAddress, "")


3.2.7 通过合约名管理合约的交易构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) MaintainByName(op int64, name, payload string) *Transaction


- 参数1 （op）      : 操作码， 1 合约升级， 2 冻结合约， 3 解冻合约
- 参数2 （name）    : 合约名
- 参数3 （payload） : 合约bin， 可为空字符串
- 返回值1          : 交易体实例

应用实例

 ::

  // 绑定合约名
  contractAddr := "0x421a1fb06bd9c9fae9b8cdaf8a662cf3c41ffa10"
  contractName := "demo"
  // setCName方法定义在下面
  setCName(contractAddr, contractName)
  trans.MaintainByName(2, contractName, "")


 ::

  func setCName(address, cName string) *bvm.Result {
    re, err := createProposalForSetCName(address, cName)
    var proposal bvm.ProposalData
    assert.NoError(err)
    result := bvm.Decode(re.Ret)
    assert.True(result.Success)
    _ = proto.Unmarshal([]byte(result.Ret), &proposal)

    voteProposal(int(proposal.Id))
    return executeProposal(int(proposal.Id))
  }

  func createProposalForSetCName(address, cName string) (*TxReceipt, StdError) {
    key, _ := account.NewAccountFromAccountJSON("", pwd)
    setCNameOpt := bvm.NewCNSSetCNameOperation(address, cName)
    cnsOpt := bvm.NewProposalCreateOperationForCNS(setCNameOpt)
    payload := bvm.EncodeOperation(cnsOpt)
    tx := NewTransaction(key.GetAddress().Hex()).Invoke(cnsOpt.Address(), payload).VMType(BVM)
    tx.Sign(key)
    return rpc.InvokeContract(tx)
  }

  func voteProposal(id int) {
    for i := 1; i < 6; i++ {
      var key account.Key
      if i < 4 {
        key, _ = account.NewAccountFromAccountJSON("", pwd)
      } else {
        key, _ = account.NewAccountSm2FromAccountJSON("", pwd)
      }
      opt := bvm.NewProposalVoteOperation(id, true)
      payload := bvm.EncodeOperation(opt)
      tx := NewTransaction(key.GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
      tx.Sign(key)
      re, err := rpc.InvokeContract(tx)
      assert.NoError(err)
      result := bvm.Decode(re.Ret)
      assert.True(result.Success)
    }
  }

  func executeProposal(id int) *bvm.Result {
    key, _ := account.NewAccountFromAccountJSON("", pwd)
    opt := bvm.NewProposalExecuteOperation(id)
    payload := bvm.EncodeOperation(opt)
    tx := NewTransaction(key.GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
    tx.Sign(key)
    re, err := rpc.InvokeContract(tx)
    assert.NoError(err)
    result := bvm.Decode(re.Ret)
    t.Log(result)
    return result
  }


3.2.8 部署合约交易构造
>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Deploy(payload string) *Transaction


- 参数1 （payload） : 编码后的合约。根据合约类型不同，传不同的值，具体参照第四章部分。（例如：若为solidity合约，该字段为合约编译后的bin，参照4.1部分；若为hvm合约，该字段为jar包处理后的值，参照4.3部分）                                                                                             |
- 返回值1          : 交易体实例                                                                                                 |

应用实例

 ::

  trans.Deploy("0x60606040526000805463ffffffff19169055341561001957fe5b5b61012a806100296000396000f300606060405263ffffffff60e060020a6000350416633ad14af38114603e57806348fe842114605c578063569c5f6d14606b578063d09de08a146091575bfe5b3415604557fe5b605a63ffffffff6004358116906024351660a0565b005b3415606357fe5b605a60c2565b005b3415607257fe5b607860d2565b6040805163ffffffff9092168252519081900360200190f35b3415609857fe5b605a60df565b005b6000805463ffffffff808216850184011663ffffffff199091161790555b5050565b6000805463ffffffff191690555b565b60005463ffffffff165b90565b6000805463ffffffff8082166001011663ffffffff199091161790555b5600a165627a7a72305820caa934a33fe993d03f87bdf39706fada68ddde78182e0110fd43e8c323d5984a0029")




3.2.9 带初始化参数的部署合约交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) DeployArgs(abiString string, args ...interface{}) *Transaction


- 参数1 （abiString） : 合约abi对应构造函数的字符串
- 参数2 （args）      : 合约初始化参数
- 返回值1            : 交易体实例

应用实例

 ::

  abi := `[
    {
      "inputs": [
        {
          "internalType": "uint32",
          "name": "a",
          "type": "uint32"
        },
        {
          "internalType": "uint32",
          "name": "b",
          "type": "uint32"
        }
      ],
      "stateMutability": "nonpayable",
      "type": "constructor"
    },
    {
      "inputs": [
        {
          "internalType": "int256",
          "name": "a",
          "type": "int256"
        },
        {
          "internalType": "int256",
          "name": "b",
          "type": "int256"
        }
      ],
      "name": "add",
      "outputs": [
        {
          "internalType": "int256",
          "name": "",
          "type": "int256"
        }
      ],
      "stateMutability": "pure",
      "type": "function"
    }
  ]

  `
  trans.DeployArgs(abi[0], 1, 2)




3.2.10 带字符串初始化参数的部署合约交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) DeployStringArgs(abiString string, args ...interface{}) *Transaction


- 参数1 （abiString） : 合约abi对应构造函数的字符串
- 参数2 （args）      : 参数列表
- 返回值1            : 交易体实例

应用实例

 ::

  abi := `[
    {
      "inputs": [
        {
          "internalType": "address",
          "name": "a",
          "type": "address"
        },
        {
          "internalType": "uint32",
          "name": "b",
          "type": "uint32"
        }
      ],
      "stateMutability": "nonpayable",
      "type": "constructor"
    },
    {
      "inputs": [
        {
          "internalType": "int256",
          "name": "a",
          "type": "int256"
        },
        {
          "internalType": "int256",
          "name": "b",
          "type": "int256"
        }
      ],
      "name": "add",
      "outputs": [
        {
          "internalType": "int256",
          "name": "",
          "type": "int256"
        }
      ],
      "stateMutability": "pure",
      "type": "function"
    }
  ]
  `
  trans.DeployStringArgs(abi[0], "1", 1)




3.2.11 调用合约交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Invoke(to string, payload []byte) *Transaction


- 参数 1 （to）     : 合约地址
- 参数2 （payload） : 合约负载，根据合约类型不同传入的值不同，具体参照第4章节（例如： 若为solidity合约，则为abi.Pack编码后的值；若为hvm合约则为hvm.GenPayload编码之后的值）                                                                                  |
- 返回值1          : 交易体实例

应用实例

 ::

  contractAddr := "0x421a1fb06bd9c9fae9b8cdaf8a662cf3c41ffa10"
  //abiContract为合约的abi
  ABI, _ := abi.JSON(strings.NewReader(abiContract))
  packed, _ := ABI.Pack("getMul")
  trans.Invoke(contractAddr, packed)




3.2.12 调用kvsql合约交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) InvokeSql(to string, payload []byte) *Transaction


- 参数1  : 合约地址
- 参数2  : sql语句的byte数组
- 返回值1 : 交易体实例

应用实例

 ::

  str := "CREATE TABLE IF NOT EXISTS testTable (id bigint(20) NOT NULL, name varchar(32) NOT NULL, exp bigint(20), money double(16,2) NOT NULL DEFAULT '99', primary key (id), unique key name (name));"
  trans.InvokeSql(contractAddr, []byte(str))




3.2.13 通过合约名字调用合约交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) InvokeByName(name string, payload []byte) *Transaction


- 参数1  : 合约名字
- 参数2  : 合约负载，设定规则与3.2.11部分相同
- 返回值1 : 交易体实例

应用实例

 ::

  name := "demo"
  contractAddr := "0x421a1fb06bd9c9fae9b8cdaf8a662cf3c41ffa10"
  // 省略绑定过程
  ABI, _ := abi.JSON(strings.NewReader(abiContract))
  packed, _ := ABI.Pack("getMul")
  trans.InvokeByName(name, packed)




3.2.14 设置extra
>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Extra(extra string) *Transaction


应用实例

 ::

  trans.Extra("demo extra")


3.2.15 设置kvextra
>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) KVExtra(kvExtra *KVExtra) *Transaction


- 参数1  : KVExtra， kv类型的extra
- 返回值1 : 交易体实例

应用实例

 ::

  kvExtra := NewKVExtra()
  kvExtra.AddKV("name", "ice")
  kvExtra.AddKV("friend", map[string]string{
    "name": "ice",
    "age":  "24",
  })
  trans.KVExtra(kvExtra)



3.2.16 设置to
>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) To(to string) *Transaction


应用实例

 ::

  trans.To("0x421a1fb06bd9c9fae9b8cdaf8a662cf3c41ffa10")




3.2.17 设置payload
>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Payload(payload string) *Transaction


应用实例

 ::

  trans.Payload("demo payload")


3.2.18 设置value
>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Value(value int64) *Transaction


应用实例

 ::

  trans.Value(123)




3.2.19 设置opcode
>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) OpCode(op int64) *Transaction


- 参数1  : 设定交易体的操作码，目前的操作码如下
- 返回值1 : 交易体实例

应用实例

 ::

  trans.OpCode(rpc.UPDATE)




3.2.20 注册DID账户的交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) Register(document *DIDDocument) *Transaction


- 参数1  : did账户结构
- 返回值1 : 交易体实例

应用实例

 ::

  password := ""
  chainID := "chainID_01"
  accountJson, _ := account.NewAccountED25519(password)
  key, _ := account.GenKeyFromAccountJson(accountJson, password)
  suffix := common.RandomString(10)
  didKey := account.NewDIDAccount(key.(account.Key), chainID, suffix)
  puKey, _ := rpc.GenDIDPublicKeyFromDIDKey(didKey)
  //admin是did账户的列表，第一次为nil;后续如果再新建就填之前的账户；
  document := rpc.NewDIDDocument(didKey.GetAddress(), puKey, admins)
  trans.Register(document)




3.2.21 管理DID账户生命周期的交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) MaintainDID(to string, op int64) *Transaction


- 参数1 : did账户地址
- 参数2 : 操作码

应用实例

 ::

  password := ""
  chainID := "chainID_01"
  accountJson, _ := account.NewAccountED25519(password)
  key, _ := account.GenKeyFromAccountJson(accountJson, password)
  suffix := common.RandomString(10)
  ac := account.NewDIDAccount(key.(account.Key), chainID, suffix)
  //这里省略了将did账户ac注册的步骤
  trans.MaintainDID(ac.GetAddress, rpc.DID_FREEZE)




3.2.22 更新DID账户publickey的交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>.

方法签名：

 ::

  func (t *Transaction) UpdatePublicKey(to string, puKey *DIDPublicKey) *Transaction


- 参数1  : did账户地址
- 参数   : did账户公钥
- 返回值1 : 交易体实例

应用实例

 ::

  // 注册DID账户ac， 然后重新生成新的did账户tempAc
  newPubkey, _ := rpc.GenDIDPublicKeyFromDIDKey(tempAc)
  trans.UpdatePublicKey(ac.GetAddress(). newPubkey)




3.2.23 更新DID账户的管理员账户交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) UpdateAdmins(to string, admins []string) *Transaction


- 参数1  : did账户地址
- 参数2  : 管理员地址列表
- 返回值1 : 交易体实例

应用实例

 ::

  // 注册一个DID账户ac， 更改其管理员为admin1， admin2， ac
  trans.UpdateAdmins(ac.GetAddress(), []string{
    admin1.GetAddress(),
    admin2.GetAddress(),
    ac.GetAddress(),
  })




3.2.24 上传DID账户凭证交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) UploadCredential(credential *DIDCredential) *Transaction


- 参数1  : 签名后的凭证结构体
- 返回值1 : 交易体实例

应用实例

 ::

  //按照上文DID账户注册的步骤，注册DID账户issuer和holder
  //issuer为签发者，holder为持有者
  cred := rpc.NewDIDCredential("type", issuer.GetAddress(), holder.GetAddress(), "credential message", time.Now().UnixNano(), time.Now().UnixNano()+1e11)
  //签发者签发凭证，即给凭证签名
  cred.Sign(issuer)
  //构建凭证上传的交易
  tx := rpc.NewTransaction(holder.GetAddress()).UploadCredential(cred)


3.2.25 下载DID账户凭证交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) DownloadCredential(credentialID string) *Transaction


- 参数1  : 凭证id
- 返回值1 : 交易体实例

应用实例

 ::

  //issuer 为签发者账户，credID为凭证id
  tx := rpc.NewTransaction(issuer.GetAddress()).DownloadCredential(credID)




3.2.26 销毁DID账户凭证交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) DestroyCredential(credentialID string) *Transaction


- 参数1  : 凭证id
- 返回值1 : 交易体实例

应用实例

 ::

  //issuer 为签发者账户，credID为凭证id
  tx := rpc.NewTransaction(issuer.GetAddress()).DestroyCredential(credID)


3.2.27 本地计算交易体hash
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

  func (t *Transaction) GetTransactionHash(gasLimit int64) string




- 参数1  : gas消耗上限
- 返回值1 : 交易体hash

 应用实例

 ::

  tx.GetTransactionHash(1000)




3.2.28 设置DID账户的Extra交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

  func (t *Transaction) DIDSetExtra(to, key, value string) *Transaction


- 参数1  : did账户地址
- 参数2  : extra信息的key
- 参数3  : extra信息value
- 返回值1 : 交易体

应用实例

 ::

  rpc := NewRPC()
  //registerDIDAccount函数详细在下面，主要作用是生成一个账户，注册成did账户；
  from := registerDIDAccount(rpc, nil)
  to := registerDIDAccount(rpc, []string{from.GetAddress()})
  key := "key_test"
  value := "value_test"

  tx := NewTransaction(from.GetAddress()).DIDSetExtra(to.GetAddress(), key, value)
  _, err := rpc.SendDIDTransaction(tx, from)


 ::

  func registerDIDAccount(rpc *RPC, admins []string) *account.DIDKey {
    rpc.SetLocalChainID()
    randNum := common.RandInt(100)
    var accountJson string
    password := "hyper"
    switch randNum % 3 {
    case 0:
      accountJson, _ = account.NewAccountED25519(password)
    case 1:
      accountJson, _ = account.NewAccountSm2(password)
    case 2:
      accountJson, _ = account.NewAccount(password)
    }
    key, _ := account.GenKeyFromAccountJson(accountJson, password)
    suffix := common.RandomString(10)
    didKey := account.NewDIDAccount(key.(account.Key), rpc.chainID, suffix)

    puKey, _ := GenDIDPublicKeyFromDIDKey(didKey)
    document := NewDIDDocument(didKey.GetAddress(), puKey, admins)
    tx := NewTransaction(didKey.GetAddress()).Register(document)
    _, err := rpc.SendDIDTransaction(tx, didKey)
    if err == nil {
      return didKey
    }
    return nil
  }


3.2.29 获取DID账户的Extra交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

  func (t *Transaction) DIDGetExtra(to, key string) *Transaction


- 参数1  : did账户地址
- 参数2  : extra信息的key
- 返回值1 : 交易体

应用实例

 ::

  rpc := NewRPC()
  from := registerDIDAccount(rpc, nil)
  to := registerDIDAccount(rpc, []string{from.GetAddress()})
  key := "key_test"
  tx := NewTransaction(from.GetAddress()).DIDGetExtra(to.GetAddress(), key)
  _, err := rpc.SendDIDTransaction(tx, from)




3.2.30 调用bvm合约交易体构造
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) InvokeBVM(to string, opt bvm.BuiltinOperation) *Transaction


- 参数1  : 合约地址
- 参数2  : 内置合约操作BuiltinOperation
- 返回值1 : 交易体实例

应用实例

::

  key, _ := account.GenKeyFromAccountJson(accountJson, "")
  opt := bvm.NewDIDSetChainIDOperation("chainID_01")
  tx := NewTransaction(key.(account.Key).GetAddress().Hex()).InvokeBVM(opt.Address(), opt)






3.3 交易体签名
------------------

3.3.1 签名
>>>>>>>>>>>>>>>>>>

方法签名：

 ::

  func (t *Transaction) Sign(key interface{})


- 参数1 : 账户key

应用实例

 ::

  trans.Sign(key)




3.3.2 批量签名
>>>>>>>>>>>>>>>>>

方法签名

 ::

  func (t *Transaction) SignWithBatchFlag(key interface{})


- 参数1 : 原始账户

应用实例

 ::

  // key 账户
  tx.SignWithBatchFlag(key)




3.3.3 聚合签名
>>>>>>>>>>>>>>>>>>

**3.3.3.1 通过普通账户生成聚合账户**

 ::

  func NewAggSignerFromAccount(key Key, publicKeys []gm.PubKey) (*AggSigner, error)


- 参数1  : 原始账户
- 参数2  : 参与聚合签名的用户的公钥列表
- 返回值1 : 聚合账户
- 返回值2 : error

 ::

  k1, _ := hex.DecodeString("04a642d77b4c8688124e343af9531de801813f1a880b664ce38a78d76ed128d07b1815004fe1de9c903b349afba9c74775ecc1692a6dd230689ea096521402e834")
  k2, _ := hex.DecodeString("042837e98f62ce38e93d41c949425b8e9a5b46107f2bd8725cc7db91b317c6d3199fd87f51883f9cc76a5535ec3b52dac4778597d26e0bc4a0b164cb4b6e87a3d5")
  k3, _ := hex.DecodeString("0445a79c4d689b9f19dbf19d58d1cb1952a8b557399f2c3a94a1e52fa21fa47c1acf7bc909e76449914a61a7ada1a855c3d3142181ce35b65cc172e5388dc494d6")

  kb := []gm.PubKey{k1, k2, k3}
  user2, err := account.NewAccountSm2FromPriv("5fa355d98ba611d56b05d18a3b665a9e276f21487043c2c88e8a0ce84c0249cb")
  assert.Nil(t, err)
  signer2, err := account.NewAggSignerFromAccount(user2, kb)




**3.3.3.2 通过DID账户生成聚合账户**

 ::

  func NewAggSignerFromDIDAccount(did *DIDKey, publicKeys []gm.PubKey) (*AggSigner, error)


- 参数1  : 原始账户
- 参数2  : 参与聚合签名的用户的公钥列表
- 返回值1 ： 聚合账户
- 返回值2 ： error

 ::

  k1, _ := hex.DecodeString("04a642d77b4c8688124e343af9531de801813f1a880b664ce38a78d76ed128d07b1815004fe1de9c903b349afba9c74775ecc1692a6dd230689ea096521402e834")
  k2, _ := hex.DecodeString("042837e98f62ce38e93d41c949425b8e9a5b46107f2bd8725cc7db91b317c6d3199fd87f51883f9cc76a5535ec3b52dac4778597d26e0bc4a0b164cb4b6e87a3d5")
  k3, _ := hex.DecodeString("0445a79c4d689b9f19dbf19d58d1cb1952a8b557399f2c3a94a1e52fa21fa47c1acf7bc909e76449914a61a7ada1a855c3d3142181ce35b65cc172e5388dc494d6")
  kb := []gm.PubKey{k1, k2, k3}
  suffix := common.RandomString(10)
  user1, err := account.NewAccountSm2FromPriv("14f02ad2f0fc12f25b9a233fb182fa064679c1077fdbbc77d1a945806ebfee35")
  did1 := account.NewDIDAccount(user1, "chainID_01", suffix)
  signer, err := account.NewAggSignerFromDIDAccount(did1, kb)




**3.3.3.3 聚合随机承诺**

 ::

  func (a *AggSigner) AggCommitment(in ...gm.Commitment) (gm.Commitment, error)


- 参数   ： 参与聚合签名的所有用户的承诺
- 返回值1 ： 聚合承诺
- 返回值2 ： error

 ::

  aggCommitment, err := first.AggCommitment(first.Commitment(), second.Commitment())


**3.3.3.4 部分签名**

 ::

  func (a *AggSigner) PartSign(msg []byte, aggCommitment gm.Commitment) (gm.Signature, error)


- 参数1  ： message
- 参数2  ： 聚合承诺
- 返回值1 ： 部分签名
- 返回值2 ： error

 ::

  s1, err := first.PartSign([]byte(transaction.NeedHashString()), aggCommitment)






**3.3.3.5 聚合签名**

 ::

  func (a *AggSigner) AggSign(signs ...gm.Signature) (gm.Signature, error)


- 参数1  ： 参与聚合签名用户的部分签名
- 返回值1 ： 聚合签名签名
- 返回值2 ： error

 ::

  aggSign, err := first.AggSign(s1, s2)




**3.3.3.6 完整实例**

***** 注意 ******

**如果最后调用AggSign的用户是DID账户，那么transaction的Participant的第一个属性initiator的值可以为以下两种**

1. **initiator为DID账户的address**

1. **initiator为DID账户的publickey，那这样调用AggSign前必须先调用SetHasDID将hasDID设置为false**

**如果最后调用AggSign的用户是非DID账户，那么transaction的Participant的第一个属性initiator的值为账户的publicKey**

 ::

  func TestAggSign(t *testing.T) {
    rp := NewRPC()
    user1, err := account.NewAccountSm2FromPriv("14f02ad2f0fc12f25b9a233fb182fa064679c1077fdbbc77d1a945806ebfee35")
    assert.Nil(t, err)

    user2, err := account.NewAccountSm2FromPriv("5fa355d98ba611d56b05d18a3b665a9e276f21487043c2c88e8a0ce84c0249cb")
    assert.Nil(t, err)
    pk1, err := user1.PublicBytes()
    assert.Nil(t, err)
    pk2, err := user2.PublicBytes()
    assert.Nil(t, err)
    pks := []gm.PubKey{pk1, pk2}

    first, err := account.NewAggSignerFromAccount(user1, pks)
    assert.Nil(t, err)

    second, err := account.NewAggSignerFromAccount(user2, pks)
    assert.Nil(t, err)

    aggCommitment, err := first.AggCommitment(first.Commitment(), second.Commitment())
    assert.Nil(t, err)

    transaction := NewTransaction(first.Address().Hex()).Participant(&Participant{
        Initiator:   pk1,
        Withholding: [][]byte{pk2},
    }).Transfer(address, int64(0))
    s1, err := first.PartSign([]byte(transaction.NeedHashString()), aggCommitment)
    assert.Nil(t, err)
    s2, err := second.PartSign([]byte(transaction.NeedHashString()), aggCommitment)
    assert.Nil(t, err)

    aggSign, err := first.AggSign(s1, s2)
    assert.Nil(t, err)
    fmt.Println(common.Bytes2Hex(aggSign))
    assert.Nil(t, first.Verify([]byte(transaction.NeedHashString()), aggSign))
    transaction.Signature(common.Bytes2Hex(aggSign))

    resp, err := rp.SignAndInvokeContract(transaction, nil)
    assert.Nil(t, err)
    fmt.Println(resp)

  }


第四章 虚拟机
==================

4.1 EVM
---------------

4.1.1 abi解析
>>>>>>>>>>>>>>>>>

EVM对应的合约为solidity，解析合约abi为如下struct

 ::

  type ABI struct {
    Constructor Method
    Methods     map[string]Method
    Events      map[string]Event

    // Fields that record overloaded methods and events
    allMethods map[string]Method
    allEvents  map[string]Event
  }


方法签名：

 ::

  func JSON(reader io.Reader) (ABI, error)


- 参数1  ： 合约的io流
- 返回值1 ： ABI结构体
- 返回值2 ： error，详见第七章

应用实例

 ::

  // abiContract 合约编码后的abi
  ABI, serr := abi.JSON(strings.NewReader(abiContract))




**4.1.1.1 合约方法获取**

 ::

  func (abi ABI) GetMethod(methodSig string) (*Method, error)




**4.1.1.2 合约event获取**

 ::

  func (abi ABI) GetEvent(eventSig string) (*Event, error)




**4.1.1.3 将方法以及参数转为binary**

 ::

  func (abi ABI) GetPayload(funcName string, args ...string) string




**4.1.1.4 编码调用的方法和参数**

 ::

  func (abi ABI) Pack(name string, args ...interface{}) ([]byte, error)


应用实例

 ::

  packed, _ = ABI.Pack("testOverload", args...)

  packed, _ = ABI.Pack("testOverload(int24,bool[])", args...)



说明：

1. 若调用时仅仅使用方法名称，则默认调用无参同名方法，若未找到无参同名方法，则调用最后一个匹配到的方法，即最后声明的同名方法；

2. 方法签名的构造方式为：

 ::

  "MethodName(inputType1,inputType2,...inputTypeN)"


即在方法名称之后使用括号将传参类型统一包括，类型之间以逗号分隔。

1. 方法签名括号中规定了方法传参类型，传参类型之间必须使用“,”分隔，传参类型必须与abi中inputs里对应的type一致，且数量和顺序必须保持一致。

2. 事件签名构造同理：

 ::

  "EventName(inputType1,inputType2,...inputTypeN)"




**4.1.1.5 返回值解析**

方法签名

 ::

  func (abi *ABI) UnpackResult(v interface{}, name, data string) (err error)


得到返回值结果后，获得状态码可以判断是否调用成功，若调用成功，解析返回值可看到调用之后的结果。

 ::

  // decode
  //返回值解析
  var result string
  if err = ABI.UnpackResult(&result, "get", txInvoke1.Ret); err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println("get返回值:", result)




**4.1.1.6 反序列化**

  ::

  func (abi *ABI) UnmarshalJSON(data []byte) error




**4.1.1.7 通过方法的id获取方法**

  ::

  func (abi *ABI) MethodById(sigdata []byte) (*Method, error)




4.1.2 abi v2
>>>>>>>>>>>>>>>>

abiv2 与 abi 在abi的结构上有了较大的变化，如下以结构ABI为例

  ::

  // abi
  type ABI struct {
    Constructor Method
    Methods     map[string]Method
    Events      map[string]Event

    // Fields that record overloaded methods and events
    allMethods map[string]Method
    allEvents  map[string]Event
  }

  // abiv2
  type ABI struct {
    Constructor Method
    Methods     map[string]Method
    Events      map[string]Event

    // Additional "special" functions introduced in solidity v0.6.0.
    // It's separated from the original default fallback. Each contract
    // can only define one fallback and receive function.
    Fallback Method // Note it's also used to represent legacy fallback before v0.6.0
    Receive  Method
  }


因此，abiv2 独立于原先的abi目录，新增abi2目录。下面列出常用的方法

**4.1.2.1 编码**

  ::

  func (abi ABI) Pack(name string, args ...interface{}) ([]byte, error)




**4.1.2.2 解码**

 ::

  func (abi *ABI) UnpackResult(v interface{}, name, data string) (err error)




4.2 BVM
-----------

BVM全称是 **built-in virtual machine** ，是用于处理内置合约的虚拟机类型。BVM的出现使得开发者可以自定义一些内置合约，提供一些固定的功能。

**内置合约**，即是合约代码由开发人员预先写好，在平台启动时就直接创建对象加载，不需要用户手动部署的合约。由于合约随着平台预先创建加载， 所以合约地址固定不变，且相比普通合约有以下特性：

- **不需部署** （也可以理解为平台刚启动就被 **”部署“** 在某个固定地址上）。

- **目前功能单一** （只支持两份），但之后可以不断地进行扩展。

- 做合约修改或增加新的内置合约需要 **平台兼容** 做调整。

- **不属于任何人** ，及任何用户都不可以对该合约进行升级或冻结、解冻的相关操作。

目前支持以下几种种内置合约

1. **提案相关** ,ProposalContract 合约地址为 `0x0000000000000000000000000000000000ffff02`

2. **hash相关** ,HashContract合约地址为  `0x0000000000000000000000000000000000ffff01`

3. **did相关** ，合约地址为 `0x0000000000000000000000000000000000ffff06`

4. **证书相关** ，合约地址为 `0x0000000000000000000000000000000000ffff05`

5.  **账户相关** ，合约地址为 `0x0000000000000000000000000000000000ffff04`

6. **mpc(多方安全计算)合约相关** 合约地址为 `0x0000000000000000000000000000000000ffff09`

7. **Root CA相关** ，合约地址为 `0x0000000000000000000000000000000000ffff0b`

8. **跨分区相关** ,合约地址为 `0x0000000000000000000000000000000000ffff0a`

9. **hash类型查询** ，合约地址为 `0x0000000000000000000000000000000000ffff0d`

10. **balanceManager相关** ，合约地址为 `0x0000000000000000000000000000000000ffff0e`





4.2.1 编解码
>>>>>>>>>>>>>>>>>>>

**4.2.1.1 将操作编码成payload**

方法签名

 ::

  func EncodeOperation(ope Operation) []byte


- 参数1  ： 操作
- 返回值1 ： 对应交易体的payload

应用实例

 ::

  opt := bvm.NewDIDSetChainIDOperation("chainID_01")
  payload := bvm.EncodeOperation(opt)




**4.2.1.2 返回值解码**

方法签名

 ::

  func Decode(ret string) *Result

  // Result represent the execute result of BVM
  type Result struct {
    Success bool
    Ret []byte
    Err string
  }



- 参数1  ： 交易回执里面的Ret
- 返回值1 ： Result结构体

应用实例

 ::

  // 发送交易拿到回执后，取ret字段
  retStr := `0x7b2253756363657373223a747275652c22526574223a22573373695932396b5a5349364d6a417766537837496d4e765a4755694f6a49774d48307365794a6a6232526c496a6f794d4442394c4873695932396b5a5349364d6a41776656303d222c22457272223a22227d`
  res := bvm.Decode(retStr)



**4.2.1.3 提案code解码**

 ::

  type ProposalData struct {
    Id        uint64              `protobuf:"varint,1,opt,name=id,proto3" json:"id,omitempty"`
    Code      []byte              `protobuf:"bytes,2,opt,name=code,proto3" json:"code,omitempty"`
    Timestamp int64               `protobuf:"varint,3,opt,name=timestamp,proto3" json:"timestamp,omitempty"`
    Timeout   int64               `protobuf:"varint,4,opt,name=timeout,proto3" json:"timeout,omitempty"`
    Status    ProposalData_Status `protobuf:"varint,5,opt,name=status,proto3,enum=bvm.ProposalData_Status" json:"status,omitempty"`
    Assentor  []*VoteInfo         `protobuf:"bytes,6,rep,name=assentor,proto3" json:"assentor,omitempty"`
    Objector  []*VoteInfo         `protobuf:"bytes,7,rep,name=objector,proto3" json:"objector,omitempty"`
    Threshold uint32              `protobuf:"varint,8,opt,name=threshold,proto3" json:"threshold,omitempty"`
    Score     uint32              `protobuf:"varint,9,opt,name=score,proto3" json:"score,omitempty"`
    Creator   string              `protobuf:"bytes,10,opt,name=creator,proto3" json:"creator,omitempty"`
    Version   string              `protobuf:"bytes,11,opt,name=version,proto3" json:"version,omitempty"`
    Type      ProposalData_PType  `protobuf:"varint,12,opt,name=type,proto3,enum=bvm.ProposalData_PType" json:"type,omitempty"`
    Completed []byte              `protobuf:"bytes,13,opt,name=completed,proto3" json:"completed,omitempty"`
    Cancel    []byte              `protobuf:"bytes,14,opt,name=cancel,proto3" json:"cancel,omitempty"`
  }


解码提案数据的code字段

方法签名

 ::

  func DecodeProposalCode(code []byte) (string, error)


应用实例

 ::

  proposalHexString := "08011284030000000a0000000f53657446696c746572456e61626c65000000010000000566616c73650000000e53657446696c74657252756c6573000000010000001f5b7b22616c6c6f775f616e796f6e65223a66616c73652c226964223a307d5d00000015536574436f6e73656e737573426174636853697a65000000010000000331303000000014536574436f6e73656e737573506f6f6c53697a65000000010000000332303000000013536574436f6e73656e73757353657453697a65000000010000000235300000001453657450726f706f73616c5468726573686f6c640000000100000001340000001253657450726f706f73616c54696d656f7574000000010000000c3438303030303030303030300000001253657450726f706f73616c54696d656f7574000000010000000c34383030303030303030303000000018536574436f6e7472616374566f74655468726573686f6c6400000001000000013300000015536574436f6e7472616374566f7465456e61626c6500000001000000047472756518d0bbb4bbebcec4d11620d0abc786c9d7c4d116280132720a2a3078303030663161376130386363633438653564333066383038353063663163663238336161336162641242307831366133313237363930393565323832633130323861376138376634336563623864623937653765643166386234663466626438393435356363353266386666180140064801522a3078303030663161376130386363633438653564333066383038353063663163663238336161336162645a05302e312e30"
  a := common.Hex2Bytes(proposalHexString)
  var pro bvm.ProposalData
  proto.Unmarshal(a, &pro) // 提案数据解码
  code, err := bvm.DecodeProposalCode(pro.Code) // 提案code字段解码




4.2.2 提案的使用
>>>>>>>>>>>>>>>>>>

    - 提案总共有六个状态：等待投票、驳回、等待执行、已完成、取消和超时。

    - 只有创建者有权取消提案

    - 提案创建后通过投票可进入等待执行、驳回状态

    - 处于等待执行的提案可有发起者发起执行操作，执行完成进入已完成状态

    - 提案处于等待投票、等待执行状态时无法创建新的提案

    - 目前提案的默认阈值为链级管理员的总个数，每个链级管理员均可以对此提案进行投票，若此提案获得的票数大于等于阈值则通过投票

    - 目前提案的默认超时时间为5分钟（当设置当超时时间小于5分钟时，会设置为5分钟），即创建提案的交易打包时间+5分钟则为提案超时时间

    - 每次执行提案交易都会拿到当前系统中最新的提案，将当前执行的交易的打包时间与提案的超时时间进行对比，如果大于超时时间，则将提案状态置为超时。（不能单纯的认为提案创建后，过了超时时长就一定超时了，没过超时时长就一定没超时。交易的打包时间是主节点中打包交易时取的当前系统时间，主节点的系统时间可能不是正常的时间序列，例如：如果提案创建后发生了viewchange，viewchange之前的主节点在当前时刻的系统时间为1:00，viewchange之后的主节点中当前时刻的系统时间为1:30，这时即使提案创建之后没有超过超时时长，对提案进行投票，由于新的主节点的时间为1:30，打包的时间戳也为1:30对应的时间戳，在执行提案对比是否超时时，判断的结果就为超时。目前主节点时间变更除了viewchange之外，还有主节点主动作恶、或根据需要主动变更系统时间。）

根据提案的状态以及对提案的操作，可得出一个提案的状态迁移如下图所示：

.. image:: ../../images/GoSDK/GoSDK2.png
    :align: center
    :alt: 提案状态迁移示意图

对于提案可根据提案内容划分为以下几类：

    - 配置类， ptype为 `config` ，data则为配置项操作列表；

    - 权限类，ptype为 `permission` ，data为权限操作列表；

    - 节点类，ptype为 `node` ，data为节点操作列表；

    - 合约命名类，ptype为 `cns` ，data为合约命名操作列表；

    - 合约生命周期管理类，ptype为 `contract` ，data为合约生命周期管理操作列表；

    - ca模式类，ptype为 `ca` ，data为ca操作列表；



**4.2.2.1 合约提案创建**

 ::

  func NewProposalCreateOperationForContract(ops ...ContractOperation) BuiltinOperation


- 参数1 （opts） ： 一组合约类型的操作

应用实例

 ::

  ope := bvm.NewContractDeployContractOperation(source, common.Hex2Bytes(bin), "evm", nil)
  contractOpt := bvm.NewProposalCreateOperationForContract(ope)




**4.2.2.2 命名提案创建**

 ::

  func NewProposalCreateOperationForCNS(ops ...CNSOperation) BuiltinOperation


- 参数1 （opts） ： 一组合约命名类型的操作

应用实例

 ::

  // addr 合约地址
  // name 命名
  setCNameOpt := bvm.NewCNSSetCNameOperation(addr, name)
  cnsOpt := bvm.NewProposalCreateOperationForCNS(setCNameOpt)




**4.2.2.3 节点类型的提案创建**

 ::

  func NewProposalCreateOperationForNode(ops ...NodeOperation) BuiltinOperation


- 参数1 （opts） ： 一组节点类型的操作

应用实例

 ::

  var operations []bvm.NodeOperation
  operations = append(operations, bvm.NewNodeAddNodeOperation([]byte("pub"), "node1", "vp", "global"))
  operations = append(operations, bvm.NewNodeAddVPOperation("node1", "global"))
  operations = append(operations, bvm.NewNodeRemoveVPOperation("node1", "global"))
  proposalCreateOperation := bvm.NewProposalCreateOperationForNode(operations...)




**4.2.2.4 权限类型的提案创建**

 ::

  func NewProposalCreateOperationForPermission(ops ...PermissionOperation) BuiltinOperation


- 参数1 （opts） ： 一组权限类型的操作

应用实例

 ::

  var operations []bvm.PermissionOperation
  operations = append(operations, bvm.NewPermissionCreateRoleOperation(managerRole))
  operations = append(operations, bvm.NewPermissionGrantOperation(managerRole, notAdminAddress))
  operations = append(operations, bvm.NewPermissionRevokeOperation(managerRole, notAdminAddress))
  operations = append(operations, bvm.NewPermissionDeleteRoleOperation(managerRole))
  proposalCreateOperation := bvm.NewProposalCreateOperationForPermission(operations...)
  payload := bvm.EncodeOperation(proposalCreateOperation)




**4.2.2.5 通过配置内容创建配置提案**

 ::

  func NewProposalCreateOperationForConfig(config []byte) (BuiltinOperation, error)


- 参数1 ： 配置内容

应用实例

 ::

  var config = `
  [filter]
      enable = false
      [[filter.rules]]
      allow_anyone = false
      authorized_roles = ["admin"]
      forbidden_roles = ["20"]
      id = 0
      name = "bvm auth"
      to = ["0x0000000000000000000000000000000000ffff02"]
      vm = ["bvm"]
  [consensus]
    algo = "RBFT"

    [consensus.pool]
      batch_size = 20
      pool_size = 3000

    [consensus.set]
      set_size = 20

  [proposal]
    timeout   = "15m"
    threshold = 3
  `

  operation, _ := bvm.NewProposalCreateOperationForConfig([]byte(config))
  payload := bvm.EncodeOperation(operation)




**4.2.2.6 通过配置类型操作创建配置提案**

 ::

  func NewProposalCreateOperationByConfigOps(ops ...ConfigOperation) BuiltinOperation


- 参数1 ： 一组配置类型的操作

应用实例

 ::

  bvm.NewProposalCreateOperationByConfigOps(bvm.NewSetProposalThreshold(7))




**4.2.2.7 提案投票**

 ::

  func NewProposalVoteOperation(proposalID int, vote bool) BuiltinOperation


- 参数1 ： 提案id
- 参数2 ： 是否开启投票

应用实例

 ::

  operation := bvm.NewProposalVoteOperation(int(proposal.Id), false)




**4.2.2.8 提案取消**

 ::

  func NewProposalCancelOperation(proposalID int) BuiltinOperation


- 参数1 ： 提案id

应用实例

 ::

  operation := bvm.NewProposalCancelOperation(2)




**4.2.2.9 提案执行**

 ::

  func NewProposalExecuteOperation(proposalID int) BuiltinOperation


- 参数1 ： 提案id

应用实例

 ::

  operation := bvm.NewProposalExecuteOperation(2)




**4.2.2.10  权限类型的操作----创建角色**

 ::

  func NewPermissionCreateRoleOperation(role string) PermissionOperation


- 参数1 ： 角色名

应用实例

 ::

  managerRole ：= "manager"
  bvm.NewPermissionCreateRoleOperation(managerRole)




**4.2.2.11 权限类型的操作----删除角色**

 ::

  func NewPermissionDeleteRoleOperation(role string) PermissionOperation


- 参数1 ： 角色名

应用实例

 ::

  managerRole ：= "manager"
  bvm.NewPermissionDeleteRoleOperation(managerRole)




**4.2.2.12  权限类型的操作----权限授予**

 ::

  func NewPermissionGrantOperation(role string, address string) PermissionOperation


- 参数1 ： 角色名
- 参数2 ： 账户地址

应用实例

 ::

  pwd := "12345678"
  nonAdminAcc, _ := account.NewAccount(pwd)
  key, _ := account.GenKeyFromAccountJson(nonAdminAcc, pwd)
  addr := key.(account.Key).GetAddress().Hex()
  createRole := bvm.NewPermissionGrantOperation(role, addr)
  operation := bvm.NewProposalCreateOperationForPermission(createRole)
  payload := bvm.EncodeOperation(operation)




**4.2.2.13 权限类型的操作----删除权限**

 ::

  func NewPermissionRevokeOperation(role string, address string) PermissionOperation


- 参数1 ： 角色名
- 参数2 ： 账户地址

应用实例

 ::

  pwd := "12345678"
  nonAdminAcc, _ := account.NewAccount(pwd)
  key, _ := account.GenKeyFromAccountJson(nonAdminAcc, pwd)
  addr := key.(account.Key).GetAddress().Hex()
  revokeRole := bvm.NewPermissionRevokeOperation(role, addr)
  operation := bvm.NewProposalCreateOperationForPermission(revokeRole)
  payload := bvm.EncodeOperation(operation)




**4.2.2.14 节点类型的操作----添加节点**

 ::

  func NewNodeAddNodeOperation(pub []byte, hostname, role, namespace string) NodeOperation


- 参数1 ： 对应节点公钥或者证书的byte数组
- 参数2 ： 节点名
- 参数3 ： 节点角色
- 参数4 ： 节点namespace

应用实例

 ::

  bvm.NewNodeAddNodeOperation([]byte("pub"), "node1", "vp", "global")




**4.2.2.15  节点类型的操作----启动vp**

 ::

  func NewNodeAddVPOperation(hostname, namespace string) NodeOperation



- 参数1 : 节点名
- 参数2 : 节点namespace

应用实例

 ::

  bvm.NewNodeAddVPOperation("node1", "global")




**4.2.2.16 节点类型的操作----删除vp**

 ::

  func NewNodeRemoveVPOperation(hostname, namespace string) NodeOperation



- 参数1 - 节点名
- 参数2 - 节点namespace

应用实例

 ::

  bvm.NewNodeRemoveVPOperation("node1", "global")


**4.2.2.17 合约命名类型的操作----设定合约名**

 ::

  func NewCNSSetCNameOperation(address string, cnsName string) CNSOperation



- 参数1 ： 合约地址
- 参数2 ： 合约名

应用实例

 ::

  addr := "0x0000000000000000000000000000000000ffff01"
  name := "HashContract"
  setCNameOpt := bvm.NewCNSSetCNameOperation(addr, name)






**4.2.2.18 合约类型的操作----部署合约**

 ::

  func NewContractDeployContractOperation(source, bin []byte, vmType string, compileOpts map[string]string) ContractOperation



- 参数1 ： 合约源文件
- 参数2 ： 合约编译后bin
- 参数3 ： 虚拟机类型
- 参数4 ： 编译选项

应用实例

 ::

  source, _ := ioutil.ReadFile("../conf/contract/Accumulator.sol")
  ope := bvm.NewContractDeployContractOperation(source, common.Hex2Bytes(binContract), "evm", nil)




**4.2.2.19 合约类型的操作----升级合约**

 ::

  func NewContractUpgradeContractOperation(source, bin []byte, vmType, contractAddress string, compileOpts map[string]string) ContractOperation



- 参数1 - 合约源文件
- 参数2 - 合约编译后bin
- 参数3 - 虚拟机类型
- 参数4 - 已部署的合约地址
- 参数5 - 编译选项

应用实例

 ::

  source, _ := ioutil.ReadFile("../conf/contract/Accumulator.sol")
  // contractAddress 已部署合约地址
  ope := bvm.NewContractUpgradeContractOperation(source, common.Hex2Bytes(binContract), "evm", contractAddr, nil)




**4.2.2.20 合约类型的操作----通过合约名升级合约**

 ::

  func NewContractUpgradeOperationByName(source, bin []byte, vmType, contractName string, compileOpts map[string]string) ContractOperation



- 参数1 ： 合约源文件
- 参数2 ： 合约编译后bin
- 参数3 ： 虚拟机类型
- 参数4 ： 已部署的合约名
- 参数5 ： 编译选项

应用实例

 ::

  source, _ := ioutil.ReadFile("../conf/contract/Accumulator.sol")
  // contractName 已部署合约名
  ope := bvm.NewContractUpgradeOperationByName(source, common.Hex2Bytes(binContract), "evm", contractName, nil)




**4.2.2.21 合约类型的操作----管理合约**

 ::

  func NewContractMaintainContractOperation(contractAddress, vmType string, opcode int) ContractOperation



- 参数1 ： 合约地址
- 参数2 ： 虚拟机类型
- 参数3 ： 操作码， 1 合约升级， 2 冻结合约， 3 解冻合约

应用实例

 ::

  // contractAddr 合约地址
  bvm.NewContractMaintainContractOperation(contractAddr, "evm", 2)


**4.2.2.22 合约类型的操作----通过合约名管理合约**

 ::

  func NewContractMaintainOperationByName(contractName, vmType string, opcode int) ContractOperation



- 参数1 : 合约名
- 参数2 : 虚拟机类型
- 参数3 : 操作码， 1 合约升级， 2 冻结合约， 3 解冻合约

应用实例

 ::

  // contractName 合约名
  bvm.NewContractMaintainOperationByName(contractName, "evm", 2)




**4.2.2.23 ca模式类型的提案创建**

当没有设置链上的ca模式时，可以通过提案设置ca模式，如果设置的ca模式与当前运行的ca模式不一致，则会停止当前链的运行。

 ::

  func NewProposalCreateOperationForCA(ops ...CAOperation) BuiltinOperation



- 参数1 （opts） : 一组ca模式类型的操作

应用实例

 ::

  mode, _ := bvm.NewCASetCAModeOperation("center")
  proposalCreateOperation := bvm.NewProposalCreateOperationForCA(mode)




**4.2.2.24 直接执行ca模式类型的操作**

查询ca模式操作可以直接执行。

 ::

  func NewProposalDirectOperationForCA(ops ...CAOperation) BuiltinOperation



- 参数1 （opts） : 一组ca模式类型的操作

应用实例



 ::

  proposalDirectOperation := bvm.NewProposalDirectOperationForCA(bvm.NewCAGetCAModeOperation())




**4.2.2.25 直接执行节点类型的操作**

当ca模式为`center` 或`none` 时，可以直接执行节点类型的操作。

- 新增节点时，节点类型的操作无需用户手动进行，即新增节点时，配置好了启动节点新增的相应操作会自动完成。

- 当ca模式为`center` 时，只有链级管理员可以直接执行节点类型的操作。

 ::

  func NewProposalDirectOperationForNode(ops ...NodeOperation) BuiltinOperation



- 参数1 （opts） : 一组节点类型的操作

应用实例



 ::

  var operations []bvm.NodeOperation
  operations = append(operations, bvm.NewNodeRemoveVPOperation("node1", "global"))
  proposalDirectOperation := bvm.NewProposalDirectOperationForNode(operations...)






4.2.3 hash的使用
>>>>>>>>>>>>>>>>>>>

**4.2.3.1 set**

 ::

  func NewHashSetOperation(key, value string) BuiltinOperation



- 参数1  ： 存储的键
- 参数2  ： 存储的值
- 返回值1 ： BuiltinOperation

应用实例

 ::

  operation := bvm.NewHashSetOperation("0x1231", "0x456")
  payload := bvm.EncodeOperation(operation）




**4.2.3.2 get**

 ::

  func NewHashGetOperation(key string) BuiltinOperation



- 参数1  : 存储的键
- 返回值1 : BuiltinOperation

应用实例

 ::

  operation := bvm.NewHashGetOperation("0x1231")
  payload := bvm.EncodeOperation(operation）


**4.2.3.3** **GenesisInfo**

 ::

  type GenesisInfo struct {
    GenesisAccount map[string]string `json:"genesisAccount,omitempty"`
    GenesisNodes   []*GenesisNode    `json:"genesisNodes,omitempty"`
  }
  // GenesisNode define the filed for genesis node.
  type GenesisNode struct {
    Hostname    string `json:"genesisNode" mapstructure:"genesisNode"`
    CertContent string `json:"certContent" mapstructure:"certContent"`
  }

  func NewSetGenesisInfoForHpcOperation(genesisInfo *GenesisInfo) BuiltinOperation



- 参数1  - genesisInfo
- 返回值1 - BuiltinOperation

应用实例

 ::

  operation := bvm.NewSetGenesisInfoForHpcOperation(&bvm.GenesisInfo{
        GenesisAccount: map[string]string{
          "0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd": "10000",
        },
        GenesisNodes: []*bvm.GenesisNode{
          {
              Hostname: "node1",
              CertContent: `-----BEGIN CERTIFICATE-----
  MIICSTCCAfWgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV
  BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD
  VQQLEwAxDjAMBgNVBAMTBW5vZGUxMQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl
  cnQwIBcNMjAwNTIxMDU1MzU2WhgPMjEyMDA0MjcwNjUzNTZaMHQxCTAHBgNVBAgT
  ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh
  dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTIxCzAJBgNVBAYTAlpIMQ4wDAYD
  VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABBI3ewNK21vHNOPG6U3X
  mKJohSNNz72QKDxUpRt0fCJHwaGYfSvY4cnqkbliclfckUTpCkFSRr4cqN6PURCF
  zkWjeTB3MA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB
  BQUHAwEGAioDBgOBCwEwDwYDVR0TAQH/BAUwAwEB/zANBgNVHQ4EBgQEAQIDBDAP
  BgNVHSMECDAGgAQBAgMEMAwGAypWAQQFZWNlcnQwCgYIKoZIzj0EAwIDQgD6Dzzv
  31kMiIlYtQRQjjs5m9pSvMZtmq2vOJW6/5J3kBuUkjTJfawqDdoxVh4yw06F/IBQ
  gSu97EZSaseY9hweAA==
  -----END CERTIFICATE-----`,
          },
          {
              Hostname: "node2",
              CertContent: `-----BEGIN CERTIFICATE-----
  MIICSTCCAfWgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV
  BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD
  VQQLEwAxDjAMBgNVBAMTBW5vZGUyMQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl
  cnQwIBcNMjAwNTIxMDU1MTE0WhgPMjEyMDA0MjcwNjUxMTRaMHQxCTAHBgNVBAgT
  ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh
  dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTExCzAJBgNVBAYTAlpIMQ4wDAYD
  VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABBI3ewNK21vHNOPG6U3X
  mKJohSNNz72QKDxUpRt0fCJHwaGYfSvY4cnqkbliclfckUTpCkFSRr4cqN6PURCF
  zkWjeTB3MA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB
  BQUHAwEGAioDBgOBCwEwDwYDVR0TAQH/BAUwAwEB/zANBgNVHQ4EBgQEAQIDBDAP
  BgNVHSMECDAGgAQBAgMEMAwGAypWAQQFZWNlcnQwCgYIKoZIzj0EAwIDQgB3Cfo8
  /Vdzzlz+MW+MIVuYQkcNkACY/yU/IXD1sHDGZQWcGKr4NR7FHJgsbjGpbUiCofw4
  4rK6biAEEAOcv1BQAA==
  -----END CERTIFICATE-----`,
          },
          {
              Hostname: "node3",
              CertContent: `-----BEGIN CERTIFICATE-----
  MIICRjCCAfKgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV
  BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD
  VQQLEwAxDjAMBgNVBAMTBW5vZGUzMQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl
  cnQwIBcNMjAwNTIxMDU1MTQ0WhgPMjEyMDA0MjcwNjUxNDRaMHQxCTAHBgNVBAgT
  ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh
  dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTExCzAJBgNVBAYTAlpIMQ4wDAYD
  VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABBI3ewNK21vHNOPG6U3X
  mKJohSNNz72QKDxUpRt0fCJHwaGYfSvY4cnqkbliclfckUTpCkFSRr4cqN6PURCF
  zkWjdjB0MA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB
  BQUHAwEGAioDBgOBCwEwDAYDVR0TAQH/BAIwADANBgNVHQ4EBgQEAQIDBDAPBgNV
  HSMECDAGgAQBAgMEMAwGAypWAQQFZWNlcnQwCgYIKoZIzj0EAwIDQgCalJzkOAqk
  IU4AMQGeWzFmdtRYJXZiElyqfrCn7Zg08Ssx14ZMO8K2cRsCncO+c/a/0IqwObEO
  wL4C2ich1g5bAA==
  -----END CERTIFICATE-----`,
          },
          {
              Hostname: "node4",
              CertContent: `-----BEGIN CERTIFICATE-----
  MIICSTCCAfWgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV
  BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD
  VQQLEwAxDjAMBgNVBAMTBW5vZGU0MQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl
  cnQwIBcNMjAwNTIxMDU1MzI0WhgPMjEyMDA0MjcwNjUzMjRaMHQxCTAHBgNVBAgT
  ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh
  dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTExCzAJBgNVBAYTAlpIMQ4wDAYD
  VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABBI3ewNK21vHNOPG6U3X
  mKJohSNNz72QKDxUpRt0fCJHwaGYfSvY4cnqkbliclfckUTpCkFSRr4cqN6PURCF
  zkWjeTB3MA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB
  BQUHAwEGAioDBgOBCwEwDwYDVR0TAQH/BAUwAwEB/zANBgNVHQ4EBgQEAQIDBDAP
  BgNVHSMECDAGgAQBAgMEMAwGAypWAQQFZWNlcnQwCgYIKoZIzj0EAwIDQgATn17d
  VxBI7s/4D/KNU0T3wvQeNrj6ZYmiNRjB/JIwPH7MhM5cTeJx0rs0K27rM/pFZtbq
  +0W3ll/JfjQ6WxAAAQ==
  -----END CERTIFICATE-----`,
          },
        },
    })
    payload := bvm.EncodeOperation(operation)






4.2.4 账户的使用
>>>>>>>>>>>>>>>>>>>>

**4.2.4.1 账户注册**

 ::

  func NewAccountRegisterOperation(address string, cert []byte) BuiltinOperation



- 参数1 ： 合约地址
- 参数2 ： 证书byte数组
-
应用实例

 ::

  addr := "0x42a815e75604dd69707ba4aa9d350a59d1e530e7"
  cert1 := []byte(`-----BEGIN CERTIFICATE-----
  MIICMTCCAd2gAwIBAgIIcB4Bo1m3X4wwCgYIKoZIzj0EAwIwdDEJMAcGA1UECBMA
  MQkwBwYDVQQHEwAxCTAHBgNVBAkTADEJMAcGA1UEERMAMQ4wDAYDVQQKEwVmbGF0
  bzEJMAcGA1UECxMAMQ4wDAYDVQQDEwVub2RlMTELMAkGA1UEBhMCWkgxDjAMBgNV
  BCoTBWVjZXJ0MB4XDTIwMTAxOTAwMDAwMFoXDTIwMTAxOTAwMDAwMFowPTELMAkG
  A1UEBhMCQ04xDjAMBgNVBAoTBWZsYXRvMQ4wDAYDVQQDEwVub2RlMTEOMAwGA1UE
  KhMFZWNlcnQwVjAQBgcqhkjOPQIBBgUrgQQACgNCAASN1aGLwcwb/1c4NCaT6vAY
  A38Z5394RgUES1SlmrYWFCxwpOkpozPgMqZ+tS5PhFRt857ChrUujXzb6PWi6XVh
  o4GSMIGPMA4GA1UdDwEB/wQEAwIB7jAxBgNVHSUEKjAoBggrBgEFBQcDAgYIKwYB
  BQUHAwEGCCsGAQUFBwMDBggrBgEFBQcDBDAMBgNVHRMBAf8EAjAAMB0GA1UdDgQW
  BBSl7MAm0apBmKIN7fyBLcS7Y/w6hTAPBgNVHSMECDAGgAQBAgMEMAwGAypWAQQF
  ZWNlcnQwCgYIKoZIzj0EAwIDQgDJ3kJ3uX/23BM9JqCwlDpympv6Eu0OPriz4KgG
  72Hr7xRJOrmZ14waO/I4jAvba7+J1uaNIv0K6EDjJplNzvPEAA==
  -----END CERTIFICATE-----
  `)

  bvm.NewAccountRegisterOperation(addr, cert1)




**4.2.4.2 删除账户**

 ::

  func NewAccountAbandonOperation(address string, sdkCert []byte) BuiltinOperation



- 参数1 ： 合约地址
- 参数2 ： 证书byte数组

 ::

  addr := "0x42a815e75604dd69707ba4aa9d350a59d1e530e7"
  cert1 := []byte(`-----BEGIN CERTIFICATE-----
  MIICMTCCAd2gAwIBAgIIcB4Bo1m3X4wwCgYIKoZIzj0EAwIwdDEJMAcGA1UECBMA
  MQkwBwYDVQQHEwAxCTAHBgNVBAkTADEJMAcGA1UEERMAMQ4wDAYDVQQKEwVmbGF0
  bzEJMAcGA1UECxMAMQ4wDAYDVQQDEwVub2RlMTELMAkGA1UEBhMCWkgxDjAMBgNV
  BCoTBWVjZXJ0MB4XDTIwMTAxOTAwMDAwMFoXDTIwMTAxOTAwMDAwMFowPTELMAkG
  A1UEBhMCQ04xDjAMBgNVBAoTBWZsYXRvMQ4wDAYDVQQDEwVub2RlMTEOMAwGA1UE
  KhMFZWNlcnQwVjAQBgcqhkjOPQIBBgUrgQQACgNCAASN1aGLwcwb/1c4NCaT6vAY
  A38Z5394RgUES1SlmrYWFCxwpOkpozPgMqZ+tS5PhFRt857ChrUujXzb6PWi6XVh
  o4GSMIGPMA4GA1UdDwEB/wQEAwIB7jAxBgNVHSUEKjAoBggrBgEFBQcDAgYIKwYB
  BQUHAwEGCCsGAQUFBwMDBggrBgEFBQcDBDAMBgNVHRMBAf8EAjAAMB0GA1UdDgQW
  BBSl7MAm0apBmKIN7fyBLcS7Y/w6hTAPBgNVHSMECDAGgAQBAgMEMAwGAypWAQQF
  ZWNlcnQwCgYIKoZIzj0EAwIDQgDJ3kJ3uX/23BM9JqCwlDpympv6Eu0OPriz4KgG
  72Hr7xRJOrmZ14waO/I4jAvba7+J1uaNIv0K6EDjJplNzvPEAA==
  -----END CERTIFICATE-----
  `)

  bvm.NewAccountAbandonOperation(addr, cert1)





4.2.5 证书的使用
>>>>>>>>>>>>>>>>>>

**4.2.5.1 证书吊销**

 ::

  func NewCertRevokeOperation(cert, priv []byte) (BuiltinOperation, error)



- 参数1  ： 证书byte数组
- 参数2  ： 证书私钥byte数组
- 返回值1 ： 内置合约操作BuiltinOperation
- 返回值2 ： error

说明：

- 当ca模式为`center` 时，删除节点后需要链级管理员手动吊销节点的证书，此时证书私钥可传`nil` 。

- 当ca模式为`none` 时，不可进行证书吊销操作。

应用实例

 ::

  ecert := []byte("-----BEGIN CERTIFICATE-----\nMIICHzCCAcSgAwIBAgIIapt5s0h7G4owCgYIKoEcz1UBg3UwPTELMAkGA1UEBhMC\nQ04xEzARBgNVBAoTCkh5cGVyY2hhaW4xDjAMBgNVBAMTBW5vZGUxMQkwBwYDVQQq\nEwAwIBcNMjEwMzI1MDAwMDAwWhgPMjEyMTAzMjUwMDAwMDBaMEQxCzAJBgNVBAYT\nAkNOMRMwEQYDVQQKEwpIeXBlcmNoYWluMQ4wDAYDVQQDEwVub2RlNTEQMA4GA1UE\nKhMHc2RrY2VydDBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABG401JscKfKj0rT3\nxN8Dwyen8mVCnXC3GBNkaENJEnqOO4jw0wT331CcX47bHMcSMRfpprbbv4cUj8cV\ncXNa9J6jgaQwgaEwDgYDVR0PAQH/BAQDAgHuMDEGA1UdJQQqMCgGCCsGAQUFBwMC\nBggrBgEFBQcDAQYIKwYBBQUHAwMGCCsGAQUFBwMEMAwGA1UdEwEB/wQCMAAwHQYD\nVR0OBBYEFPT6cvqWN9MBuhhlnmPrCQZG2iKoMB8GA1UdIwQYMBaAFJq1kzm0Q76P\nxf84+ZRlfrWBKy27MA4GAypWAQQHc2RrY2VydDAKBggqgRzPVQGDdQNJADBGAiEA\n3vcQvDi91E5GTsvV/IhKqrfuLkrnudN+3+QtocUX2IMCIQC6Ct1CS4c60SaE59tI\n3a/wjXSyWIYGN6Rwt0k0KFbF+w==\n-----END CERTIFICATE-----\n")
  priv := []byte("-----BEGIN EC PRIVATE KEY-----\nMHgCAQECIQClNEoZsGgZLfdMgYyMCWH8I0PLZynFp2U+wnsSzJ6z+6AKBggqgRzP\nVQGCLaFEA0IABG401JscKfKj0rT3xN8Dwyen8mVCnXC3GBNkaENJEnqOO4jw0wT3\n31CcX47bHMcSMRfpprbbv4cUj8cVcXNa9J4=\n-----END EC PRIVATE KEY-----\n")
  operation, ferr := bvm.NewCertRevokeOperation(ecert, priv)


**4.2.5.2 证书冻结**

text
func NewCertFreezeOperation(cert, priv []byte) (BuiltinOperation, error)



- 参数1  ： 证书byte数组
- 参数2  ： 证书私钥byte数组
- 返回值1 ： 内置合约操作BuiltinOperation
- 返回值2 ： error

应用实例

 ::

  ecert := []byte("-----BEGIN CERTIFICATE-----\nMIICHzCCAcSgAwIBAgIIapt5s0h7G4owCgYIKoEcz1UBg3UwPTELMAkGA1UEBhMC\nQ04xEzARBgNVBAoTCkh5cGVyY2hhaW4xDjAMBgNVBAMTBW5vZGUxMQkwBwYDVQQq\nEwAwIBcNMjEwMzI1MDAwMDAwWhgPMjEyMTAzMjUwMDAwMDBaMEQxCzAJBgNVBAYT\nAkNOMRMwEQYDVQQKEwpIeXBlcmNoYWluMQ4wDAYDVQQDEwVub2RlNTEQMA4GA1UE\nKhMHc2RrY2VydDBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABG401JscKfKj0rT3\nxN8Dwyen8mVCnXC3GBNkaENJEnqOO4jw0wT331CcX47bHMcSMRfpprbbv4cUj8cV\ncXNa9J6jgaQwgaEwDgYDVR0PAQH/BAQDAgHuMDEGA1UdJQQqMCgGCCsGAQUFBwMC\nBggrBgEFBQcDAQYIKwYBBQUHAwMGCCsGAQUFBwMEMAwGA1UdEwEB/wQCMAAwHQYD\nVR0OBBYEFPT6cvqWN9MBuhhlnmPrCQZG2iKoMB8GA1UdIwQYMBaAFJq1kzm0Q76P\nxf84+ZRlfrWBKy27MA4GAypWAQQHc2RrY2VydDAKBggqgRzPVQGDdQNJADBGAiEA\n3vcQvDi91E5GTsvV/IhKqrfuLkrnudN+3+QtocUX2IMCIQC6Ct1CS4c60SaE59tI\n3a/wjXSyWIYGN6Rwt0k0KFbF+w==\n-----END CERTIFICATE-----\n")
  priv := []byte("-----BEGIN EC PRIVATE KEY-----\nMHgCAQECIQClNEoZsGgZLfdMgYyMCWH8I0PLZynFp2U+wnsSzJ6z+6AKBggqgRzP\nVQGCLaFEA0IABG401JscKfKj0rT3xN8Dwyen8mVCnXC3GBNkaENJEnqOO4jw0wT3\n31CcX47bHMcSMRfpprbbv4cUj8cVcXNa9J4=\n-----END EC PRIVATE KEY-----\n")
  operation, ferr := bvm.NewCertFreezeOperation(ecert, priv)


**4.2.5.3 证书解冻**

 ::

  func NewCertUnfreezeOperation(cert, priv []byte) (BuiltinOperation, error)



- 参数1  ： 证书byte数组
- 参数2  ： 证书私钥byte数组
- 返回值1 ： 内置合约操作BuiltinOperation
- 返回值2 ： error

应用实例

 ::

  ecert := []byte("-----BEGIN CERTIFICATE-----\nMIICHzCCAcSgAwIBAgIIapt5s0h7G4owCgYIKoEcz1UBg3UwPTELMAkGA1UEBhMC\nQ04xEzARBgNVBAoTCkh5cGVyY2hhaW4xDjAMBgNVBAMTBW5vZGUxMQkwBwYDVQQq\nEwAwIBcNMjEwMzI1MDAwMDAwWhgPMjEyMTAzMjUwMDAwMDBaMEQxCzAJBgNVBAYT\nAkNOMRMwEQYDVQQKEwpIeXBlcmNoYWluMQ4wDAYDVQQDEwVub2RlNTEQMA4GA1UE\nKhMHc2RrY2VydDBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABG401JscKfKj0rT3\nxN8Dwyen8mVCnXC3GBNkaENJEnqOO4jw0wT331CcX47bHMcSMRfpprbbv4cUj8cV\ncXNa9J6jgaQwgaEwDgYDVR0PAQH/BAQDAgHuMDEGA1UdJQQqMCgGCCsGAQUFBwMC\nBggrBgEFBQcDAQYIKwYBBQUHAwMGCCsGAQUFBwMEMAwGA1UdEwEB/wQCMAAwHQYD\nVR0OBBYEFPT6cvqWN9MBuhhlnmPrCQZG2iKoMB8GA1UdIwQYMBaAFJq1kzm0Q76P\nxf84+ZRlfrWBKy27MA4GAypWAQQHc2RrY2VydDAKBggqgRzPVQGDdQNJADBGAiEA\n3vcQvDi91E5GTsvV/IhKqrfuLkrnudN+3+QtocUX2IMCIQC6Ct1CS4c60SaE59tI\n3a/wjXSyWIYGN6Rwt0k0KFbF+w==\n-----END CERTIFICATE-----\n")
  priv := []byte("-----BEGIN EC PRIVATE KEY-----\nMHgCAQECIQClNEoZsGgZLfdMgYyMCWH8I0PLZynFp2U+wnsSzJ6z+6AKBggqgRzP\nVQGCLaFEA0IABG401JscKfKj0rT3xN8Dwyen8mVCnXC3GBNkaENJEnqOO4jw0wT3\n31CcX47bHMcSMRfpprbbv4cUj8cVcXNa9J4=\n-----END EC PRIVATE KEY-----\n")
  operation, ferr := bvm.NewCertUnfreezeOperation(ecert, priv)


**4.2.5.4 证书检查**

 ::

  func NewCertCheckOperation(cert []byte) BuiltinOperation


- 参数1  ： 证书byte数组
- 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  ecert := []byte("-----BEGIN CERTIFICATE-----\nMIICHzCCAcSgAwIBAgIIapt5s0h7G4owCgYIKoEcz1UBg3UwPTELMAkGA1UEBhMC\nQ04xEzARBgNVBAoTCkh5cGVyY2hhaW4xDjAMBgNVBAMTBW5vZGUxMQkwBwYDVQQq\nEwAwIBcNMjEwMzI1MDAwMDAwWhgPMjEyMTAzMjUwMDAwMDBaMEQxCzAJBgNVBAYT\nAkNOMRMwEQYDVQQKEwpIeXBlcmNoYWluMQ4wDAYDVQQDEwVub2RlNTEQMA4GA1UE\nKhMHc2RrY2VydDBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABG401JscKfKj0rT3\nxN8Dwyen8mVCnXC3GBNkaENJEnqOO4jw0wT331CcX47bHMcSMRfpprbbv4cUj8cV\ncXNa9J6jgaQwgaEwDgYDVR0PAQH/BAQDAgHuMDEGA1UdJQQqMCgGCCsGAQUFBwMC\nBggrBgEFBQcDAQYIKwYBBQUHAwMGCCsGAQUFBwMEMAwGA1UdEwEB/wQCMAAwHQYD\nVR0OBBYEFPT6cvqWN9MBuhhlnmPrCQZG2iKoMB8GA1UdIwQYMBaAFJq1kzm0Q76P\nxf84+ZRlfrWBKy27MA4GAypWAQQHc2RrY2VydDAKBggqgRzPVQGDdQNJADBGAiEA\n3vcQvDi91E5GTsvV/IhKqrfuLkrnudN+3+QtocUX2IMCIQC6Ct1CS4c60SaE59tI\n3a/wjXSyWIYGN6Rwt0k0KFbF+w==\n-----END CERTIFICATE-----\n")
  operation, ferr := bvm.NewCertCheckOperation(ecert, priv)


4.2.6 DID的使用
>>>>>>>>>>>>>>>>>>>

**4.2.6.1 设置chainId**

 ::

  func NewDIDSetChainIDOperation(chainID string) BuiltinOperation


说明：通过创世账户来设置链的chainId，来激活DID服务

应用实例

 ::

  accountJson := `{"address":"0xfbca6a7e9e29728773b270d3f00153c75d04e1ad","version":"4.0","algo":"0x13","publicKey":"049c330d0aea3d9c73063db339b4a1a84d1c3197980d1fb9585347ceeb40a5d262166ee1e1cb0c29fd9b2ef0e4f7a7dfb1be6c5e759bf411c520a616863ee046a4","privateKey":"5f0a3ea6c1d3eb7733c3170f2271c10c1206bc49b6b2c7e550c9947cb8f098e3"}`
  key, _ := account.GenKeyFromAccountJson(accountJson, "")
  opt := bvm.NewDIDSetChainIDOperation("chainID_01")
  payload := bvm.EncodeOperation(opt)
  // 发送交易
  tx := rpc.NewTransaction(key.(account.Key).GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
  rpc.SignAndInvokeContract(tx, key)




4.2.7 MPC的使用
>>>>>>>>>>>>>>>>>

**4.2.7.1 获取mpc历史记录**

 ::

  func NewMPCHistoryOperation(ct string) BuiltinOperation


： 参数1  ： mcp使用的曲线类型
： 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  accountJson := `{"address":"0xfbca6a7e9e29728773b270d3f00153c75d04e1ad","version":"4.0","algo":"0x13","publicKey":"049c330d0aea3d9c73063db339b4a1a84d1c3197980d1fb9585347ceeb40a5d262166ee1e1cb0c29fd9b2ef0e4f7a7dfb1be6c5e759bf411c520a616863ee046a4","privateKey":"5f0a3ea6c1d3eb7733c3170f2271c10c1206bc49b6b2c7e550c9947cb8f098e3"}`
  key, _ := account.GenKeyFromAccountJson(accountJson, "")
  opt := bvm.NewMPCHistoryOperation(common.CurveNameSM9)
  payload := bvm.EncodeOperation(opt)
  // 发送交易
  tx := NewTransaction(key.GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
  res, serr := rpc.SignAndInvokeContract(tx, key)


**4.2.7.2 获取mpc某一记录具体信息**

 ::

  func NewMPCInfoOperation(tag, ct string) BuiltinOperation



- 参数1  ： 该记录指定tag，填空值""默认获取最新记录信息
- 参数2  ： mcp使用的曲线类型
- 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  accountJson := `{"address":"0xfbca6a7e9e29728773b270d3f00153c75d04e1ad","version":"4.0","algo":"0x13","publicKey":"049c330d0aea3d9c73063db339b4a1a84d1c3197980d1fb9585347ceeb40a5d262166ee1e1cb0c29fd9b2ef0e4f7a7dfb1be6c5e759bf411c520a616863ee046a4","privateKey":"5f0a3ea6c1d3eb7733c3170f2271c10c1206bc49b6b2c7e550c9947cb8f098e3"}`
  key, _ := account.GenKeyFromAccountJson(accountJson, "")
  opt := bvm.NewMPCInfoOperation("", common.CurveNameBN254)
  payload := bvm.EncodeOperation(opt)
  // 发送交易
  tx := NewTransaction(key.GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
  res, serr := rpc.SignAndInvokeContract(tx, key)


**4.2.7.3 新增mpc记录**

 ::

  func NewMPCBeaconOperation(ptau []byte) BuiltinOperation


- 参数1  ： 使用zkp工具生成的contribution内容
- 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  const srsCon1Base = "AQAAAAwAAAAAAAAAYm4yNTQAAAAFAAAAAgAAAMAPAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAiD8NbtSTbhAjF7a2A1plNW1QWXGdQ4DqhexXq5tlSGTKHghqdiaB3pDAKrvpsQWR2j+OHusYAS0Lb7ip9b5RBcZA/2JvyFhTikl1sL9PBepE0t4TXZsdzZbwgUL73/rDyn1pS+YmHh8v/LaoC/zmmYXvGaAaO+d3oBzI2FJ9d6wDeQwUd8e8RtfcygPyWMpncmtJRjp+MXRQKHWvHSM4+Al2VVG6lWD4WSSF0B3olCI9jSlibST8WVsulHf0Su6dxKM9kF6RrY4Td+iRSfe9ibz4hYEXN4O6CNMGgafKwQBGLZPRFL8MpRSzp/Vgml6X6JI5k1pnKXkClocTnODfNkcXuoeBYr3vvPfgcuDleDpUnFH3075qrJk1xrwCRohIxATXum9/HhDwarrYVpQrMppWvu7hFDgNT/RIyYEKqmDGJAgobdR5iIyUrJiDB7g7ecfWshIYMq12v0rcQCcbycJ9HPKseMnBAltRoasrB6QVGBCbRHGRItUVTDCrUU4LCrHxyk4ZIuxyW545kd02CI4LhIHUvwPJQSretdNiFjpIu58BVibxTWUj7tCj34ZmiZqwB9I0YFL0oFuRltkdtkm0DQc+ReY91YQvpkKo97Y2OFFXV9c+GjCHw7IrtWGhRjq6P4VK65ps97g7FXUjaR082Hdp2EdFBs9xVM8dwfyLI+IVyXZnNhyfo6ZEI+Ex1caaqUL2wfd4WO7g/Qm+wEvVP4DwPzOIcvQNuJp32kW8J8rxkrA7NkR9vYaX90IcxqaZLXVi26GjWk5mZSHlynfmhKtdFjrkhFe3i34wmHGCAG0G24Q21uIZRQIgvXWt6jDKnFK1YG4Mw625aNiXgQeezebBzslzNTuyejEwVYs2EbGlNNk14j6klcPaWd5cimRLN+9iS8TR1EMnfVcBxV6A0SWIg9BY+bebql4CIFMAAmueybmv7x8R1DAv/1D+T+NwDf07+YNLf6pLTVC+YYmbbYVKWH+DFrzyvWKut/O6HewUFVK9vlyIB0kjENNVy2feKk8GVyD96sSrJD4ORqfGlad6T/tivmJa9Lzg3GGL+Vo27lslYx9iNG8KcgiUveJHaaWtASw1o75HgNtI8wSPDCt0nRLSx7YRM5psFcNZPDwgoPBc+Twtgfl+xXLuSEvF2YrjC7XMbey/mIWys8lSFc9XxWEv5RfhNzufRa9DK7yZl4e2QuQAahnOGsLWyQuw4iU1+ODfyQSWbEnTm0GHUr+y4F4lNz5AZ5idRNqJtY+sMyZuMoYRs20L7zKTwG66l1Tfbb9G5iLjB/5CWLR9pgTvpD2QUUUYijw0pFeJ4cDfPRCiPwHGkwWgJxMQsGGnVrskHKUmpRj5BOKWIoN3voTQtBasjRdYNkzhWhXX71m9x6Z186Q6nBGwZgR2RAJ6xYubF+Wi/g73TmeQ6b716owx8iri+xVyCAUDaqVLnbQgUMcM9hq+Bn+4jOGR3wnOngaeUOsPgM7GoI1unEmZZ9mjlArSGcRE0BQh50aCCneTCLQUY08lPrDb3FhcRWBYz/JYxM4HM8DObkwQyIPUatU5fWIedaDI38+EHygAkEZtCGLoKz/NdJcVJK/HJ1veIEu+4LRqSwIKzTDkYgC80K9WGHNtbNv3Tluy6nHgZNp0gGaaf1y2Vu7Ob/sfQlfaTyfBJfqpGzCG2jOA7nunh88tXPprQyWlqSte1TuGDefeCgkYPg+OHDV7DdyWsWw16PAaoqUrerlJnvvNwEh4DUEsNvw7Q+WuFtIMydZuA3WN5wRe6E3RB8d/fDcDx7AU9q13LOczxxhglETSMCnfuSqYfiXQ94/Wk0TByUMEOTeGnCnUsixZaIHDbX7bAcs6880QfhjjHeejtfzJMwsqJU1z81QFSV8eQZge4ljYU7Qqy++1cLNOxWbBmIFnSQg0Sw1Kg0P/cOUv0a2/AmtkbL29Y968PlSju29DCJ8Ilajp7ZO2wJRZkyI19yvJvLFMbdasGBJ959ox8zR3KII87IfOZm9tvoOlbuuhxWDu8scDntqPg8gSx9FpNYTpBHAFOczCIxV9r7FUOja7sL+P6BNy/nmXEa4YCHoovACMFtjt9yHrNKxCMHmkzpICutDAkA3LKd9IgpjMKlvE4wGHVvXBmyn0RYhyqdO+Di1zjpj8eQ3Cokfj+Q3tqf4FRC+lP8XjuV8FGzskRNpuP650ZTGXl7g9c8ZvBywSI7iAmqdhrLm4iqJ0s0lIjLqQ52XW0f8wmYU6oXmcrVhOKMf14sK0X/pgFU4P3G4dn/1UOrospLhy3QVOmKXuvhhARNvdRs3BFibQYDr+9xMgAQKFVioh3sK1iptxtN5FixSEN0YKqBffhQ684ROeTsiKH7Z/EDVulaK97JLjCPCcEgfooCCY895nE9Ec5I4yxoAHltTiHi9I8U42g7QaKvuVRckgeuXtW9aH7Rfj+WPDZJzkV5izj0Z7PoUHcK3KCrNHeOlSbCJEaR7oN8q20RTwDnWYHK2BIoI009oRQTV1RsSwCE3HSI6eVwMpZEgJwueTw3bjathP4COMBSJEp5xSgcooc6tZyC6y41ywrzV6xAt2pce6H9tGkCscySGTPIrEqK2vwQdOXDCOwSP3bhzJ3QfEXvHjpj/fi3RXtDQ+pEFk/M9onUX9gkOKvsBJjiuoHJt3ZV7kHi3IZ/8iyMO5hiv52giyJBmeuYEa/InIVSnICK/ByXi7l4DxqF5FzWXIedvKRpStsGlBf99PuLPptSJrSmLaaLCKw+Ij50RxU4dj4Ia5S9YGP36KkQ7UrydQG9SdzHnqSBxUvR55fv1RwLx/4jI1N9ykTaHARu40THb9f2fRVcTnHz2yLSdd1t2J4BzB5nw4xURrbDWPp4Mxeme59p8RVEZpL/ZqM2sMQsMSvyGpDUd5BPDOlCBOe2I36YOqdB65HmuvdPYVDpuWQF7EZHP2N2DiuZjY1loamlufeHKFywoIuVeTV/q8hNfCBK5M5de+EG+ki/u/qtmwMaxV+lWXIOxPkD9MHBGapgTIRdPhPHReT0m50K3KW7dMb0ovOYl5fcQKmN8siVhXQSO0eEyJXASEUDowOGlDO96FYIiuTgjuH7zUlFZFiAwLrvfoWODRmEAEGIzR2B4C25ehCLwdNtZOPBudyYZlaUOSosQIDJn9GZL6yta3wVN9laApBDQgQtHBKO6By4w9iIUzuWcbbxVltdzYhr19Pi7KnTWJaFxdbP0gzfycxJxKguEsXGbfmKpA0kBhUPvowAnRmKZgeVcPVQKTCZDy24U683n3MIOC4KnKEVFTU96I+tM5UYrJ3TkfwhWIJxjnQUgAcDGUBhaFCUH+KZ3lEeCegu2xCYPGvi0uanpPkkCJcgab85f3AU9u+AtaUBlHZnxYGmKCjo/XgWwgVnVT+Iuiyfvd2HNP/vrYb0r1OqUM/TsM4n+CbdqROovGbfwowbi+gBYATJ3y9XIJyN1YHJyOr5s9AoxCz3hyr8NnTQxDkSxKrx73MxvpIbb1gde2hz6W5nWfCiEWswipQAc2L0BnsaBl2XKYoZcf5fXyQoJmN/Ffen6kAYVGsVqSt59aRXG56skZdyK6XS/6n/KciLUESuXCkqPDwEPpFLJXudiENfQz0OswmQed9mw9ImO1GkrpEQpwRWoLLePrMUdiGQoLnoIiyTZAa+4J0FVa/q0/27OgUH2EZi8BlFWH4F6dAvkjt5f3fZ2ut7UM/UXvenSW15iRfT770aZDFiVjBKADc5D2W3qFFT2iGT2Iv3gDhaBNIkEU/vHcWCktZ2x8q4NcDrJ+P7X+Pmg+xzxk9KbS+Vwg9hakeeMQxp4mxsdlSVAlzAfJRjtKZK/3xqes84Q8JgJ+9M31CoI2QC2PdB/ILd1bVeLAvG28UkCKQRqic74fWSK7uDIlelQVu03c14J/6cI8R4m3qJszrCGLEoH3hH9RsawN2ACC7H19QjP3BkLuDKjVcQMqm/QrwkbKAAKx1Xtj4EjXh4Zl3Nq4TMAFrfVmvSS68lU1z8+6vXD3/wFlnpRChBlNhzBN2xrMz0F5Y+aFVJkqFtW99hFAwjCMnA2SZ5R565jIZ2BZR/YPAL8rOiz4IevGqb3eyaXjSrp0CEQNoHY5TV0D+PYUFigByzDCYHYw0IWgRPHFRZyVgM2xIKWIeOdevNhEqU32oop2wPYYjb1IKw80eIdEmgMznruA/ZZQl5QY6MYe+yo3jALxCB7IEoOCTOWptG4Y3WPi4NynsexoF3fo80OzyzkCw9CHcWOnlPVYKpf3IAfzVpMd6TIQjCYpuK2UQkMomQPiLzU1m8UQw7REWjCA8FEvfW5uuupcJ2J5u07/L8wbAI8YvclWQoJ/2tGQaJnOiPp+Jz3dzBl82/BfxTXpACqLo6Gon6xoham5BIjwkcw0Y+P6B4KgfOk8eh658qnw1IrZ6e73hdRFpnejuzcKKnh6Gmo3xt72mx+eQUg7Ex5cwE/ThiDz/whB90jCRDIXLW0hmhVE1ycfh3jvS84ZcDiAFwrV3+9macEoaaFTmb0rnHn3h9kr3MkzH7li14lA5IEny0N3/F6KdY8MqC0USOF61YlmwSwvb01fFNtNccW9htYZgO3PGmdYpbhBXJ02qS1QLb5FO/5XC1bS61cQIiMLZP81U1ntApsbhTsqy22k7p5CrhRPDbskBcjt0PfNp0T+aTXzYnfTyFdRZCAkqm5OAUbdkY1EHe4lmNX+qfhGxXLgi8zM3cD5NXNKeT1jdkVIlbx9SbPHQDF46fKNcYmGgjjEZHM08J25J/dQss4sg135HMoZEokiCPQA2S1spMlJ1/Nwv0ausUNAbynzs6lVZClGhVSCIUiIfUGpqyRbRPEqeRtHZHQA71HdVA2FLLQTTMEJcuhx2vFerWMtcffEU4govaTkFbMSUrJTFijzUFP4dxaW1iNtNUqwTd8nNYZb18ttgQ/G7497cBz5fRXOggbtwEkKMc6YOCmrjEt8BAf7bUr7pdEko+/fYw355QdD9HpWRcUgH0IAeMxDeR4E0b8sSeCZavb2VcNmuj91HwJAfcXg/0L2RVqhQ1vp4sX44wgm1tIG9rMltVHymydKx694S7MnyhkAvQNounUiCkSnru+ZJVVTd+afQwAbhjBICa+VYi75+zzoicecmJuC7P2Q3eY5rR6Doiq0OSr/MiGH/pkM+1IhUJ3sLnoAdQVflcGpb3Qp/0qrrgMxJQ2SmXnyyI+SwSFdm86yvfcaBWAF+ZzeU7A15MajMdM5QK6xs2sNyICdYBgg71CqfXrAwAAAAABAAAAAAAAGY6Tk5INSDpyYL+3MftdJfGqSTM1qecSl+SFt67zEsIYAN7vEh8edkJqAGZeXER5Z0Mi1Pde2t1G3r1c2ZL27QkGidBYX/B17J6ZrWkMM5W8SzEzcLOO81Ws2tzRIpdbEshepduMbetKq3GAjctAj+PR52kMQ9N7TObMAWb6faoDQSJmp9+U7qYpJTmOfnKQQPgJtxzexEkjqZI0Xlsn3B+0j0sfmZ8DeryxfIvOflg0eB2E0pEbxW131BcRf9hcDjd30953YKwGKJbePsoUDJZFQiCelPUPg+ckWBhM4u4gI3jmqpSNnrpZRwn13fEnF69P+jDBX+FlmGVGx7SW+QQAAACWAgAAAAAAAJICAAAAAAAAAAAAAAYAAAAxX2NvbjENwnw9D5tibZwnkQWSspX1dcQWVKW/8kjdVg56eHACaRZWuXzNdxdrB7qFg2T2yWoHdd6/jknrqndkAtXMOBJgB7fjVdMygKDkfEPzwZaMuiFopueSbaPu2dLnx1EUpV4WbMtCx5iJHvBQ8ajbDQY0UsDXvHvRPC9/91uYT/qryieewE9mkPbqU4amOhF6wJbPdSZLL92jgZI7ZBdns3p6KeoXBNYxzX4lFMTUTuSoozWHK1tCMoeLwoPruqyvky8rcQLT5oaGlSAgHouHy4x5Eo40tGFkEse+Mi+WkfMyyBqvMppk5RWIytuo7nSBe2Mbcv7jl5nJSdhEpXlE6vLcKOCLBETiM7rLUN4W6GqDMaOSV28XRwTyHGE7zm2KgdAwLXpWbqi73/lsjz3hFKinrZGGP3XcF/zpcVphoXVXvhAqFLfJZGX0tTQq1Kx+ST6ZKq2FOiqiTwg02jxUx+dZBwVAwPw/18U0pUypd69zNnxJvI1q8tHghMAlk9OFS0Mg/DW7Uk24QIxe2tgNaZTVtUFlxnUOA6oXsV6ubZUhkyh4IanYmgd6QwCq76bEFkdo/jh7rGAEtC2+4qfW+UQXA0EiZqfflO6mKSU5jn5ykED4Cbcc3sRJI6mSNF5bJ9wftI9LH5mfA3q8sXyLzn5YNHgdhNKRG8Vtd9QXEX/YXA43d9Ped2CsBiiW3j7KFAyWRUIgnpT1D4PnJFgYTOLuICN45qqUjZ66WUcJ9d3xJxevT/owwV/hZZhlRse0lvnHmL98fHd4DXMZEV0xjUTBYta5spLdDU5oEXsaVX7vLRwXXXieRiPb16SKlPZhXx7x1s5UVFLKEflMF/f23K5I"

  accountJson := `{"address":"0xfbca6a7e9e29728773b270d3f00153c75d04e1ad","version":"4.0","algo":"0x13","publicKey":"049c330d0aea3d9c73063db339b4a1a84d1c3197980d1fb9585347ceeb40a5d262166ee1e1cb0c29fd9b2ef0e4f7a7dfb1be6c5e759bf411c520a616863ee046a4","privateKey":"5f0a3ea6c1d3eb7733c3170f2271c10c1206bc49b6b2c7e550c9947cb8f098e3"}`
  key, _ := account.GenKeyFromAccountJson(accountJson, "")
  resb, err := base64.StdEncoding.DecodeString(srsCon1Base)
  opt := bvm.NewMPCBeaconOperation(resb)
  payload := bvm.EncodeOperation(opt)
  // 发送交易
  tx := NewTransaction(key.GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
  res, serr := rpc.SignAndInvokeContract(tx, key)




4.2.8 Root CA的使用
>>>>>>>>>>>>>>>>>>>>>>>

**4.2.8.1 新增root ca**

当ca模式为`center` 时，链级管理员可以新增root ca。

 ::

  func NewRootCAAddOperation(rootCA string) BuiltinOperation


- 参数1  ： 新增加的root ca内容
- 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  accountJson := `{"address":"0xfbca6a7e9e29728773b270d3f00153c75d04e1ad","version":"4.0","algo":"0x13","publicKey":"049c330d0aea3d9c73063db339b4a1a84d1c3197980d1fb9585347ceeb40a5d262166ee1e1cb0c29fd9b2ef0e4f7a7dfb1be6c5e759bf411c520a616863ee046a4","privateKey":"5f0a3ea6c1d3eb7733c3170f2271c10c1206bc49b6b2c7e550c9947cb8f098e3"}`
  key, _ := account.GenKeyFromAccountJson(accountJson, "")
  rootCA := `-----BEGIN CERTIFICATE-----
  MIICODCCAeSgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV
  BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD
  VQQLEwAxDjAMBgNVBAMTBW5vZGU1MQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl
  cnQwIBcNMjAwNTIyMDUyNTQ4WhgPMjEyMDA0MjgwNjI1NDhaMHQxCTAHBgNVBAgT
  ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh
  dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTUxCzAJBgNVBAYTAlpIMQ4wDAYD
  VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABLs/ih4HBDxs0nHNliXt
  sZKAaW2fdgi51H07eMNtHib/8R55GYFinvjmeJa9OvbyYsWAbCTQsBu3LV7aPpOT
  0QGjaDBmMA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB
  BQUHAwEGAioDBgOBCwEwDwYDVR0TAQH/BAUwAwEB/zANBgNVHQ4EBgQEAQIDBDAM
  BgMqVgEEBWVjZXJ0MAoGCCqGSM49BAMCA0IA/9MkrtXUMni8THIhym5t2manIffB
  xpW51rRuxPjB/0wZvgKPDWuOcq8eqsOOX/d4wFtMwJ4e7MqLZu6RWSc2MQA=
  -----END CERTIFICATE-----
  `
  opt := bvm.NewRootCAAddOperation(rootCA)
  payload := bvm.EncodeOperation(opt)
  // 发送交易
  tx := NewTransaction(key.GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
  res, serr := rpc.SignAndInvokeContract(tx, key)




**4.2.8.2 查询root ca**

当ca模式为`center` 时，可以查询root ca。

 ::

  func NewRootCAGetOperation() BuiltinOperation



- 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  accountJson := `{"address":"0xfbca6a7e9e29728773b270d3f00153c75d04e1ad","version":"4.0","algo":"0x13","publicKey":"049c330d0aea3d9c73063db339b4a1a84d1c3197980d1fb9585347ceeb40a5d262166ee1e1cb0c29fd9b2ef0e4f7a7dfb1be6c5e759bf411c520a616863ee046a4","privateKey":"5f0a3ea6c1d3eb7733c3170f2271c10c1206bc49b6b2c7e550c9947cb8f098e3"}`
  key, _ := account.GenKeyFromAccountJson(accountJson, "")
  opt := bvm.NewRootCAGetOperation()
  payload := bvm.EncodeOperation(opt)
  // 发送交易
  tx := NewTransaction(key.GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
  res, serr := rpc.SignAndInvokeContract(tx, key)

4.2.9 跨分区交易
>>>>>>>>>>>>>>>>>>>>>

（暂不开放）

..
 4.2.9 跨分区交易
    >>>>>>>>>>>>>>>>>>>>>

    flato平台为跨分区交易暴露了一种新的API，前缀为`crosschain_`，可使用的API接口包括3种，分别对应三类跨分区相关交易：

    | API名称                            | 交易类别        |
    | -------------------------------- | ----------- |
    | crosschain_invokeAnchorContract  | 锚节点相关交易     |
    | crosschain_invokeContract        | 跨分区事务交易     |
    | crosschain_invokeTimeoutContract | 触发跨分区事务超时交易 |

    各类跨分区交易都通过hvm合约或者bvm合约完成。接下来的章节将介绍各个功能对应的合约类型和合约调用接口。

    此外，为方便用户使用，平台内置合约提供了获取链上锚节点状态/链上跨分区事务状态的接口，这一类接口是普通的bvm合约调用，通过contract_invokeContract服务来实现。

    **4.2.9.1 注册锚节点（global）**

    - vm类型：BVM

    - 合约地址：“0x0000000000000000000000000000000000ffff0a”

    - API接口：crosschain_invokeAnchorContract

    - 目标分区：global

    - 合约内容构造函数：

     ::

      func NewSystemAnchorOperation(method ContractMethod, args ...string) BuiltinOperation


    - 当前功能合约调用具体参数：

     ::

      NewSystemAnchorOperation(bvm.RegisterAnchor, "hostname", "namespace")


    - 参数1  - 注册锚节点合约函数名称
    - 参数2  - 锚节点的hostname
    - 参数3  - 锚节点预期管理的分区名称
    - 返回值1 - 内置合约操作BuiltinOperation

    应用实例

     ::

      key, _ := account.NewAccountFromAccountJSON(accountJsons[0], "")
      operation1 := bvm.NewSystemAnchorOperation(bvm.RegisterAnchor, "node1", "ns2")
      payload1 := bvm.EncodeOperation(operation1)
      tx1 := NewTransaction(key.GetAddress().Hex()).Invoke(operation1.Address(), payload1).VMType(BVM)
      rpc.namespace = "global"
      re1, err := rpc.SignAndInvokeCrossChainContract(tx1, "invokeAnchorContract", key)


    **4.2.9.2 注册锚节点（普通分区）**

    - vm类型：BVM

    - 合约地址：“0x0000000000000000000000000000000000ffff08”

    - API接口：crosschain_invokeAnchorContract

    - 目标分区：普通分区

    - 合约内容构造函数：

     ::

      func NewNormalAnchorOperation(method ContractMethod, args []string) BuiltinOperation


    - 当前功能合约调用具体参数：

     ::

      NewNormalAnchorOperation(bvm.RegisterAnchor, []string{"hostname"})


    - 参数1  : 注册锚节点合约函数名称
    - 参数2  : string类型数组，包含一个元素：锚节点的hostname
    - 返回值1 : 内置合约操作BuiltinOperation

    应用实例

     ::

      key, _ := account.NewAccountFromAccountJSON(accountJsons[0], "")
      operation := bvm.NewNormalAnchorOperation(bvm.RegisterAnchor, []string{"node1"})
      payload := bvm.EncodeOperation(operation)
      tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
      rpc.namespace = "ns1"
      res, err := rpc.SignAndInvokeCrossChainContract(tx, "invokeAnchorContract", key)


    **4.2.9.3 替换锚节点（global）**

    - vm类型：BVM

    - 合约地址：“0x0000000000000000000000000000000000ffff0a”

    - API接口：crosschain_invokeAnchorContract

    - 目标分区：global

    - 合约内容构造函数：

     ::

      func NewSystemAnchorOperation(method ContractMethod, args ...string) BuiltinOperation


    - 当前功能合约调用具体参数：

     ::

      NewSystemAnchorOperation(bvm.ReplaceAnchor, "old_hostname", "old_namespace", "new_hostname", "new_namespace")



    - 参数1  - 替换锚节点合约函数名称
    - 参数2  - 旧的hostname
    - 参数3  - 旧的分区名称
    - 参数4  - 新的hostname
    - 参数5  - 新的分区名称
    - 返回值1 - 内置合约操作BuiltinOperation

    应用实例

     ::

      key, _ := account.NewAccountFromAccountJSON(accountJsons[0], "")
      operation := bvm.NewSystemAnchorOperation(bvm.ReplaceAnchor, "node2", "ns1", "node1", "ns1")
      payload := bvm.EncodeOperation(operation)
      tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
      rpc1 := rpc.NewRPC().Namespace("global")
      res, err := rpc1.SignAndInvokeCrossChainContract(tx, "invokeAnchorContract", key)


    **4.2.9.4 global解注册锚节点**

    - vm类型：BVM

    - 合约地址：“0x0000000000000000000000000000000000ffff0a”

    - API接口：crosschain_invokeAnchorContract

    - 目标分区：global

    - 合约内容构造函数：

     ::

      func NewSystemAnchorOperation(method ContractMethod, args ...string) BuiltinOperation


    - 当前功能合约调用具体参数：

     ::

      NewSystemAnchorOperation(bvm.UnRegisterAnchor, "hostname", "namespace")



    - 参数1  ： 解注册锚节点合约函数名称
    - 参数2  ： 锚节点的hostname
    - 参数3  ： 锚节点预期管理的分区名称
    - 返回值1 ： 内置合约操作BuiltinOperation

    应用实例

     ::

      key, _ := account.NewAccountFromAccountJSON(accountJsons[0], "")
      operation := bvm.NewSystemAnchorOperation(bvm.UnRegisterAnchor, "node1", "ns2")
      payload := bvm.EncodeOperation(operation)
      tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
      rpc1 := rpc.NewRPC().Namespace("global")
      res, err := rpc1.SignAndInvokeCrossChainContract(tx, "invokeAnchorContract", key)


    **4.2.9.5 查询锚节点注册状态**

    - vm类型：BVM

    - 合约地址：“0x0000000000000000000000000000000000ffff0a”

    - API接口：contract_invokeContract

    - 目标分区：global

    - 合约内容构造函数：

     ::

      func NewSystemAnchorOperation(method ContractMethod, args ...string) BuiltinOperation


    - 当前功能合约调用具体参数：

     ::

      NewSystemAnchorOperation(bvm.ReadAnchor, "namespace")



    - 参数1  ： 查询锚节点注册状态合约函数名称
    - 参数2  ： 分区名称
    - 返回值1 ： 内置合约操作BuiltinOperation

    应用实例

     ::

      operation := bvm.NewSystemAnchorOperation(bvm.ReadAnchor, "ns3")
      payload := bvm.EncodeOperation(operation)
      tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
      rpc1 := rpc.NewRPC().Namespace("global")
      res, err := rpc1.SignAndInvokeContract(tx, key)


    > 注：这里使用的是`rpc.SignAndInvokeContract`函数来构造法发往平台的请求。

    **4.2.9.6 跨分区事务超时**

    - vm类型：BVM

    - 合约地址：“0x0000000000000000000000000000000000ffff0a”

    - API接口：crosschain_invokeTimeoutContract

    - 目标分区：global

    - 合约内容构造函数：

     ::

      func NewSystemAnchorOperation(method ContractMethod, args ...string) BuiltinOperation


    - 当前功能合约调用具体参数：

     ::

      NewSystemAnchorOperation(bvm.Timeout, "crossChainID")



    - 参数1  ： 触发跨分区事务超时合约函数名称
    - 参数2  ： 跨分区事务ID
    - 返回值1 ： 内置合约操作BuiltinOperation

    应用实例

     ::

      operation := bvm.NewSystemAnchorOperation(bvm.Timeout, "ns1__@__0x16b22229e93fdad80cef0920cc26d2fb6acb32945f8f5fe2248f0adcc3aa2c29__@__ns2")
      payload := bvm.EncodeOperation(operation)
      tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
      rpc1 := rpc.NewRPC().Namespace("global")
      res, err := rpc1.SignAndInvokeCrossChainContract(tx, "invokeTimeoutContract", key)




    **4.2.9.7 查询跨分区事务状态**

    - vm类型：BVM

    - 合约地址：“0x0000000000000000000000000000000000ffff0a”

    - API接口：contract_invokeContract

    - 目标分区：global

    - 合约内容构造函数：

     ::

      func NewSystemAnchorOperation(method ContractMethod, args ...string) BuiltinOperation


    - 当前功能合约调用具体参数：

     ::

      NewSystemAnchorOperation(bvm.ReadCrossChain, "crossChainID")


    - 参数1  ： 获取跨分区事务状态合约函数名称
    - 参数2  ： 跨分区事务ID
    - 返回值1 ： 内置合约操作BuiltinOperation

    应用实例

     ::

      operation := bvm.NewSystemAnchorOperation(bvm.ReadCrossChain, "ns1__@__0x16b22229e93fdad80cef0920cc26d2fb6acb32945f8f5fe2248f0adcc3aa2c29__@__ns2")
      payload := bvm.EncodeOperation(operation)
      tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
      rpc1 := rpc.NewRPC().Namespace("global")
      res, err := rpc1.SignAndInvokeContract(tx, key)


    > 注：这里使用的是`rpc.SignAndInvokeContract`函数来构造法发往平台的请求。


4.2.10 hash类型查询
>>>>>>>>>>>>>>>>>>>>>>

**4.2.10.1 获取目前支持的算法类型**

- vm类型：BVM

- 合约地址：“0x0000000000000000000000000000000000ffff0d”

- API接口：contract_invokeContract

- 目标分区：global

- 合约内容构造函数：

 ::

  func NewHashGetSupportOperation() BuiltinOperation


- 当前功能合约调用具体参数：

 ::

  NewHashGetSupportOperation()


- 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  operation := bvm.NewHashGetSupportOperation()
  payload := bvm.EncodeOperation(operation)
  tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
  rpc1 := rpc.NewRPC().Namespace("global")
  res, err := rpc1.SignAndInvokeContract(tx, key)


**4.2.10.2 获取已变更的算法记录**

- vm类型：BVM

- 合约地址：“0x0000000000000000000000000000000000ffff0d”

- API接口：contract_invokeContract

- 目标分区：global

- 合约内容构造函数：

 ::

  func NewHashGetAlgoOperation() BuiltinOperation


- 当前功能合约调用具体参数：

 ::

  NewHashGetAlgoOperation()


- 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  operation := bvm.NewHashGetAlgoOperation()
  payload := bvm.EncodeOperation(operation)
  tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
  rpc1 := rpc.NewRPC().Namespace("global")
  res, err := rpc1.SignAndInvokeContract(tx, key)


**4.2.10.3 变更算法**

- vm类型：BVM

- 合约地址：“0x0000000000000000000000000000000000ffff0d”

- API接口：contract_invokeContract

- 目标分区：global

- 合约内容构造函数：

 ::

  func NewHashChangeHashAlgo(data []byte) BuiltinOperation


- 当前功能合约调用具体参数：

 ::

  NewHashGetAlgoOperation(data)


- 返回值1 ： 内置合约操作BuiltinOperation

应用实例

 ::

  ha := bvm.AlgoSet{
      HashAlgo: "SHA2_512",
  }
  data, _ := json.Marshal(ha)
  operation := bvm.NewHashChangeHashAlgo(data)
  payload := bvm.EncodeOperation(operation)
  tx := NewTransaction(key.GetAddress().Hex()).Invoke(operation.Address(), payload).VMType(BVM)
  rpc1 := rpc.NewRPC().Namespace("global")
  res, err := rpc1.SignAndInvokeContract(tx, key)


4.2.11 Balance管理
>>>>>>>>>>>>>>>>>>>>>>

**4.2.11.1 设置balance**

 ::

  func NewIssueBalanceSetOperation(addr string, value int64) BuiltinOperation



- addr  ： 用户地址
- value ： 用户balance
- 返回值1  ： 内置合约操作BuiltinOperation
- 返回值2  ： error



4.3 HVM
--------------



4.3.1 abi解析
>>>>>>>>>>>>>>>>>>>>

abi解析后的结构如下

 ::

  type BeanAbi struct {
    BeanVersion Version  `json:"version"`
    BeanName    string   `json:"beanName"`
    Inputs      []Entry  `json:"inputs"`
    Output      Entry    `json:"output"`
    ClassBytes  string   `json:"classBytes"`
    Structs     []Entry  `json:"structs"`
    BeanType    BeanType `json:"beanType"`
  }

  type Entry struct {
    Name       string  `json:"name"`
    EntryType  Type    `json:"type"`
    Properties []Entry `json:"properties,omitempty"`
    StructName string  `json:"structName,omitempty"`
  }



**4.3.1.1 通过abiJson字符串解析合约**

 ::

  func GenAbi(abiJson string) (Abi, error)


应用实例

 ::

  abiPath := "../hvmtestfile/fibonacci/hvm.abi"
  abiJson, rerr := common.ReadFileAsString(abiPath)
  abi, gerr := hvm.GenAbi(abiJson)




**4.3.1.2 获取待调用的方法**

::

  func (abi Abi) GetBeanAbi(beanName string) (*BeanAbi, error)





- 参数1  ： 方法在abi中的名字
- 返回值1 ： beanAbi结构体
- 返回值2 ： error

应用实例

 ::

  methodInvokeAbi := "../hvmtestfile/methodInvoke/hvm.abi"
  abiJson, err := common.ReadFileAsString(methodInvokeAbi)
  abi, err := hvm.GenAbi(abiJson)
  easyBean := "invoke.InvokeFibonacci"
  beanAbi, err := abi.GetBeanAbi(easyBean)


***

 ::

  func (abi Abi) GetMethodAbi(methodName string) (*BeanAbi, error)



- 参数1  ： 合约方法签名
- 返回值1 ： beanAbi结构体
- 返回值2 ： error

应用实例

 ::

  methodInvokeAbi := "../hvmtestfile/methodInvoke/hvm.abi"
  abiJson, err := common.ReadFileAsString(methodInvokeAbi)
  abi, err := hvm.GenAbi(abiJson)
  _, err = abi.GetMethodAbi("Hello")
  methodAbi3, err := abi.GetMethodAbi("Hello(java.lang.String)")
  methodAbi4, err := abi.GetMethodAbi("Hello(int,java.lang.String)")


**4.3.1.3 编码beanAbi**

 ::

  func GenPayload(beanAbi *BeanAbi, params ...interface{}) ([]byte, error)


- 参数1  ： beanAbi的实例
- 参数2  ： beanAbi对应的合约方法的参数
- 返回值1 ： 编码后的beanAbi
- 返回值2 ： error



应用实例

 ::

  methodInvokeAbi := "../hvmtestfile/methodInvoke/hvm.abi"
  abiJson, err := common.ReadFileAsString(methodInvokeAbi)
  abi, err := hvm.GenAbi(abiJson)
  easyBean := "invoke.InvokeFibonacci"
  beanAbi, err := abi.GetBeanAbi(easyBean)
  payload, err := hvm.GenPayload(beanAbi)




**4.3.1.4 payload解码**

 ::

  type PayLoad struct {
    InvokeBeanName string   `json:"invokeBeanName"`
    InvokeArgs     string   `json:"invokeArgs"`
    InvokeMethods  []string `json:"invokeMethods"`
  }


  func DecodePayload(pl string) (*PayLoad, error)


- 参数1  : payload的十六进制字符串
- 返回值1 : PayLoad结构体
- 返回值2 : error

应用实例

 ::

  result, err := hvm.DecodePayload("0xfefffbce00087472616e736665720010000000036a6176612e6c616e672e537472696e674141410010000000036a6176612e6c616e672e537472696e674242420004000000036c6f6e67313030")




4.3.2 jar包部署
>>>>>>>>>>>>>>>>>>>>>

jar包部署合约需要对jar包进行编码

 ::

  func DecompressFromJar(filepath string) ([]byte, error)


- 参数1  : jar包文件路径
- 返回值1 ： 编码后的byte数组
- 返回值2 ： error

应用实例

 ::

  jarPath := "../hvmtestfile/hvm-first-1.0.jar"
  payload, sysErr := rpc.DecompressFromJar(jarPath)
  transaction := rpc.NewTransaction(ecKey.GetAddress().Hex()).Deploy(common.Bytes2Hex(payload)).VMType(rpc.HVM)




4.4 KVSQL
---------------

4.4.1 回执解码
>>>>>>>>>>>>>>>>>>>>

 ::

  type ResultSet struct {
    columnCount  uint32
    lastInsertID uint64
    updatedCount uint32

    rowNumber  int // the number of rows
    resChunk   *resultChunk
    columnInfo []*Field
  }


**4.4.1.1 解回执为ResultSet**

 ::

  func DecodeRecordSet(buf []byte) *ResultSet


- 参数1  : 交易回执的ret字段解码成byte数组
- 返回值1 : sql执行结果的结构体实例

应用实例

 ::

  b, _ := hexutil.Decode(res.Ret)
  rs := kvsql.DecodeRecordSet(b)


**4.4.1.2 获取列数**

 ::

  `func (c ResultSet) ColumnCount() uint32`

 ::

  answer := "0x0035000000053654333354053654333354066175746f4964066175746f49643f000b000000030502000536543333540536543333540574797065300574797065303f000b000000030300000536543333540536543333540574797065310574797065313f000a000000032000000536543333540536543333540574797065320574797065323f0014000000080000000536543333540536543333540574797065330574797065333f0014000000082800000536543333540536543333540574797065340574797065343f000c0000000408001f0536543333540536543333540574797065350574797065353f00ff0000000408001e0536543333540536543333540574797065360574797065363f00ff0000000408001e0536543333540536543333540574797065370574797065373f00160000000508001f0536543333540536543333540574797065380574797065383f00ff0000000500001e0536543333540536543333540574797065390574797065393f000b000000f600000005365433335405365433335406747970653130067479706531303f0020000000f600001e05365433335405365433335406747970653131067479706531313f000b000000f600000005365433335405365433335406747970653132067479706531323f0016000000f600001405365433335405365433335406747970653133067479706531333f00090000000900000005365433335405365433335406747970653134067479706531343f00080000000920000005365433335405365433335406747970653135067479706531353f00060000000200000005365433335405365433335406747970653136067479706531363f00040000000100000005365433335405365433335406747970653137067479706531373f00030000000120000005365433335405365433335406747970653138067479706531383f00050000000220000005365433335405365433335406747970653139067479706531393f000a0000000a80000005365433335405365433335406747970653230067479706532303f00130000000c80000005365433335405365433335406747970653231067479706532313f000a0000000b80000005365433335405365433335406747970653232067479706532323f00130000000780000005365433335405365433335406747970653233067479706532333f00040000000d60000005365433335405365433335406747970653234067479706532342e0000000000fe00000005365433335405365433335406747970653235067479706532352e0004000000fe00000005365433335405365433335406747970653236067479706532362e00fc030000fe00000005365433335405365433335406747970653237067479706532372e0000000000fd00000005365433335405365433335406747970653238067479706532382e0004000000fd00000005365433335405365433335406747970653239067479706532392e00fc030000fd00000005365433335405365433335406747970653330067479706533303f0000000000fe80000005365433335405365433335406747970653331067479706533313f0001000000fe80000005365433335405365433335406747970653332067479706533323f00ff000000fe80000005365433335405365433335406747970653333067479706533333f00ffff0000fc80000005365433335405365433335406747970653334067479706533343f00ff000000f980000005365433335405365433335406747970653335067479706533353f00ffffff00fa80000005365433335405365433335406747970653336067479706533362e0058020000fc00000005365433335405365433335406747970653337067479706533372e00fcffff03fa00000005365433335405365433335406747970653338067479706533383f0001000000fd80000005365433335405365433335406747970653339067479706533392e0000000000fe00000005365433335405365433335406747970653430067479706534302e0004000000fe00000005365433335405365433335406747970653431067479706534312e0010000000fe00000005365433335405365433335406747970653432067479706534322e0000000000fe00000005365433335405365433335406747970653433067479706534332e0004000000fe00000005365433335405365433335406747970653434067479706534342e00d4030000fe00000005365433335405365433335406747970653435067479706534353f00010000001020000005365433335405365433335406747970653436067479706534363f00400000001020000005365433335405365433335406747970653437067479706534373f00010000000100000005365433335405365433335406747970653438067479706534382e00fcffff03fa00000005365433335405365433335406747970653439067479706534392e00fcffff03fa00000005365433335405365433335406747970653530067479706535302e0004000000fe00000005365433335405365433335406747970653531067479706535312e0004000000fd00000001000000040907000001010100000000049e732b7b010101000000000415d0870701010100000000083aa1ffffffffffff010101000000000871575519bfcd04020101010000000004ba0f9f3e01010100000000044d6d6e3f0101010000000004d66cc3be0101010000000008a4a6c8fae881e83f0101010000000008aba76123a72be63f01010100000000052d313731340101020000000005000000212d302e3233363530303030303030303030303030303030303030303030303030300101020000000021000000062d3531343132010102000000000600000016302e383433303030303030303030303030303030303001010200000000160000000453d955000101010000000004c3d1180001010100000000024b2d0101010000000001660101010000000001170101010000000002282d01010100000000080e0000000054c91d0101010000000008000000287342f21d0101010000000008001ef1cdf239000001010100000000080100007524a8b01f0101010000000002770701010100000000000101020000000000000000013001010200000000010000006436383036323439363639373332383534303836393332373339363837343432383138343539323234343837313936333534323934343333343031373936333438363830353736303134373133383232343632323236353730393639323939303039353035010102000000006400000000010102000000000000000001640101020000000001000000fcff007a4f3979704e654c456e63684f69784944304b527637704134794e6638346e6649317076333252367a737a6246486a34527479447645574e644772557738715161576573774552307464454f50746b4d5a506b50673669564c585551445464486d71674e6736487351665639637134636772735436686e557063736463416e3041584f684f596c65636f34426a44464c7a62424e5276516c6c644632684b413355636c4a7350757738726c616363517872763663655851556c74427a414861773264456b4c4252597a53444e4d5a3033735534456b52666e6a48454e564e6730416f6c5a66355356753059486a426768563148355930654b4b7661735a473101010200000000ff000000000101020000000000000000012a0101020000000001000000fcff00ccd82dbce2a48abe65f391308024cff5bb3971c2aa2761f10c52476cc9e3235206b47528a36eb3767dc50852ed3ebfa5b17a0b47ed2a5a2b56ead26c5b96c805ca65e22eaabbc7af1560e71e57ca734ca952ed17f54285d4feecd11c0d5ba0e679c464cf1196373f427c41a8e9f5c157f78834f2e57c06aa5d5af0682504349f796518eca8f4f33dbf53a9dfa30e1f4232106825d8695817ae992d70fc0a30c8b4f9d232701d948d6a9199acd8e5ce94b5aa50af5929555b91f124167fde826315ba922120d8d4f6c5d1fcbdb08708521f9f91a7e84366f08b90bccbb18186660fe49d5ca270dbe476b3f4c7a7418b51d37cf38c7c55e5c784257dde277a4f01010200000000ff00000064279e9d01d72ec83ad3ae46ce057466371ba263f62a726018347135ec8619bc835e5f215431902b181ff2e41c6a28115a1e01b50ed33faad7601416032c3d2911b00b25e051f1f3676b082aa98d618f251c5b984976c4a89f7ed84ad8ef0ff22cdf05f1c50101020000000064000000189bc7836679d2fb8bf55d01daa0c7b7791a01b2d9359ec8ea010102000000001800000064892df41245164975050a48fdc2a262b012f29e95763cb507bd5df48b339d1539a988202f99d31cfe072df1696a58529b7813d848a4edcf9133bdbe67b4449b0718b07839471e35a6a798f9f17d6db70aee9b989e88d19dc9e363fc5906c4abb7d1bae32a0101020000000064000000143635363636373638363937303731373237333734010102000000001400000014363836393730373137323733373437353736373701010200000000140000000116010102000000000100000000010002000000000000000001610101020000000001000000046132373501010200000000040000000001000200000000000000000001000200000000000000000c61302c61312c61322c613630010102000000000c00000001000101020000000001000000087fffffffffffffff01010200000000080000000150010101000000001437323733373437353736373737383739383038310101020000000014000000143635363636373638363937303731373237333734010102000000001400000001390101020000000001000000016b0101020000000001000000"
  b, _ := hexutil.Decode(answer)
  rs := kvsql.DecodeRecordSet(b)
  rs.ColumnCount()


**4.4.1.3 获取lastInsertId**

 ::

  `func (c ResultSet) LastInsertID() uint64`

 ::

  answer := "0x0035000000053654333354053654333354066175746f4964066175746f49643f000b000000030502000536543333540536543333540574797065300574797065303f000b000000030300000536543333540536543333540574797065310574797065313f000a000000032000000536543333540536543333540574797065320574797065323f0014000000080000000536543333540536543333540574797065330574797065333f0014000000082800000536543333540536543333540574797065340574797065343f000c0000000408001f0536543333540536543333540574797065350574797065353f00ff0000000408001e0536543333540536543333540574797065360574797065363f00ff0000000408001e0536543333540536543333540574797065370574797065373f00160000000508001f0536543333540536543333540574797065380574797065383f00ff0000000500001e0536543333540536543333540574797065390574797065393f000b000000f600000005365433335405365433335406747970653130067479706531303f0020000000f600001e05365433335405365433335406747970653131067479706531313f000b000000f600000005365433335405365433335406747970653132067479706531323f0016000000f600001405365433335405365433335406747970653133067479706531333f00090000000900000005365433335405365433335406747970653134067479706531343f00080000000920000005365433335405365433335406747970653135067479706531353f00060000000200000005365433335405365433335406747970653136067479706531363f00040000000100000005365433335405365433335406747970653137067479706531373f00030000000120000005365433335405365433335406747970653138067479706531383f00050000000220000005365433335405365433335406747970653139067479706531393f000a0000000a80000005365433335405365433335406747970653230067479706532303f00130000000c80000005365433335405365433335406747970653231067479706532313f000a0000000b80000005365433335405365433335406747970653232067479706532323f00130000000780000005365433335405365433335406747970653233067479706532333f00040000000d60000005365433335405365433335406747970653234067479706532342e0000000000fe00000005365433335405365433335406747970653235067479706532352e0004000000fe00000005365433335405365433335406747970653236067479706532362e00fc030000fe00000005365433335405365433335406747970653237067479706532372e0000000000fd00000005365433335405365433335406747970653238067479706532382e0004000000fd00000005365433335405365433335406747970653239067479706532392e00fc030000fd00000005365433335405365433335406747970653330067479706533303f0000000000fe80000005365433335405365433335406747970653331067479706533313f0001000000fe80000005365433335405365433335406747970653332067479706533323f00ff000000fe80000005365433335405365433335406747970653333067479706533333f00ffff0000fc80000005365433335405365433335406747970653334067479706533343f00ff000000f980000005365433335405365433335406747970653335067479706533353f00ffffff00fa80000005365433335405365433335406747970653336067479706533362e0058020000fc00000005365433335405365433335406747970653337067479706533372e00fcffff03fa00000005365433335405365433335406747970653338067479706533383f0001000000fd80000005365433335405365433335406747970653339067479706533392e0000000000fe00000005365433335405365433335406747970653430067479706534302e0004000000fe00000005365433335405365433335406747970653431067479706534312e0010000000fe00000005365433335405365433335406747970653432067479706534322e0000000000fe00000005365433335405365433335406747970653433067479706534332e0004000000fe00000005365433335405365433335406747970653434067479706534342e00d4030000fe00000005365433335405365433335406747970653435067479706534353f00010000001020000005365433335405365433335406747970653436067479706534363f00400000001020000005365433335405365433335406747970653437067479706534373f00010000000100000005365433335405365433335406747970653438067479706534382e00fcffff03fa00000005365433335405365433335406747970653439067479706534392e00fcffff03fa00000005365433335405365433335406747970653530067479706535302e0004000000fe00000005365433335405365433335406747970653531067479706535312e0004000000fd00000001000000040907000001010100000000049e732b7b010101000000000415d0870701010100000000083aa1ffffffffffff010101000000000871575519bfcd04020101010000000004ba0f9f3e01010100000000044d6d6e3f0101010000000004d66cc3be0101010000000008a4a6c8fae881e83f0101010000000008aba76123a72be63f01010100000000052d313731340101020000000005000000212d302e3233363530303030303030303030303030303030303030303030303030300101020000000021000000062d3531343132010102000000000600000016302e383433303030303030303030303030303030303001010200000000160000000453d955000101010000000004c3d1180001010100000000024b2d0101010000000001660101010000000001170101010000000002282d01010100000000080e0000000054c91d0101010000000008000000287342f21d0101010000000008001ef1cdf239000001010100000000080100007524a8b01f0101010000000002770701010100000000000101020000000000000000013001010200000000010000006436383036323439363639373332383534303836393332373339363837343432383138343539323234343837313936333534323934343333343031373936333438363830353736303134373133383232343632323236353730393639323939303039353035010102000000006400000000010102000000000000000001640101020000000001000000fcff007a4f3979704e654c456e63684f69784944304b527637704134794e6638346e6649317076333252367a737a6246486a34527479447645574e644772557738715161576573774552307464454f50746b4d5a506b50673669564c585551445464486d71674e6736487351665639637134636772735436686e557063736463416e3041584f684f596c65636f34426a44464c7a62424e5276516c6c644632684b413355636c4a7350757738726c616363517872763663655851556c74427a414861773264456b4c4252597a53444e4d5a3033735534456b52666e6a48454e564e6730416f6c5a66355356753059486a426768563148355930654b4b7661735a473101010200000000ff000000000101020000000000000000012a0101020000000001000000fcff00ccd82dbce2a48abe65f391308024cff5bb3971c2aa2761f10c52476cc9e3235206b47528a36eb3767dc50852ed3ebfa5b17a0b47ed2a5a2b56ead26c5b96c805ca65e22eaabbc7af1560e71e57ca734ca952ed17f54285d4feecd11c0d5ba0e679c464cf1196373f427c41a8e9f5c157f78834f2e57c06aa5d5af0682504349f796518eca8f4f33dbf53a9dfa30e1f4232106825d8695817ae992d70fc0a30c8b4f9d232701d948d6a9199acd8e5ce94b5aa50af5929555b91f124167fde826315ba922120d8d4f6c5d1fcbdb08708521f9f91a7e84366f08b90bccbb18186660fe49d5ca270dbe476b3f4c7a7418b51d37cf38c7c55e5c784257dde277a4f01010200000000ff00000064279e9d01d72ec83ad3ae46ce057466371ba263f62a726018347135ec8619bc835e5f215431902b181ff2e41c6a28115a1e01b50ed33faad7601416032c3d2911b00b25e051f1f3676b082aa98d618f251c5b984976c4a89f7ed84ad8ef0ff22cdf05f1c50101020000000064000000189bc7836679d2fb8bf55d01daa0c7b7791a01b2d9359ec8ea010102000000001800000064892df41245164975050a48fdc2a262b012f29e95763cb507bd5df48b339d1539a988202f99d31cfe072df1696a58529b7813d848a4edcf9133bdbe67b4449b0718b07839471e35a6a798f9f17d6db70aee9b989e88d19dc9e363fc5906c4abb7d1bae32a0101020000000064000000143635363636373638363937303731373237333734010102000000001400000014363836393730373137323733373437353736373701010200000000140000000116010102000000000100000000010002000000000000000001610101020000000001000000046132373501010200000000040000000001000200000000000000000001000200000000000000000c61302c61312c61322c613630010102000000000c00000001000101020000000001000000087fffffffffffffff01010200000000080000000150010101000000001437323733373437353736373737383739383038310101020000000014000000143635363636373638363937303731373237333734010102000000001400000001390101020000000001000000016b0101020000000001000000"
  b, _ := hexutil.Decode(answer)
  rs := kvsql.DecodeRecordSet(b)
  rs.LastInsertID()


**4.4.1.4 获取修改的行数**

 ::

  `func (c ResultSet) UpdatedCount() uint32`

 ::

  answer := "0x0035000000053654333354053654333354066175746f4964066175746f49643f000b000000030502000536543333540536543333540574797065300574797065303f000b000000030300000536543333540536543333540574797065310574797065313f000a000000032000000536543333540536543333540574797065320574797065323f0014000000080000000536543333540536543333540574797065330574797065333f0014000000082800000536543333540536543333540574797065340574797065343f000c0000000408001f0536543333540536543333540574797065350574797065353f00ff0000000408001e0536543333540536543333540574797065360574797065363f00ff0000000408001e0536543333540536543333540574797065370574797065373f00160000000508001f0536543333540536543333540574797065380574797065383f00ff0000000500001e0536543333540536543333540574797065390574797065393f000b000000f600000005365433335405365433335406747970653130067479706531303f0020000000f600001e05365433335405365433335406747970653131067479706531313f000b000000f600000005365433335405365433335406747970653132067479706531323f0016000000f600001405365433335405365433335406747970653133067479706531333f00090000000900000005365433335405365433335406747970653134067479706531343f00080000000920000005365433335405365433335406747970653135067479706531353f00060000000200000005365433335405365433335406747970653136067479706531363f00040000000100000005365433335405365433335406747970653137067479706531373f00030000000120000005365433335405365433335406747970653138067479706531383f00050000000220000005365433335405365433335406747970653139067479706531393f000a0000000a80000005365433335405365433335406747970653230067479706532303f00130000000c80000005365433335405365433335406747970653231067479706532313f000a0000000b80000005365433335405365433335406747970653232067479706532323f00130000000780000005365433335405365433335406747970653233067479706532333f00040000000d60000005365433335405365433335406747970653234067479706532342e0000000000fe00000005365433335405365433335406747970653235067479706532352e0004000000fe00000005365433335405365433335406747970653236067479706532362e00fc030000fe00000005365433335405365433335406747970653237067479706532372e0000000000fd00000005365433335405365433335406747970653238067479706532382e0004000000fd00000005365433335405365433335406747970653239067479706532392e00fc030000fd00000005365433335405365433335406747970653330067479706533303f0000000000fe80000005365433335405365433335406747970653331067479706533313f0001000000fe80000005365433335405365433335406747970653332067479706533323f00ff000000fe80000005365433335405365433335406747970653333067479706533333f00ffff0000fc80000005365433335405365433335406747970653334067479706533343f00ff000000f980000005365433335405365433335406747970653335067479706533353f00ffffff00fa80000005365433335405365433335406747970653336067479706533362e0058020000fc00000005365433335405365433335406747970653337067479706533372e00fcffff03fa00000005365433335405365433335406747970653338067479706533383f0001000000fd80000005365433335405365433335406747970653339067479706533392e0000000000fe00000005365433335405365433335406747970653430067479706534302e0004000000fe00000005365433335405365433335406747970653431067479706534312e0010000000fe00000005365433335405365433335406747970653432067479706534322e0000000000fe00000005365433335405365433335406747970653433067479706534332e0004000000fe00000005365433335405365433335406747970653434067479706534342e00d4030000fe00000005365433335405365433335406747970653435067479706534353f00010000001020000005365433335405365433335406747970653436067479706534363f00400000001020000005365433335405365433335406747970653437067479706534373f00010000000100000005365433335405365433335406747970653438067479706534382e00fcffff03fa00000005365433335405365433335406747970653439067479706534392e00fcffff03fa00000005365433335405365433335406747970653530067479706535302e0004000000fe00000005365433335405365433335406747970653531067479706535312e0004000000fd00000001000000040907000001010100000000049e732b7b010101000000000415d0870701010100000000083aa1ffffffffffff010101000000000871575519bfcd04020101010000000004ba0f9f3e01010100000000044d6d6e3f0101010000000004d66cc3be0101010000000008a4a6c8fae881e83f0101010000000008aba76123a72be63f01010100000000052d313731340101020000000005000000212d302e3233363530303030303030303030303030303030303030303030303030300101020000000021000000062d3531343132010102000000000600000016302e383433303030303030303030303030303030303001010200000000160000000453d955000101010000000004c3d1180001010100000000024b2d0101010000000001660101010000000001170101010000000002282d01010100000000080e0000000054c91d0101010000000008000000287342f21d0101010000000008001ef1cdf239000001010100000000080100007524a8b01f0101010000000002770701010100000000000101020000000000000000013001010200000000010000006436383036323439363639373332383534303836393332373339363837343432383138343539323234343837313936333534323934343333343031373936333438363830353736303134373133383232343632323236353730393639323939303039353035010102000000006400000000010102000000000000000001640101020000000001000000fcff007a4f3979704e654c456e63684f69784944304b527637704134794e6638346e6649317076333252367a737a6246486a34527479447645574e644772557738715161576573774552307464454f50746b4d5a506b50673669564c585551445464486d71674e6736487351665639637134636772735436686e557063736463416e3041584f684f596c65636f34426a44464c7a62424e5276516c6c644632684b413355636c4a7350757738726c616363517872763663655851556c74427a414861773264456b4c4252597a53444e4d5a3033735534456b52666e6a48454e564e6730416f6c5a66355356753059486a426768563148355930654b4b7661735a473101010200000000ff000000000101020000000000000000012a0101020000000001000000fcff00ccd82dbce2a48abe65f391308024cff5bb3971c2aa2761f10c52476cc9e3235206b47528a36eb3767dc50852ed3ebfa5b17a0b47ed2a5a2b56ead26c5b96c805ca65e22eaabbc7af1560e71e57ca734ca952ed17f54285d4feecd11c0d5ba0e679c464cf1196373f427c41a8e9f5c157f78834f2e57c06aa5d5af0682504349f796518eca8f4f33dbf53a9dfa30e1f4232106825d8695817ae992d70fc0a30c8b4f9d232701d948d6a9199acd8e5ce94b5aa50af5929555b91f124167fde826315ba922120d8d4f6c5d1fcbdb08708521f9f91a7e84366f08b90bccbb18186660fe49d5ca270dbe476b3f4c7a7418b51d37cf38c7c55e5c784257dde277a4f01010200000000ff00000064279e9d01d72ec83ad3ae46ce057466371ba263f62a726018347135ec8619bc835e5f215431902b181ff2e41c6a28115a1e01b50ed33faad7601416032c3d2911b00b25e051f1f3676b082aa98d618f251c5b984976c4a89f7ed84ad8ef0ff22cdf05f1c50101020000000064000000189bc7836679d2fb8bf55d01daa0c7b7791a01b2d9359ec8ea010102000000001800000064892df41245164975050a48fdc2a262b012f29e95763cb507bd5df48b339d1539a988202f99d31cfe072df1696a58529b7813d848a4edcf9133bdbe67b4449b0718b07839471e35a6a798f9f17d6db70aee9b989e88d19dc9e363fc5906c4abb7d1bae32a0101020000000064000000143635363636373638363937303731373237333734010102000000001400000014363836393730373137323733373437353736373701010200000000140000000116010102000000000100000000010002000000000000000001610101020000000001000000046132373501010200000000040000000001000200000000000000000001000200000000000000000c61302c61312c61322c613630010102000000000c00000001000101020000000001000000087fffffffffffffff01010200000000080000000150010101000000001437323733373437353736373737383739383038310101020000000014000000143635363636373638363937303731373237333734010102000000001400000001390101020000000001000000016b0101020000000001000000"
  b, _ := hexutil.Decode(answer)
  rs := kvsql.DecodeRecordSet(b)
  rs.UpdatedCount()


**4.4.1.5 获取列**

 ::

  `func (c ResultSet) Columns() []string`

 ::

  answer := "0x0035000000053654333354053654333354066175746f4964066175746f49643f000b000000030502000536543333540536543333540574797065300574797065303f000b000000030300000536543333540536543333540574797065310574797065313f000a000000032000000536543333540536543333540574797065320574797065323f0014000000080000000536543333540536543333540574797065330574797065333f0014000000082800000536543333540536543333540574797065340574797065343f000c0000000408001f0536543333540536543333540574797065350574797065353f00ff0000000408001e0536543333540536543333540574797065360574797065363f00ff0000000408001e0536543333540536543333540574797065370574797065373f00160000000508001f0536543333540536543333540574797065380574797065383f00ff0000000500001e0536543333540536543333540574797065390574797065393f000b000000f600000005365433335405365433335406747970653130067479706531303f0020000000f600001e05365433335405365433335406747970653131067479706531313f000b000000f600000005365433335405365433335406747970653132067479706531323f0016000000f600001405365433335405365433335406747970653133067479706531333f00090000000900000005365433335405365433335406747970653134067479706531343f00080000000920000005365433335405365433335406747970653135067479706531353f00060000000200000005365433335405365433335406747970653136067479706531363f00040000000100000005365433335405365433335406747970653137067479706531373f00030000000120000005365433335405365433335406747970653138067479706531383f00050000000220000005365433335405365433335406747970653139067479706531393f000a0000000a80000005365433335405365433335406747970653230067479706532303f00130000000c80000005365433335405365433335406747970653231067479706532313f000a0000000b80000005365433335405365433335406747970653232067479706532323f00130000000780000005365433335405365433335406747970653233067479706532333f00040000000d60000005365433335405365433335406747970653234067479706532342e0000000000fe00000005365433335405365433335406747970653235067479706532352e0004000000fe00000005365433335405365433335406747970653236067479706532362e00fc030000fe00000005365433335405365433335406747970653237067479706532372e0000000000fd00000005365433335405365433335406747970653238067479706532382e0004000000fd00000005365433335405365433335406747970653239067479706532392e00fc030000fd00000005365433335405365433335406747970653330067479706533303f0000000000fe80000005365433335405365433335406747970653331067479706533313f0001000000fe80000005365433335405365433335406747970653332067479706533323f00ff000000fe80000005365433335405365433335406747970653333067479706533333f00ffff0000fc80000005365433335405365433335406747970653334067479706533343f00ff000000f980000005365433335405365433335406747970653335067479706533353f00ffffff00fa80000005365433335405365433335406747970653336067479706533362e0058020000fc00000005365433335405365433335406747970653337067479706533372e00fcffff03fa00000005365433335405365433335406747970653338067479706533383f0001000000fd80000005365433335405365433335406747970653339067479706533392e0000000000fe00000005365433335405365433335406747970653430067479706534302e0004000000fe00000005365433335405365433335406747970653431067479706534312e0010000000fe00000005365433335405365433335406747970653432067479706534322e0000000000fe00000005365433335405365433335406747970653433067479706534332e0004000000fe00000005365433335405365433335406747970653434067479706534342e00d4030000fe00000005365433335405365433335406747970653435067479706534353f00010000001020000005365433335405365433335406747970653436067479706534363f00400000001020000005365433335405365433335406747970653437067479706534373f00010000000100000005365433335405365433335406747970653438067479706534382e00fcffff03fa00000005365433335405365433335406747970653439067479706534392e00fcffff03fa00000005365433335405365433335406747970653530067479706535302e0004000000fe00000005365433335405365433335406747970653531067479706535312e0004000000fd00000001000000040907000001010100000000049e732b7b010101000000000415d0870701010100000000083aa1ffffffffffff010101000000000871575519bfcd04020101010000000004ba0f9f3e01010100000000044d6d6e3f0101010000000004d66cc3be0101010000000008a4a6c8fae881e83f0101010000000008aba76123a72be63f01010100000000052d313731340101020000000005000000212d302e3233363530303030303030303030303030303030303030303030303030300101020000000021000000062d3531343132010102000000000600000016302e383433303030303030303030303030303030303001010200000000160000000453d955000101010000000004c3d1180001010100000000024b2d0101010000000001660101010000000001170101010000000002282d01010100000000080e0000000054c91d0101010000000008000000287342f21d0101010000000008001ef1cdf239000001010100000000080100007524a8b01f0101010000000002770701010100000000000101020000000000000000013001010200000000010000006436383036323439363639373332383534303836393332373339363837343432383138343539323234343837313936333534323934343333343031373936333438363830353736303134373133383232343632323236353730393639323939303039353035010102000000006400000000010102000000000000000001640101020000000001000000fcff007a4f3979704e654c456e63684f69784944304b527637704134794e6638346e6649317076333252367a737a6246486a34527479447645574e644772557738715161576573774552307464454f50746b4d5a506b50673669564c585551445464486d71674e6736487351665639637134636772735436686e557063736463416e3041584f684f596c65636f34426a44464c7a62424e5276516c6c644632684b413355636c4a7350757738726c616363517872763663655851556c74427a414861773264456b4c4252597a53444e4d5a3033735534456b52666e6a48454e564e6730416f6c5a66355356753059486a426768563148355930654b4b7661735a473101010200000000ff000000000101020000000000000000012a0101020000000001000000fcff00ccd82dbce2a48abe65f391308024cff5bb3971c2aa2761f10c52476cc9e3235206b47528a36eb3767dc50852ed3ebfa5b17a0b47ed2a5a2b56ead26c5b96c805ca65e22eaabbc7af1560e71e57ca734ca952ed17f54285d4feecd11c0d5ba0e679c464cf1196373f427c41a8e9f5c157f78834f2e57c06aa5d5af0682504349f796518eca8f4f33dbf53a9dfa30e1f4232106825d8695817ae992d70fc0a30c8b4f9d232701d948d6a9199acd8e5ce94b5aa50af5929555b91f124167fde826315ba922120d8d4f6c5d1fcbdb08708521f9f91a7e84366f08b90bccbb18186660fe49d5ca270dbe476b3f4c7a7418b51d37cf38c7c55e5c784257dde277a4f01010200000000ff00000064279e9d01d72ec83ad3ae46ce057466371ba263f62a726018347135ec8619bc835e5f215431902b181ff2e41c6a28115a1e01b50ed33faad7601416032c3d2911b00b25e051f1f3676b082aa98d618f251c5b984976c4a89f7ed84ad8ef0ff22cdf05f1c50101020000000064000000189bc7836679d2fb8bf55d01daa0c7b7791a01b2d9359ec8ea010102000000001800000064892df41245164975050a48fdc2a262b012f29e95763cb507bd5df48b339d1539a988202f99d31cfe072df1696a58529b7813d848a4edcf9133bdbe67b4449b0718b07839471e35a6a798f9f17d6db70aee9b989e88d19dc9e363fc5906c4abb7d1bae32a0101020000000064000000143635363636373638363937303731373237333734010102000000001400000014363836393730373137323733373437353736373701010200000000140000000116010102000000000100000000010002000000000000000001610101020000000001000000046132373501010200000000040000000001000200000000000000000001000200000000000000000c61302c61312c61322c613630010102000000000c00000001000101020000000001000000087fffffffffffffff01010200000000080000000150010101000000001437323733373437353736373737383739383038310101020000000014000000143635363636373638363937303731373237333734010102000000001400000001390101020000000001000000016b0101020000000001000000"
  b, _ := hexutil.Decode(answer)
  rs := kvsql.DecodeRecordSet(b)
  rs.Columns()


**4.4.1.6 获取行数**

 ::

  `func (c ResultSet) RowNumber() int`

 ::

  answer := "0x0035000000053654333354053654333354066175746f4964066175746f49643f000b000000030502000536543333540536543333540574797065300574797065303f000b000000030300000536543333540536543333540574797065310574797065313f000a000000032000000536543333540536543333540574797065320574797065323f0014000000080000000536543333540536543333540574797065330574797065333f0014000000082800000536543333540536543333540574797065340574797065343f000c0000000408001f0536543333540536543333540574797065350574797065353f00ff0000000408001e0536543333540536543333540574797065360574797065363f00ff0000000408001e0536543333540536543333540574797065370574797065373f00160000000508001f0536543333540536543333540574797065380574797065383f00ff0000000500001e0536543333540536543333540574797065390574797065393f000b000000f600000005365433335405365433335406747970653130067479706531303f0020000000f600001e05365433335405365433335406747970653131067479706531313f000b000000f600000005365433335405365433335406747970653132067479706531323f0016000000f600001405365433335405365433335406747970653133067479706531333f00090000000900000005365433335405365433335406747970653134067479706531343f00080000000920000005365433335405365433335406747970653135067479706531353f00060000000200000005365433335405365433335406747970653136067479706531363f00040000000100000005365433335405365433335406747970653137067479706531373f00030000000120000005365433335405365433335406747970653138067479706531383f00050000000220000005365433335405365433335406747970653139067479706531393f000a0000000a80000005365433335405365433335406747970653230067479706532303f00130000000c80000005365433335405365433335406747970653231067479706532313f000a0000000b80000005365433335405365433335406747970653232067479706532323f00130000000780000005365433335405365433335406747970653233067479706532333f00040000000d60000005365433335405365433335406747970653234067479706532342e0000000000fe00000005365433335405365433335406747970653235067479706532352e0004000000fe00000005365433335405365433335406747970653236067479706532362e00fc030000fe00000005365433335405365433335406747970653237067479706532372e0000000000fd00000005365433335405365433335406747970653238067479706532382e0004000000fd00000005365433335405365433335406747970653239067479706532392e00fc030000fd00000005365433335405365433335406747970653330067479706533303f0000000000fe80000005365433335405365433335406747970653331067479706533313f0001000000fe80000005365433335405365433335406747970653332067479706533323f00ff000000fe80000005365433335405365433335406747970653333067479706533333f00ffff0000fc80000005365433335405365433335406747970653334067479706533343f00ff000000f980000005365433335405365433335406747970653335067479706533353f00ffffff00fa80000005365433335405365433335406747970653336067479706533362e0058020000fc00000005365433335405365433335406747970653337067479706533372e00fcffff03fa00000005365433335405365433335406747970653338067479706533383f0001000000fd80000005365433335405365433335406747970653339067479706533392e0000000000fe00000005365433335405365433335406747970653430067479706534302e0004000000fe00000005365433335405365433335406747970653431067479706534312e0010000000fe00000005365433335405365433335406747970653432067479706534322e0000000000fe00000005365433335405365433335406747970653433067479706534332e0004000000fe00000005365433335405365433335406747970653434067479706534342e00d4030000fe00000005365433335405365433335406747970653435067479706534353f00010000001020000005365433335405365433335406747970653436067479706534363f00400000001020000005365433335405365433335406747970653437067479706534373f00010000000100000005365433335405365433335406747970653438067479706534382e00fcffff03fa00000005365433335405365433335406747970653439067479706534392e00fcffff03fa00000005365433335405365433335406747970653530067479706535302e0004000000fe00000005365433335405365433335406747970653531067479706535312e0004000000fd00000001000000040907000001010100000000049e732b7b010101000000000415d0870701010100000000083aa1ffffffffffff010101000000000871575519bfcd04020101010000000004ba0f9f3e01010100000000044d6d6e3f0101010000000004d66cc3be0101010000000008a4a6c8fae881e83f0101010000000008aba76123a72be63f01010100000000052d313731340101020000000005000000212d302e3233363530303030303030303030303030303030303030303030303030300101020000000021000000062d3531343132010102000000000600000016302e383433303030303030303030303030303030303001010200000000160000000453d955000101010000000004c3d1180001010100000000024b2d0101010000000001660101010000000001170101010000000002282d01010100000000080e0000000054c91d0101010000000008000000287342f21d0101010000000008001ef1cdf239000001010100000000080100007524a8b01f0101010000000002770701010100000000000101020000000000000000013001010200000000010000006436383036323439363639373332383534303836393332373339363837343432383138343539323234343837313936333534323934343333343031373936333438363830353736303134373133383232343632323236353730393639323939303039353035010102000000006400000000010102000000000000000001640101020000000001000000fcff007a4f3979704e654c456e63684f69784944304b527637704134794e6638346e6649317076333252367a737a6246486a34527479447645574e644772557738715161576573774552307464454f50746b4d5a506b50673669564c585551445464486d71674e6736487351665639637134636772735436686e557063736463416e3041584f684f596c65636f34426a44464c7a62424e5276516c6c644632684b413355636c4a7350757738726c616363517872763663655851556c74427a414861773264456b4c4252597a53444e4d5a3033735534456b52666e6a48454e564e6730416f6c5a66355356753059486a426768563148355930654b4b7661735a473101010200000000ff000000000101020000000000000000012a0101020000000001000000fcff00ccd82dbce2a48abe65f391308024cff5bb3971c2aa2761f10c52476cc9e3235206b47528a36eb3767dc50852ed3ebfa5b17a0b47ed2a5a2b56ead26c5b96c805ca65e22eaabbc7af1560e71e57ca734ca952ed17f54285d4feecd11c0d5ba0e679c464cf1196373f427c41a8e9f5c157f78834f2e57c06aa5d5af0682504349f796518eca8f4f33dbf53a9dfa30e1f4232106825d8695817ae992d70fc0a30c8b4f9d232701d948d6a9199acd8e5ce94b5aa50af5929555b91f124167fde826315ba922120d8d4f6c5d1fcbdb08708521f9f91a7e84366f08b90bccbb18186660fe49d5ca270dbe476b3f4c7a7418b51d37cf38c7c55e5c784257dde277a4f01010200000000ff00000064279e9d01d72ec83ad3ae46ce057466371ba263f62a726018347135ec8619bc835e5f215431902b181ff2e41c6a28115a1e01b50ed33faad7601416032c3d2911b00b25e051f1f3676b082aa98d618f251c5b984976c4a89f7ed84ad8ef0ff22cdf05f1c50101020000000064000000189bc7836679d2fb8bf55d01daa0c7b7791a01b2d9359ec8ea010102000000001800000064892df41245164975050a48fdc2a262b012f29e95763cb507bd5df48b339d1539a988202f99d31cfe072df1696a58529b7813d848a4edcf9133bdbe67b4449b0718b07839471e35a6a798f9f17d6db70aee9b989e88d19dc9e363fc5906c4abb7d1bae32a0101020000000064000000143635363636373638363937303731373237333734010102000000001400000014363836393730373137323733373437353736373701010200000000140000000116010102000000000100000000010002000000000000000001610101020000000001000000046132373501010200000000040000000001000200000000000000000001000200000000000000000c61302c61312c61322c613630010102000000000c00000001000101020000000001000000087fffffffffffffff01010200000000080000000150010101000000001437323733373437353736373737383739383038310101020000000014000000143635363636373638363937303731373237333734010102000000001400000001390101020000000001000000016b0101020000000001000000"
  b, _ := hexutil.Decode(answer)
  rs := kvsql.DecodeRecordSet(b)
  rs.RowNumber()




**4.4.1.7 获取指定行数据**

 ::

  `func (rs *ResultSet) GetRow(rowIndex int) Row`



 ::

  answer := "0x0035000000053654333354053654333354066175746f4964066175746f49643f000b000000030502000536543333540536543333540574797065300574797065303f000b000000030300000536543333540536543333540574797065310574797065313f000a000000032000000536543333540536543333540574797065320574797065323f0014000000080000000536543333540536543333540574797065330574797065333f0014000000082800000536543333540536543333540574797065340574797065343f000c0000000408001f0536543333540536543333540574797065350574797065353f00ff0000000408001e0536543333540536543333540574797065360574797065363f00ff0000000408001e0536543333540536543333540574797065370574797065373f00160000000508001f0536543333540536543333540574797065380574797065383f00ff0000000500001e0536543333540536543333540574797065390574797065393f000b000000f600000005365433335405365433335406747970653130067479706531303f0020000000f600001e05365433335405365433335406747970653131067479706531313f000b000000f600000005365433335405365433335406747970653132067479706531323f0016000000f600001405365433335405365433335406747970653133067479706531333f00090000000900000005365433335405365433335406747970653134067479706531343f00080000000920000005365433335405365433335406747970653135067479706531353f00060000000200000005365433335405365433335406747970653136067479706531363f00040000000100000005365433335405365433335406747970653137067479706531373f00030000000120000005365433335405365433335406747970653138067479706531383f00050000000220000005365433335405365433335406747970653139067479706531393f000a0000000a80000005365433335405365433335406747970653230067479706532303f00130000000c80000005365433335405365433335406747970653231067479706532313f000a0000000b80000005365433335405365433335406747970653232067479706532323f00130000000780000005365433335405365433335406747970653233067479706532333f00040000000d60000005365433335405365433335406747970653234067479706532342e0000000000fe00000005365433335405365433335406747970653235067479706532352e0004000000fe00000005365433335405365433335406747970653236067479706532362e00fc030000fe00000005365433335405365433335406747970653237067479706532372e0000000000fd00000005365433335405365433335406747970653238067479706532382e0004000000fd00000005365433335405365433335406747970653239067479706532392e00fc030000fd00000005365433335405365433335406747970653330067479706533303f0000000000fe80000005365433335405365433335406747970653331067479706533313f0001000000fe80000005365433335405365433335406747970653332067479706533323f00ff000000fe80000005365433335405365433335406747970653333067479706533333f00ffff0000fc80000005365433335405365433335406747970653334067479706533343f00ff000000f980000005365433335405365433335406747970653335067479706533353f00ffffff00fa80000005365433335405365433335406747970653336067479706533362e0058020000fc00000005365433335405365433335406747970653337067479706533372e00fcffff03fa00000005365433335405365433335406747970653338067479706533383f0001000000fd80000005365433335405365433335406747970653339067479706533392e0000000000fe00000005365433335405365433335406747970653430067479706534302e0004000000fe00000005365433335405365433335406747970653431067479706534312e0010000000fe00000005365433335405365433335406747970653432067479706534322e0000000000fe00000005365433335405365433335406747970653433067479706534332e0004000000fe00000005365433335405365433335406747970653434067479706534342e00d4030000fe00000005365433335405365433335406747970653435067479706534353f00010000001020000005365433335405365433335406747970653436067479706534363f00400000001020000005365433335405365433335406747970653437067479706534373f00010000000100000005365433335405365433335406747970653438067479706534382e00fcffff03fa00000005365433335405365433335406747970653439067479706534392e00fcffff03fa00000005365433335405365433335406747970653530067479706535302e0004000000fe00000005365433335405365433335406747970653531067479706535312e0004000000fd00000001000000040907000001010100000000049e732b7b010101000000000415d0870701010100000000083aa1ffffffffffff010101000000000871575519bfcd04020101010000000004ba0f9f3e01010100000000044d6d6e3f0101010000000004d66cc3be0101010000000008a4a6c8fae881e83f0101010000000008aba76123a72be63f01010100000000052d313731340101020000000005000000212d302e3233363530303030303030303030303030303030303030303030303030300101020000000021000000062d3531343132010102000000000600000016302e383433303030303030303030303030303030303001010200000000160000000453d955000101010000000004c3d1180001010100000000024b2d0101010000000001660101010000000001170101010000000002282d01010100000000080e0000000054c91d0101010000000008000000287342f21d0101010000000008001ef1cdf239000001010100000000080100007524a8b01f0101010000000002770701010100000000000101020000000000000000013001010200000000010000006436383036323439363639373332383534303836393332373339363837343432383138343539323234343837313936333534323934343333343031373936333438363830353736303134373133383232343632323236353730393639323939303039353035010102000000006400000000010102000000000000000001640101020000000001000000fcff007a4f3979704e654c456e63684f69784944304b527637704134794e6638346e6649317076333252367a737a6246486a34527479447645574e644772557738715161576573774552307464454f50746b4d5a506b50673669564c585551445464486d71674e6736487351665639637134636772735436686e557063736463416e3041584f684f596c65636f34426a44464c7a62424e5276516c6c644632684b413355636c4a7350757738726c616363517872763663655851556c74427a414861773264456b4c4252597a53444e4d5a3033735534456b52666e6a48454e564e6730416f6c5a66355356753059486a426768563148355930654b4b7661735a473101010200000000ff000000000101020000000000000000012a0101020000000001000000fcff00ccd82dbce2a48abe65f391308024cff5bb3971c2aa2761f10c52476cc9e3235206b47528a36eb3767dc50852ed3ebfa5b17a0b47ed2a5a2b56ead26c5b96c805ca65e22eaabbc7af1560e71e57ca734ca952ed17f54285d4feecd11c0d5ba0e679c464cf1196373f427c41a8e9f5c157f78834f2e57c06aa5d5af0682504349f796518eca8f4f33dbf53a9dfa30e1f4232106825d8695817ae992d70fc0a30c8b4f9d232701d948d6a9199acd8e5ce94b5aa50af5929555b91f124167fde826315ba922120d8d4f6c5d1fcbdb08708521f9f91a7e84366f08b90bccbb18186660fe49d5ca270dbe476b3f4c7a7418b51d37cf38c7c55e5c784257dde277a4f01010200000000ff00000064279e9d01d72ec83ad3ae46ce057466371ba263f62a726018347135ec8619bc835e5f215431902b181ff2e41c6a28115a1e01b50ed33faad7601416032c3d2911b00b25e051f1f3676b082aa98d618f251c5b984976c4a89f7ed84ad8ef0ff22cdf05f1c50101020000000064000000189bc7836679d2fb8bf55d01daa0c7b7791a01b2d9359ec8ea010102000000001800000064892df41245164975050a48fdc2a262b012f29e95763cb507bd5df48b339d1539a988202f99d31cfe072df1696a58529b7813d848a4edcf9133bdbe67b4449b0718b07839471e35a6a798f9f17d6db70aee9b989e88d19dc9e363fc5906c4abb7d1bae32a0101020000000064000000143635363636373638363937303731373237333734010102000000001400000014363836393730373137323733373437353736373701010200000000140000000116010102000000000100000000010002000000000000000001610101020000000001000000046132373501010200000000040000000001000200000000000000000001000200000000000000000c61302c61312c61322c613630010102000000000c00000001000101020000000001000000087fffffffffffffff01010200000000080000000150010101000000001437323733373437353736373737383739383038310101020000000014000000143635363636373638363937303731373237333734010102000000001400000001390101020000000001000000016b0101020000000001000000"
  b, _ := hexutil.Decode(answer)
  rs := kvsql.DecodeRecordSet(b)
  rs.GetRow(0)




**4.4.1.8 解具体的列(字段)的值**

解具体字段的值，需要弄清楚自己定义的**表字段类型**以及希望获取的字段**下标，**并调用以下方法与之**一一对应**。比如字符串类型字段调用GetString(index)来解。若类型对应错误会导致解出来的值不对，更严重的会导致panic。


- tinyint                   : GetInt8
- tinyint unsigned          : GetUint8
- smallint                  : GetInt16
- smallint unsigned         : GetUint16
- int                       : GetInt32
- unsigned int              : GetUint32
- bigint                    : GetInt64
- unsigned bigint           : GetUint64
- float                     : GetFloat32
- dobule                    : GetFloat64
- decimal                   : GetDecimal
- varchar                   : GetString
- bit, blob(字节类型)        : GetBytes
- DateTime, timestamp, date : GetTime
- time                      : GetDuration
- enum                      : GetEnum
- set                       : GetSet

 ::

  func (r Row) GetInt8(colIdx int) int8
  func (r Row) GetUint8(colIdx int) uint8
  func (r Row) GetInt16(colIdx int) int16
  func (r Row) GetUint16(colIdx int) uint16
  func (r Row) GetInt32(colIdx int) int32
  func (r Row) GetUint32(colIdx int) uint32
  func (r Row) GetInt64(colIdx int) int64
  func (r Row) GetUint64(colIdx int) uint64
  func (r Row) GetFloat32(colIdx int) float32
  func (r Row) GetFloat64(colIdx int) float64
  func (r Row) GetString(colIdx int) string
  func (r Row) GetBytes(colIdx int) []byte
  func (r Row) GetTime(colIdx int) types.Time
  func (r Row) GetDuration(colIdx int, fillFsp int) types.Duration
  func (r Row) GetEnum(colIdx int) types.Enum
  func (r Row) IsNull(colIdx int) bool
  func (r Row) GetDecimal(colIdx int) *types.MyDecimal
  func (r Row) GetSet(colIdx int) types.Set


示例：

 ::

  answer := "0x0035000000053654333354053654333354066175746f4964066175746f49643f000b000000030502000536543333540536543333540574797065300574797065303f000b000000030300000536543333540536543333540574797065310574797065313f000a000000032000000536543333540536543333540574797065320574797065323f0014000000080000000536543333540536543333540574797065330574797065333f0014000000082800000536543333540536543333540574797065340574797065343f000c0000000408001f0536543333540536543333540574797065350574797065353f00ff0000000408001e0536543333540536543333540574797065360574797065363f00ff0000000408001e0536543333540536543333540574797065370574797065373f00160000000508001f0536543333540536543333540574797065380574797065383f00ff0000000500001e0536543333540536543333540574797065390574797065393f000b000000f600000005365433335405365433335406747970653130067479706531303f0020000000f600001e05365433335405365433335406747970653131067479706531313f000b000000f600000005365433335405365433335406747970653132067479706531323f0016000000f600001405365433335405365433335406747970653133067479706531333f00090000000900000005365433335405365433335406747970653134067479706531343f00080000000920000005365433335405365433335406747970653135067479706531353f00060000000200000005365433335405365433335406747970653136067479706531363f00040000000100000005365433335405365433335406747970653137067479706531373f00030000000120000005365433335405365433335406747970653138067479706531383f00050000000220000005365433335405365433335406747970653139067479706531393f000a0000000a80000005365433335405365433335406747970653230067479706532303f00130000000c80000005365433335405365433335406747970653231067479706532313f000a0000000b80000005365433335405365433335406747970653232067479706532323f00130000000780000005365433335405365433335406747970653233067479706532333f00040000000d60000005365433335405365433335406747970653234067479706532342e0000000000fe00000005365433335405365433335406747970653235067479706532352e0004000000fe00000005365433335405365433335406747970653236067479706532362e00fc030000fe00000005365433335405365433335406747970653237067479706532372e0000000000fd00000005365433335405365433335406747970653238067479706532382e0004000000fd00000005365433335405365433335406747970653239067479706532392e00fc030000fd00000005365433335405365433335406747970653330067479706533303f0000000000fe80000005365433335405365433335406747970653331067479706533313f0001000000fe80000005365433335405365433335406747970653332067479706533323f00ff000000fe80000005365433335405365433335406747970653333067479706533333f00ffff0000fc80000005365433335405365433335406747970653334067479706533343f00ff000000f980000005365433335405365433335406747970653335067479706533353f00ffffff00fa80000005365433335405365433335406747970653336067479706533362e0058020000fc00000005365433335405365433335406747970653337067479706533372e00fcffff03fa00000005365433335405365433335406747970653338067479706533383f0001000000fd80000005365433335405365433335406747970653339067479706533392e0000000000fe00000005365433335405365433335406747970653430067479706534302e0004000000fe00000005365433335405365433335406747970653431067479706534312e0010000000fe00000005365433335405365433335406747970653432067479706534322e0000000000fe00000005365433335405365433335406747970653433067479706534332e0004000000fe00000005365433335405365433335406747970653434067479706534342e00d4030000fe00000005365433335405365433335406747970653435067479706534353f00010000001020000005365433335405365433335406747970653436067479706534363f00400000001020000005365433335405365433335406747970653437067479706534373f00010000000100000005365433335405365433335406747970653438067479706534382e00fcffff03fa00000005365433335405365433335406747970653439067479706534392e00fcffff03fa00000005365433335405365433335406747970653530067479706535302e0004000000fe00000005365433335405365433335406747970653531067479706535312e0004000000fd00000001000000040907000001010100000000049e732b7b010101000000000415d0870701010100000000083aa1ffffffffffff010101000000000871575519bfcd04020101010000000004ba0f9f3e01010100000000044d6d6e3f0101010000000004d66cc3be0101010000000008a4a6c8fae881e83f0101010000000008aba76123a72be63f01010100000000052d313731340101020000000005000000212d302e3233363530303030303030303030303030303030303030303030303030300101020000000021000000062d3531343132010102000000000600000016302e383433303030303030303030303030303030303001010200000000160000000453d955000101010000000004c3d1180001010100000000024b2d0101010000000001660101010000000001170101010000000002282d01010100000000080e0000000054c91d0101010000000008000000287342f21d0101010000000008001ef1cdf239000001010100000000080100007524a8b01f0101010000000002770701010100000000000101020000000000000000013001010200000000010000006436383036323439363639373332383534303836393332373339363837343432383138343539323234343837313936333534323934343333343031373936333438363830353736303134373133383232343632323236353730393639323939303039353035010102000000006400000000010102000000000000000001640101020000000001000000fcff007a4f3979704e654c456e63684f69784944304b527637704134794e6638346e6649317076333252367a737a6246486a34527479447645574e644772557738715161576573774552307464454f50746b4d5a506b50673669564c585551445464486d71674e6736487351665639637134636772735436686e557063736463416e3041584f684f596c65636f34426a44464c7a62424e5276516c6c644632684b413355636c4a7350757738726c616363517872763663655851556c74427a414861773264456b4c4252597a53444e4d5a3033735534456b52666e6a48454e564e6730416f6c5a66355356753059486a426768563148355930654b4b7661735a473101010200000000ff000000000101020000000000000000012a0101020000000001000000fcff00ccd82dbce2a48abe65f391308024cff5bb3971c2aa2761f10c52476cc9e3235206b47528a36eb3767dc50852ed3ebfa5b17a0b47ed2a5a2b56ead26c5b96c805ca65e22eaabbc7af1560e71e57ca734ca952ed17f54285d4feecd11c0d5ba0e679c464cf1196373f427c41a8e9f5c157f78834f2e57c06aa5d5af0682504349f796518eca8f4f33dbf53a9dfa30e1f4232106825d8695817ae992d70fc0a30c8b4f9d232701d948d6a9199acd8e5ce94b5aa50af5929555b91f124167fde826315ba922120d8d4f6c5d1fcbdb08708521f9f91a7e84366f08b90bccbb18186660fe49d5ca270dbe476b3f4c7a7418b51d37cf38c7c55e5c784257dde277a4f01010200000000ff00000064279e9d01d72ec83ad3ae46ce057466371ba263f62a726018347135ec8619bc835e5f215431902b181ff2e41c6a28115a1e01b50ed33faad7601416032c3d2911b00b25e051f1f3676b082aa98d618f251c5b984976c4a89f7ed84ad8ef0ff22cdf05f1c50101020000000064000000189bc7836679d2fb8bf55d01daa0c7b7791a01b2d9359ec8ea010102000000001800000064892df41245164975050a48fdc2a262b012f29e95763cb507bd5df48b339d1539a988202f99d31cfe072df1696a58529b7813d848a4edcf9133bdbe67b4449b0718b07839471e35a6a798f9f17d6db70aee9b989e88d19dc9e363fc5906c4abb7d1bae32a0101020000000064000000143635363636373638363937303731373237333734010102000000001400000014363836393730373137323733373437353736373701010200000000140000000116010102000000000100000000010002000000000000000001610101020000000001000000046132373501010200000000040000000001000200000000000000000001000200000000000000000c61302c61312c61322c613630010102000000000c00000001000101020000000001000000087fffffffffffffff01010200000000080000000150010101000000001437323733373437353736373737383739383038310101020000000014000000143635363636373638363937303731373237333734010102000000001400000001390101020000000001000000016b0101020000000001000000"
  b, _ := hexutil.Decode(answer)
  rs := kvsql.DecodeRecordSet(b)
  row := rs.GetRow(0)
  row.GetString(49)




4.5 FVM
-------------

4.5.1 abi编解码
>>>>>>>>>>>>>>>>>>>>>>

**规约**

虚拟机内部所有的abi类型均实现接口Compact

 ::

  // Compact compact mode
  type Compact interface {
    Encode() ([]byte, error)
    Decode(val []byte) (int, error)
    GetVal() interface{}
    GetType() TypeString
  }


目前FVM针对rust合约，类型映射关系如下

- FixI8         : i8
- FixU8         : u8
- FixI16        : i16
- FixU16        : u16
- FixI32        : i32
- FixU32        : u32
- FixI64        : i64
- FixU64        : u64
- FixI128       : i128
- FixU128       : u128
- CompactU8     : Compact<u8>(暂不支持)
- CompactU16    : Compact<u16>(暂不支持)
- CompactU32    : Compact<u32>(暂不支持)
- CompactU64    : Compact<u64>(暂不支持)
- CompactU128   : Compact<u128>(暂不支持)
- CompactString : str
- CompactVec    : vec
- CompactArray  : array([; 3])
- CompactStruct : struct
- CompactBool   : bool



**需要特殊说明的是:**

1. **128位的整数由于go原生不支持，这里利用big.Int(16位byte)来表示与编解码**

1. **CompactVec, CompactArray， 由于存在可能的嵌套，我们必须要指定NextTypeList, 表明其嵌套的深度，例如vec<vec<string>>, 他的NextTypeList为[]TypeString{VecName, String}**

1. **不要出现循环引用的结构**



****规约部分仅作了解，我们的真实使用使用请使用以下方法****

示例结构体

 ::

  #[derive(TypeInfo)]
  pub struct Student {
      id: u32,
      name: String,
      list: Vec<Vec<String>>,
  }

  #[derive(TypeInfo)]
  pub struct Info {
      ages: Vec<[Student; 10]>,
  }


示例abi

 ::

  {
    "contract": {
      "name": "MyContract",
      "constructor": {
        "input": []
      }
    },
    "methods": [
      {
        "name": "add",
        "input": [
          {
            "type_id": 0
          }
        ],
        "output": [
          {
            "type_id": 0
          }
        ]
      },
      {
        "name": "make_student",
        "input": [
          {
            "type_id": 1
          },
          {
            "type_id": 6
          }
        ],
        "output": [
          {
            "type_id": 0
          }
        ]
      }
    ],
    "types": [
      {
        "id": 0,
        "type": "primitive",
        "primitive": "u64"
      },
      {
        "id": 1,
        "type": "struct",
        "fields": [
          {
            "type_id": 2
          },
          {
            "type_id": 3
          },
          {
            "type_id": 4
          }
        ]
      },
      {
        "id": 2,
        "type": "primitive",
        "primitive": "u32"
      },
      {
        "id": 3,
        "type": "primitive",
        "primitive": "str"
      },
      {
        "id": 4,
        "type": "vec",
        "fields": [
          {
            "type_id": 5
          }
        ]
      },
      {
        "id": 5,
        "type": "vec",
        "fields": [
          {
            "type_id": 3
          }
        ]
      },
      {
        "id": 6,
        "type": "struct",
        "fields": [
          {
            "type_id": 7
          }
        ]
      },
      {
        "id": 7,
        "type": "vec",
        "fields": [
          {
            "type_id": 8
          }
        ]
      },
      {
        "id": 8,
        "type": "array",
        "fields": [
          {
            "type_id": 1
          }
        ],
        "array_len": 10
      }
    ]
  }





**4.5.1.1 解析abi文件**

 ::

  func GenAbi(reader io.Reader) (*scale.Abi, error)


 ::

  // currentABI 为abi的字符串
  a, err := fvm.GenAbi(strings.NewReader(currentABI))
  // path 为abi文件路径
  by, err := ioutil.ReadFile(path)
  a, err := JSON(bytes.NewReader(by))


**4.5.1.2 获取payload**

 ::

  func Encode(abi *scale.Abi, method string, params ...interface{}) ([]byte, error)


**特别注意：**

**1.对于params的值严格按照abi文件对应的参数顺序，以及各个参数的格式进行传参**

**2.对于结构体类型，按照其每个属性的值以及顺序组成一个[]interface{}进行传输**

**3.对于数组按照顺序组成一个[]interface{}进行传输**

**4.对于整数，可以方便的传int或者字符串（我们依据abi文件对其正确性做了校验），但如果开发者自己指定类型，例如uint16(1), 这个时候我们会对该数字做严格的类型校验(开发者应该明白，uint16(1)与uint8(1)是不同的)**

 ::

  fvm.Encode(abi, "make_student", []interface{}{
        uint32(1),
        "test",
        []interface{}{
          []interface{}{
            "hello",
            "world",
          },
        },
      }, []interface{}{
        []interface{}{},
      })
  fvm.Encode(abi, "make_student", []interface{}{
        1,
        "test",
        []interface{}{
          []interface{}{
            "hello",
            "world",
          },
        },
      }, []interface{}{
        []interface{}{},
      })

  fvm.Encode(abi, "make_student", []interface{}{
        "1",
        "test",
        []interface{}{
          []interface{}{
            "hello",
            "world",
          },
        },
      }, []interface{}{
        []interface{}{},
      })




**4.5.1.3 解析返回值**

 ::

  type InvokeBean struct {
    // 方法名
    MethodName string  `json:"method_name"`
    // 返回值列表
    Params     []Compact `json:"params"`
  }

  func DecodeRet(abi *scale.Abi, method string, val []byte) (*scale.InvokeBean, error)


***** 开发者应当自己在应用中定义对应的结构来映射放回值, 对于复杂的嵌套类型，开发者应当自己清楚的知道自己在做什么**

 ::

  type Student struct {
    Name string
    Age int
    Nums []int
    Friend Base
  }

  type Base struct {
    Name string
    Age int
  }
  res, err := fvm.DecodeRet(abi, "make_student", common.Hex2Bytes("xxx"))
  var nums []int
  for _, v := range res.Params[2].GetVal().([]scale.Compact) {
    nums = append(nums, v.GetVal())
  }
  friendCompact := res.Params[3].GetVal().([]scale.Compact)
  f := Base{
    Name: friendCompact[0].GetVal(),
    Age: friendComapct[1].GetVal(),
  }
  newStudent := &Student{
    Name: res.Params[0].GetVal(),
    Age: res.Params[1].GetVal(),
    Nums: num,
    Friend: f,
  }


第五章 RPC 接口
======================





第六章 GRPC 接口
=========================



第七章 报错说明
=====================

GoSDK中和网络相关的RPC、grpc调用均返回StdError类型的错误。StdError为一个接口。

 ::

  type StdError interface {
    fmt.Stringer
    error
    Code() int
  }


所以，该结构提供三种方法：

`func (re *RetError) Error() string`

- 说明：实现了error接口。

- 返回【返回值1】：返回错误描述。

`func (re *RetError) String() string`

- 说明：实现了fmt.Stringer接口。

- 返回【返回值1】：将会返回以下格式字符串"error code: %d, error reason: %s", re.Code(), re.Error()，即错误码加错误描述。

`func (re *RetError) Code() int {`

- 说明：返回错误码。

- 返回【返回值1】：错误码。错误码表如下。


- 0      : 请求成功
- -32700 : 服务端接收到无效的json。该错误发送于服务器尝试解析json文本
- -32600 : 无效的请求（比如非法的JSON格式）
- -32601 : 方法不存在或者无效
- -32602 : 无效的方法参数
- -32603 : JSON-RPC内部错误
- -32605 : cvp在报错前的拦截器
- -32606 : cvp在报错后的拦截器
- -32000 : Hyperchain内部错误或者空指针或者节点未安装solidity环境
- -32001 : 请求的数据不存在
- -32002 : 余额不足
- -32003 : 签名非法
- -32004 : 合约部署出错
- -32005 : 合约调用出错
- -32006 : 系统繁忙
- -32007 : 交易重复
- -32008 : 合约操作权限不够
- -32009 : (合约)账户不存在
- -32010 : namespace不存在
- -32011 : 账本上无区块产生，查询最新区块的时候可能抛出该错误
- -32012 : 订阅不存在
- -32013 : 数据归档、快照相关错误
- -32014 : 未能解锁本地节点帐户
- -32015 : 调用原生合约失败
- -32016 : 无效的参与者节点哈希
- -32017 : 不可用的peer
- -32018 : 不一致的peer
- -32019 : 没有collection信息的非法的私有交易
- -32020 : ACO服务不可用
- -32021 : 废弃的接口
- -32022 : did调用失败
- -32023 : cvp不支持此方法
- -32024 : 共识状态非正常
- -32025 : 进程已满
- -32026 : QPS 受限
- -32027 : simulate受限
- -32028 : sql执行出错
- -32096 : http请求处理超时
- -32097 : Hypercli用户令牌无效
- -32098 : 请求未带cert或者错误cert导致认证失败
- -32099 : 请求tcert失败
- -9995  : 请求失败(通常是请求体过长)
- -9996  : 请求失败(通常是请求消息错误)
- -9997  : 异步请求失败
- -9998  : 请求超时
- -9999  : 获取平台响应失败







第八章 示例
======================

8.1 EVM
----------------

8.1.1 编写合约代码
>>>>>>>>>>>>>>>>>>>>>>>

 ::

  pragma solidity 0.8.0;

  contract SetHash{
      mapping(string => string) hashMapping;

      function insert(string memory key,string memory value) public {
          hashMapping[key] = value;
      }

      function get(string memory key) public view returns(string memory) {
          return hashMapping[key];
      }
  }




8.1.2 编译合约
>>>>>>>>>>>>>>>>>>>>>>>

 ::

  // abi
  [
    {
      "inputs": [
        {
          "internalType": "string",
          "name": "key",
          "type": "string"
        }
      ],
      "name": "get",
      "outputs": [
        {
          "internalType": "string",
          "name": "",
          "type": "string"
        }
      ],
      "stateMutability": "view",
      "type": "function"
    },
    {
      "inputs": [
        {
          "internalType": "string",
          "name": "key",
          "type": "string"
        },
        {
          "internalType": "string",
          "name": "value",
          "type": "string"
        }
      ],
      "name": "insert",
      "outputs": [],
      "stateMutability": "nonpayable",
      "type": "function"
    }
  ]

  // bin
  608060405234801561001057600080fd5b5061056a806100206000396000f3fe608060405234801561001057600080fd5b50600436106100365760003560e01c806306e63ff81461003b578063693ec85e14610057575b600080fd5b610055600480360381019061005091906102ba565b610087565b005b610071600480360381019061006c9190610279565b6100be565b60405161007e91906103a7565b60405180910390f35b806000836040516100989190610390565b908152602001604051809103902090805190602001906100b992919061016e565b505050565b60606000826040516100d09190610390565b908152602001604051809103902080546100e990610493565b80601f016020809104026020016040519081016040528092919081815260200182805461011590610493565b80156101625780601f1061013757610100808354040283529160200191610162565b820191906000526020600020905b81548152906001019060200180831161014557829003601f168201915b50505050509050919050565b82805461017a90610493565b90600052602060002090601f01602090048101928261019c57600085556101e3565b82601f106101b557805160ff19168380011785556101e3565b828001600101855582156101e3579182015b828111156101e25782518255916020019190600101906101c7565b5b5090506101f091906101f4565b5090565b5b8082111561020d5760008160009055506001016101f5565b5090565b600061022461021f846103fa565b6103c9565b90508281526020810184848401111561023c57600080fd5b610247848285610451565b509392505050565b600082601f83011261026057600080fd5b8135610270848260208601610211565b91505092915050565b60006020828403121561028b57600080fd5b600082013567ffffffffffffffff8111156102a557600080fd5b6102b18482850161024f565b91505092915050565b600080604083850312156102cd57600080fd5b600083013567ffffffffffffffff8111156102e757600080fd5b6102f38582860161024f565b925050602083013567ffffffffffffffff81111561031057600080fd5b61031c8582860161024f565b9150509250929050565b60006103318261042a565b61033b8185610435565b935061034b818560208601610460565b61035481610523565b840191505092915050565b600061036a8261042a565b6103748185610446565b9350610384818560208601610460565b80840191505092915050565b600061039c828461035f565b915081905092915050565b600060208201905081810360008301526103c18184610326565b905092915050565b6000604051905081810181811067ffffffffffffffff821117156103f0576103ef6104f4565b5b8060405250919050565b600067ffffffffffffffff821115610415576104146104f4565b5b601f19601f8301169050602081019050919050565b600081519050919050565b600082825260208201905092915050565b600081905092915050565b82818337600083830152505050565b60005b8381101561047e578082015181840152602081019050610463565b8381111561048d576000848401525b50505050565b600060028204905060018216806104ab57607f821691505b602082108114156104bf576104be6104c5565b5b50919050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052602260045260246000fd5b7f4e487b7100000000000000000000000000000000000000000000000000000000600052604160045260246000fd5b6000601f19601f830116905091905056fea264697066735822122022f216ab2a903b000b9ef84705051fed53aecd3e515879d46fd8372ba6f2cef364736f6c63430008000033


8.1.3 使用gosdk
>>>>>>>>>>>>>>>>>>>>>>>

 ::

  package main

  import (
    "fmt"
    "github.com/hyperchain/gosdk/abi"
    "github.com/hyperchain/gosdk/account"
    "github.com/hyperchain/gosdk/rpc"
    "strings"
  )

  const (
    abiContract = `[
    {
        "constant": false,
        "inputs": [
          {
              "name": "key",
              "type": "string"
          },
          {
              "name": "value",
              "type": "string"
          }
        ],
        "name": "insert",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
    },
    {
        "constant": true,
        "inputs": [
          {
              "name": "key",
              "type": "string"
          }
        ],
        "name": "get",
        "outputs": [
          {
              "name": "",
              "type": "string"
          }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
    }
  ]`
    binContract = "608060405234801561001057600080fd5b50610444806100206000396000f30060806040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306e63ff814610051578063693ec85e14610100575b600080fd5b34801561005d57600080fd5b506100fe600480360381019080803590602001908201803590602001908080601f0160208091040260200160405190810160405280939291908181526020018383808284378201915050505050509192919290803590602001908201803590602001908080601f01602080910402602001604051908101604052809392919081815260200183838082843782019150505050505091929192905050506101e2565b005b34801561010c57600080fd5b50610167600480360381019080803590602001908201803590602001908080601f0160208091040260200160405190810160405280939291908181526020018383808284378201915050505050509192919290505050610266565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156101a757808201518184015260208101905061018c565b50505050905090810190601f1680156101d45780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b806000836040518082805190602001908083835b60208310151561021b57805182526020820191506020810190506020830392506101f6565b6001836020036101000a03801982511681845116808217855250505050505090500191505090815260200160405180910390209080519060200190610261929190610373565b505050565b60606000826040518082805190602001908083835b6020831015156102a0578051825260208201915060208101905060208303925061027b565b6001836020036101000a03801982511681845116808217855250505050505090500191505090815260200160405180910390208054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156103675780601f1061033c57610100808354040283529160200191610367565b820191906000526020600020905b81548152906001019060200180831161034a57829003601f168201915b50505050509050919050565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106103b457805160ff19168380011785556103e2565b828001600101855582156103e2579182015b828111156103e15782518255916020019190600101906103c6565b5b5090506103ef91906103f3565b5090565b61041591905b808211156104115760008160009055506001016103f9565b5090565b905600a165627a7a7230582001a711c918e9f0c995f05f63b0f98e6ee752d4e014717372d11bab848cbe4bae0029"
  )


  func main() {
    // new account
    //创建账户
    accountJson, err := account.NewAccountSm2("123")
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("accountJson:", accountJson)
    key, err := account.GenKeyFromAccountJson(accountJson, "123")
    if err != nil {
        fmt.Println(err)
        return
    }
    newAddress := key.(*account.SM2Key).GetAddress()
    fmt.Println("account address:", newAddress)
    rpcAPI := rpc.NewRPCWithPath("./conf")

    // deploy
    //部署合约
    tranDeploy := rpc.NewTransaction(newAddress.Hex()).Deploy(binContract)
    txDeploy, stdErr := rpcAPI.SignAndDeployContract(tranDeploy, key)
    if stdErr != nil {
        fmt.Println(stdErr)
        return
    }
    fmt.Println("contract address:", txDeploy.ContractAddress)
    // invoke contract
    //调用合约
    ABI, _ := abi.JSON(strings.NewReader(abiContract))
    packed, err := ABI.Pack("insert", "key1", "value1")
    tranInvoke := rpc.NewTransaction(newAddress.Hex()).Invoke(txDeploy.ContractAddress, packed)
    txInvoke, stdErr := rpcAPI.SignAndInvokeContract(tranInvoke, key)
    if stdErr != nil {
        fmt.Println(stdErr)
        return
    }
    fmt.Println("invoke transaction hash:", txInvoke.TxHash)
    // invoke contract
    //调用合约
    packed1, err := ABI.Pack("get", "key1")
    tranInvoke1 := rpc.NewTransaction(newAddress.Hex()).Invoke(txDeploy.ContractAddress, packed1)
    txInvoke1, err := rpcAPI.SignAndInvokeContract(tranInvoke1, key)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("invoke transaction hash:", txInvoke1.TxHash)

    // decode
    //返回值解析
    var result string
    if err = ABI.UnpackResult(&result, "get", txInvoke1.Ret); err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("get返回值:", result)

  }



8.2 HVM
-----------

本节展示了使用GoSDK操作HVM合约的完整例子。在使用gosdk之前，请到 http://hvm.internal.hyperchain.cn/](http://hvm.internal.hyperchain.cn/ 上学习如何编写HVM合约

通过一个SetHash 合约来展示如果通过gosdk 发送hvm合约交易。更多的HVM合约Demo 可参考下面链接中的示例

http://thoughts.hyperchain.cn:8099/workspaces/5b6c3babbe825b41b446a307/docs/5ed787cac98cc900016b726e,

http://thoughts.hyperchain.cn:8099/workspaces/5b6c3babbe825b41b446a307/docs/5ed787cac98cc900016b726e



8.2.1 HVM合约invokeBean 如下所示
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

  package invoke;

  import cn.hyperchain.contract.BaseInvoke;
  import logic.ISetHash;

  /**
  * @author Lam
  * @ClassName InvokeSetHash
  * @date 2019-10-11
  */
  public class InvokeSetHash implements BaseInvoke<String, ISetHash> {
      private String key;
      private String value;

      public InvokeSetHash() {
      }

      public InvokeSetHash(String key, String value) {
          this.key = key;
          this.value = value;
      }

      @Override
      public String invoke(ISetHash iSetHash) {
          iSetHash.setHash(key, value);
          return null;
      }
  }



8.2.2 HVM 合约接口定义以及合约实现如下所示：
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

  package logic;

  import cn.hyperchain.contract.BaseContractInterface;

  /**
  * @author Lam
  * @ClassName ISetHash
  * @date 2019-10-11
  */
  public interface ISetHash extends BaseContractInterface {
      void setHash(String key, String value);
  }



 ::

  package logic;

  import cn.hyperchain.annotations.StoreField;
  import cn.hyperchain.contract.BaseContract;
  import cn.hyperchain.core.HyperMap;

  /**
  * @author Lam
  * @ClassName SetHash
  * @date 2019-10-11
  */
  public class SetHash extends BaseContract implements ISetHash {
      @StoreField
      HyperMap<String, String> map = new HyperMap<String, String>();

      @Override
      public void setHash(String key, String value) {
          map.put(key, value);
          if (key.equals("error")) {
              throw new RuntimeException("error!!!");
          }
      }

      public String getHash(String key) {
          return map.get(key);
      }
  }



8.2.3 合约编译以及ABI生成
>>>>>>>>>>>>>>>>>>>>>>>>>>

合约编译方式可以参考HVM上述HVM合约使用链接，这里只介绍HVM合约ABI 生成方式。

利用我们提供的HVM ABI 生成maven 插件即可编译生成HVM合约的ABI , 需要在合约项目pom 文件中加入下面的依赖

 ::

              <plugin>
                  <groupId>cn.hyperchain.hvm</groupId>
                  <artifactId>hvm-maven-plugin</artifactId>
                  <version>0.0.2</version>
                  <configuration>
                      <jarFile>${project.basedir}/target/sethash-1.0-sethash.jar</jarFile>
                      <invokeBeanPath>${project.basedir}/target/classes</invokeBeanPath>
                      <invokeBeanPackages>
                          <param>invoke.InvokeSetHash</param>
                      </invokeBeanPackages>
                      <outputFile>${project.basedir}/target/hvm.abi</outputFile>
                  </configuration>
              </plugin>



其中

- <jarFile> 中是hvm合约的jar包

- <invokeBeanPath> 是hvm合约的class字节码文件夹

- <invokeBeanPackages>中包含很多<param>标签，每个<param>标签中都是一个实现*BaseInvoke*接口的类的全路径。

- <outputFile>中存放的是生成的.abi的存放目录

在合约编译完成后，执行下面的命令，即可生成HVM合约的ABI

 ::

  mvn hvm:abi


在生成的hvm.abi中

- version代表hvm.abi的版本

- beanName代表实现*BaseInvoke*接口的类的名称

- inputs代表invoke方法的参数  其中

- name 代表变量名称

- type代表变量类型

- 如果是结构体则structname代表结构体类的名称

- output代表invoke方法的返回值

- structs里面包含HVM合约中的结构体

示例hvm.abi如下

 ::

  [{"version":"v1","beanName":"invoke.InvokeSetHash","inputs":[{"name":"key","type":"String","structName":"java.lang.String"},{"name":"value","type":"String","structName":"java.lang.String"}],"output":{"name":"java.lang.String","type":"String","structName":"java.lang.String"},"classBytes":"cafebabe0000003400280a0008001e090007001f09000700200b000500210700220a000700230700240700250700260100036b65790100124c6a6176612f6c616e672f537472696e673b01000576616c75650100063c696e69743e010003282956010004436f646501000f4c696e654e756d6265725461626c650100124c6f63616c5661726961626c655461626c65010004746869730100164c696e766f6b652f496e766f6b65536574486173683b010027284c6a6176612f6c616e672f537472696e673b4c6a6176612f6c616e672f537472696e673b2956010006696e766f6b65010024284c6c6f6769632f49536574486173683b294c6a6176612f6c616e672f537472696e673b01000869536574486173680100104c6c6f6769632f49536574486173683b010042284c636e2f6879706572636861696e2f636f6e74726163742f42617365436f6e7472616374496e746572666163653b294c6a6176612f6c616e672f4f626a6563743b0100095369676e61747572650100594c6a6176612f6c616e672f4f626a6563743b4c636e2f6879706572636861696e2f636f6e74726163742f42617365496e766f6b653c4c6a6176612f6c616e672f537472696e673b4c6c6f6769632f49536574486173683b3e3b01000a536f7572636546696c65010012496e766f6b65536574486173682e6a6176610c000d000e0c000a000b0c000c000b0c0027001401000e6c6f6769632f49536574486173680c00150016010014696e766f6b652f496e766f6b65536574486173680100106a6176612f6c616e672f4f626a656374010021636e2f6879706572636861696e2f636f6e74726163742f42617365496e766f6b65010007736574486173680021000700080001000900020002000a000b00000002000c000b000000040001000d000e0001000f0000003300010001000000052ab70001b10000000200100000000a00020000000f0004001000110000000c0001000000050012001300000001000d00140001000f00000059000200030000000f2ab700012a2bb500022a2cb50003b1000000020010000000120004000000120004001300090014000e001500110000002000030000000f0012001300000000000f000a000b00010000000f000c000b00020001001500160001000f0000004800030002000000102b2ab400022ab40003b90004030001b00000000200100000000a000200000019000e001a001100000016000200000010001200130000000000100017001800011041001500190001000f0000003300020002000000092a2bc00005b60006b00000000200100000000600010000000b00110000000c0001000000090012001300000002001a00000002001b001c00000002001d","structs":[]}]



8.2.4 调用上述合约的 gosdk代码如下所示
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

  func main() {

    //创建账户
    accountJson, err := account.NewAccountSm2("123")
    if err != nil {
      fmt.Println(err)
      return
    }
    fmt.Println("accountJson:", accountJson)
    key, err := account.GenKeyFromAccountJson(accountJson, "123")
    if err != nil {
      fmt.Println(err)
      return
    }
    newAddress := key.(*account.SM2Key).GetAddress()
    fmt.Println("account address:", newAddress)
    rpcAPI := rpc.NewRPCWithPath("./conf")
    jarPath := "./conf/contract/sethash-1.0-sethash.jar"
    payload, sysErr := rpc.DecompressFromJar(jarPath)
    if sysErr != nil {
      fmt.Println(sysErr)
      return
    }
    transaction := rpc.NewTransaction(newAddress.Hex()).Deploy(common.Bytes2Hex(payload)).VMType(rpc.HVM)
    receipt, sysErr := rpcAPI.SignAndDeployContract(transaction, key)
    if sysErr != nil {
      fmt.Println(sysErr)
      return
    }
    fmt.Println("contract address:", receipt.ContractAddress)
    //read abi
    abiPath := "./conf/contract/hvm.abi"
    abiJson, sysErr := common.ReadFileAsString(abiPath)
    if sysErr != nil {
      fmt.Println(sysErr)
      return
    }
    abi, sysErr := hvm.GenAbi(abiJson)
    if sysErr != nil {
      fmt.Println(sysErr)
      return
    }
    invokeBean := "invoke.InvokeSetHash"
    beanAbi, sysErr := abi.GetBeanAbi(invokeBean)
    if sysErr != nil {
      fmt.Println(sysErr)
      return
    }
    invokePayload, sysErr := hvm.GenPayload(beanAbi, "key1", "value1")
    invokeTx := rpc.NewTransaction(newAddress.Hex()).Invoke(receipt.ContractAddress, invokePayload).VMType(rpc.HVM)
    invokeRe, sysErr := rpcAPI.SignAndInvokeContract(invokeTx, key)
    if sysErr != nil {
      fmt.Println(sysErr)
      return
    }
    fmt.Printf("java log 解码前: %s\n", invokeRe.TxHash)
    fmt.Printf("java log 解码后：%s\n", java.DecodeJavaResult(invokeRe.Ret))
  }



8.3 KVSQL
---------------

 ::

  hrpc := rpc.NewRPCWithPath("../conf")
  js, err := account.NewAccountSm2("12345678")
  gmAcc, err := account.GenKeyFromAccountJson(js, "12345678")
  newAddress := gmAcc.(*account.SM2Key).GetAddress()
  transaction := NewTransaction(newAddress.Hex()).
    VMType(KVSQL).
    Deploy(hexutil.Encode([]byte("KVSQL")))
  // 建库
  txReceipt, err := hrpc.SignAndDeployContract(transaction, gmAcc)

  // 建表
  str := "CREATE TABLE IF NOT EXISTS testTable (id bigint(20) NOT NULL, name varchar(32) NOT NULL, exp bigint(20), money double(16,2) NOT NULL DEFAULT '99', primary key (id), unique key name (name));"
  tranInvoke := NewTransaction(newAddress.Hex()).InvokeSql(txReceipt.ContractAddress, []byte(str))
  tranInvoke.VMType(KVSQL)
  res, err := hrpc.SignAndInvokeContract(tranInvoke, gmAcc)

  // 插入
  in := "insert into testTable (id, name, exp, money) values (1, 'test', 1, 1.1);"
  tranInvoke2 := NewTransaction(newAddress.Hex()).InvokeSql(txReceipt.ContractAddress, []byte(in))
  tranInvoke2.VMType(KVSQL)
  _, err = hrpc.SignAndInvokeContract(tranInvoke2, gmAcc)


  // 获取
  sl := "select * from testTable where id = 1;"
  tranInvoke3 := NewTransaction(newAddress.Hex()).InvokeSql(txReceipt.ContractAddress, []byte(sl))
  tranInvoke3.VMType(KVSQL)
  res, err = hrpc.SignAndInvokeContract(tranInvoke3, gmAcc)
  b, _ := hexutil.Decode(res.Ret)
  rs := kvsql.DecodeRecordSet(b)


8.4 BVM
--------------

 ::

  rpc := NewRPC()
  const genesisAccountJson = `{"address":"0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd","version":"4.0", "algo":"0x03","publicKey":"0400ddbadb932a0d276e257c6df50599a425804a3743f40942d031f806bf14ab0c57aed6977b1ad14646672f9b9ce385f2c98c4581267b611f48f4b7937de386ac","privateKey":"16acbf6b4f09a476a35ebd4c01e337238b5dceceb6ff55ff0c4bd83c4f91e11b"}`
  key, _ := account.GenKeyFromAccountJson(genesisAccountJson, "")
  opt := bvm.NewDIDSetChainIDOperation("chainID_01")
  payload := bvm.EncodeOperation(opt)
  tx := NewTransaction(key.(account.Key).GetAddress().Hex()).Invoke(opt.Address(), payload).VMType(BVM)
  _, err := rpc.SignAndInvokeContract(tx, key)



8.5 TRANSFER
--------------

 ::

  hrp := rpc.NewRPC()
  accountJSON, _ := account.NewAccountSm2("12345678")
  aKey, syserr := account.GenKeyFromAccountJson(accountJSON, "12345678")
  newAddress2 := aKey.(*account.SM2Key).GetAddress()
  transaction1 := rpc.NewTransaction(newAddress2.Hex()).Transfer(address, int64(0))
  transaction1.Sign(aKey)
  receipt1, err := hrp.SignAndSendTx(transaction1, aKey)


8.6 FVM
--------------

 ::

  func Demo() {
    const currentABI = `{
    "contract": {
      "name": "SetHash",
      "constructor": {
        "input": []
      }
    },
    "methods": [
      {
        "name": "set_hash",
        "input": [
          {
            "type_id": 0
          },
          {
            "type_id": 0
          }
        ],
        "output": []
      },
      {
        "name": "get_hash",
        "input": [
          {
            "type_id": 0
          }
        ],
        "output": [
          {
            "type_id": 1
          }
        ]
      }
    ],
    "types": [
      {
        "id": 0,
        "type": "primitive",
        "primitive": "str"
      },
      {
        "id": 1,
        "type": "primitive",
        "primitive": "str"
      }
    ]
  }`
    rp := rpc.NewRPCWithPath("../../conf")
    wasmPath1 := "./wasm/SetHash-gc.wasm"
    buf, err := ioutil.ReadFile(wasmPath1)
    if err != nil {
      return
    }
    guomiPri := "6153af264daa4763490f2a51c9d13417ef9f579229be2141574eb339ee9b9d2a"
    pri := new(gm.SM2PrivateKey)
    pri.FromBytes(common.FromHex(guomiPri), 0)
    guomiKey := &account.SM2Key{
      &gm.SM2PrivateKey{
        K:         pri.K,
        PublicKey: pri.CalculatePublicKey().PublicKey,
      },
    }

    transaction := rpc.NewTransaction(guomiKey.GetAddress().Hex()).Deploy(common.Bytes2Hex(buf)).VMType(rpc.FVM)
    transaction.Sign(guomiKey)
    rep, err := rp.SignAndDeployContract(transaction, guomiKey)
    if err != nil {
      return
    }
    a, err := fvm.GenAbi(strings.NewReader(currentABI))
    if err != nil {
      return
    }
    invokeInput, err := fvm.Encode(a, "set_hash", "key", "value")
    if err != nil {
      return
    }
    invokeTrans := rpc.NewTransaction(guomiKey.GetAddress().Hex()).Invoke(rep.ContractAddress, invokeInput).VMType(rpc.FVM)
    invokeTrans.Sign(guomiKey)
    _, err = rp.SignAndInvokeContract(invokeTrans, guomiKey)
    if err != nil {
      return
    }

    invokeInput2, err := fvm.Encode(a, "get_hash", "key")
    if err != nil {
      return
    }
    invokeTrans2 := rpc.NewTransaction(guomiKey.GetAddress().Hex()).Invoke(rep.ContractAddress, invokeInput2).VMType(rpc.FVM)
    invokeTrans2.Sign(guomiKey)
    recipt2, err := rp.SignAndInvokeContract(invokeTrans2, guomiKey)
    if err != nil {
      return
    }

    res, err := fvm.DecodeRet(a, "get_hash", common.Hex2Bytes(recipt2.Ret))
    if err != nil {
      return
    }
    fmt.Println(res)
  }



