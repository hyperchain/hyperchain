.. _LP-User-Manual:

轻节点介绍
^^^^^^^^^^^

1. 引言
=========

1.1 编写目的
---------------

本文档用于向开发人员介绍LP功能和使用方式。

LP（Light Peer）不参与共识，仅通过信任的VP来同步区块头数据，并对外提供区块头数据查询、交易转发上链等服务。LP同一时刻只能连接一个VP，但可动态替换VP，VP可以连接多个LP。

1.2 相关术语
--------------

VP（Validate Peer）：验证节点

LP（Light Peer）：轻节点

1.3 连接方式
----------------

vp和lp节点互相连接一共四种方式，分别是：

1. 在lp的配置文件里增加vp信息，启动后自动连接；
2. 在vp的配置文件里增加lp信息，启动后自动连接；
3. 通过ipc命令，在vp上动态添加lp；
4. 通过ipc命令，在lp上动态添加vp；

2. 启动节点&检查证书配置
========================

2.1 VP节点部署
---------------

可以参照部署文档，启动vp节点。

2.2 LP节点部署
-----------------

LP与NVP共享一套证书体系，LP的证书配置以及生成方式与NVP一致，可以参照NVP使用手册的节点部署以及生成证书章节

3. 节点连接
=============

对于LP节点来说，若其未连接任何VP节点，也能正常工作，但不能转发交易，不能同步新数据。当要与VP节点连接时，LP节点需要提前准备和VP相连的证书文件，还需要进行相应的配置：

1. LP作为独立节点启动，需要LP节点的 `configuration/dynamic.toml` 文件中 **修改self节点名称、port端口、域配置。**

- 注意： **LP的配置信息不能和VP节点以及NVP节点的配置信息重复**

2. 在LP节点的 `configuration/global/ns_dynamic.toml` 文件 **修改[self]信息** ，具体的配置根据不同的连接方式略有不同，在下面详细介绍。

3. 配置完成之后，可以通过 `start.sh` 脚本来启动节点。由于VP和LP可以动态地建立连接，因此无须同时启动。

VP和LP节点互相连接方式一共有四种，选择任一一种方式都可以完成LP和VP的连接，下文将详细介绍每种连接方式。

3.1 在VP上配置LP
-----------------

如果要在VP启动时能够自动去连接指定的LP，需要在VP节点的 `configuration/global/ns_dynamic.toml` 文件做相应修改。在四个VP节点启动并相互连接的场景下，增加两个与node1相连接的LP需要修改的配置项如下所示::

 [[lps]]				# 运行时修改。lps数组在节点运行过程中实时变化。
 hostname	= "lp1"    # lp节点的名称
 score       = 10        # 默认为10，表示节点的优先级

 [[lps]]				# 运行时修改。lps数组在节点运行过程中实时变化。
 hostname	= "lp2"    # lp节点的名称
 score       = 10        # 默认为10，表示节点的优先级

 [p2p]

  [p2p.ip]

    [p2p.ip.remote]

      hosts = [
		"node1 127.0.0.1:50011",
		"node2 127.0.0.1:50012",
		"node3 127.0.0.1:50013",
		"node4 127.0.0.1:50014",
		"lp1 127.0.0.1:50215",
		"lp2 127.0.0.1:50216",
		]

由于LP是namespace级别的节点，所以只需要增加lp列表字段：

- [[lps]]表示lp数组，数组中记录有lp的信息。

- [p2p]的hosts列表中，增加lp的名字和地址信息字段。

其他部分的内容无需修改，[self]中的n数量指的是连接的vp数量，无需变化。

3.2 在LP上配置VP
-----------------

同理，在LP节点启动前需要指定连接的VP，先按照部署文档进行更改，然后在LP节点的 `configuration/global/ns_dynamic.toml` 文件做相应修改。 **LP只能连接一个VP** ，需要修改的配置项如下::

 [[nodes]]            # 运行时修改。nodes数组记录vp节点信息
	                 # 在节点运行过程中实时变化。
  hostname = "node1" # vp节点的名称
  score = 10         #节点的优先级，默认为10

 [p2p]

  [p2p.ip]

    [p2p.ip.remote]
		# hosts里面需要配置lp连接的vp的节点名称和ip地址
      hosts = ["node1 127.0.0.1:50011"]

 [self]
  hostname = "lp1"   # lp节点的名称
  n = 1              # 连接的vp数量(0或者1)
  type = "lp"       # 节点类型

- [[nodes]]字段，添加与LP节点连接的VP节点信息；

- [p2p]字段，同时配置VP的节点名称和ip地址；

- [self]字段，修改对应的LP信息；

  - hostname为LP节点的名称；

  - n表示要连接的VP数量为1（LP上n的数值<=1)；

  - type字段用来记录节点的类型为LP；

通过ipc命令新增LP
-------------------

上述通过配置的方式需要进行节点的启停从而使配置生效，实际上也可以通过ipc命令动态地进行LP的增删操作。

命令格式： `lp add <namespace> <hostname> <address>` 具体操作可以查看 **4.2节内容** 。

3.4 通过ipc命令新增VP
-----------------------

在lp上也可以通过3.3节介绍的命令动态地增删VP，这里不再赘述。

4.IPC命令
=============

平台提供查询LP状态、新增LP、删除LP三类IPC运维命令。

1. `lp status` ：查询LP的当前状态，在LP和VP执行返回信息不同；

2. `lp add` ：在VP上执行为新增LP，在LP上执行为新增VP；

3. `lp remove` ：在VP上执行为删除LP，在LP上执行为删除VP。

IPC命令格式如下::

 lp add <namespace> <hostname> <addr>    //新增
 lp remove <namespace> <hostname>        //删除
 lp status <nameapce> [hostname]         //状态查询

调用IPC命令前，需要先启动IPC交互式命令行::

./hyperchain -s --ipc=hpc_1.ipc

4.1 lp status
---------------

该命令可以用于查询LP和VP的连接状态。 **VP节点上可以同时得知连接的LP的状态和自己的状态；LP节点只能查询到自己的状态信息** 。

命令： `lp status <namespace> [hostname]` （该条命令可在VP和LP上执行）

**上述命令格式的具体含义如下所示：**

- `<namespace>` ：<>表示必要参数，由于LP是namespace级别的节点，因此需指定所在的namespace；

- `[hostname]` ：[]表示可选参数，在VP上调用时，若不指定hostname，将以列表形式返回所有与VP相连的LP状态信息；在LP上调用时，由于LP只能和一个VP相连，是否指定hostname对返回结果没有影响。

在LP上调用 `lp status` 命令，返回信息如下表所示：

========= ==============
返回信息  含义
========= ==============
hostname  LP连接的VP名称
lp_status LP当前的状态
height    LP当前区块高度
========= ==============

在VP上调用 `lp status` 命令，返回信息如下表所示：

========= ==========================================================
返回信息  含义
========= ==========================================================
hostname  VP连接的LP名称
vp_status VP当前的状态
lp_status LP当前的状态
height    LP当前区块高度
msg       对当前LP的描述，这份描述内容是LP的握手或区块事件的回复信息
========= ==========================================================

**以下是一些正常的状态指令实例：**

LP节点查询状态::

 # lp节点和一个vp节点相连接:格式 lp status <namespace>
 >>> lp status global
 {hostname: node1, lp_status: IDLE, height: 0}

 # lp节点和一个vp节点相连接:格式 lp status <namespace> [hostname]
 >>> lp status global node1
 {hostname: node1, lp_status: IDLE, height: 0}

VP节点查询状态::

 # 场景：vp节点，和多个lp节点相连接。
 # 使用命令格式 lp status <namespace> 查询
 >>> lp status global
 {hostname: lp1, vp_status: IDLE, lp_status: NORMAL, height: 0, msg: NULL}
 {hostname: lp2, vp_status: IDLE, lp_status: NORMAL, height: 0, msg: NULL}

 # 使用命令格式 lp status <namespace> [hostname]查询其中一个节点信息
 >>> lp status global lp2
 {hostname: lp2, vp_status: IDLE, lp_status: NORMAL, height: 0, msg: NULL}

4.2 lp add
-------------

该命令用于动态增加节点场景，VP和LP都可以使用该指令动态增加节点。

命令： `lp add <namespace> <hostname> <address>`

**上述命令格式的具体含义如下所示：**

- `<namespace>` ：<>表示必要参数，由于LP是namespace级别的节点，因此需指定所在的namespace；

- `<hostname>` ：<>表示必要参数，指定要连接节点的名称；

- `<address>` ：<>表示必要参数，指定要连接节点的ip地址；

**注意：新增命令的成功返回并不意味着连接建立成功**，若在VP上执行新增命令，则可以通过4.1节介绍的 `lp status` 命令查询LP状态，若查询结果显示 `lp_status=NORMAL` 则代表新增成功，而若一直处于 `ABNORMAL` 状态，则需要根据日志进一步排查问题；而由于LP端无法查询VP状态，因此若在LP上执行新增命令，是否连接成功只能通过日志来进行确认。

**LP连接VP示例** ::

 # 初始阶段，lp没有连接任何vp
 >>> lp status global
 this lp connects no vp

 # 正确使用add命令，连接node1，此时不代表新增node1成功
 >>> lp add global node1 127.0.0.1:50011
 success

 # 使用状态查询，发现成功连接vp节点
 >>> lp status global
 {hostname: node1, lp_status: IDLE, height: 0}


4.3 lp remove
-----------------

该指令用于VP和LP之间断开连接，VP和LP都可以使用该指令动态删除节点。

ipc命令格式： `lp remove <namespace> <hostname> `

- `<namespace>` ：<>表示必要参数，由于LP是namespace级别的节点，因此需指定其所在的namespace；

- `<hostname>` ：<>表示必要参数，指定要连接节点的名称；

该命令的行为如下：

- 通知对端进行同样的删除操作；

- 断开网络逻辑连接；

- 删除对端配置文件信息；

- 清空相应的缓存；

- 进行证书吊销；

一般情况下该命令有如下两种返回值：

- `remove [hostname] success:` 这意味着成功通知到对方，双方都会执行上面提到的删除流程；

- `inform [hostname] to delete failed, maybe need manual operation:` 这意味着由于网络或其他节点异常问题，导致未能成功通知到对端进行删除，在这种情况下，本地仍然会执行上述删除流程， **可能带来的影响是** 对端的配置文件或内存中仍然保留本节点的信息，因此仍然会尝试进行连接，但由于本节点已经进行了证书吊销，连接不会建立成功， **解决方法是在对端重新执行删除命令** 。

**VP删除LP示例** ::

 # vp节点查询状态，显示和多个lp节点连接
 >>> lp status global
 {hostname: lp1, vp_status: IDLE, lp_status: NORMAL, height: 0, msg: NULL}
 {hostname: lp2, vp_status: IDLE, lp_status: NORMAL, height: 0, msg: NULL}

 # 删除其中一个LP节点
 >>> lp remove global lp1
 remove [lp1] success

 # 删除成功，查询状态
 >>> lp status global
 {hostname: lp2, vp_status: IDLE, kp_status: NORMAL, height: 0, msg: NULL}

 # 删除第二个LP节点
 >>> lp remove global lp2
 remove [lp2] success

 # 删除成功，查询状态
 >>> lp status global
 this vp connects no nvp or lp

5. 操作实例
================

这章会列举一些实际操作场景。

5.1 LP动态切换VP
-------------------

场景：LP通过IPC命令动态的切换VP。

IPC命令如下::

 >>> lp status global
 {hostname: node1, lp_status: IDLE, height: 0}

 >>> lp remove global node1
 remove [node1] success

 >>> lp status global
 this lp connects no vp

 >>> lp add global node2 127.0.0.1:50012
 add node2 success

 >>> lp status global
 {hostname: node2, lp_status: IDLE, height: 0}

**使用过程介绍** ::

1. LP目前已经有连接的VP，通过状态查询命令可以查询到当前连接的VP节点信息；
2. LP执行指令删除自己所连接的VP节点： `lp remove global node1` ；
3. LP节点收到返回信息 `remove [node1] success` 表明成功删除；
4. 使用状态查询命令查询当前状态，验证已经删除成功，此时应该没有连接任何VP节点，结果应返回 `this lp connects no vp` ；
5. 通过add指令，添加新的想要连接的VP节点；添加新的节点之前需要根据 **第二章节进行证书配置** ；
6. 如果VP节点未启动，那么需要先启动该VP节点，节点配置 **参考部署文档** ；
7. 在LP节点执行 `lp add global node2 127.0.0.1:50012` 命令；
8. 等待返回 `add node2 success` ，说明和node2连接的准备工作完成，但此时还不证明已经连接成功；
9. 使用状态查询命令查询当前状态，验证已经添加成功，此时应该成功连接新的VP节点，结果应返回LP的状态信息： `{hostname: node2, lp_status: IDLE, height: 0}` ；

6.异常处理
===========

6.1 指令格式输入错误
---------------------

- 命令长度/类型出现问题::

 # 命令长度不够，小于3
 >>> lp status
 Error: invalid command

 # 命令中的namespace不存在
 >>> lp status g
 Error: namespace [g] not exists

 # 命令中长度正确但是指令类型不支持，目前仅支持add、remove、status三种
 >>> lp type global
 Error: invald command

- add指令格式有误::

 # 如namespace或者hostname的参数为空，或者输入的ip地址/端口号有问题
 >>> lp add global   127.0.0.1:500231
 Error: invalid command

 # 命令格式不正确，add命令长度必须为5，即lp add namespace hostname addr
 >>> lp add global
 Error: invalid command: expected length: 5

 >>> lp add global node1
 Error: invalid command: expected length: 5

 # 端口号异常
 >>> lp add global node1 127.0.0.1：50011
 Error: Uknown rune: 65306


**解决方案：** 参照 **第4章节** ，正确输入指令

6.2 查询状态时VP与LP并未互相连接
-------------------------------

 在状态查询指令之中，输入 `lp status <namespace> [hostname]` 指令，如果查询询不到，根据节点类型返回 `this vp connects no nvp or lp/this lp connects no vp`

**解决方案：** 使用添加连接命令，进行节点连接。

6.3 重复添加存在节点
--------------------

 ::

 # 如果输入指令中的hostname和现在已经启动的vp/lp的hostname重复将会报错
 >>> lp add global node1 127.0.0.1:50023
 Error: hostname collide with existed vp


**解决方案：** 检查LP，VP，CVP各个节点的配置文件，查看是否有重名现象，进行更改。

**6.4 删除并不存在的节点**
---------------------------

 ::

 # 删除并不存在的节点
 >>> lp remove global lp2
 Error: not existed: lp2

**解决方案：** 该节点不存在或已经删除成功，无法再进行重复操作，可以进行下一步操作，无需处理。

6.5 添加节点失败
----------------

在添加节点之后如果出现以下情况说明添加节点失败：

1）如果日志信息一直在显示反复连接，出现timeout等日志信息，此时表明建立连接失败；

2）如果是vp节点使用状态查询的指令，如果连接的lp节点状态一直都是 `ABNORMAL` 状态，也表明建立连接出现问题。

**解决方案：**

- 此时不做处理也可以正常运行，但是后台会一直尝试连接该节点，反复打印timeout日志信息；

- 如果想要停止打印日志信息，用户可以调用remove指令撤回上一条连接add指令。

 ::

 # 删除节点成功
 >>> lp remove global node1
 remove [node1] success

 # vp节点和lp之间的连接存在问题，需要人工介入
 >>> lp remove global lp1
 inform [lp1] to delete failed, maybe need manual operation

针对remove出错，没有成功返回success信息的场景

**解决方案：** 需要人工进行介入，lp和vp之间的连接状态不正常，需要根据日志信息进行人工介入。

6.6 查询到空区块
----------------

当调用查询区块接口后如果发现区块各值均为空或者零值，说明该区块为空区块，出现这种情况是因为vp在归档后再去连接lp，导致lp缺失了归档高度前的区块头数据::

 >>> curl http://127.0.0.1:8087 --data '[{"jsonrpc": "2.0", "id": 0, "method": "block_getBlockByNumber", "params": [10, false], "namespace": "global"}]'
 response: {'jsonrpc': '2.0', 'namespace': 'global', 'id': 0, 'code': 0, 'message': 'SUCCESS', 'result': {'version': '', 'number': '0x0', 'hash': '0x0000000000000000000000000000000000000000000000000000000000000000', 'parentHash': '0x0000000000000000000000000000000000000000000000000000000000000000', 'writeTime': 0, 'avgTime': '0x0', 'txcounts': '0x0', 'merkleRoot': '0x0000000000000000000000000000000000000000000000000000000000000000', 'txRoot': '0x0000000000000000000000000000000000000000000000000000000000000000'}}

解决方案：lp应尽可能连接有完整数据的vp，才可以同步尽可能多的区块头数据


