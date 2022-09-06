.. _litesdk:

Litesdk
^^^^^^^^^^^^

 `完整Litesdk使用手册 <https://upload.filoop.com/RTD-Hyperchain%2FLiteSDK.zip>`_

第一章. 前言
============

**LiteSDK**\ 是一个\ **轻量JavaSDK工具**\ ，提供与Hyperchain区块链平台交互的接口以及一些处理工具。该文档⾯向Hyperchain区块链平台的应⽤开发者，提供hyperchain
Java SDK的 使⽤指南。

如需尝试SDK的demo，可将项目clone到本地，配置好Java环境(推荐使用1.8)和maven构建工具，可运行对应的发送交易和查询接口，例如合约调用的demo可在\ `此处 <https://github.com/hyperchain/javasdk/tree/master/src/test/java/cn/hyperchain/sdk>`__\ 找到，对应的资源文件和合约demo也在resource文件夹下。

同时如需更好的使用SDK来操作区块链平台发送交易，建议阅读Hyperchain区块链底层平台\ `介绍文档 <http://docs.hyperchain.cn/>`__\ ，如需详细文档介绍可联系运维人员。

同时对于EVM、HVM合约调用，也需详细阅读对应的合约介绍了解相关概念后再使用SDK进行操作。

EVM Solidity合约文档链接：https://solidity.readthedocs.io/en/latest/

HVM 合约文档链接见Hyperchain介绍文档。

第二章. 初始化
==============

2.1 创建HttpProvider对象
------------------------

``HttpProvider``\ 是一个接口，负责管理与节点的连接，实现\ ``HttpProvider``\ 接口的类需要提供底层的通信实现，目前\ **LiteSDK**\ 已有默认的实现类\ ``DefaultHttpProvider``\ ，创建\ ``DefaultHttpProvider``\ 需要通过\ **Builder**\ 模式创建，示例如下：

.. code:: java

   public static final String node1 = "localhost:8081";

   // 方式1
   HttpProvider httpProvider = new DefaultHttpProvider.Builder()
                   .setUrl(node1)
                   .https(tlsca, tls_peer_cert, tls_peer_priv)
                   .build();

   // 方式2，自定义超时时间
   HttpProvider httpProvider = new DefaultHttpProvider.Builder(10, 10, 10)
                   .setUrl(node1)
                   .https(tlsca, tls_peer_cert, tls_peer_priv)
                   .build();

-  ``Builder(int readTimeout, int writeTimeout, int connectTimeout)``\ 自定义\ **https协议**\ 的读取超时时间、写超时时间和连接超时时间，单位为s。
-  ``setUrl()``\ 可以设置连接的节点\ **URL**\ （格式为\ **ip+jsonRPC端口**\ ）;
-  ``https()``\ 设置启动\ **https协议**\ 连接并设置使用的证书(需要传的参数类型为输入流)。

2.2 创建GrpcProvider对象
------------------------

``GrpcProvider``\ 是接口\ ``HttpProvider``\ 的一个实现类，\ ``DefaultHttpProvider``\ 通过jsonrpc与节点进行通信，而\ ``GrpcProvider``\ 通过grpc双向流与节点进行通信，创建\ ``GrpcProvider``\ 需要通过Builder模式创建，示例如下：

.. code:: java

   public static final String node1 = "localhost:11001";

   // 方式一
   GrpcProvider grpcProvider = new GrpcProvider.Builder()
     							.setUrl("localhost:11001")
     							.build();

   // 方式二
   GrpcProvider grpcProvider = new GrpcProvider.Builder(3000)
     							.setUrl("localhost:11001")
     							.setStreamNum(3)
     							.build();

-  ``Builder(long time)``\ 自定义grpc向节点请求的超时时间，单位为ms。
-  ``setUrl``\ 可以设置连接的节点\ **URL**\ （格式为\ **ip+grpc**\ 端口）
-  ``setStreamNum``\ 设置本连接与节点之间同一种类型的grpc双向流的最大数量，默认值为1，通常来说其值可设置为使用该\ ``GrpcProvider``\ 的线程数量。

2.3 创建ProviderManager对象
---------------------------

每个节点的连接都需要一个\ ``HttpProvider``\ ，而\ ``ProviderManager``\ 负责集成、管理这些\ ``HttpProvider``\ ，创建\ ``ProviderManager``\ 有两种方式，一种是通过\ ``createManager()``\ 创建，另一种是和\ ``HttpProvider``\ 一样通过\ **Builder**\ 模式创建。使用前者创建会使用\ ``ProvideManager``\ 的默认配置参数，而如果想定制更多的属性则需要通过后者的方式创建，示例如下：

另外，每个节点都有一个对应的TxVersion，可通过\ ``getTxVersion(nodeId)``\ 接口获取对应节点的TxVersion，发送到节点的transaction的TxVersion必须与节点一致才能通过验签。\ ``providerManager``\ 对象在创建时会通过\ ``TxVersion.setGlobalTxVersion``\ 设置全局的TxVersion。\ ``Transaction``\ 对象也可通过\ ``setTxVersion``\ 函数设置单次交易的TxVersion。

为全局设置txVersion(不推荐修改)：

.. code:: java

   static {
       TxVersion.setGlobalTxVersion(TxVersion.TxVersion10);
   }

或者为单个交易设置txVersion：

.. code:: java

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).deploy(payload).txVersion(TxVersion.TxVersion10).build();

一般而言，使用新版本LiteSDK访问hyperchain默认txVersion为1.0，对于hyperchain2.0来说，sdk将自动识别平台txVersion，\ **不需要手动进行设置**\ 。

节点平台与TxVersion对应关系如下：

================ =============
平台使用版本     TxVersion版本
================ =============
hyperchain 1.x   1.0
hyperchain 2.0.0 2.3
================ =============

.. code:: java

   // 方式1
   ProviderManager providerManager = ProviderManager.createManager(HttpProvider);

   // 方式2
   providerManager = new ProviderManager.Builder()
                   .namespace("global")
                   .providers(httpProvider1, httpProvider2, httpProvider3, httpProvider4)
     							.grpcProviders(grpcProvider1, grpcProvider2, grpcProvider3, grpcProvider4)
                   .enableTCert(sdkcert_cert, sdkcert_priv, unique_pub, unique_priv)
                   .build();

方式1：

只需要传\ ``HttpProvider``\ 对象，其他都使用\ ``ProvideManager``\ 的默认配置，如不启用证书、使用的\ **namespace**\ 配置项为\ **global**\ 。

方式2： \* ``namespace()``\ 可以设置对应的\ **namespace名**; \*
``providers()``\ 设置需要管理的\ ``HttpProvider``\ 对象们; \*
``grpcProviders``\ 设置需要管理的\ ``GrpcProvider``\ 对象； \*
``enableTCert()``\ 设置使用的证书(**需要传的参数类型为输入流)**\ 。注：例子中未出现的方法还有一个\ ``cfca(InputStream sdkCert, InputStream sdkCertPriv)``\ ，功能与\ ``enableTCert()``\ 相同，两者的区别是证书校验是否通过\ **cfca机构**\ ，且在创建\ ``ProvideManager``\ 对象过程中两个方法只能使用其中一个。

注：enableTcert里面的sdkcert_cert，sdkcert_priv，unique_pub，unique_priv，分别对应证书目录下的sdkcert_cert，key_priv，unique_pub，unique_priv文件。

2.4 创建服务
------------

相关的一类服务集合由一个专门的\ ``Service``\ 接口管理，并通过对应的实现类实现具体的创建过程（如封装发送请求需要附带的参数）。\ **LiteSDK**\ 通过\ ``ServiceManager``\ 类负责管理创建所有的\ ``Service``\ 对象，以下是一个创建获取节点信息的服务的例子：

.. code:: java

   // 将ProviderManager对象作为参数，通过getNodeService()创建NodeService类型的对象
   // NodeService为声明的接口， 实际类型为NodeServiceImpl
   NodeService nodeService = ServiceManager.getNodeService(providerManager);

   // 通过调用NodeService提供的方法创建相应的服务，类型为Request<NodeResponse>
   NodeRequest nodeRequest = nodeService.getNodes();

实际上每个服务创建对应创建一个请求，这个请求都继承了共同的父类——``Request``\ ，\ **LiteSDK**\ 将根据不同的\ ``Service``\ 接口，返回不同\ ``Request``\ 子类，同时将用户调用接口的参数\ ``params``\ 封装到\ ``Request``\ 请求中，而在创建\ ``Request``\ 的过程中会附带一个具体的响应类型的声明，该响应类型也将根据不同的\ ``Service``\ 接口与\ ``Request``\ 绑定。

``Request``\ 拥有\ ``send()``\ 和\ ``sendAsync()``\ 同步发送和异步发送两个方法：

-  ``send()``:
   同步发送返回\ ``Request``\ 根据不同接口绑定的\ ``Response``
-  ``sendAsync()``:
   异步发送返回\ ``Request``\ 根据不同接口绑定了\ ``Response``\ 的\ ``Future``\ 接口

2.5 获取结果
------------

同样地，响应也都继承了共同的父类——``Response``\ ，通过调用\ ``Request``\ 的\ ``send()``\ 方法得到，\ **LitesSDK**\ 会将不同的返回结果\ ``result``\ 根据接口封装成不同的\ ``Response``\ 子类，如
**2.3**
所说\ ``Response``\ 类型在生成\ ``Request``\ 时绑定。\ ``Response``\ 可以获取状态码、状态消息等，而不同的\ ``Response``\ 可以获取到不同的结果，有时也需要进一步获取到更具体的信息。示例如下：

.. code:: java

   NodeResponse nodeResponse = nodeRequest.send();
   System.out.println(nodeResponse.getResult());

当\ ``ProvideManager``\ 管理多个节点连接时，返回的节点信息应该是一个数组，这时就需要调用示例中的\ ``getResult()``\ 方法将返回结果转换成更准确的类型。

第三章. 交易
============

**LiteSDK**\ 的交易接口需要用到交易体，交易体的应用场景分为两类：一类\ **是普通的转账交易，不涉及虚拟机**\ ，一类是\ **合约交易，和虚拟机相关**\ 。两者虽然都名为交易，但实际执行的功能和应用场景都不同，且转账交易的实现由\ ``TxService``\ 提供，合约交易的实现由\ ``ContractService``\ 提供。

合约接口
--------

以交易体结构为核心的交易主要应用在合约交易上，即将想要执行的操作和数据封装成一笔交易体，再调用合约服务(``ContractService``)的接口去执行。

绑定合约接口的\ ``Response``\ 子类只有\ ``TxHashResponse``\ 和\ ``ReceiptResponse``\ ，前者封装了\ ``ReceiptResponse``\ 类型的参数，实际是\ **tx
hash**\ ，拿到\ ``TxHashResponse``\ 后调用\ **polling**\ 方法可通过\ **tx
hash**\ 去查找获取真正的交易回执；后者\ ``ReceiptResponse``\ 即为交易回执，无需再调用\ **polling**\ 查询。

``TxHashResponse``\ 的主要方法如下：

.. code:: java

   /**
    * 通过交易hash获取交易回执.
    *
    * @return 返回 ReceiptResponse
    * @throws RequestException -
    */
   public ReceiptResponse polling() throws RequestException;

   /**
    * 获取交易hash.
    *
    * @return 交易hash
    */
   public String getTxHash();

LiteSDK的合约接口较特殊，交易相关的接口目前提供了\ **部署合约、调用合约、管理合约、通过投票管理合约**\ 四种接口。其中以grpc开头的接口表示该接口只有在创建\ ``ProviderManager``\ 对象时，设置了\ ``GrpcProvider``\ 与节点通信才可使用，且绑定了\ ``ReceiptResponse``\ 。

.. code:: java

   public interface ContractService {
       // 部署合约
       Request<TxHashResponse> deploy(Transaction transaction, int... nodeIds);

       // 调用合约
       Request<TxHashResponse> invoke(Transaction transaction, int... nodeIds);

       // 管理合约，包括升级，冻结，解冻
       Request<TxHashResponse> maintain(Transaction transaction, int... nodeIds);

       Request<TxHashResponse> manageContractByVote(Transaction transaction, int... nodeIds);

     	Request<ReceiptResponse> grpcDeployReturnReceipt(Transaction transaction, int... nodeIds);

     	Request<ReceiptResponse> grpcInvokeReturnReceipt(Transaction transaction, int... nodeIds);

       Request<ReceiptResponse> grpcMaintainReturnReceipt(Transaction transaction, int... nodeIds);

       Request<ReceiptResponse> grpcManageContractByVoteReturnReceipt(Transaction transaction, int... nodeIds);

   }

根据要创建的合约服务不同，封装的\ ``Transaction``\ 交易体也会不同。\ **并且LiteSDK支持HVM、EVM、BVM、FVM三种形式的合约**\ ，这几种也会影响到交易体的创建。

转账交易
--------

转账交易的实现主要是TxService提供，主要有三个接口。

.. code:: java

   Request<TxHashResponse> sendTx(Transaction transaction, int... nodeIds);

   Request<TxHashesResponse> sendBatchTxs(ArrayList<Transaction> transactions, ArrayList<String> methods, int... nodeIds);

   Request<ReceiptResponse> grpcSendTxReturnReceipt(Transaction transaction, int... nodeIds);

前两个接口分别绑定了\ ``TxHashResponse``\ 和\ ``TxHashesResponse``\ ，当拿到这两个响应时调用\ ``polling()``\ 方法就可以获取真正的交易回执，前者返回\ ``ReceiptResponse``\ ，后者返回\ ``ArrayList<ReceiptResponse>``\ 。第三个接口\ ``grpcSendTxReturnReceipt``\ 绑定了\ ``ReceiptResponse``\ ，即可以直接获得交易回执。转账交易和合约接口类似，主要的不同在于交易体的创建，转账交易通过内部类\ ``Builder``\ 调用\ ``transfer()``\ 方法创建。

.. code:: java

   class Builder {
       public Builder transfer(String to, long value);
   }

   // example:
   Transaction transaction = new Transaction.Builder(account.getAddress()).transfer("794BF01AB3D37DF2D1EA1AA4E6F4A0E988F4DEA5", 0).build();

**创建交易体并调用服务的具体流程如下。**

创建账户
~~~~~~~~

这个过程分为两步，先创建\ ``AccountService``\ 对象，再利用该对象创建账户，示例如下：

.. code:: java

   AccountService accountService = ServiceManager.getAccountService(providerManager);
   Account account = accountService.genAccount(Algo.SMRAW);

如第二章所说，创建\ ``Service``\ 对象需要指定\ ``ProviderManager``\ 对象，且使用\ ``genAccount()``\ 创建账户时需要指定加密算法，如示例中使用\ **SMRAW算法**\ （只有\ **ECRAW**\ 、\ **SMRAW**\ 、\ **ED25519RAW**\ 不需要密码参数，其余的加密算法需要手动设置\ **password**\ ）。另外，对于要使用\ **PKI算法**\ 创建的账户，需要传入其使用的\ **PFX证书**\ 的输入流以及该证书对应的密码。原因是PFX证书内包含生成其的私钥，解密该私钥需要对应密码。

``AccountService``\ 提供的接口如下：

.. code:: java

   public interface AccountService {
       Account genAccount(Algo algo);

       Account genAccount(Algo algo, String password);

       Account genAccount(Algo algo, String password, FileInputStream input);

       Account fromAccountJson(String accountJson);

       Account fromAccountJson(String accountJson, String password);

       Request<BalanceResponse> getBalance(String address, int... nodeIds);

       Request<RolesResponse> getRoles(String address, int... nodeIds);

       Request<AccountsByRoleResponse> getAccountsByRole(String role, int... nodeIds);

       Request<StatusResponse> getStatus(String address, int... nodeIds);
   }

前五个接口是用于生成账户。余下接口是查询账户相关信息，其说明如下：

-  ``getBalance``\ 方法则可以查询该账户所有的余额，需要传一个\ **合约地址**\ 为参数。
-  ``getRoles``\ 方法则可以查询该账户所有的角色，需要传一个\ **合约地址**\ 为参数。
-  ``getAccountsByRole``\ 方法则可以查询具有改角色的账户列表，需要传一个\ **角色名称**\ 为参数。
-  ``getStatus``
   方法则可以查询普通账户的状态，需要穿一个普通\ **账户地址**\ 为参数。

目前Account服务支持的所有加密算法如下：

.. code:: java

   public enum Algo {
       ECDES("0x02"),
       ECRAW("0x03"),
       ECAES("0x04"),
       EC3DES("0x05"),

       SMSM4("0x11"),
       SMDES("0x12"),
       SMRAW("0x13"),
       SMAES("0x14"),
       SM3DES("0x15"),

       ED25519DES("0x21"),
       ED25519RAW("0x22"),
       ED25519AES("0x23"),
       ED255193DES("0x24"),

       PKI("0x41");
   }

交易体创建
~~~~~~~~~~

**LiteSDK**\ 使用\ **Builder**\ 模式来负责对\ ``Transaction``\ 的创建，通过调用\ ``build()``\ 函数来获取到\ ``Transaction``\ 实例。HVM、EVM、BVM和FVM分别有各自的\ **Builder**\ ：\ ``HVMBuilder``\ 、\ ``EVMBuilder、BVMBuilder``\ 、\ ``FVMBuilder``\ ，继承同一个父类\ ``Builer``\ 。目前\ **Builder**\ 模式提供了五种交易体的封装，分别对应\ **部署合约、调用合约、升级合约、冻结合约、解冻合约**\ ，其中前两个服务的交易体分别定义在HVM、EVM、BVM、FVM各自的\ ``Builder``\ 子类中，后三者都是\ **管理合约**\ 这一服务的子服务，定义在父类\ ``Builder``\ 中。

.. code:: java

   class Builder {
       Builder upgrade(String contractAddress, String payload);
       Builder freeze(String contractAddress);
       Builder unfreeze(String contractAddress);
       Transaction build();
   }

   class HVMBuilder extends Builder {
       Builder deploy(InputStream fis);
       Builder invoke(String contractAddress, BaseInvoke baseInvoke);
   }

   class EVMBuilder extends Builder {
       // 当合约无构造参数时使用，不需abi参数
       Builder deploy(String bin);
       // 当合约需要提供abi解析构造方法参数时使用
       Builder deploy(String bin, Abi abi, FuncParams params);
       Builder invoke(String contractAddress, String methodName, Abi abi, FuncParams params);
   }

   class BVMBuilder extends Builder {
       Builder invoke(BuiltinOperation opt)
   }

下面是创建各个服务的交易体\ ``Transaction``\ 的实例。

部署合约
~~~~~~~~

HVM
------

.. code:: java

   InputStream payload = FileUtil.readFileAsStream("hvm-jar/hvmbasic-1.0.0-student.jar");

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).deploy(payload).build();

创建交易体时需要指定要\ **部署的jar包(封装成流)**\ 。

EVM
-----

.. code:: java

   InputStream inputStream1 = FileUtil.readFileAsStream("solidity/sol2/TestContract_sol_TypeTestContract.bin");
   InputStream inputStream2 = FileUtil.readFileAsStream("solidity/sol2/TestContract_sol_TypeTestContract.abi");
   String bin = FileUtil.readFile(inputStream1);
   String abiStr = FileUtil.readFile(inputStream2);
   Abi abi = Abi.fromJson(abiStr);

   FuncParams params = new FuncParams();
   params.addParams("contract01");
   Transaction transaction = new Transaction.EVMBuilder(account.getAddress()).deploy(bin, abi, params).build();
   // 如果要部署的合约无构造函数，则调用如下
   // Transaction transaction = new Transaction.EVMBuilder(account.getAddress()).deploy(bin).build();

创建交易体时需要指定要\ **部署的合约的bin、abi文件的字符串内容以及合约名**\ 。

FVM
-----

待部署的合约构造函数不带参数使用方式如下

.. code:: java

   InputStream inputStream1 = Thread.currentThread().getContextClassLoader().getResourceAsStream("fvm-contract/set_hash/SetHash-gc.wasm");
   Transaction transaction = new Transaction.FVMBuilder(account.getAddress()).deploy(inputStream1).build();

如果待部署的合约构造函数有参数那么使用方式如下

.. code:: java

   FVMAbi abi = FVMAbi.fromJson(abiStr);
   FuncParams params = new FuncParams();
   params.addParams(2);
   Transaction transaction = new Transaction.FVMBuilder(account.getAddress()).deploy(inputStream1, abi, params).build();

创建交易体时需要指定要\ **部署的合约的wasm文件**

调用合约
~~~~~~~~

.. _hvm-1:

HVM
-----

hvm调用合约有四种方式：

-  **InvokeBean**\ 调用
-  直接调用合约方法（类似evm）
-  通过hvm-abi文件调用
-  通过hvm-abi文件进行并行合约调用

1. InvokeBean调用如下：

.. code:: java

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).invoke(receiptResponse.getContractAddress(), invoke).build();

创建交易体时需要指定\ **合约地址**\ 和\ **InvokeBean**\ (HVM中新提出的概念，请先通过HVM文档了解)。

2. 直接调用合约方法如下：

.. code:: java

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).invokeDirectly(receiptResponse.getContractAddress(), params).build();

params类型为\ ``InvokeDirectlyParams``\ ，具体的构造方式见附录。

3. 通过hvm.abi文件调用合约

.. code:: java

   InputStream inputStream = FileUtil.readFileAsStream("hvm-abi/hvm.abi");
   String abiJson = FileUtil.readFile(inputStream);
   //通过invokeBean调用
   InvokeHVMAbiParams.ParamBuilder params = new InvokeHVMAbiParams.ParamBuilder(abiJson,  HVMBeanAbi.BeanType.InvokeBean,"invoke.InvokeBid");
   params.addParam(100);
   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).invokeByBeanAbi(contractAddress, params.build()).build();

   // MethodBean 通过methodBean调用
   InvokeHVMAbiParams.ParamBuilder params = new InvokeHVMAbiParams.ParamBuilder(abiJson,  HVMBeanAbi.BeanType.MethodBean,"bid");
   params.addParam(100);
   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).invokeByBeanAbi(contractAddress, params.build()).build();

4. 通过hvm.abi文件进行合约并行调用

hvm支持区块间的hvm合约调用并行执行，hvm合约并行调用的具体内容请参考HVM使用手册，这里只给出使用示例。

.. code:: java

   // 事先读取abi文件
   InputStream inputStream = FileUtil.readFileAsStream("hvm-abi/parallel-nestedMap/parallel-contract.abi");
   String abiJson = FileUtil.readFile(inputStream);
   HVMAbi hvmAbi = new HVMAbi(abiJson);
   // 创建HVMAbiGetter的一个实例，传入调用合约的abi
   HVMAbiMap hvmAbiMap = new HVMAbiMap();
   hvmAbiMap.setHVMAbi(contractAddress, hvmAbi);
   //  基于InvokeHVMAbiParams构建用于并行调用的payload，其中需要将isParallel设置为true
   InvokeHVMAbiParams.ParamBuilder builder = new InvokeHVMAbiParams.ParamBuilder(hvmAbiMap, contractAddress, HVMBeanAbi.BeanType.MethodBean, "transfer", true);
   // 传入指定abiBean的参数，构建交易
   InvokeHVMAbiParams params = builder.addParam("k1").addParam("k2").addParam(100).build();
   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).invokeByBeanAbi(contractAddress, params).build();

.. _evm-1:

EVM
-----

.. code:: java

   FuncParams params = new FuncParams();
   params.addParams("10".getBytes());
   Transaction transaction = new Transaction.EVMBuilder(account.getAddress()).invoke(contractAddress, "TestBytes32(bytes1)", abi, params).build();

创建交易体时需要指定\ **调用方法**\ 、\ **abi文件**\ 和\ **方法参数**\ 。

.. _fvm-1:

FVM
-----

.. code:: java

   InputStream inputStream2 = Thread.currentThread().getContextClassLoader().getResourceAsStream("fvm-contract/set_hash/contract.json");
   String abiStr = FileUtil.readFile(inputStream2);
   FVMAbi abi = FVMAbi.fromJson(abiStr);
   FuncParams params = new FuncParams();
   params.addParams("key001");
   params.addParams("this is the value of 0001");
   Transaction transaction1 = new Transaction.FVMBuilder(account.getAddress()).invoke(contractAddress, "set_hash", abi, params).build();

创建交易体时需要指定\ **调用方法，abi以及参数**

升级合约
~~~~~~~~

升级合约使用ContractService的maintain接口。

.. _hvm-2:

HVM
-----

.. code:: java

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).upgrade(contractAddress, payload).build();

创建交易体时需要指定\ **合约地址**\ 和\ **读取新合约jar包得到的字符串**

.. _evm-2:

EVM
-----

.. code:: java

   Transaction transaction = new Transaction.EVMBuilder(account.getAddress()).upgrade(contractAddress, payload).build();

创建交易体时需要指定\ **合约地址**\ 和\ **升级的新合约的bin文件字符串**\ 。

.. _fvm-2:

FVM
-----

.. code:: java

   Transaction transaction3 = new Transaction.FVMBuilder(account.getAddress()).upgrade(contractAddress, Encoder.encodeDeployWasm(inputStream3)).build();

创建交易体时需要指定\ **合约地址**\ 和\ **升级的新合约的wasm文件**\ 。

冻结合约
~~~~~~~~

冻结合约使用ContractService的maintain接口。

.. _hvm-3:

HVM
-----

.. code:: java

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).freeze(contractAddress).build();

创建交易体时需要指定\ **合约地址**\ 。

.. _evm-3:

EVM
-----

.. code:: java

   Transaction transaction = new Transaction.EVMBuilder(account.getAddress()).freeze(contractAddress).build();

创建交易体时需要指定\ **合约地址**\ 。

.. _fvm-3:

FVM
-----

.. code:: java

   Transaction transaction = new Transaction.FVMBuilder(account.getAddress()).freeze(contractAddress).build();

创建交易体时需要指定\ **合约地址**\ 。

解冻合约
~~~~~~~~

解冻合约使用ContractService的maintain接口。

.. _hvm-4:

HVM
-----

.. code:: java

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).unfreeze(contractAddress).build();

创建交易体时需要指定\ **合约地址**\ 。

.. _evm-4:

EVM
-----

.. code:: java

   Transaction transaction = new Transaction.EVMBuilder(account.getAddress()).unfreeze(contractAddress).build();

创建交易体时需要指定\ **合约地址**\ 。

.. _fvm-4:

FVM
-----

.. code:: java

   Transaction transaction = new Transaction.FVMBuilder(account.getAddress()).unfreeze(contractAddress).build();

创建交易体时需要指定\ **合约地址**\ 。

simulate交易
------------

simulate交易执行不会更改区块链的账本状态，这是其和普通交易最大的不同。在使用方式上，两者没有什么区别，只需要在构建simulate交易时设置其\ ``simulate``\ 标志，即可如发送普通交易一样，发送simulate交易。

构建simulate交易示例如下

.. code:: java

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress())
     .invoke(receiptResponse.getContractAddress(), invoke)
     .simulate()
     .build();

交易体的payload
---------------

在创建交易体时，会根据传入的参数生成payload。如果是HVM合约相关的\ ``transaction``\ ，可通过\ ``Decoder``\ 提供的\ ``decodeHVMPayload(String payload)``\ 方法对payload进行解析，返回\ ``HVMPayload``\ 对象。

.. code:: java

   HVMPayload decodeHVMPayload(String payload)

``HVMPayload``\ 结构如下：

.. code:: java

   public class HVMPayload {
       private String invokeBeanName;
       private String invokeArgs;
       private Set<String> invokeMethods;
   }

其中\ ``invokeBeanName``\ 为调用的HVM合约的名字，\ ``invokeArgs``\ 为调用的参数，\ ``invokeMethods``\ 为调用的合约方法。

如果是BVM合约相关的\ ``transaction``\ ，可通过\ ``Decoder``\ 提供的\ ``decodeBVMPayload(String payload)``\ 方法对payload进行解析，返回\ ``Operation``\ 对象，从\ ``Operation``\ 中可以获取BVM交易调用的内置合约方法(methodName)和参数(args)。

交易体设置TxVersion
-------------------

``transaction``\ 对象在创建时，其TxVersion属性值默认为全局的TxVersion，也可通过\ ``setTxVersion``\ 函数来设置交易体的TxVersion属性。

.. code:: java

   Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).deploy(payload).txVersion(TxVersion.TxVersion10).build();

交易体签名
----------

通过\ ``Transaction``\ 提供的\ ``sign()``\ 方法，需要指定\ ``Account``\ 对象。

.. code:: java

   transaction.sign(account);

创建请求
--------

这个过程分为两步，先创建\ ``ContractService``\ 对象，再指定之前构造的交易体调用相应的服务接口，示例如下：

.. code:: java

   ContractService contractService = ServiceManager.getContractService(providerManager);

   //方式一
   Request<TxHashResponse> contractRequest = contractService.deploy(transaction);

   //方式二
   Request<ReceiptResponse> contractGrpcRequest = contractService.grpcDeployReturnReceipt(transaction);

发送交易体
----------

如果创建请求调用的是普通接口，不是\ **grpc**\ 的服务接口，那么这个过程实际分为两步，调用\ ``send()``\ 部署合约拿到响应，再对响应解析拿到\ ``ReceiptResponse``\ （执行结果）。如果创建请求调用的是\ **grpc**\ 服务接口，只需要调用\ ``send()``\ 方法拿到\ ``ReceiptResponse``\ 响应就结束了。

.. code:: java

   //方式一
   ReceiptResponse receiptResponse = contractRequest.send().polling();

   //方式二
   ReceiptResponse receiptResponse2 = contractGrpcRequest.send();

合约辅助接口
------------

LiteSDK除了提供上述与合约交易相关的接口，还提供了以下编译合约、获取合约状态等查询接口，其响应类型如下：

-  CompileContractResponse
-  StringResponse
-  DeployerListResponse

**CompileContractResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是内部类\ ``CompileCode``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class CompileContractResponse extends Response {
     	private CompileCode result;
   		public class CompileCode {

           private List<String> abi;

           private List<String> bin;

           private List<String> types;
       }
   }

**StringResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是String，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class StringResponse extends Response {

       private String result;
   }

**DeployerListResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是List，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class DeployerListResponse extends Response {

       private List<String> result;
   }

以下为合约的相关查询接口

.. code:: java

   public interface ContractService {

       Request<CompileContractResponse> compileContract(String code, int... nodeIds);

       Request<StringResponse> getCode(String addr, int... nodeIds);

       Request<StringResponse> getContractCountByAddr(String addr, int...nodeIds);

       Request<DeployerListResponse> getDeployedList(String address, int... nodeIds);

       Request<StringResponse> getStatus(String addr, int...nodeIds);

       Request<StringResponse> getCreator(String addr, int...nodeIds);

       Request<StringResponse> getCreateTime(String addr, int...nodeIds);

       Request<StringResponse> getStatusByCName(String cname, int...nodeIds);

       Request<StringResponse> getCreatorByCName(String cname, int...nodeIds);

       Request<StringResponse> getCreateTimeByCName(String cname, int...nodeIds);

   }

编译Solidity合约
~~~~~~~~~~~~~~~~

参数：

-  code solidity合约源码
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<CompileContractResponse> compileContract(String code, int... nodeIds);

获取合约源码
~~~~~~~~~~~~

参数：

-  addr 合约地址
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<StringResponse> getCode(String addr, int... nodeIds);

获取账户部署的合约数量
~~~~~~~~~~~~~~~~~~~~~~

参数：

-  addr 账户地址
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<StringResponse> getContractCountByAddr(String addr, int...nodeIds);

获取账户部署的合约地址列表
~~~~~~~~~~~~~~~~~~~~~~~~~~

参数：

-  address 账户地址
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<DeployerListResponse> getDeployedList(String address, int... nodeIds);

获取合约状态
~~~~~~~~~~~~

参数：

-  addr 合约地址
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<StringResponse> getStatus(String addr, int...nodeIds);

获取合约的部署账户
~~~~~~~~~~~~~~~~~~

参数：

-  addr 合约地址
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<StringResponse> getCreator(String addr, int...nodeIds);

获取合约的部署时间
~~~~~~~~~~~~~~~~~~

参数：

-  addr 合约地址
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<StringResponse> getCreateTime(String addr, int...nodeIds);

获取合约状态by cname
~~~~~~~~~~~~~~~~~~~~

参数：

-  cname 合约名
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<StringResponse> getStatusByCName(String cname, int...nodeIds);

获取合约的部署账户by cname
~~~~~~~~~~~~~~~~~~~~~~~~~~

参数：

-  cname 合约名
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<StringResponse> getCreatorByCName(String cname, int...nodeIds);

获取合约的部署时间by cname
~~~~~~~~~~~~~~~~~~~~~~~~~~

参数：

-  cname 合约名
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<StringResponse> getCreateTimeByCName(String cname, int...nodeIds);

第四章. Transaction接口(TxService)
==================================

**注：该章的Transaction与第三章的交易体概念不同，该章的接口主要主要用于查询之前在链上的执行信息，将返回的信息封装为Transaction结构体。**

TxService接口繁多，返回的执行结果根据情况封装共对应五种响应：

-  TxResponse
-  TxCountWithTSResponse
-  TxCountResponse
-  TxAvgTimeResponse
-  ReceiptListResponse

详细结构请参考第十一章

4.1 查询交易by transaction hash(getTransactionByHash)
-----------------------------------------------------

参数： \* txHash 交易hash \* nodeIds 请求向哪些节点发送

.. code:: java

   Request<TxResponse> getTxByHash(String txHash, int... nodeIds);

4.2 查询交易by block hash(getTxByBlockHashAndIndex)
---------------------------------------------------

参数：

-  blockHash 区块哈希值
-  index 区块内的交易索引值
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<TxResponse> getTxByBlockHashAndIndex(String blockHash, int index, int... nodeIds);

4.3 查询交易by block number(getTxByBlockNumAndIndex)
----------------------------------------------------

参数：

-  blockNumber 区块号
-  index 区块内的交易索引值
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<TxResponse> getTxByBlockNumAndIndex(int blockNumber, int idx, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxResponse> getTxByBlockNumAndIndex(String blockNumber, String idx, int... nodeIds);

4.4 查询交易by time with limit(getTransactionsByTimeWithLimit)
--------------------------------------------------------------

参数：

-  startTime 起始时间戳
-  endTime 结束时间戳
-  metaData 分页相关参数
-  filter 交易过滤条件
-  nodeIds 请求向哪些节点发送

.. code:: java

   Request<TxLimitResponse> getTransactionsByTimeWithLimit(BigInteger startTime, BigInteger endTime, MetaDataParam metaData, FilterParam filter, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxLimitResponse> getTransactionsByTimeWithLimit(BigInteger startTime, BigInteger endTime, MetaDataParam metaData, int... nodeIds);

   Request<TxLimitResponse> getTransactionsByTimeWithLimit(String startTime, String endTime, MetaDataParam metaData, FilterParam filter, int... nodeIds);

   Request<TxLimitResponse> getTransactionsByTimeWithLimit(String startTime, String endTime, MetaDataParam metaData, int... nodeIds);

   Request<TxLimitResponse> getTransactionsByTimeWithLimit(BigInteger startTime, BigInteger endTime, int... nodeIds);

   Request<TxLimitResponse> getTransactionsByTimeWithLimit(String startTime, String endTime, int... nodeIds);

4.5 查询交易 with limit(getTxsWithLimit)
----------------------------------------

参数：

-  from 区块区间起点
-  to 区块区间终点
-  metaData 分页相关参数
-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<TxLimitResponse> getTxsWithLimit(String from, String to, MetaDataParam metaData, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxLimitResponse> getTxsWithLimit(String from, String to, int... nodeIds);

4.6 查询指定区块区间交易平均处理时间(getTxAvgTimeByBlockNumber)
---------------------------------------------------------------

参数：

-  from 区块区间起点
-  to 区块区间终点
-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<TxAvgTimeResponse> getTxAvgTimeByBlockNumber(BigInteger from, BigInteger to, int... nodeIds);

重载方法如下:

.. code:: java

   Request<TxAvgTimeResponse> getTxAvgTimeByBlockNumber(String from, String to, int... nodeIds);

4.7 查询链上所有交易量(getTransactionsCount)
--------------------------------------------

参数：

-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxCountWithTSResponse> getTransactionsCount(int... nodeIds);

4.8 查询交易回执信息by transaction hash(getTransactionReceipt)
--------------------------------------------------------------

参数：

-  txHash 交易hash。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<ReceiptResponse> getTransactionReceipt(String txHash, int... nodeIds);

4.9 查询上链的交易回执信息(getConfirmedTransactionReceipt)
----------------------------------------------------------

参数：

-  txHash 交易hash。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<ReceiptResponse> getConfirmedTransactionReceipt(String txHash, int... nodeIds);

4.10 查询区块交易数量by block hash(getBlockTxCountByHash)
---------------------------------------------------------

参数：

-  blockHash 区块哈希值
-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<TxCountWithTSResponse> getBlockTxCountByHash(String blockHash, int... nodeIds);

4.11 查询区块交易数量by block number(getBlockTxCountByNumber)
-------------------------------------------------------------

参数：

-  blockNumber 区块号。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxCountWithTSResponse> getBlockTxCountByNumber(String blockNumber, int... nodeIds);

4.12 查询平台当前的交易版本号(getTxVersion)
-------------------------------------------

getTxVersion接口会在创建ProviderManager对象时调用，并设置全局的TxVersion。

参数：

-  nodeId 说明请求哪个节点平台的交易版本号

.. code:: java

   Request<TxVersionResponse> getTxVersion(int nodeId) throws RequestException;

4.13 查询链上所有非法交易交易量(getInvalidTransactionsCount)
------------------------------------------------------------

参数：

-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxCountWithTSResponse> getInvalidTransactionsCount(int... nodeIds);

4.14 查询链上指定时间段内的非法交易交易量(getInvalidTxsCountByTime)
-------------------------------------------------------------------

参数：

-  startTime 开始时间
-  endTime 截止时间
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxCountResponse> getInvalidTxsCountByTime(BigInteger startTime, BigInteger endTime, int... nodeIds);

4.15 查询一个区块中的所有非法交易 by block number(getInvalidTxsByBlockNumber)
-----------------------------------------------------------------------------

参数：

-  blockNumber 区块号
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxResponse> getInvalidTxsByBlockNumber(BigInteger blockNumber, int... nodeIds);

4.16 查询一个区块中的所有非法交易 by block hash(getInvalidTxsByBlockHash)
-------------------------------------------------------------------------

参数：

-  blockHash 区块哈希
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxResponse> getInvalidTxsByBlockHash(String blockHash, int... nodeIds);

4.17 查询非法交易 with limit(getInvalidTxsWithLimit)
----------------------------------------------------

参数：

-  from 区块区间起点
-  to 区块区间终点
-  metaData 分页相关参数
-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<TxLimitResponse> getInvalidTxsWithLimit(String from, String to, MetaDataParam metaData, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxLimitResponse> getInvalidTxsWithLimit(String from, String to, int... nodeIds);

   Request<TxLimitResponse> getInvalidTxsWithLimit(Integer from, Integer to, int... nodeIds);

4.18 查询下一页交易(getNextPageTransactions)
--------------------------------------------

注意：当输入的区块范围较大并且这个范围内符合条件的交易数量非常大时，\ **请求响应延迟将非常高**\ 。存在服务器资源被该请求处理长时间占用的风险，应尽量
避免使用。

参数：

-  blkNumber 从该区块开始计数。
-  txIndex 起始交易在blkNumber号区块的位置偏移量。
-  minBlkNumber 截止计数的最小区块号。
-  maxBlkNumber 截止计数的最大区块号。
-  separated 表示要跳过的交易条数（一般用于跳页查询）。
-  pageSize 表示要返回的交易条数。
-  containCurrent
   true表示返回的结果中包括blkNumber区块中位置为txIndex的交易，如果该条交易不是合约地址为address合约的交易，则不算入。
-  address 合约地址。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxResponse> getNextPageTransactions(BigInteger blkNumber, BigInteger txIndex, BigInteger minBlkNumber, BigInteger maxBlkNumber, BigInteger separated, BigInteger pageSize, boolean containCurrent, String address, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxResponse> getNextPageTransactions(String blkNumber, String txIndex, String minBlkNumber, String maxBlkNumber, String separated, String pageSize, boolean containCurrent, String address, int... nodeIds);

4.19 查询上一页交易(getPrevPageTransactions)
--------------------------------------------

注意：当输入的区块范围较大并且这个范围内符合条件的交易数量非常大时，\ **请求响应延迟将非常高**\ 。存在服务器资源被该请求处理长时间占用的风险，应尽量
避免使用。

参数：

-  blkNumber 从该区块开始计数。
-  txIndex 起始交易在blkNumber号区块的位置偏移量。
-  minBlkNumber 截止计数的最小区块号。
-  maxBlkNumber 截止计数的最大区块号。
-  separated 表示要跳过的交易条数（一般用于跳页查询）。
-  pageSize 表示要返回的交易条数。
-  containCurrent
   true表示返回的结果中包括blkNumber区块中位置为txIndex的交易，如果该条交易不是合约地址为address合约的交易，则不算入。
-  address 合约地址。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxResponse> getPrevPageTransactions(BigInteger blkNumber, BigInteger txIndex, BigInteger minBlkNumber, BigInteger maxBlkNumber, BigInteger separated, BigInteger pageSize, boolean containCurrent, String address, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxResponse> getPrevPageTransactions(String blkNumber, String txIndex, String minBlkNumber, String maxBlkNumber, String separated, String pageSize, boolean containCurrent, String address, int... nodeIds);

4.20 查询下一页非法交易(getNextPageInvalidTransactions)
-------------------------------------------------------

注意：当输入的区块范围较大并且这个范围内符合条件的交易数量非常大时，\ **请求响应延迟将非常高**\ 。存在服务器资源被该请求处理长时间占用的风险，应尽量
避免使用。

参数：

-  blkNumber 从该区块开始计数。
-  txIndex 起始交易在blkNumber号区块的位置偏移量。
-  minBlkNumber 截止计数的最小区块号。
-  maxBlkNumber 截止计数的最大区块号。
-  separated 表示要跳过的交易条数（一般用于跳页查询）。
-  pageSize 表示要返回的交易条数。
-  containCurrent
   true表示返回的结果中包括blkNumber区块中位置为txIndex的交易，如果该条交易不是合约地址为address合约的交易，则不算入。
-  address 合约地址。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxResponse> getNextPageInvalidTransactions(String blkNumber, String txIndex, String minBlkNumber, String maxBlkNumber, String separated, String pageSize, boolean containCurrent, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxResponse> getNextPageInvalidTransactions(BigInteger blkNumber, BigInteger txIndex, BigInteger minBlkNumber, BigInteger maxBlkNumber, BigInteger separated, BigInteger pageSize, boolean containCurrent, int... nodeIds);

4.21 查询上一页非法交易(getPrevPageInvalidTransactions)
-------------------------------------------------------

注意：当输入的区块范围较大并且这个范围内符合条件的交易数量非常大时，\ **请求响应延迟将非常高**\ 。存在服务器资源被该请求处理长时间占用的风险，应尽量
避免使用。

参数：

-  blkNumber 从该区块开始计数。
-  txIndex 起始交易在blkNumber号区块的位置偏移量。
-  minBlkNumber 截止计数的最小区块号。
-  maxBlkNumber 截止计数的最大区块号。
-  separated 表示要跳过的交易条数（一般用于跳页查询）。
-  pageSize 表示要返回的交易条数。
-  containCurrent
   true表示返回的结果中包括blkNumber区块中位置为txIndex的交易，如果该条交易不是合约地址为address合约的交易，则不算入。
-  address 合约地址。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxResponse> getPrevPageInvalidTransactions(String blkNumber, String txIndex, String minBlkNumber, String maxBlkNumber, String separated, String pageSize, boolean containCurrent, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxResponse> getPrevPageInvalidTransactions(BigInteger blkNumber, BigInteger txIndex, BigInteger minBlkNumber, BigInteger maxBlkNumber, BigInteger separated, BigInteger pageSize, boolean containCurrent, int... nodeIds);

4.22 查询区块区间交易数量by contract address(getTransactionsCountByContractAddr)
--------------------------------------------------------------------------------

注意：当输入的区块范围较大并且这个范围内符合条件的交易数量非常大时，\ **请求响应延迟将非常高**\ 。存在服务器资源被该请求处理长时间占用的风险，应尽量避免使用。

参数：

-  from 起始区块号。
-  to 终止区块号。
-  address 合约地址。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxCountResponse> getTransactionsCountByContractAddr(String from, String to, String address, int... nodeIds);

重载方法如下：

.. code:: java

   Request<TxCountResponse> getTransactionsCountByContractAddr(BigInteger from, BigInteger to, String address, int... nodeIds);

4.23 查询指定时间区间内的交易数量(getTxsCountByTime)
----------------------------------------------------

注意：当输入的时间范围较大并且这个范围内的区块较多时，\ **请求响应延迟将升高**\ 。存在服务器资源被该请求处理长时间占用的风险，应尽量避免使用。

参数：

-  startTime 起起始时间戳(单位ns)。
-  endTime 结束时间戳(单位ns)。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxResponse> getTxsCountByTime(BigInteger startTime, BigInteger endTime, int... nodeIds);

4.24 查询指定extraID的交易by extraID(getTxsByExtraID)
-----------------------------------------------------

该接口只要在访问的节点开启数据索引功能时才可用。

参数：

-  mode [可选]
   表示本次查询请求的查询模式，目前有0、1、2三个值可选，默认为0。0
   表示按序精确查询模式，即筛选出的的交易 extraId
   数组的数值和顺序都与查询条件完全一致。1
   表示非按序精确查询模式，即筛选出的交易 extraId
   数组包含查询条件里指定的全部数值，顺序无要求。2
   表示非按序匹配查询模式，即筛选出的交易 extraId
   数组包含部分或全部查询条件指定的值，且顺序无要求。。
-  detail [可选] 是否返回详细的交易内容，默认为false。
-  metaData [可选]
   分页相关参数。指定本次查询的起始位置、查询方向以及返回的条数。若未指定，则默认从最新区块开始向前查询，默认返回条数上限是5000条。
-  filter [必选] 指定本次查询过滤条件。包括交易extraId和交易接收方地址。
-  nodeIds 说明请求向哪些节点发送。

MetaDataParam 结构如下：

-  pagesize [可选] 表示本次查询返回多少
   条交易。如果未指定，则pagesize默认
   值为5000，如果超过5000，则使用节点默认值5000。如果符合条件的交易数量实际上超过pagesize，则返回结果里hasmore为true。
-  bookmark [可选]
   表示本次查询的书签位置，即起始位置，返回的结果里不包含用户指定的书签所对应的交易。如果未指定且backward为false，则默认从最新区块开始向前遍历，如果未指定且backward为true，则默认从创世区块开始向后遍历。
-  backward [可选]
   表示本次查询的方向，false表示以起始位置为起点从高区块往低区块遍历，true表示以起
   始位 置为起点从低区块往高区块遍历，默认查询方向为false。

Bookmark 结构如下：

-  blkNum 交易所在区块号。
-  txIndex 交易索引号，即交易在区块内的位置。

FilterParam 结构如下：

-  extraId [必选]
   指定交易extraId的值，类型为数组，数组元素可以为Long或者string。
-  txTo [可选] 指定交易接收方的地址。

客户端可以利用该接口实现区块链的“分页查询”，根据返回结果里的hasmore来判断是否要继续查询剩下的数据。下面对该接口的参数做进一步说明：

如果查询条件未指定 metadata，则 metadata.backward 默认为
false、书签位置默认为最新区块最后一条交易，从书签位置开始往前遍历，limit默认为5000条。

如果查询条件里未指定 metadata.bookmark，若 metadata.backward 为
false，则默认书签位置为最新区块的最后一条交易，若metadata.backward 为
true，则默认书签位置为第一个区块的第一条交易。

如果查询条件里指定的书签位置 metadata.bookmark 位于区块区间 [1, latest]
里，则我们需要根据 metadata.backward 的值来调整遍历的区块区间。如果
metadata.backward 为false，则区块区间调整为 [1,
metadata.bookmark.blkNum]，如果 metadata.backward
为true，则区块区间调整为 [metadata.bookmark.blkNum, latest]。

当 backward 为 false 的时
候，如果指定的书签位置在区块1之前，则接口返回error。当 backward 为 true
的时候，如果指定的书签位置在最新区块之后，则接口返回error。

.. code:: java

   Request<TxLimitResponse> getTxsByExtraID(int mode, boolean detail, MetaDataParam metaData, FilterParam filter, int... nodeIds);

4.25 查询指定filter的交易by filter(getTxsByFilter)
--------------------------------------------------

该接口只要在访问的节点开启数据索引功能时才可用。

参数：

-  mode [可选] 表示本次查询请求的查询模式，目前有 0、1
   两个值可选，默认为0。0
   表示多条件与查询模式，即交易对应字段的值与查询条件里所有指定的字段值都完全一致。1
   表示多条件或询模式，即交易对应字段的值至少有一个等于查询条件里指定的字段值。
-  detail [可选] 是否返回详细的交易内容，默认为false。
-  metaData [可选]
   指定本次查询的起始位置、查询方向以及返回的条数。若未指定，则默认从最新区块开始向前查询，默认返回条数上限是5000条。
-  filter [必选] 指定本次查询过滤条件。
-  nodeIds 说明请求向哪些节点发送。

MetaDataParam 结构如下：

-  pagesize [可选] 表示本次查询返回多少
   条交易。如果未指定，则pagesize默认
   值为5000，如果超过5000，则使用节点默认值5000。如果符合条件的交易数量实际上超过pagesize，则返回结果里hasmore为true。
-  bookmark [可选]
   表示本次查询的书签位置，即起始位置，返回的结果里不包含用户指定的书签所对应的交易。如果未指定且backward为false，则默认从最新区块开始向前遍历，如果未指定且backward为true，则默认从创世区块开始向后遍历。
-  backward [可选]
   表示本次查询的方向，false表示以起始位置为起点从高区块往低区块遍历，true表示以起
   始位 置为起点从低区块往高区块遍历，默认查询方向为false。

Bookmark 结构如下：

-  blkNum 交易所在区块号。
-  txIndex 交易索引号，即交易在区块内的位置。

FilterParam 结构如下：

-  txHash [可选] 指定交易的哈希值。
-  blkNumber [可选] 指定交易所在的区块号。
-  txIndex [可选] 指定交易在区块内的索引位置。
-  txFrom [可选] 指定交易发送方的地址。
-  txTo [可选] 指定交易接收方的地址。
-  extraId [可选]
   指定交易extraId的值，类型为数组，数组元素可以为long或者String。

客户端可以利用该接口实现区块链的“分页查询”，根据返回结果里的 hasmore
来判断是否要继续查询剩下的数据。下面对该接口的参数做进一步说明：

如果查询条件未指定 metadata，则 metadata.backward 默认为
false、书签位置默认为最新区块最后一条交易，从书签位置开始往前遍历，limit默认为5000条。

如果查询条件里未指定 metadata.bookmark，若 metadata.backward 为
false，则默认书签位置为最新区块的最后一条交易，若 metadata.backward 为
true，则默认书签位置为第一个区块的第一条交易。

如果查询条件里指定的书签位置 metadata.bookmark 位于区块区间 [1, latest]
里， 则我们需要根据 metadata.backward 的值来调整遍历的区块区间。如果
metadata.backward 为false，则区块区间调整为 [1,
metadata.bookmark.blkNum]，如果 metadata.backward
为true，则区块区间调整为 [metadata.bookmark.blkNum, latest]。

当 backward 为 false
的时候，如果指定的书签位置在区块1之前，则接口返回error。当 backward 为
true 的时候，如果指定的书签位置在最新区块之后，则接口返回error。

.. code:: java

   Request<TxLimitResponse> getTxsByFilter(int mode, boolean detail, MetaDataParam metaData, FilterParam filter, int... nodeIds);

4.26 查询批量交易by hash list(getBatchTxByHash)
-----------------------------------------------

注意：当输入的交易哈希非常多时，\ **请求响应延迟将升高**\ 。如果返回的数据量超过节点所在服务器内存大小时，将导致处理查询请求的节点出现\ **OOM（Out
Of Memory）**\ 风险，可使用 **tx_getTransactionByHash** 接口替代。

参数：

-  txHashList 交易的哈希数组, 哈希值为32字节的十六进制字符串。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<TxResponse> getBatchTxByHash(ArrayList<String> txHashList, int... nodeIds);

4.27 查询批量回执by hash list(getBatchReceipt)
----------------------------------------------

注意：当输入的交易哈希非常多时，\ **请求响应延迟将升高**\ 。如果返回的数据量超过节点所在服务器内存大小时，将导致处理查询请求的节点出现\ **OOM（Out
Of Memory）**\ 风险，可使用 **tx_getTransactionReceipt** 接口替代。

参数：

-  txHashList 交易的哈希数组, 哈希值为32字节的十六进制字符串。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<ReceiptListResponse> getBatchReceipt(ArrayList<String> txHashList, int... nodeIds);

第五章. BlockService相关接口
============================

BlockService接口与TxService相似，只是获取的对象是区块信息。同样地，BlockService对象也有很多对应的响应类型：

-  BlockResponse
-  BlockNumberResponse
-  BlockAvgTimeResponse
-  BlockCountResponse

详细结构请参考第十章。

5.1 获取最新区块(getLastestBlock)
---------------------------------

参数：

-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockResponse> getLastestBlock(int... nodeIds);

5.2 查询区块by block hash(getBlockByHash)
-----------------------------------------

参数：

-  blockHash 区块的哈希值,32字节的十六进制字符串。
-  isPlain (可选)
   默认为false，表示返回的区块\ **包括**\ 区块内的交易信息，如果指定为true，表示返回的区块\ **不包括**\ 区块内的交易。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockResponse> getBlockByHash(String blockHash, int... nodeIds);

   Request<BlockResponse> getBlockByHash(String blockHash, boolean isPlain, int... nodeIds);

5.3 查询区块by block number(getBlockByNum)
------------------------------------------

参数：

-  blockNumber 区块号。
-  isPlain (可选)
   默认为false，表示返回的区块\ **包括**\ 区块内的交易信息，如果指定为true，表示返回的区块\ **不包括**\ 区块内的交易。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockResponse> getBlockByNum(BigInteger blockNumber, int... nodeIds);

   Request<BlockResponse> getBlockByNum(BigInteger blockNumber, boolean isPlain, int... nodeIds);

重载方法如下：

.. code:: java

   Request<BlockResponse> getBlockByNum(String blockNumber, int... nodeIds);

   Request<BlockResponse> getBlockByNum(String blockNumber, boolean isPlain, int... nodeIds);

5.4 查询区块平均生成时间(getAvgGenerateTimeByBlockNumber)
---------------------------------------------------------

参数：

-  from 起始区块号。
-  to 终止区块号。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockAvgTimeResponse> getAvgGenerateTimeByBlockNumber(BigInteger from, BigInteger to, int... nodeIds);

重载方法如下：

.. code:: java

   Request<BlockAvgTimeResponse> getAvgGenerateTimeByBlockNumber(String from, String to, int... nodeIds);

5.5 查询最新区块号，即链高(getChainHeight)
------------------------------------------

参数：

-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockNumberResponse> getChainHeight(int... nodeIds);

5.6 查询创世区块号(getGenesisBlock)
-----------------------------------

参数：

-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockNumberResponse> getGenesisBlock(int... nodeIds);

5.7 查询指定区间区块with limit(getBlocksWithLimit)
--------------------------------------------------

参数：

-  from 起始区块号。
-  to 终止区块号。
-  size 查询的区块数量
-  isPlain
   (可选)，默认为false，表示返回的区块\ **包括**\ 区块内的交易信息，如果指定为true，表示返回的区块\ **不包括**\ 区块内的交易。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockLimitResponse> getBlocksWithLimit(String from, String to, int size, boolean isPlain, int... nodeIds);

重载方法如下：

.. code:: java

   Request<BlockLimitResponse> getBlocksWithLimit(String from, String to, boolean isPlain, int... nodeIds);

5.8 查询指定时间区间内的区块数量(getBlocksByTime)
-------------------------------------------------

注意：当输入的时间范围较大时，\ **请求响应延迟将升高**\ 。存在服务器资源被该请求处理长时间占用的风险，应尽量避免使用此接口。

参数：

-  startTime 起始时间戳(单位ns)。
-  endTime 结束时间戳(单位ns)。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockCountResponse> getBlocksByTime(BigInteger startTime, BigInteger endTime, int... nodeIds);

重载方法如下：

.. code:: java

   Request<BlockCountResponse> getBlocksByTime(String startTime, String endTime, int... nodeIds);

5.9 查询批量区块by block hash list(getBatchBlocksByHash)
--------------------------------------------------------

注意：当输入的区块哈希非常多时，\ **请求响应延迟将升高**\ 。如果返回的数据量超过节点所在服务器内存大小时，将导致处理查询请求的节点出现\ **OOM（Out
Of Memory）**\ 风险，可使用 **tx_getBlockByHash** 接口替代。

参数：

-  blockHashList 要查询的区块哈希数组，哈希值为32字节的十六进制字符串。
-  isPlain (可选)
   默认为false，表示返回的区块\ **包括**\ 区块内的交易信息，如果指定为true，表示返回的区块\ **不包括**\ 区块内的交易。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockResponse> getBatchBlocksByHash(ArrayList<String> blockHashList, int... nodeIds);

   Request<BlockResponse> getBatchBlocksByHash(ArrayList<String> blockHashList, boolean isPlain, int... nodeIds);

5.10 查询批量区块by block number list(getBatchBlocksByNum)
----------------------------------------------------------

注意：当输入的区块号非常多时，\ **请求响应延迟将升高**\ 。如果返回的数据量超过节点所在服务器内存大小时，将导致处理查询请求的节点出现\ **OOM（Out
Of Memory）**\ 风险，可使用 **tx_getBlockByNumber** 接口替代。

参数：

-  blockNumberList 要查询的区块号数组。
-  isPlain (可选)
   默认为false，表示返回的区块\ **包括**\ 区块内的交易信息，如果指定为true，表示返回的区块\ **不包括**\ 区块内的交易。
-  nodeIds 说明请求向哪些节点发送。

.. code:: java

   Request<BlockResponse> getBatchBlocksByNum(ArrayList<Integer> blockNumberList, int... nodeIds);

   Request<BlockResponse> getBatchBlocksByNum(ArrayList<Integer> blockNumberList, boolean isPlain, int... nodeIds);

重载方法如下：

.. code:: java

   Request<BlockResponse> getBatchBlocksByStrNum(ArrayList<String> blockNumberList, int... nodeIds);

   Request<BlockResponse> getBatchBlocksByStrNum(ArrayList<String> blockNumberList, boolean isPlain, int... nodeIds);

第六章. Node相关接口（NodeService）
===================================

NodeService接口用于获取节点信息。NodeService对象对应的响应类型如下：

-  NodeResponse
-  NodeHashResponse
-  NodeStateResponse

分别对应的结构如下。

**NodeResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是内部类\ ``Node``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class NodeResponse extends Response {
       public class Node {
           private int id;
           private String ip;
           private String port;
           private String namespace;
           private String hash;
           private String hostname;
           private boolean isPrimary;
           private boolean isvp;
           private int status;
           private int delay;
       }
       private JsonElement result;
   }

**NodeHashResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是\ ``String``

.. code:: java

   public class NodeHashResponse extends Response {
       @Expose
       private String result;
   }

**NodeStateResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是内部类\ ``NodeState``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class NodeStateResponse extends Response {
       public class NodeState {
           @Expose
           private int id;
          	@Expose
           private String hostname;
           @Expose
           private String hash;
           @Expose
           private String status;
           @Expose
           private int view;
         	@Expose
           private int epoch;
         	@Expose
           private int checkpoint;
           @Expose
           private String blockHeight;
           @Expose
           private String blockHash;
       }
       private JsonElement result;
   }

6.1 获取节点信息
----------------

参数：

-  ids 说明请求向哪些节点发送。

.. code:: java

   Request<NodeResponse> getNodes(int... ids);

6.2 获取节点哈希
----------------

参数：

-  nodeIds 节点ID

.. code:: java

   Request<NodeHashResponse> getNodeHash(int... nodeIds);

   Request<NodeHashResponse> getNodeHashByID(int nodeId);

6.3 获取节点状态信息
--------------------

参数：

-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<NodeStateResponse> getNodeStates(int... nodeIds);

第七章. MQ相关接口(MQService)
=============================

MQService接口用于与\ **RabbitMQ**\ 进行交互。由于开发时间较早，\ ``MQService``\ 对应的响应类型只有\ ``MQResponse``\ 一种，这与之前提到的接口都不太相同：

``MQResponse``\ 接口结构如下：

.. code:: java

   public class MQResponse extends Response {
       private JsonElement result;
    	public List<String> getQueueNames();
       public String getExchanger();
   }

7.1 通知MQ服务器正常工作
------------------------

参数：

-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> informNormal(int... nodeIds)

7.2 注册队列
------------

参数：

-  from 调用该接口的账户地址
-  queueName 队列名称
-  routingkeys 想要订阅的消息类型
-  isVerbose 推送区块时是否推送交易列表，true表示是
-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> registerQueue(String from, String queueName, List<String> routingkeys, Boolean isVerbose, int... nodeIds);

7.3 注册队列(with mqParam)
--------------------------

参数：

-  mqParam
   注册队列所需参数，除了7.2中的参数外，新增了合约event事件的相关过滤参数

.. code:: java

   Request<MQResponse> registerQueue(MQParam mqParam, int... nodeIds);

7.4 注销队列
------------

参数：

-  from 调用该接口的账户地址
-  queueName 队列名称
-  exchangerName exchanger 名称
-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> unRegisterQueue(String from, String queueName, String exchangerName, int... nodeIds);

7.5 获取所有队列名称
--------------------

参数

-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> getAllQueueNames(int... nodeIds);

7.6 获取所有exchanger名称
-------------------------

参数：

-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> getExchangerName(int... nodeIds);

7.7 删除exchanger
-----------------

参数：

-  exchangerName exchanger名称
-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> deleteExchanger(String exchangerName, int... nodeIds);

第八章. MQGrpcService相关接口
=============================

MQGrpcService依赖grpc服务提供MQ相关功能，无需依赖第三方软件进行消息推送。MQGrpcService的响应类型沿用了MQService的\ ``MQResponse``\ ，同时新增了\ ``MQGrpcConsumeResponse``\ 作为消费接口的响应类型。

注意：在使用此服务时，创建ProviderManager对象时，必须提供grpcProvider。

MQResponse为注册队列、注销队列、获取队列名称、停止消费接口的响应类型

.. code:: java

   public class MQResponse extends Response {
       @Expose
       private JsonElement result;
   }

``MQGrpcConsumeResponse``\ 为消费MQ信息接口的响应类型，仅提供一个\ ``getResult``\ 方法，返回值为实现了Iterator接口的\ ``ServerStreamManager``\ 对象。用户可以通过\ ``next``\ 方法不停的获取来自平台的MQ消息，如果平台没有新的消息产生，则next方法会阻塞。MQ消息可通过\ ``Decoder.decodeMQMessage``\ 方法进行解析。

注意：队列A，在同一时刻，只能被一个客户端消费。即对于在一号节点注册的队列A，如果客户端甲已经调用consume接口消费了该队列，那么在这个时刻，客户端乙将无法同时消费队列A。只有当客户端甲调用stopConsume接口，停止消费队列A之后，客户端乙才可以消费队列A。

.. code:: java

   public class MQGrpcConsumeResponse extends Response {
       private ServerStreamManager manager;
       public ServerStreamManager getResult();
   }

   public class ServerStreamManager extends Manager implements Iterator {
       private Iterator<Transaction.CommonRes> commonResIterator;
     	@Override
       public boolean hasNext();

       @Override
       public Object next();

       @Override
       public void remove();
   }

   // mq消费示例
   @Test
   public void testMQ_GRPC_Consume() throws RequestException {
   		Request<MQGrpcConsumeResponse> request = mqGrpcService.consume("queueName", nodeId);
       MQGrpcConsumeResponse response = request.send();
       ServerStreamManager manager = response.getResult();
       while (manager.hasNext()) {
       	String res = (String) manager.next();
         System.out.println(Decoder.decodeMQMessage(res));
         System.out.println(Decoder.decodeMQMessage(res).getBody());
       }
   }

.. _注册队列-1:

8.1 注册队列
------------

参数：

-  from 调用该接口的账户地址
-  queueName 队列名称
-  routingkeys 想要订阅的消息类型
-  isVerbose 推送区块时是否推送交易列表，true表示是
-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> registerQueue(String from, String queueName, List<String> routingkeys, Boolean isVerbose, int... nodeIds);

参数：

-  mqParam
   注册队列所需参数，除了上述方法中的参数外，新增了合约event事件的相关过滤参数

.. code:: java

   Request<MQResponse> registerQueue(MQParam mqParam, int... nodeIds);

.. _注销队列-1:

8.2 注销队列
------------

参数：

-  from 调用该接口的账户地址
-  queueName 队列名称
-  exchangerName exchanger 名称
-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> unRegisterQueue(String queueName, int... nodeIds);

.. _获取所有队列名称-1:

8.3 获取所有队列名称
--------------------

参数

-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> getAllQueueNames(int... nodeIds);

8.4 消费MQ信息
--------------

注意：本消费接口使用完之后，应及时调用停止消费MQ信息接口，不然该队列无法被再次消费。

参数

-  queueName 队列名称

-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQGrpcConsumeResponse> consume(String queueName, int... nodeIds);

8.5 停止消费MQ信息
------------------

参数

-  queueName 队列名称

-  nodeIds 说明请求向某个节点发送，nodeIds有且只能有一个

.. code:: java

   Request<MQResponse> stopConsume(String queueName, int... nodeIds);

第九章. ArchiveService相关接口
==============================

``ArchiveService``\ 接口用于快照和归档相关工作，对应的响应类型如下：

-  ArchiveResponse
-  ArchiveFilterIdResponse
-  ArchiveBoolResponse
-  ArchiveStringResponse

详细结构请参考第十章。

9.1 列出所有快照
----------------

参数：

-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<ArchiveResponse> listSnapshot(int... nodeIds);

9.2 数据归档（直接归档）
------------------------

参数：

-  blkNumber 区块号
-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<ArchiveStringResponse> archiveNoPredict(BigInteger blkNumber, int... nodeIds);

9.3 查询归档数据状态
--------------------

参数：

-  filterId 快照id
-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<ArchiveStringResponse> queryArchive(String filterId, int... nodeIds);

9.4 查询归档数据是否存在
------------------------

参数：

-  filterId 快照id
-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<ArchiveBoolResponse> queryArchiveExist(String filterId, int... nodeIds);

9.5 查询最近一次归档的进度
--------------------------

参数：

-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<ArchiveLatestResponse> queryLatestArchive(int... nodeIds);

9.6 制作快照
------------

参数：

-  blockNumber 区块号
-  nodeIds 说明请求向哪些节点发送

.. code:: java

   Request<ArchiveFilterIdResponse> snapshot(BigInteger blockNumber, int... nodeIds);

第十章. SqlService相关接口
==========================

10.1 创建SQL交易体
------------------

**LiteSDK**\ 使用\ **Builder**\ 模式来负责对\ ``Transaction``\ 的创建，通过调用\ ``build()``\ 函数来获取到\ ``Transaction``\ 实例。SQL交易体由SQLBuilder负责创建，其提供了5种SQL交易体的封装，包括\ **创建数据库**\ 、\ **冻结数据库**\ 、\ **解冻数据库**\ 、\ **删除数据库**\ 、\ **调用SQL**\ 。

**创建数据库**

.. code:: java

   Transaction transaction = new Transaction.SQLBuilder(account.getAddress()).
     create().
     build();

**冻结数据库**

.. code:: java

   Transaction transaction = new Transaction.SQLBuilder(account.getAddress()).
     freeze(databaseAddr).
     build();

**解冻数据库**

.. code:: java

   Transaction transaction = new Transaction.SQLBuilder(account.getAddress()).
     unfreeze(databaseAddr).
     build();

**删除数据库**

.. code:: java

   Transaction transaction = new Transaction.SQLBuilder(account.getAddress()).
     destroy(databaseAddr).
     build();

**调用SQL**

.. code:: java

   Transaction transaction = new Transaction.SQLBuilder(account.getAddress()).
     invoke(databaseAddr, SQLStr).
     build();

10.2 创建数据库
---------------

创建数据库交易成功后，可通过交易回执查询创建的数据库地址。

参数：

-  transaction 创建数据库的交易体

-  nodeIds 说明请求向哪些节点发送

.. code:: java

   //方式一
   Request<TxHashResponse> create(Transaction transaction, int... nodeIds);

   //方式二（设置GrpcProvider方可使用）
   Request<ReceiptResponse> grpcInvokeReturnReceipt(Transaction transaction, int... nodeIds);

10.3 管理数据库生命周期
-----------------------

参数：

-  transaction
   管理数据库生命周期的交易体，即冻结数据库、解冻数据库或者删除数据库的交易体

-  nodeIds 说明请求向哪些节点发送

.. code:: java

   //方式一
   Request<TxHashResponse> maintain(Transaction transaction, int... nodeIds);

   //方式二（设置GrpcProvider方可使用）
   Request<ReceiptResponse> grpcMaintainReturnReceipt(Transaction transaction, int... nodeIds);

10.4 调用SQL
------------

参数：

-  transaction 调用SQL的交易体

-  nodeIds 说明请求向哪些节点发送

.. code:: java

   //方式一
   Request<TxHashResponse> invoke(Transaction transaction, int... nodeIds);

   //方式二（设置GrpcProvider方可使用）
   Request<ReceiptResponse> grpcInvokeReturnReceipt(Transaction transaction, int... nodeIds);

10.5 SQL执行结果解码
--------------------

类似于HVM，SQL的执行结果解码方法在Decoder类中

参数：

-  encode sql的执行结果，即sql交易的回执

.. code:: java

   public static Chunk decodeKVSQL(String encode);

10.6 Chunk结构的使用
--------------------

chunk结构保存了sql的执行结果，使用方法类似于jdbc的ResultSet

**10.6.1 DML语句**

对于DML语句，通过getUpdateCount方法可以获得sql语句对应的修改行数

.. code:: java

   public long getUpdateCount();

也可以通过getLastInsertID获得插入的自增键

.. code:: java

   public long getLastInsertID();

**10.6.2 DQL语句**

对于DQL语句，chunk的使用方法类似于jdbc的ResultSet

调用next接口将游标移动到查询结果的下一行，游标初始位置为第一行之前。

.. code:: java

   public boolean next();

如果返回结果为false表示游标已经移动到最后一行之后了。

调用getValue接口将获得游标指向的当前行以及参数对应列所指向的数据的值。返回结果的类型取决于该列在数据库表中的数据类型。

参数

-  columnIdex 列坐标，范围为[0,n-1]，n为列的数量

.. code:: java

   public Object getValue(int columnIndex);

chunk同时也提供指定类型的数据获取的方法，对应方法如下，XXX表示类型，如getFloat，getTime等。该类使用方法与getValue一致，不过需要注意的是，XXX类型必须和数据库表中该列的类型一致或者能过相互转换，否则无法获得正确的结果或者会抛出异常。

.. code:: java

   public XXX getXXX(int columnIndex);

第十一章. 接口响应类型结构体介绍
================================

11.1 TxService接口对应的响应类型
--------------------------------

-  TxResponse
-  TxCountWithTSResponse
-  TxCountResponse
-  TxAvgTimeResponse
-  ReceiptListResponse

分别对应的结构如下：

**TxResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际结构是内部类\ ``Transaction``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class TxResponse extends Response {
       public class Transaction {
           private String version;
           private String hash;
           private String blockNumber;
           private String blockHash;
           private String txIndex;
           private String from;
           private String to;
           private String amount;
           private String timestamp;
           private String nonce;
           private String extra;
           private String executeTime;
           private String payload;
           private String signature;
           private String blockTimestamp;
           private String blockWriteTime;
       }
       private JsonElement result;
   }

**TxCountWithTSResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是内部类\ ``TxCount``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class TxCountWithTSResponse extends Response {
       public class TxCount {
           private String count;
           private long timestamp;
         	private String lastIndex;
         	private String lastBlockNum;
       }
       protected JsonElement result;
   }

**TxCountResponse**

继承\ ``TxCountWithTSResponse``\ 通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是\ ``TxCount``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class TxCountResponse extends TxCountWithTSResponse {
   }

**TxAvgTimeResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是\ ``String``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class TxAvgTimeResponse extends Response {
       private String result;
   }

**ReceiptListResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是内部类\ ``Receipt``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class ReceiptListResponse extends Response {
       private ArrayList<ReceiptResponse.Receipt> result;
   }

``Receipt``\ 结构如下

.. code:: java

   public class Receipt {
           private String contractAddress;
           private String ret;
           private String txHash;
           private EventLog[] log;
           private String vmType;
           private long gasUsed;
           private String version;
   }

11.2 BlockService接口对应的响应类型
-----------------------------------

-  BlockResponse
-  BlockNumberResponse
-  BlockAvgTimeResponse
-  BlockCountResponse

分别对应的结构如下。

**BlockResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是内部类\ ``Block``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class BlockResponse extends Response {
       public class Block {
           private String version;
           private String number;
           private String hash;
           private String parentHash;
           private String writeTime;
           private String avgTime;
           private String txcounts;
           private String merkleRoot;
       }
       private JsonElement result;
   }

**BlockNumberResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是\ ``String``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class BlockNumberResponse extends Response {
       private String result;
   }

**BlockAvgTimeResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是\ ``String``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class BlockAvgTimeResponse extends Response {
       @Expose
       private String result;
   }

**BlockCountResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际类型是内部类\ ``BlockCount``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class BlockCountResponse extends Response {
       public class BlockCount {
           private String sumOfBlocks;
           private String startBlock;
           private String endBlock;
       }
       private BlockCount result;
   }

11.3 ArchiveService接口对应的响应类型
-------------------------------------

-  ArchiveResponse
-  ArchiveFilterIdResponse
-  ArchiveBoolResponse
-  ArchiveStringResponse

分别对应的结构如下：

**ArchiveResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际结构是内部类\ ``Archive``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class ArchiveResponse extends Response {
       public class Archive {
           private String height;
           private String hash;
           private String filterId;
           private String merkleRoot;
           private String date;
           private String namespace;
       }

       private JsonElement result;
   }

**ArchiveFilterIdResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际结构是\ ``String``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class ArchiveFilterIdResponse extends Response {
       private String result;
   }

**ArchiveBoolResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际结构是\ ``Boolean``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class ArchiveBoolResponse extends Response {
       private Boolean result;
   }

**ArchiveStringResponse**

通过\ ``result``\ 接收返回结果，\ ``result``\ 实际结构是\ ``String``\ ，可通过\ ``getResult()``\ 方法得到。

.. code:: java

   public class ArchiveBoolResponse extends Response {
       private String result;
   }

附录
====

附录 A Solidity与Java的编码解码
-------------------------------

类型对应
~~~~~~~~

当使用\ **LiteSDK**\ 编译solidity合约时，由于java和solidity本身类型的不兼容，所以在调用solidity方法传参数的时候需要对java类型进行相应的编码解码，LiteSDK内部的\ ``Abi``\ 类，\ **与solidity的abi文件对应，用来提供solidity合约的函数入参、返回值等信息**\ ，方便我们对solidity类型和java类型做转换，目前Litesdk支持的对应类型如下：

=================== ==================================
JAVA                SOLIDITY
=================== ==================================
``boolean/Boolean`` ``bool``
``BigInteger``      ``int、int8、int16……int256``
``BigInteger``      ``uint、uint8、uint16……uint256``
``String``          ``string``
``byte[]/Byte[]``   ``bytes、bytes1、bytes2……bytes32``
``string``          ``address``
``Array``/``List``  ``array``
=================== ==================================

编码
~~~~

编码时需要提供以下信息：

-  solidity合约对应的abi对象，
-  调用方法名
-  封装后的java参数

实现java与solidity之间的类型转换。（注：如果是部署需要提供bin文件，具体参照\ `部署合约 <#部署合约>`__\ 一节）

Abi对象
------------

通过\ **LiteSDK**\ 提供的\ ``FileUtil``\ 工具类读取文件内容得到abi字符串，并利用\ ``Abi``\ 类的\ ``fromJson``\ 方法生成封装的Abi对象，使用方法如下：

.. code:: java

   InputStream abiIs = Thread.currentThread().getContextClassLoader().getResourceAsStream("xxx.abi");
   String abiStr = FileUtil.readFile(abiIs);
   Abi abi = Abi.fromJson(abiStr);

调用方法名
------------

调用方法名需要按格式\ ``$(method_name)(type1[,type2…])``\ 填，假如solidity的函数签名为

.. code:: solidity

   function TestUint(uint8 a) returns (uint8) {
       return a;
   }

则我们提供的调用方法名为\ ``TestUint(uint8)``\ ，如果函数多个参数，则调用方法名的类型之间用\ **,**\ 分隔。

封装的java参数
------------

**LiteSDK**\ 提供了\ ``FuncParams``\ 工具类封装\ **需要转换成solidity类型的java参数**\ ，使用方法如下：

.. code:: java

   FuncParams params = new FuncParams();
   // param 是类型对应表里对应的java参数
   params.addParams(param1);
   params.addParams(param2);

   // 构造交易时将构造好的FuncParams对象传进去
   Transaction transaction = new Transaction.EVMBuilder(account.getAddress()).invoke(contractAddress, <method_name>, abi, params).build();

解码
~~~~

解码与编码类似，需要提供\ **Abi对象、方法名和编码的solidity结果**\ ，具体可见\ `编码 <#编码>`__\ 一节。

调用evm合约得到交易回执\ ``ReceiptResponse``\ 后，需要对solidity合约的返回值进行解析，使用方法如下：

.. code:: java

   String ret = receiptResponse.getRet();
   byte[] fromHex = ByteUtil.fromHex(ret);

   // 通过abi的方法名解码，由于返回值可能有多个，所以解码得到的其实是一个List<?>，当中的每个对象
   // 对应一个返回值。如该例子返回值为 int256，在java中对应的是BigInter，所以对返回的decodeResult
   // 遍历强转为BigInteger
   List<?> decodeResult = abi.getFunction("TestInt(int256)").decodeResult(fromHex);
   for (Object result : decodeResult) {
       System.out.println(result.getClass());
       System.out.println(((BigInteger) result).toString());
   }

附录B HVM合约相关使用方式
-------------------------

直接调用HVM合约方法的参数封装
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

直接调用HVM合约方法封装参数需要用到类\ ``InvokeDirectlyParams``\ 。

示例如下：

假设调用合约方法\ ``add(int a, int b)``\ ，传入参数（10，100）；

.. code:: java

   // 构造函数传入想要调用的方法名
   InvokeDirectlyParams.ParamBuilder params = new InvokeDirectlyParams.ParamBuilder("add");
   // 方法addxxx分别构造不同类型的参数
   params.addint(10);
   params.addint(100);
   InvokeDirectlyParams.params.build();

HVM合约交易回执解码
~~~~~~~~~~~~~~~~~~~

调用HVM（Java）合约后，获取到的交易回执可以使用`Decoder.decodeHVM进行解码。

示例如下：

假设调用合约方法\ ``public Integer add(int a, int b)``\ ，返回值类型是Integer；

.. code:: java

   ReceiptResponse receiptResponse = contractService.deploy(transaction).send().polling();
   Integer result = Decoder.decodeHVM(receiptResponse.getRet(), Integer.class);//传入合约方法的返回值类型

附录C 平台错误码和对应原因
--------------------------

======== ==========================================================
**code** **含义**
======== ==========================================================
0        请求成功
-32700   服务端接收到无效的json。该错误发送于服务器尝试解析json文本
-32600   无效的请求（比如非法的JSON格式）
-32601   方法不存在或者无效
-32602   无效的方法参数
-32000   Hyperchain内部错误或者空指针或者节点未安装solidity环境
-32001   查询的数据不存在
-32002   余额不足
-32003   签名非法
-32004   合约部署出错
-32005   合约调用出错
-32006   系统繁忙(平台需要处理交易量达到限制)
-32007   交易重复
-32008   合约操作权限不够
-32009   账户不存在
-32010   namespace不存在
-32011   账本上无区块产生，查询最新区块的时候可能抛出该错误
-32012   订阅不存在
-32013   数据归档、快照相关错误
-32021   过时接口
-32097   Hypercli用户令牌无效
-32098   请求未带cert或者错误cert导致认证失败
-32099   请求tcert失败
\        参数错误(指定节点发送时，指定index错误)
-9993    文件下载失败
-9994    FileMgrHttpProvider不支持的request类型请求
-9995    请求失败(通常是请求体过长)
-9996    请求失败(通常是请求消息错误)
-9997    异步请求失败
-9998    请求超时(轮询结束未获得回执)
-9999    获取平台响应失败
======== ==========================================================

上述为平台api和sdk接口可能返回的状态码的说明，其中-999x的状态码为sdk对平台返回状态码或网络请求结果的封装，简化上层处理逻辑；其余状态码为平台api接口的原生返回结果。

在通过LiteSDK调用查询接口时，例如查询交易Hash对应的交易回执或者通过区块号查询区块内容时，LiteSDK将不会对查询接口进行交易状态码的封装，返回原生状态码，查询结果即为平台返回结果；当发生网络断连问题导致查询接口无法获得Response时，将返回-999x状态码。

当通过LiteSDK发送交易时，由于平台执行交易为异步执行，通过先返回交易Hash，在通过交易Hash查询回执的方式，所以LiteSDK将发送交易和查询交易回执进行了拆分，一个完整的发送交易并获得回执过程如下：

.. code:: java

   Request<TxHashResponse> request = sendTxService.sendTx(transaction);
   TxHashResponse txHashResponse = request.send();
   ReceiptResponse response = txHashResponse.polling();

1. 通过调用\ ``request.send()``\ 将交易发送到链上，

   1. 返回状态码为0并获取交易Hash表示交易已成功上链
   2. 当出现-9995或者-9996时表示请求返送失败，交易未上链
   3. **当出现-9999时表示网络出现断连，此时无法确定是交易还未发送成功还是获取Response时出现错误，不明确错误原因**
   4. 其余情况均为平台返回交易上链失败错误，交易未上链

2. 通过调用\ ``txHashResponse.polling()``\ 可以
   通过交易Hash获取交易回执：

   1. 返回状态码为0时表示查找回执成功，交易执行成功
   2. **由于轮询查找回执时可能平台尚未完成交易执行(-32001)、平台达到流量限制(-32006)或网络抖动(-9996,-9999)等原因，轮询过程将持续到轮询次数结束，此时若任未获取到回执，将抛出-9998的错误，此时表示轮询查询回执不成功，可能平台尚未执行完该笔交易，不明确错误原因**
   3. 其余情况下轮询获取到回执均表示查找回执成功，但交易执行失败，成为非法交易