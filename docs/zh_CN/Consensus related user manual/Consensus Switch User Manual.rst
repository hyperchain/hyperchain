.. _Consensus-Switch-User-Manual:

共识线下切换使用手册
^^^^^^^^^^^^^^^^^^^^^

1. 引言
==========

1.1 编写目的
--------------

此文档描述Hyperchain中如何线下完成共识算法切换，在v2.7.0版本中，目前仅支持 **RBFT到NoxBFT** 的 **线下切换** 步骤

1.2 术语
-------------

+------------+--------------+-----------------------------------------+
| 英文       | 中文         | 含义                                    |
+============+==============+=========================================+
| /          | 共识算法切换 | 区块链平台在保留账本数                  |
|            |              | 据的前提下，从一种共识切换到另一种共识  |
+------------+--------------+-----------------------------------------+
| NoxBFT     | /            | Hyperchain新型拜占庭共识算法            |
+------------+--------------+-----------------------------------------+
| RBFT       | 鲁棒性拜占庭 | Hyperch                                 |
|            | 容错共识算法 | ain实现的一种强鲁棒性拜占庭容错共识算法 |
+------------+--------------+-----------------------------------------+
| epoch      | 世代         | 共识算法                                |
|            |              | 在发生集群配置变更后会触发“世代”的更替  |
+------------+--------------+-----------------------------------------+

2. 配置说明
===============

注意：只有在epoch变更后没有提交交易的场景下，才支持共识算法切换

链未进行过升级操作的场景下，共识从RBFT切换到NoxBFT仅需修改 `configuration/<分区名>/ns_dynamic.toml` 中的共识算法类型而无需额外的配置文件。

但若系统从历史版本升级到v2.7.0，则需要额外配置 `upgrade.toml` （详见 **《v2.7.0 升级说明》** ）

2.1 ns_dynamic.toml
------------------------

将 `consensus.algo` 配置项的 **RBFT** 修改为 **NoxBFT**

 ::

    [consensus]
    algo = "NoxBFT" # Support RBFT or NoxBFT

2.2 NoxBFT共识配置
---------------------

详见《 **v2.7.0 NoxBFT使用手册** 》

3. 流程说明
================

说明RBFT线下切换为NoxBFT的具体操作流程

3.1 停止RBFT集群
------------------

从历史版本升级到v2.7.0的详见3.1.1，从v2.7.0版本切换的场景详见3.1.2

3.1.1 历史版本的RBFT算法切换v2.7.0的NoxBFT
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

1. 保证集群所有节点在相同的高度停止RBFT节点
2. 保证不再有新的交易提交

例：在集群中所有节点高度都为19时停止节点

 ::

     # 通过rpc查看集群当前状态
     curl <ip地址>:<端口> --data '{"jsonrpc":"2.0","method": "node_getNodeStates","id": 1, "namespace":"<分区名>"}'

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

1. 使用v2.7.0的hyperchain二进制文件生成配置 `upgrade.toml`

在集群中每个节点根目录使用命令生成upgrade.toml

 ::

    ./hyperchain --gqc configuration


3.1.2 v2.7.0版本的RBFT算法切换到NoxBFT
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

1. 在epoch变更的区块高度停止节点，可通过配置交易触发
2. 保证epoch变更后不再有新的交易提交

例：RBFT集群在区块高度13时将epoch变更为2，此时不再提交新交易并停止节点

 ::

     # 通过rpc查看集群当前状态
     curl <ip地址>:<端口> --data '{"jsonrpc":"2.0","method": "node_getNodeStates","id": 1, "namespace":"<分区名>"}'

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

3.2 修改共识算法类型
-----------------------

修改 `configuration/<分区名>/ns_dynamic.toml` 中 `consensus.algo` 为 **NoxBFT** ::

 [consensus]
  # algo = "RBFT"
  algo = "NoxBFT"

  [consensus.pool]
    batch_size = 500
    pool_size = 50000

  [consensus.set]
    set_size = 25

 ...

3.3 重启节点
---------------

节点重启后集群即可以NoxBFT算法进行共识

