快速入门
========

如果您还没有完成上一篇中提到的所有\ `准备工作 <prerequisites.html>`__,
请先完成它们，再继续下一步的操作。本快速入门告诉您如何从源代码构建Hyperchain，如何启动一个Hyperchain集群。

编译Hyperchain
--------------

拉取代码
````````

克隆代码到您的\ ``GOPATH``\ 工作目录下：

.. code:: bash

    mkdir -p $GOPATH/src/github.com/hyperchain
    cd $GOPATH/src/github.com/hyperchain
    git clone https://github.com/hyperchain/hyperchain

编译代码
````````

请确保您已经安装了正确的Go工具，如有问题，请参见\ `准备工作 <prerequisites.html>`__.

编译Hyperchain：

.. code:: bash

    cd $GOPATH/src/github.com/hyperchain/hyperchain
    govendor build

您也可以执行 ``go build``\ 来编译。

启动 Hyperchain
---------------

由于Hyperchain集群需要至少4个节点建立一个BFT系统，我们建议用以下几种模式启动Hyperchain节点：

- 单服务器模式 - 本地运行4个节点 
- 多服务器模式 - 多服务器运行4个节点

单服务器模式 - 本地运行4个节点
``````````````````````````````

我们提供了一个工具脚本名为\ ``local.sh``,
可以用来快速部署运行本地4个Hyperchain节点。

.. code:: bash

    cd $GOPATH/src/github.com/hyperchain/hyperchain/scripts
    ./local.sh

如果脚本输出以下信息，说明Hyperchian节点已经正常运行了。

.. code:: bash

    $./local.sh
    ...
    ...
    start up node 1 ... done
    start up node 2 ... done
    start up node 3 ... done
    start up node 4 ... done

多服务器模式 - 多服务器运行4个节点
``````````````````````````````````

SSH免密通路
'''''''''''

因为我们使用的\ ``server.sh``\ 工具脚本在执行ssh操作时，会提示输入远程服务器
的密码，所以我们建议您打通与远程服务器之间的SSH免密通路。

1. 本地节点生成SSH秘钥对，密码请设置为空：

.. code:: bash

    ssh-keygen

    Generating public/private key pair.
    Enter file in which to save the key (/home/hyperchain/.ssh/id_rsa):
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/hyperchain/.ssh/id_rsa.
    Your public key has been saved in /home/hyperchain/.ssh/id_rsa.pub.

2. 将SSH公钥拷贝到 Hyperchain 节点,
请用您远程服务器上的用户名代替以下命令中的 ``{username}``

.. code:: bash

    ssh-copy-id {username}@node1
    ssh-copy-id {username}@node2
    ssh-copy-id {username}@node3
    ssh-copy-id {username}@node4

分发部署 Hyperchain
'''''''''''''''''''

我们提供了一个工具脚本名为\ ``server.sh``,
可以用来快速分发到4个节点部署运Hyperchain。

1. 首先请您将4台服务器的IP地址填入到
hyperchain/scripts目录下的serverlist.txt文件中。

格式如下所示：

.. code:: bash

    $ cat $GOPATH/src/github.com/hyperchain/hyperchain/scripts/serverlist.txt
    172.16.1.101
    172.16.1.102
    172.16.1.103
    172.16.1.104

2. 使用server.sh启动远程多个Hyperchain节点。

.. code:: bash

    cd $GOPATH/src/github.com/hyperchain/hyperchain/scripts
    ./server.sh

如果脚本输出以下信息，说明Hyperchian节点已经正常运行了。

.. code:: bash

    $./server.sh
    ...
    ...
    start up node 1 ... done
    start up node 2 ... done
    start up node 3 ... done
    start up node 4 ... done
