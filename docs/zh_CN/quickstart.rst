快速入门
^^^^^^^^^^^^

本章将介绍如何在本地部署一条4节点的底层链。

下载安装包
-------------

请根据您使用的操作系统下载相应的安装包。

**登录飞洛官方资源库下载试用版：**

https://www.hyperchain.cn/products/hyperchain

点击右上角 “登录服务平台” 按钮了解

安装节点
----------

首先解压安装包，并拷贝分发至4个节点目录下::

 # 创建工作目录
 mkdir /opt/workspace && cd /opt/workspace

 # 解压缩安装包
 tar xvf hpc-flato-1.0.6-fa3ca76-20210316-CentOS-6.10.tar.gz

 # 根据实际情况，将解压后的目录拷贝至四个节点目录
 cp -r flato-fa3ca76 node1/
 cp -r flato-fa3ca76 node2/
 cp -r flato-fa3ca76 node3/
 cp -r flato-fa3ca76 node4/


**注：“Flato”为趣链区块链平台新版英文名称**

通过执行安装脚本依次安装四个节点::

 cd /opt/workspace/node1/
 ./deploy-local.sh -d ./

 cd /opt/workspace/node2/
 ./deploy-local.sh -d ./

 cd /opt/workspace/node3/
 ./deploy-local.sh -d ./

 cd /opt/workspace/node4/
 ./deploy-local.sh -d ./

  source ~/.bashrc


修改配置文件
-------------

所有配置文件均保存在 `node/configuration` 中，按照本步骤对相关的配置进行修改，未在本步骤中说明的配置使用默认配置，即可完成节点的正常启动。

单服务器部署模式下，各节点互相通信的IP可以直接使用 `localhost` 或 `127.0.0.1` ，但各节点所使用的端口则必须保证不冲突，如四节点分别为 `xxxx1~xxxx4` 。

**依次修改每个节点的 `configuration/dynamic.toml` ：**

修改node1::

 self = "node1"

 [port]
 jsonrpc     = 8081
 grpc        = 50011

 [p2p]
	[p2p.ip.remote]
		hosts = [
		 "node1 127.0.0.1:50011",
		 "node2 127.0.0.1:50012",
		 "node3 127.0.0.1:50013",
		 "node4 127.0.0.1:50014",
	    ]

	[p2p.ip.self]
	   domain = "domain1",
	   addrs = ["domain1 127.0.0.1:50011"]

修改node2::

 self = "node2"

 [port]
 jsonrpc     = 8082
 grpc        = 50012

 [p2p]
	[p2p.ip.remote]
		hosts = [
		 "node1 127.0.0.1:50011",
		 "node2 127.0.0.1:50012",
		 "node3 127.0.0.1:50013",
		 "node4 127.0.0.1:50014",
	    ]

	[p2p.ip.self]
	    domain = "domain1",
	    addrs = ["domain1 127.0.0.1:50012"]

修改node3::

 self = "node3"

 [port]
 jsonrpc     = 8083
 grpc        = 50013

 [p2p]
	[p2p.ip.remote]
		hosts = [
		 "node1 127.0.0.1:50011",
		 "node2 127.0.0.1:50012",
		 "node3 127.0.0.1:50013",
		 "node4 127.0.0.1:50014",
	    ]

	[p2p.ip.self]
	    domain = "domain1",
	    addrs = ["domain1 127.0.0.1:50013"]

修改node4::

 self = "node4"

 [port]
 jsonrpc     = 8084
 grpc        = 50014

 [p2p]
	[p2p.ip.remote]
		hosts = [
		 "node1 127.0.0.1:50011",
		 "node2 127.0.0.1:50012",
		 "node3 127.0.0.1:50013",
		 "node4 127.0.0.1:50014",
	    ]

	[p2p.ip.self]
	    domain = "domain1",
	    addrs = ["domain1 127.0.0.1:50014"]

**依次修改每个节点的** `configuration/global/ns_dynamic.toml`

修改node1::

 [self]
 n           = 4,     
 hostname    = "node1",
 new         = false

修改node2::

 [self]
 n           = 4,     
 hostname    = "node2",
 new         = false

修改node3::

 [self]n           = 4 ,
 hostname    = "node3",
 new         = false

修改node4::

 [self]n           = 4,
 hostname    = "node4",
 new         = false

启动节点
---------

执行启动脚本，依次启动节点::

 # 启动节点1cd /opt/workspace/node1 && ./start.sh
 # 启动节点2cd /opt/workspace/node2 && ./start.sh
 # 启动节点3cd /opt/workspace/node3 && ./start.sh
 # 启动节点4cd /opt/workspace/node4 && ./start.sh

部署成功
----------

按照上述步骤启动节点之后，若在节点控制台日志输出中观察到如下字样，则代表节点启动成功::

 NOTI [2021-05-13T19:04:19.429] [consensus] flato-rbft@v0.2.31/exec.go:226 ======== Replica 1 finished recovery, epoch=0/view=1/height=0.NOTI [2021-05-13T19:04:19.429] [consensus] flato-rbft@v0.2.31/exec.go:227  +==============================================+  |                                              |  |            RBFT Recovery Finished            |  |                                              |  +==============================================+


