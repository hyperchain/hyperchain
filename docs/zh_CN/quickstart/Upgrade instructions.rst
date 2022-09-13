.. _upgrade_instructions:

升级说明
^^^^^^^^^^

本文是v1.8.x 或2.1.0以前到 v2.7.0的版本升级说明文档

..

   低版本升级至高版本需注意应用侧也需要升级对应的sdk版本

参照《SDK&版本对应关系》文档

1. v1.8.x到v2.7.0的版本升级
===========================

1.1 节点等信息上链
------------------

在hyperchain还在运行的状态下，提前准备好配置中的genesis账户配置信息、当前所有节点的节点名及其对应的证书文件。

通过sdk发送交易将上述信息上链，以litesdk为例：

.. code:: java

   public void testSetGenesisInfo() throws Exception {
           // 准备创世账户、创世节点信息
           // 需要按照实际情况准备创世账户、创世节点以及节点证书
           Map<String, String> genesisAccount= new HashMap<>();
           genesisAccount.put("0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd", "1000000000");
           genesisAccount.put("e93b92f1da08f925bdee44e91e7768380ae83307", "1000000000");
           List<GenesisNode> genesisNodes = new ArrayList<>();
           String node1Cert = "node1 cert content";
           genesisNodes.add(new GenesisNode("node1", node1Cert));
           GenesisInfo genesisInfo = new GenesisInfo(genesisAccount, genesisNodes);
           // 发送交易将创世信息上链
           Account ac = accountService.fromAccountJson(accountJson);
           Transaction transaction = new Transaction.
                   BVMBuilder(ac.getAddress()).
                   invoke(new HashOperation.HashBuilder().setGenesisInfoForHpc(genesisInfo).build()).
                   // 注意，向hyperchain发交易，txVersion需要设置成1.0
                   txVersion(TxVersion.TxVersion10).
                   build();
           transaction.sign(ac);
           ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
           System.out.println(receiptResponse.getRet());
           Result result = Decoder.decodeBVM(receiptResponse.getRet());
           Assert.assertTrue(result.isSuccess());

           // 查询上链的创世信息
           transaction = new Transaction.
                   BVMBuilder(ac.getAddress()).
                   invoke(new HashOperation.HashBuilder().getGenesisInfoForHpc().build()).
                   // 注意，向hyperchain发交易，txVersion需要设置成1.0
                   txVersion(TxVersion.TxVersion10).
                   build();
           transaction.sign(ac);
           receiptResponse = contractService.invoke(transaction).send().polling();
           System.out.println(receiptResponse.getRet());

           result = Decoder.decodeBVM(receiptResponse.getRet());
           Assert.assertTrue(result.isSuccess());
           Gson gson = new Gson();
           Assert.assertEquals(gson.toJson(genesisInfo), result.getRet());
       }

1.2 进行数据备份
----------------

升级之前先停止当前hyperchain进程，将旧版本数据备份，若升级失败可以及时恢复，并将数据打包，供迁移使用：

.. code:: shell

   # 检查当前​磁盘空间是否足够，如磁盘空间不足需额外挂载磁盘
   $ df

   # 停止旧版本进程，按照实际安装路径以及安装包名称替换
   $ cd /path/to/hyperchain-1.8.6
   $ ./stop.sh
   ​

   # 备份旧目录
   $ tar zcvf /path/to/hyperchain-1.8.6-backup.tar.gz /path/to/hyperchain-1.8.6

1.3. 部署Flato
--------------

.. code:: text

   # 在flato安装文件目录执行deploy_local脚本
   $ ./deploy-local.sh -d /flato-1.0.0

之后请参考《平台部署手册》对所有节点的网络配置进行修改。

1.4. 使用flato-dbcli进行数据升级
--------------------------------

dbcli是升级数据、保证数据格式在大版本之间兼容的命令行工具。dbcli和flato的版本对应关系如下表所示：

==== ===== ===========================
\    dbcli flato
==== ===== ===========================
版本 1.1.1 1.0.6
\    1.1.3 1.0.8（包含）-1.1.0（包含）
\    1.2.0 1.2.0及以上
==== ===== ===========================

**请通过**\ ``**./flato-dbcli --version**``\ **确认版本，请勿使用上表以外的版本进行升级。**

dbcli是升级数据、保证数据格式在大版本之间兼容的命令行工具。dbcli的输入参数列表以及相应的说明如下：

+-------------+-------------------------------------------+------------+
| 参          | 介绍                                      | Required / |
| 数名(逗号前 |                                           | Optional   |
| 后两者等效) |                                           |            |
+=============+===========================================+============+
| h           | hyperchain的配置文件路径，如              | Required   |
| pcConfig,hc | namespace.toml                            |            |
+-------------+-------------------------------------------+------------+
| flatoArch   | 清洗                                      | Required   |
| ivePath,fap | 线下数据完成后归档/快照数据的目标路径，如 |            |
|             | “archive”                                 |            |
+-------------+-------------------------------------------+------------+
| flat        | flato的配置文件目录，                     | Required   |
| oConfig,fcp | 即flato安装包中的整个configuration目录；  |            |
+-------------+-------------------------------------------+------------+
| n           | hyperchain的分区名，如”global”            | Required   |
| amespace,ns |                                           |            |
+-------------+-------------------------------------------+------------+
| fla         | 升级后flato的数据目录                     | Optional   |
| toDBPath,fp |                                           |            |
+-------------+-------------------------------------------+------------+
| onlin       | hyperchai                                 | Optional   |
| eBlockDB,ob | n线上区块数据目录，如”data/filelog/block” |            |
+-------------+-------------------------------------------+------------+
| onlineJ     | hyperchain线上                            | Optional   |
| ournalDB,oj | journal数据目录，如”data/filelog/journal” |            |
+-------------+-------------------------------------------+------------+
| onlineBlock | hyperchain线上bl                          | Optional   |
| ChainDB,obc | ockchain目录，如”data/leveldb/blockchain” |            |
+-------------+-------------------------------------------+------------+
| mqPath,mq   | hyperchain线上MQ目录，如”data/leveldb/MQ” | O          |
|             |                                           | ptional,仅 |
|             |                                           | 当使用过MQ |
|             |                                           | 功能时需要 |
+-------------+-------------------------------------------+------------+
| snaps       | hyperch                                   | Optional   |
| hotDBDir,sd | ain的快照db目录，如”data/leveldb/snapshot |            |
|             | s/“注意此处需要的不是单个snapshotdb的目录 |            |
+-------------+-------------------------------------------+------------+
| snap        | 记录快照的snapshot.meta                   | Optional   |
| shotMeta,sm |                                           |            |
+-------------+-------------------------------------------+------------+
| f           | 快照的filter id                           | Optional   |
| ilterID,fid |                                           |            |
+-------------+-------------------------------------------+------------+
| arc         | 记录归档的archive.meta                    | Optional   |
| hiveMeta,am |                                           |            |
+-------------+-------------------------------------------+------------+
| offline     | hyperchain                                | Optional   |
| BlockDB,ofb | 线下区块数据目录，如”data/filelog/backup” |            |
+-------------+-------------------------------------------+------------+
| offlineJo   | hyperchain线下journal                     | Optional   |
| urnalDB,ofj | 数据目录，如”data/filelog/journal_backup” |            |
+-------------+-------------------------------------------+------------+
| offlineAr   | hyperchain                                | Optional   |
| chiveDB,ofa | 线下archive目录，如”data/leveldb/archive” |            |
+-------------+-------------------------------------------+------------+
| log         | 日志级别，可选                            | Optional   |
|             | debug,info,notice,warnin                  |            |
|             | g,error,critical,默认notice，大小写不敏感 |            |
+-------------+-------------------------------------------+------------+

dbcli同时支持已归档的线下数据的升级与未归档的线上数据的升级，并且会根据输入的参数自动选择升级的模式。

需要注意flato-dbcli工具不支持断点续传，如数据升级中途退出则需要从头开始。

为避免此问题 需要在工具命令前加上 nohup 命令锁定进程，在命令的最后加上
&>/dev/null & 使进程后台运行。\ **（非常重要！！！）**

示例：

.. code:: text

   nohup ./flato-dbcli migrate \
    -ns global \
    -fp /flato-1.5.0/namespaces/global/data/public/ \
    -fap /flato-1.5.0/namespaces/global/data/archive \
    -fcp /flato-1.5.0/configuration/ \
    -hc /hyperchain-1.8.6/namespaces/global/config/namespace.toml \
    -ob /hyperchain-1.8.6/namespaces/global/data/filelog/block \
    -oj /hyperchain-1.8.6/namespaces/global/data/filelog/journal \
    -obc /hyperchain-1.8.6/namespaces/global/data/leveldb/blockchain \
    -log debug \
    &>/dev/null &

1.4.1 未归档的线上数据
~~~~~~~~~~~~~~~~~~~~~~

在这种场景下，除了需要提供上述\ **Required**\ 的参数之外，还\ **至少**\ 需要提供以下参数：

1. -fp:
   升级后flato数据的目标目录，在完成升级之后需要将该目录拷贝至flato节点下；

2. -ob:
   hyperchain中的区块数据目录，在默认配置下，这个路径为data/filelog/block；

3. -oj:
   hyperchain中的journal数据目录，在默认配置下，这个路径为data/filelog/journal；

4. -obc:
   hyperchain中的账本数据目录，在默认配置下，这个路径为data/leveldb/blockchain；

具体的使用示例如下：

.. code:: shell

   nohup ./flato-dbcli migrate \
    -ns global \
    -fp /flato-1.5.0/namespaces/global/data/public/ \
    -fap /flato-1.5.0/namespaces/global/data/archive \
    -fcp /flato-1.5.0/configuration/ \
    -hc /hyperchain-1.8.6/namespaces/global/config/namespace.toml \
    -ob /hyperchain-1.8.6/namespaces/global/data/filelog/block \
    -oj /hyperchain-1.8.6/namespaces/global/data/filelog/journal \
    -obc /hyperchain-1.8.6/namespaces/global/data/leveldb/blockchain \
    -log debug \
    &>/dev/null &

**若使用过MQ功能，则需要指定mq数据库路径，请参考参数列表。**

在执行为上述指令后，flato目标目录(即上例子的-fp路径)会呈现如下结构：

.. code:: text

   . <- /flato-1.0.8/namespaces/global/data/
   ├── archive
   │   ├── 0xaf57419bd97530d5384debe86e24eec6
   │   ├── archive.meta
   │   ├── archive.record
   │   └── snapshot.meta
   └── public
       ├── accountdb
       ├── blockdb
       ├── cadb
       ├── chaindb
       ├── consensusdb
       ├── invalidtx
       ├── journaldb
       ├── minifile
       ├── mqdb
       ├── radardb
       ├── receiptdb
       ├── statedb
       ├── sync_chain.meta
   	├── config.meta
   	└── rollback.meta

**值得注意的是，若曾进行过归档操作，则必须提供archive.meta文件，并且确保归档点小于最近的检查点之前，即若Hyperchain中的区块高度为58，那么不能在[51,58]进行归档，示例：**

.. code:: bash

   nohup ./flato-dbcli migrate \
    -ns global \
    -fp /flato-1.0.8/namespaces/global/data/public/ \
    -fap /flato-1.0.8/namespaces/global/data/archive \
    -fcp /flato-1.0.8/configuration/ \
    -hc /hyperchain-1.8.6/namespaces/global/config/namespace.toml \
    -ob /hyperchain-1.8.6/namespaces/global/data/filelog/block \
    -oj /hyperchain-1.8.6/namespaces/global/data/filelog/journal \
    -obc /hyperchain-1.8.6/namespaces/global/data/leveldb/blockchain
    -am /hyperchain-1.8.6/namespaces/global/data/archive.meta ##必须提供\
    &>/dev/null &


若未做过归档，那么请忽略该参数。

1.4.2 已归档的线下数据
~~~~~~~~~~~~~~~~~~~~~~

若要升级已归档的线下数据，除了要提供参数表格中\ **Required**\ 的参数，还需要提供以下参数：

1. -ofb:
   需要升级的归档区块数据库，默认配置下，路径为/hyperchain-1.8.6/namespaces/global/data/filelog/backup；

2. -ofj:
   需要升级的归档journal数据库，默认配置下，路径为/hyperchain-1.8.6/namespaces/global/data/filelog/journal_backup；

3. -ofa:
   需要升级的归档账本数据库，默认配置下，路径为/hyperchain-1.8.6/namespaces/global/data/leveldb/archive；

4. -am:
   归档记录archive.meta,默认配置下，路径为/hyperchain-1.8.6/namespaces/global/data/archive.meta；

5. -sm:
   归档产生的snapshot.meta文件路径，默认配置下，路径为/hyperchain-1.8.6/namespaces/global/data/snapshot.meta；

6. -sd:
   快照数据库目录，默认配置下，路径为/hyperchain-1.8.6/namespaces/global/data/leveldb/snapshots；

7. -fid: 归档快照对应的filter
   id，在调用归档接口之后可以从返回结果中查询到该值。

具体的使用示例如下：

.. code:: shell

   nohup ./flato-dbcli migrate \
    -ns global \
    -fap /flato-1.0.0/namespaces/global/data/archive \
    -fcp /flato-1.0.0/configuration/ \
    -hc /hyperchain-1.8.6/namespaces/global/config/namespace.toml \
    -ofb /hyperchain-1.8.6/namespaces/global/data/filelog/backup \
    -ofj /hyperchain-1.8.6/namespaces/global/data/filelog/journal_backup \
    -ofa /hyperchain-1.8.6/namespaces/global/data/leveldb/archive \
    -sm /hyperchain-1.8.6/namespaces/global/data/snapshot.meta \
    -sd /hyperchain-1.8.6/namespaces/global/data/leveldb/snapshots \
    -fid 0xdcbb0bd0b2cc9c746f476f7bbd031ad0 \
    -am /hyperchain-1.8.6/namespaces/global/data/archive.meta \
    -log debug \
    &>/dev/null &

升级成功后的数据目录如下：

.. code:: text

   . <- /flato-1.0.8/namespaces/global/data/
   └── archive
       ├── 0x7d6d371262caef0fb21f417dae92254d
       │   ├── accountdb
       │   ├── blockdb
       │   ├── chaindb
       │   ├── journaldb
       │   ├── receiptdb
       │   └── statedb
       ├── archive.meta
       ├── archive.record
       └── snapshot.meta

通过我们的archive-reader组件可以进行归档数据的查询。

1.5. 启动Flato并验证状态
------------------------

在升级完线上数据、完成数据的替换之后，就可以启动Flato进程了：

.. code:: shell

   # 启动flato节点
   /flato-1.0.8/start.sh

   检查节点状态
   $ ./status.sh

   # 检查namespace日志确认是否启动成功
   $ tail -300f namespaces/global/data/logs/最新的log文件

若节点启动成功，则说明本次版本升级成功。

在进行下一步操作之前，请启动所有的区块链节点，并确保所有节点都输出如下日志，代表节点共识恢复成功：

.. code:: text

    +==============================================+
     |                                              |
     |            RBFT Recovery Finished            |
     |                                              |
     +==============================================+

1.6. 失败恢复
-------------

若数据升级过程发生失败，或flato节点启动失败，可通过备份的数据恢复hyperchain节点：

.. code:: shell

   # 关闭节点
   $ cd /flato-1.0.8/
   $ ./stop.sh

   # 删除或重命名原先的文件
   $ rm -rf /flato-1.0.8

   # 解压备份的数据
   $ cd /hyperchain-1.8.6
   $ tar zxvf hyperchain-1.8.6-backup.tar.gz
   # 待四个节点均复原后启动平台
   $ cd /hyperchain-1.8.6/hyperchain-1.8.6
   $ ./start.sh

   # 检查恢复日志
   $ tail -300f namespaces/global/data/logs/最新的log文件

2. 从v2.0.0~v2.3.0版本升级到v2.7.0+
===================================

2.1 节点初始化
--------------

在升级前，需要确保已经进行过节点初始化，若没有进行，需要先进行节点初始化。

检查是否进行过节点初始化：(自行替换ip及端口)

.. code:: text

   curl 127.0.0.1:8081 --data '{"jsonrpc":"2.0","namespace":"global","method":"config_getVSet","params":[""],"id":1}'

若返回的结果如下，则表示进行过初始化。

.. code:: text

   {
       "jsonrpc":"2.0",
       "namespace":"global",
       "id":1,
       "code":0,
       "message":"SUCCESS",
       "result":[
           "node1",
           "node2",
           "node3",
           "node4"
       ]
   }

若返回的结果如下，则表示没有进行过初始化，需要进行节点初始化。

.. code:: text

   {
       "jsonrpc":"2.0",
       "namespace":"global",
       "id":1,
       "code":-32001,
       "message":"DB not found: not found VSet"
   }

通过Litesdk进行节点初始化的示例如下：

.. code:: java

   // 每个字符串为ns_genesis.toml中的genesis.alloc配置的每个账户私钥，用于创建提案、提案投票等
   private static String[] accountJsons = new String[]{
               ""
               , ""
               , ""
               , ""
               , ""
               , ""};
   public void testInit() throws RequestException {

       // 准备创世节点的hostname以及对应节点的公钥
   	// 替换nodex为对应节点的hostname，pubx为对应节点的公钥

       // create、vote and execute proposal for init nodes
       completeProposal(new ProposalOperation.ProposalBuilder().createForNode(
               new NodeOperation.NodeBuilder().addNode("pub1", "node1","vp", "global").build(),
               new NodeOperation.NodeBuilder().addVP("node1","global").build(),
               new NodeOperation.NodeBuilder().addNode("pub2", "node2","vp", "global").build(),
               new NodeOperation.NodeBuilder().addVP("node2","global").build(),
               new NodeOperation.NodeBuilder().addNode("pub3", "node3","vp", "global").build(),
               new NodeOperation.NodeBuilder().addVP("node3","global").build(),
               new NodeOperation.NodeBuilder().addNode("pub4", "node4","vp", "global").build(),
               new NodeOperation.NodeBuilder().addVP("node4","global").build()
       ).build());
   }

   public String completeProposal(BuiltinOperation opt) throws RequestException {
       // create
       invokeBVMContract(opt, accountService.fromAccountJson(accountJsons[0]));

       Request<ProposalResponse> proposal = configService.getProposal();
       ProposalResponse proposalResponse = proposal.send();
       ProposalResponse.Proposal prop = proposalResponse.getProposal();

       // vote
       for (int i = 1; i < accountJsons.length; i++) {
           invokeBVMContract(new ProposalOperation.ProposalBuilder().vote(prop.getId(), true).build(), accountService.fromAccountJson(accountJsons[i]));
       }

       // execute
       Result result = invokeBVMContract(new ProposalOperation.ProposalBuilder().execute(prop.getId()).build(), accountService.fromAccountJson(accountJsons[0]));
       Assert.assertEquals("", result.getErr());

       System.out.println(result.getRet());
       List<OperationResult> resultList = Decoder.decodeBVMResult(result.getRet());
       for (OperationResult or : resultList) {
   	    System.out.println(or.getCode());
       }
       if (resultList.size() > 0) {
           return resultList.get(0).getMsg();
       }
       return null;
   }

   public Result invokeBVMContract(BuiltinOperation opt, Account acc) throws RequestException {
       Transaction transaction = new Transaction.
               BVMBuilder(acc.getAddress()).
               invoke(opt).
               build();
       transaction.sign(acc);

       ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
       Result result = Decoder.decodeBVM(receiptResponse.getRet());
       System.out.println(result);
       return result;
   }

2.2 配置文件（2.1.0之后的版本需要新增\ ``[gensis]``\ 模块）
-----------------------------------------------------------

新增配置文件ns_genesis.toml

.. code:: yaml

   [genesis]
   ca_mode = "Center" # ca模式，配置后不支持修改。支持的ca模式有：中心化ca，即Center；分布式ca，即Distributed；无ca，即none
   root_ca = ["""-----BEGIN CERTIFICATE-----
   MIICODCCAeSgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV
   BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD
   VQQLEwAxDjAMBgNVBAMTBW5vZGUxMQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl
   cnQwIBcNMjAwNTIxMDQyNTQ0WhgPMjEyMDA0MjcwNTI1NDRaMHQxCTAHBgNVBAgT
   ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh
   dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTExCzAJBgNVBAYTAlpIMQ4wDAYD
   VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABDoBjgQsvY4xhyIy3aWh
   4HLOTTY6te1VbmZaH5EZnKzqjU1f436bVsfi9HLE3/MCeZD6ISe1U5giM5NuwF6T
   ZEOjaDBmMA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB
   BQUHAwEGAioDBgOBCwEwDwYDVR0TAQH/BAUwAwEB/zANBgNVHQ4EBgQEAQIDBDAM
   BgMqVgEEBWVjZXJ0MAoGCCqGSM49BAMCA0IAuVuDqguvjPPveimWruESBYqMJ1qq
   ryhXiMhlYwzH1FgUz0TcayuY+4KebRhFhb14ZDXBBPXcn9CYdtbbSxXTogE=
   -----END CERTIFICATE-----
   """] # 当ca模式问中心化ca，即Center时，需要配置中心化ca使用的root ca，其他模式下无需配置
   [genesis.alloc] # 创世账户，用户需用自己生成的账户地址替换创世账户地址，后续妥善保管这些账户的私钥
   "000f1a7a08ccc48e5d30f80850cf1cf283aa3abd" = "1000000000"
   "e93b92f1da08f925bdee44e91e7768380ae83307" = "1000000000"
   "6201cb0448964ac597faf6fdf1f472edf2a22b89" = "1000000000"
   "b18c8575e3284e79b92100025a31378feb8100d6" = "1000000000"
   "856E2B9A5FA82FD1B031D1FF6863864DBAC7995D" = "1000000000"
   "fbca6a7e9e29728773b270d3f00153c75d04e1ad" = "1000000000"

   [[genesis.nodes]] # 创世节点列表
   genesisNode = "node1" # 创世节点名称，与hostname对应
   certContent = """-----BEGIN CERTIFICATE-----
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
   -----END CERTIFICATE-----""" # 创世节点证书，即此节点的ecert

   [[genesis.nodes]]
   genesisNode = "node2"
   certContent = """-----BEGIN CERTIFICATE-----
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
   -----END CERTIFICATE-----"""

   [[genesis.nodes]]
   genesisNode = "node3"
   certContent = """-----BEGIN CERTIFICATE-----
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
   -----END CERTIFICATE-----"""

   [[genesis.nodes]]
   genesisNode = "node4"
   certContent = """-----BEGIN CERTIFICATE-----
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
   -----END CERTIFICATE-----"""

由于在v2.7.0版本开始支持checkpoint上链，对于历史版本，需要指定额外配置来生成初始信任的检查点

2.3 停止旧版本集群
------------------

保证旧版本集群中的所有节点在\ **同一高度**\ 下停止

例：在集群中所有节点高度都为19时停止节点

.. code:: shell

   # 通过rpc查看集群当前状态
   curl <ip地址>:<端口> --data '{"jsonrpc":"2.0","method": "node_getNodeStates","id": 1, "namespace":"<分区名>"}'

.. code:: json

   {
       "code": 0,
       "id": 1,
       "jsonrpc": "2.0",
       "message": "SUCCESS",
       "namespace": "global",
       "result": [
           {
               "blockHash": "0000000000000013eec2cc0a0866491f8d031283298a9a45a60e55eb65ca58e5",
               "blockHeight": 19,
               "checkpoint": 10,
               "epoch": 2,
               "hash": "fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a",
               "hostname": "node1",
               "id": 1,
               "status": "NORMAL",
               "view": 0
           },
           {
               "blockHash": "0000000000000013eec2cc0a0866491f8d031283298a9a45a60e55eb65ca58e5",
               "blockHeight": 19,
               "checkpoint": 10,
               "epoch": 2,
               "hash": "c82a71a88c58540c62fc119e78306e7fdbe114d9b840c47ab564767cb1c706e2",
               "hostname": "node2",
               "id": 2,
               "status": "NORMAL",
               "view": 0
           },
           {
               "blockHash": "0000000000000013eec2cc0a0866491f8d031283298a9a45a60e55eb65ca58e5",
               "blockHeight": 19,
               "checkpoint": 10,
               "epoch": 2,
               "hash": "0c89dc7d8bdf45d1fed89fdbac27463d9f144875d3d73795f64f35dc204480fd",
               "hostname": "node3",
               "id": 3,
               "status": "NORMAL",
               "view": 0
           },
           {
               "blockHash": "0000000000000013eec2cc0a0866491f8d031283298a9a45a60e55eb65ca58e5",
               "blockHeight": 19,
               "checkpoint": 10,
               "epoch": 2,
               "hash": "34d299742260716bab353995fe98727004b5c27bde52489f61de093176e82088",
               "hostname": "node4",
               "id": 4,
               "status": "NORMAL"
   		}
       ]
   }

2.4 版本升级
------------

2.4.1 二进制文件替换
~~~~~~~~~~~~~~~~~~~~

1. 在hyperchain安装目录执行stop.sh脚本

.. code:: text

   ./stop.sh

2. 将安装目录中老版本的hyperchain文件替换为要升级到的版本的hyperchain文件

2.4.2 自动生成upgrade.toml
~~~~~~~~~~~~~~~~~~~~~~~~~~

1. 在节点安装目录下，使用命令自动生成\ ``upgrade.toml``

.. code:: text

   ./hyperchain --gqc configuration

2. 查看\ ``configuration/<分区名>/upgrade.toml``\ 文件，确保配置文件已自动生成

.. code:: text

   > cat configuration/<分区名>/upgrade.toml

   [genesis_checkpoint]
     data = "0ad3031a42124030303030303030303030303030303030636165373565343962356635326231383337623133353430326536636437356231616131353363336666343231316336228c030a610a056e6f64653112583056301006072a8648ce3d020106052b8104000a0342000412377b034adb5bc734e3c6e94dd798a26885234dcfbd90283c54a51b747c2247c1a1987d2bd8e1c9ea91b9627257dc9144e90a415246be1ca8de8f511085ce450a610a056e6f64653212583056301006072a8648ce3d020106052b8104000a0342000412377b034adb5bc734e3c6e94dd798a26885234dcfbd90283c54a51b747c2247c1a1987d2bd8e1c9ea91b9627257dc9144e90a415246be1ca8de8f511085ce450a610a056e6f64653312583056301006072a8648ce3d020106052b8104000a0342000412377b034adb5bc734e3c6e94dd798a26885234dcfbd90283c54a51b747c2247c1a1987d2bd8e1c9ea91b9627257dc9144e90a415246be1ca8de8f511085ce450a610a056e6f64653412583056301006072a8648ce3d020106052b8104000a0342000412377b034adb5bc734e3c6e94dd798a26885234dcfbd90283c54a51b747c2247c1a1987d2bd8e1c9ea91b9627257dc9144e90a415246be1ca8de8f511085ce45"
   >

3. 在集群所有节点上查看\ ``upgrade.toml``\ 文件，确保\ *genesis_checkpoint*\ 配置项\ **每个节点一致**

2.4.3 启动节点
~~~~~~~~~~~~~~

使用新版本二进制启动节点。

使用\ ``start.sh``\ 启动hyperchain进程:

.. code:: bash

   #根据实际情况修改/data/hyperchain
   cd /data/hyperchain
   ./start.sh
   #或者如果上面命令失败，尝试下面这个命令，待服务正常后，再使用脚本重启
   ./hyperchain

2.4.4 ca模式管理
~~~~~~~~~~~~~~~~

链完成升级后，需要通过提案设置ca模式，如果为中心ca，设置完ca模式后还需要通过新增root
ca的方式将所有使用的root ca上链。具体设置ca模式以及通过新增root
ca的方式将使用的root ca上链的流程详见《ca模式管理》

2.5 版本升级后加入新节点
------------------------

历史版本区块链集群在通过此方式升级至v2.7.0版本后，加入新节点时需要进行的额外操作

（若集群是从v2.7.0才启动的，\ **不带有历史版本的数据**\ ，则新加入的节点无需以下操作）

2.5.1 从集群现有节点获取upgrade.toml
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

拷贝集群现有节点安装目录下的\ ``configuration/<分区名>/upgrade.toml``\ 到新节点安装目录下的分区目录\ ``configuration/<分区名>/``

2.5.2 配置ns_static.toml
~~~~~~~~~~~~~~~~~~~~~~~~

1. 查询集群现有节点的区块与世代信息

.. code:: shell

   # 通过rpc查看集群当前状态
   curl <ip地址>:<端口> --data '{"jsonrpc":"2.0","method": "node_getNodeStates","id": 1, "namespace":"<分区名>"}'

**例：**\ 当前查询到区块高度\ ``blockHeight``\ 为\ *13*\ ，区块哈希\ ``blockHash``\ 为\ *000000000000000dafd619e625e629e4f79d2acf4782220aa030bf411e8aef21*\ ，世代\ ``epoch``\ 为\ *2*

.. code:: json

   {
       "code": 0,
       "id": 1,
       "jsonrpc": "2.0",
       "message": "SUCCESS",
       "namespace": "global",
       "result": [
           {
               "blockHash": "000000000000000dafd619e625e629e4f79d2acf4782220aa030bf411e8aef21",
               "blockHeight": 13,
               "checkpoint": 13,
               "epoch": 2,
               "hash": "fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a",
               "hostname": "node1",
               "id": 1,
               "status": "NORMAL",
               "view": 0
           },
           {
               "blockHash": "000000000000000dafd619e625e629e4f79d2acf4782220aa030bf411e8aef21",
               "blockHeight": 13,
               "checkpoint": 13,
               "epoch": 2,
               "hash": "c82a71a88c58540c62fc119e78306e7fdbe114d9b840c47ab564767cb1c706e2",
               "hostname": "node2",
               "id": 2,
               "status": "NORMAL",
               "view": 0
           },
           {
               "blockHash": "000000000000000dafd619e625e629e4f79d2acf4782220aa030bf411e8aef21",
               "blockHeight": 13,
               "checkpoint": 13,
               "epoch": 2,
               "hash": "0c89dc7d8bdf45d1fed89fdbac27463d9f144875d3d73795f64f35dc204480fd",
               "hostname": "node3",
               "id": 3,
               "status": "NORMAL",
               "view": 0
           },
           {
               "blockHash": "000000000000000dafd619e625e629e4f79d2acf4782220aa030bf411e8aef21",
               "blockHeight": 13,
               "checkpoint": 13,
               "epoch": 2,
               "hash": "34d299742260716bab353995fe98727004b5c27bde52489f61de093176e82088",
               "hostname": "node4",
               "id": 4,
               "status": "NORMAL",
               "view": 0
           },

2. 修改\ ``configuration/<分区名>/ns_static.toml``

修改[**sync.target**]配置项：

::

   - 将epoch设置为步骤1查询到的epoch

   - 将height设置为步骤1查询到的blockHeight

   - 将hash设置为步骤1查询到的blockHash

.. code:: text

   [sync]
   	[sync.target]
   		epoch  = 0
   		height = -1
   		hash = ""

2.5.3 启动节点
~~~~~~~~~~~~~~

使用新版本二进制启动节点

使用\ ``start.sh``\ 启动hyperchain进程:

.. code:: bash

   #根据实际情况修改/data/hyperchain
   cd /data/hyperchain
   ./start.sh
   #或者如果上面命令失败，尝试下面这个命令，待服务正常后，再使用脚本重启
   ./hyperchain


