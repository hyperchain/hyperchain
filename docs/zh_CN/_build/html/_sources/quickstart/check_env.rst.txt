.. _check_env:

####################
检查系统环境
####################

首先以 `node1` 服务器为例，完成以下的检查步骤。

检查服务器时间
------------------

检查Flato节点所在服务器的时间是否与标准时间同步，如果不同步请联络系管理员同步系统时钟::

 #查看服务器时间命令
 date

检查服务器配置
------------------

检查服务器配置是否与预期的配置一致，如果不一致请联系系统管理调整配置::

 #查看CPU主频
 cat /proc/cpuinfo | grep 'model name' | uniq
 #查看CPU核数
 cat /proc/cpuinfo | grep 'model name' | wc -l
 #查看内存大小
 #如果free -h执行失败，可以直接调用free查看
 q
 #查看挂载的文件系统大小
 df -h

检查端口占用情况
--------------------

检查Flato节点所需的端口是否被其他进程占用，如已被占用请联络系统管理员进行调整。

检查端口是否被监听，以查看8001端口为例::

 #查看端口是否被占用的命令
 netstat -nap | grep 8001

如果存在被占用的情况，上述命令会打印出以下类似信息::

 (Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
 tcp6       0      0 :::8001                 :::*                    LISTEN      30207/./process1

检查网络连通性
------------------

检查网络连通性的目的，就是为了检查Flato节点所监听的端口能否被其他节点访问到，如果其他节点访问不到请联络系统管理做处理。

可以使用以下三种方法检查网络连通性， `选择任意一种即可` 。

- nt工具
- nc命令
- Python HTTP模块

使用nt工具测试连通性
>>>>>>>>>>>>>>>>>>>>>>>

nt是一个专门用于测试网络连通性的工具。

假设Flato节点IP地址node1~node4，需要验证node2~node4与node1上8001端口的连通性，使用方法如下::

 #登录node1
 #具体操作时将node1换成服务器IP地址
 ssh flato@node1
 #解压nt工具包
 tar xvf nt-linux64.tar.gz
 cd nt-linux64
 #启动nt监听
 ./nt server -l 0.0.0.0:8001
 #登录node2
 #具体操作时将node2换成服务器IP地址
 ssh flato@node2
 #解压nt工具包
 tar xvf nt-linux64.tar.gz
 #编辑servers.txt，向servers.txt中加入需要检测的IP:Port，本例中填入一下内容
 #具体操作时将node1换成服务器IP地址
 echo 'node1:8001' > servers.txt
 #检查servers.txt内容是否符合预期
 cat servers.txt
 #启动客户端测试
 ./nt client
 #看到类似如下带SUCCESS字样的输出，即表明测试成功
 [CLIENT] TEST node1:8001    [SUCCESS] RESP: s: server_resp [0.0.0.0:8001], C->S: 0 ms, RTT: 0 ms
 #在node3、node4上重复在node2上操作即可
 #测试完之后返回到node1
 #按 CTRL-C 结束server监听
 CTRL-C

**nt工具支持同时检查多个IP:Port的连通性，只要在servers.txt中以每行一个IP:Port的格式填写即可。**

使用nc命令测试连通性
>>>>>>>>>>>>>>>>>>>>>>>>>>

还可以用nc命令测试连通性，此方法的优点是操作步骤简单，但缺点是有些系统不会自带安装nc命令。

::

 #安装nc命令如下：
 sudo yum install -y nc

假设Flato节点IP地址node1~node4，需要验证node2~node4与node1上8001端口的连通性，使用方法如下::

 #登录node1
 #具体操作时将node1换成服务器IP地址
 ssh flato@node1
 #启动nc监听, -l设置开启监听模式，-k开启支持多客户端同时连接模式，-p指定监听端口
 nc -l -k -p 8001
 #登录node2
 #具体操作时将node2换成服务器IP地址
 ssh flato@node2
 #使用nc命令测试连通性，-w选项设置3秒等待时间,-i选项设置连接成功后空闲等待时间(空闲超3秒即退出)
 #具体操作时将node1换成服务器IP地址
 nc -w 3 -i 3 -v node1 8001
 #如果出现以下带Connected字样的输出，表示测试成功。
 Ncat: Connected to node1:8001.
 Ncat: Idle timeout expired (3000 ms).
 #在node3、node4上重复在node2上操作即可
 #测试完之后返回到node1
 #按 CTRL-C 结束nc监听
 CTRL-C

使用Python的HTTP模块测试连通性
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

使用Python自带的HTTP模块也能快速开启对一个端口的监听，如果在使用上述两种方法时遇到问题，可以考虑使用此方法快速测试网络连通性。

假设Flato节点IP地址node1~node4，需要验证node2~node4与node1上8001端口的连通性，使用方法如下::

 #登录node1
 #具体操作时将node1换成服务器IP地址
 ssh flato@node1
 #启动Python HTTP模块监听，命令如下(注意大小写)
 python -m SimpleHTTPServer 8001
 #登录node2
 #具体操作时将node2换成服务器IP地址
 ssh flato@node2
 #使用curl命令测试连通性
 #具体操作时将node1换成服务器IP地址
 curl node1:8001 >& /dev/null && echo yes || echo no
 #如果测试成功就打印yes，否则打印no
 #在node3、node4上重复在node2上操作即可
 #测试完之后返回到node1
 #按 CTRL-C 结束Python监听
 CTRL-C

检查系统字符集
---------------------

`flato` 节点默认使用的字符集为 `UTF-8` ，请检查 `SDK` 或者应用服务器的默认字符集是否为 `UTF-8` ，如果不是，有可能造成签名非法。

::

 Linux系统字符集查看
 echo $LANG

 Linux修改字符集
 vim /etc/sysconfig/i18n

 LANG="zh_CN.UTF-8"

 修改文件保存退出之后要生效要执行如下命令才可生效
 source /etc/sysconfig/i18n

检查最大文件句柄数
---------------------

启动flato之前，需要保证文件句柄数至少为65535，否则有可能会由于文件句柄数不足引发系统宕机。

::

 Linux检查文件句柄数
 ulimit -n

查询到的数值应至少为65535，否则，建议联系当前服务器的管理员进行修改。

重复操作
-----------

在完成以上步骤后， `node1`服务器的系统环境就检查完毕了。请按照 **2.1~2.5** 中的步骤，再分别登录到 `node2~node4` 上做一次检查。