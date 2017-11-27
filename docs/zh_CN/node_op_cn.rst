节点操作
========

1. 添加节点
-----------

添加节点需要涉及以下三个文件的配置： 

- ``peerconfig.toml``: 对端配置文件，用于配置逻辑对端节点连接信息。 
- ``hosts.toml``: 主机配置文件，用于配置物理对端节点连接信息，包括节点主机与IP地址的映射。
- ``addr.toml``: 地址配置文件，用于配置物理互连地址信息，包括域名与IP地址的映射。当对端节点反向连接的时候需要用到此文件的配置。

Example1: 在已有4个VP节点的基础上，添加第五个VP节点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

新增的VP节点在启动前需配置好以下配置文件：

1. ``peerconfig.toml``

.. code:: yaml


    [[nodes]]
      hostname = "node1"
      id = 1
      static = true

    [[nodes]]
      hostname = "node2"
      id = 2
      static = true

    [[nodes]]
      hostname = "node3"
      id = 3
      static = true

    [[nodes]]
      hostname = "node4"
      id = 4
      static = true

    [[nodes]]
      hostname = "node5"
      id = 5
      static = true

    [self]
      caconf = "config/namespace.toml"
      hostname = "node5"
      id = 5        # 节点ID  
      n = 5         # 需要连接的vp的个数
      new = true    # 是否为新加入节点
      org = false   # 是否为创世节点
      rec = false   # 是否重连
      vp = true     # 是否为VP节点

2. ``hosts.toml``

本文件需要配置\ **所有需要连接的节点的物理地址**\ ，hyperchain节点之间通过hostname进行相互通信，所以hostname可以配置为任意的节点通信地址。

.. code:: yaml

    hosts = [
    "node1 127.0.0.1:50011",
    "node2 127.0.0.1:50012",
    "node3 127.0.0.1:50013",
    "node4 127.0.0.1:50014",
    "node5 127.0.0.1:50015"
    ]

3. ``addr.toml``

addr采用域的形式进行声明，例如hostname为\ ``node1``\ 和\ ``node5``\ 的两个节点同属\ ``domain1``\ 而其他节点都分别属于不同的domain下，则配置文件应该按照如下配置（本配置文件为\ ``node5``\ 的\ ``addr.toml``\ ）：

.. code:: yaml

    addrs = [
    "domain1 127.0.0.1:50015",
    "domain2 192.168.100.20:50015",
    "domain3 202.110.20.13:50015",
    "domain4 127.0.0.1:50015",
    ]
    domain = "domain1"

.. Note:: 重申一下，\ ``addr.toml``\ 是为了让别的节点知道自己所在的域以及让不同的域的节点通过不同的网络地址进行互连的配置，能够允许复杂网段之间的节点互连。

Example2: 在已有4个VP节点的基础上，添加一个NVP节点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

新增的NVP节点在启动前需配置好以下配置文件：

1. ``peerconfig.toml``

.. code:: yaml

    [[nodes]]
      hostname = "node1"
      id = 1
      static = true

    [[nodes]]
      hostname = "node2"
      id = 2
      static = true

    [[nodes]]
      hostname = "node3"
      id = 3
      static = true

    [[nodes]]
      hostname = "node4"
      id = 4
      static = true

    [self]
      caconf = "config/namespace.toml"
      hostname = "node5"
      id = 0
      n = 4     # 需要连接的vp的个数
      new = true    # 是否为新加入节点
      org = false   # 是否为创世节点
      rec = false   # 是否重连
      vp = false    # 是否为VP节点

2. ``hosts.toml``

.. code:: yaml

    hosts = [
    "node1 127.0.0.1:50011",
    "node2 127.0.0.1:50012",
    "node3 127.0.0.1:50013",
    "node4 127.0.0.1:50014",
    "node5 127.0.0.1:50015"
    ]

3. ``addr.toml``

.. code:: yaml

    addrs = [
    "domain1 127.0.0.1:50015",
    "domain2 127.0.0.1:50015",
    "domain3 127.0.0.1:50015",
    "domain4 127.0.0.1:50015",
    "domain5 127.0.0.1:50015"
    ]
    domain = "domain5"

2. 删除节点
-----------

我们将删除节点分为三种情况：

1. VP断开与某个VP的连接；
2. VP主动断开与NVP的连接；
3. NVP主动断开与VP的连接；

第二种和第三种的结果是一样的，均能使NVP不再同步VP数据、NVP不再转发交易给VP。

在下面的例子中，我们均假设各个节点的JSON-RPC API服务端口映射如下：

-  1号节点：\ ``8081``
-  2号节点：\ ``8082``
-  3号节点：\ ``8083``
-  4号节点：\ ``8084``
-  5号节点：\ ``8085``

Example1: VP断开与其他VP的连接
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

例如，当前有5个VP节点，现在要删除5号VP节点。

首先，获取要删除的5号VP节点的哈希，

.. code:: bash

    # Request
    curl -X POST -d '{"jsonrpc":"2.0","method":"node_getNodeHash","params":[],"id":1, "namespace":"global"}' localhost:8085

    # Response
    {
      "jsonrpc": "2.0",
      "namespace": "global",
      "id": 1,
      "code": 0,
      "message": "SUCCESS",
      "result": "55d3c05f2c24c232a47a1f1963ace172b21d3a2ec0ac83ea075da2d2427603bc"
    }

然后，分别向1、2、3、4、5号VP节点发送删除节点请求，

.. code:: bash

    # Request
    curl -X POST -d '{"jsonrpc":"2.0","method":"node_deleteVP","params":[{"nodehash":"55d3c05f2c24c232a47a1f1963ace172b21d3a2ec0ac83ea075da2d2427603bc"}],"id":1, "namespace":"global"}' localhost:8081/8082/8083/8084/8085

    # Response
    {
      "jsonrpc": "2.0",
      "namespace": "global",
      "id": 1,
      "code": 0,
      "message": "SUCCESS",
      "result": "successful request to delete vp node, hash 55d3c05f2c24c232a47a1f1963ace172b21d3a2ec0ac83ea075da2d2427603bc"
    }

当在终端看到以下日志时，说明VP节点删除成功。

.. code:: bash

    global::p2p 12:33:17.709 DELETE NODE 55d3c05f2c24c232a47a1f1963ace172b21d3a2ec0ac83ea075da2d2427603bc 
    global::p2p 12:33:17.709 delete validate peer 5

Example2: VP断开与NVP的连接
^^^^^^^^^^^^^^^^^^^^^^^^^^^

例如，当前有4个VP节点和1个NVP节点，NVP节点与1号节点相连。

首先，获取NVP节点的哈希，

.. code:: bash

    # Request
    curl -X POST -d '{"jsonrpc":"2.0","method":"node_getNodeHash","params":[],"id":1, "namespace":"global"}' localhost:8085

    # Response
    {
      "jsonrpc": "2.0",
      "namespace": "global",
      "id": 1,
      "code": 0,
      "message": "SUCCESS",
      "result": "4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad"
    }

然后，向1号节点发送删除NVP的请求，

.. code:: bash

    # Request
    curl -X POST -d '{"jsonrpc":"2.0","method":"node_deleteNVP","params":[{"nodehash":"4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad"}],"id":1, "namespace":"global"}' localhost:8081

    # Response
    {
      "jsonrpc": "2.0",
      "namespace": "global",
      "id": 1,
      "code": 0,
      "message": "SUCCESS",
      "result": "successful request to delete nvp node, hash 4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad"
    }

在1号VP节点终端看到以下日志，

.. code:: bash

    global::p2p 13:28:02.857 delete NVP peer, hash 4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad, vp pool size(4) nvp pool size(0) 

同时，NVP节点也会打印以下日志说明与1号节点已经断开连接。

.. code:: bash

    global::p2p 13:28:02.858 peers_pool.go:244 delete validate peer 1 

说明NVP节点删除成功。

Example3: NVP断开与VP的连接
^^^^^^^^^^^^^^^^^^^^^^^^^^^

这种情况与Example2差不多，不同的是我们这次是向NVP节点发送删节点请求。

例如，当前有4个VP节点和1个NVP节点，NVP节点与1号节点相连。

首先，获取1号VP节点的哈希，

.. code:: bash

    # Request
    curl -X POST -d '{"jsonrpc":"2.0","method":"node_getNodeHash","params":[],"id":1, "namespace":"global"}' localhost:8081

    # Response
    {
        "jsonrpc": "2.0",
        "namespace": "global",
        "id": 1,
        "code": 0,
        "message": "SUCCESS",
        "result": "fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a"
    }

然后，向NVP节点发送删除VP的请求，

.. code:: bash

    # Request
    curl -X POST -d '{"jsonrpc":"2.0","method":"node_deleteVP","params":[{"nodehash":"fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a"}],"id":1, "namespace":"global"}' localhost:8085

    # Response
    {
        "jsonrpc": "2.0",
        "namespace": "global",
        "id": 1,
        "code": 0,
        "message": "SUCCESS",
        "result": "successful request to delete vp node, hash fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a"
    }

在NVP节点终端看到以下日志，

.. code:: bash

    global::p2p 13:47:17.744 delete validate peer 1 

同时，在1号VP节点终端看到以下日志，

.. code:: bash

    global::p2p 13:47:17.744 delete NVP peer, hash 4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad, vp pool size(4) nvp pool size(0)

说明VP节点删除成功。
