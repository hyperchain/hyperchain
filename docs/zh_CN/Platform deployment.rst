平台部署
^^^^^^^^^^^^

前言
---------

该文档将介绍如何部署一个拥有4个节点的Flato集群，操作步骤会较其他系统的部署稍繁琐一些，用户需要 **分别登录到4台服务器** 上进行操作。

这里假设4台服务器的IP分别为 `node1` 、 `node2` 、 `node3` 和 `node4` ，操作用户都是 `flato` 。 

**该部署文档是为模拟平台上线部署的场景而编写的，所有的操作都只有普通用户权限，因此不涉及任何sudo权限的操作。不推荐使用root用户部署平台，容易养成不好的操作习惯，也不利于发现操作步骤中的错误。**

第一章 获取安装包以及用户登录
--------------------------------

1.1 获取安装包
>>>>>>>>>>>>>>>>>>>

如果您已通过其他方式获取安装包请忽略此步骤。

【线下获取】对接趣链科技相关人员

【公司外部】登录服务平台官方资源库下载

- https://baas.hyperchain.cn

【公司内部】登录OA：[__https://moffi.hyperchain.cn__](https://moffi.hyperchain.cn/)

- 点击签发->平台组件->组件列表->flato->下载，选择使用您平台的flato版本下载

- 点击签发->证书->创建证书，选择V1.7+证书体系，按步骤点击后下载得到证书zip文件

- 点击签发->许可->我的许可，申请适合您需求的LICENSE，审批通过后在许可状态中下载LICENSE文件

至此，您已经获得了所有需要的安装包，注意：**证书和LICENSE文件将在flato部署完成时用到，详见3.2节**。



## 1.2 创建使用用户及文件权限

创建平台部署所需的用户，例如创建如下用户：

```bash
 用户名：flato
 密码：flato
```

可用如下命令创建新用户：

```bash
s
sudo passwd flato
```

修改部署路径及数据存放路径的目录权限，例如`/opt/flato`及`/data/hype`

```bash
sudo chown -R flato: /data/flato
```

## 1.3 上传安装包

登录服务器前需要上传Flato安装包和另外的小工具。

以服务器地址`node1`，用户名`flato`为例，操作步骤如下：

```bash
#上传Flato安装包
#具体操作时将flato-install.tar.gz换成实际安装包名，将node1换成实际服务器IP地址
scp flato-installer.tar.gz flato@node1:~
#上传nt工具包
#具体操作时将node1换成实际服务器IP地址
scp nt-linux64.tar.gz flato@node1:~
```

## 1.4 登录操作用户

```bash
#具体操作时将node1换成服务器IP地址
ssh flato@node1
Password:
#输入登录密码
```

## 1.5 重复操作

请按照**1.2~1.3**中的步骤，再分别登录到`node2~node4`上，以继续之后的操作。

# 第二章 检查系统环境

首先以`node1`服务器为例，完成以下的检查步骤。

## 2.1 检查服务器时间

检查Flato节点所在服务器的时间是否与标准时间同步，如果不同步请联络系管理员同步系统时钟。

```bash
#查看服务器时间命令
date
```

## 2.2 检查服务器配置

检查服务器配置是否与预期的配置一致，如果不一致请联系系统管理调整配置。

```bash
#查看CPU主频
cat /proc/cpuinfo | grep 'model name' | uniq
#查看CPU核数
cat /proc/cpuinfo | grep 'model name' | wc -l
#查看内存大小
#如果free -h执行失败，可以直接调用free查看
q
#查看挂载的文件系统大小
df -h
```

## 2.3 检查端口占用情况

检查Flato节点所需的端口是否被其他进程占用，如已被占用请联络系统管理员进行调整。

/检查端口是否被监听，以查看8001端口为例：

```bash
#查看端口是否被占用的命令
netstat -nap | grep 8001
```

如果存在被占用的情况，上述命令会打印出以下类似信息：

```bash
(Not all processes could be identified, non-owned process info will not be shown, you would have to be root to see it all.)tcp6       0      0 :::8001                 :::*                    LISTEN      30207/./process1
```

## 2.4 检查网络连通性

检查网络连通性的目的，就是为了检查Flato节点所监听的端口能否被其他节点访问到，如果其他节点访问不到请联络系统管理做处理。

可以使用以下三种方法检查网络连通性，`选择任意一种即可`。

- nt工具

- nc命令

- Python HTTP模块

### 2.4.1 使用nt工具测试连通性

nt是一个专门用于测试网络连通性的工具。

假设Flato节点IP地址node1~node4，需要验证node2~node4与node1上8001端口的连通性，使用方法如下：

```bash
#登录node1#具体操作时将node1换成服务器IP地址ssh flato@node1#解压nt工具包tar xvf nt-linux64.tar.gzcd nt-linux64#启动nt监听./nt server -l 0.0.0.0:8001#登录node2#具体操作时将node2换成服务器IP地址ssh flato@node2#解压nt工具包tar xvf nt-linux64.tar.gz#编辑servers.txt，向servers.txt中加入需要检测的IP:Port，本例中填入一下内容#具体操作时将node1换成服务器IP地址echo 'node1:8001' > servers.txt#检查servers.txt内容是否符合预期cat servers.txt#启动客户端测试./nt client#看到类似如下带SUCCESS字样的输出，即表明测试成功[CLIENT] TEST node1:8001    [SUCCESS] RESP: s: server_resp [0.0.0.0:8001], C->S: 0 ms, RTT: 0 ms#在node3、node4上重复在node2上操作即可#测试完之后返回到node1#按 CTRL-C 结束server监听CTRL-C
```

**nt工具支持同时检查多个IP:Port的连通性，只要在servers.txt中以每行一个IP:Port的格式填写即可。**

### 2.4.2 使用nc命令测试连通性

还可以用nc命令测试连通性，此方法的优点是操作步骤简单，但缺点是有些系统不会自带安装nc命令。

```text
#安装nc命令如下：sudo yum install -y nc
```

假设Flato节点IP地址node1~node4，需要验证node2~node4与node1上8001端口的连通性，使用方法如下：

```bash
#登录node1#具体操作时将node1换成服务器IP地址ssh flato@node1#启动nc监听, -l设置开启监听模式，-k开启支持多客户端同时连接模式，-p指定监听端口nc -l -k -p 8001#登录node2#具体操作时将node2换成服务器IP地址ssh flato@node2#使用nc命令测试连通性，-w选项设置3秒等待时间,-i选项设置连接成功后空闲等待时间(空闲超3秒即退出)#具体操作时将node1换成服务器IP地址nc -w 3 -i 3 -v node1 8001#如果出现以下带Connected字样的输出，表示测试成功。Ncat: Connected to node1:8001.Ncat: Idle timeout expired (3000 ms).#在node3、node4上重复在node2上操作即可#测试完之后返回到node1#按 CTRL-C 结束nc监听CTRL-C
```

### 2.4.3 使用Python的HTTP模块测试连通性

使用Python自带的HTTP模块也能快速开启对一个端口的监听，如果在使用上述两种方法时遇到问题，可以考虑使用此方法快速测试网络连通性。

假设Flato节点IP地址node1~node4，需要验证node2~node4与node1上8001端口的连通性，使用方法如下：

```bash
#登录node1#具体操作时将node1换成服务器IP地址ssh flato@node1#启动Python HTTP模块监听，命令如下(注意大小写)python -m SimpleHTTPServer 8001#登录node2#具体操作时将node2换成服务器IP地址ssh flato@node2#使用curl命令测试连通性#具体操作时将node1换成服务器IP地址curl node1:8001 >& /dev/null && echo yes || echo no#如果测试成功就打印yes，否则打印no#在node3、node4上重复在node2上操作即可#测试完之后返回到node1#按 CTRL-C 结束Python监听CTRL-C
```

## 2.5 检查系统字符集

`flato`节点默认使用的字符集为`UTF-8` ，请检查`SDK`或者应用服务器的默认字符集是否为`UTF-8`，如果不是，有可能造成签名非法。

```bash
Linux系统字符集查看echo $LANGLinux修改字符集vim /etc/sysconfig/i18nLANG="zh_CN.UTF-8"修改文件保存退出之后要生效要执行如下命令才可生效source /etc/sysconfig/i18n
```

## 2.6 检查最大文件句柄数

启动flato之前，需要保证文件句柄数至少为65535，否则有可能会由于文件句柄数不足引发系统宕机。

```bash
> ulimit -n65535
```

查询到的数值应至少为65535，否则，建议联系当前服务器的管理员进行修改。

## 2.7 重复操作

在完成以上步骤后，`node1`服务器的系统环境就检查完毕了。请按照**2.1~2.5**中的步骤，再分别登录到`node2~node4`上做一次检查。

# 第三章 安装节点

首先以node1服务器为例，完成以下的安装步骤。

## 3.1 备份数据

在做安装操作之前，需要先检查目标目录是否有数据，如果不是首次安装，请先备份一下历史数据。

## 3.2 安装节点

以下步骤以安装node1上的Flato为例

首先解压安装包

```bash
#回到用户主目录，解压安装包cd#根据实际情况修改flato-install.tar.gztar xvf flato-installer.tar.gz#根据实际情况修改flato-abcdefcd flato-abcdef
```

假设目标安装目录是`/opt/flato`, 请先对照操作步骤**2.2**中的文件系统检查结果，再次确认目标目录的大小满足需求。

```bash
df -h
```

若安装目录尚不存在，且登陆用户为非root用户，则需要使用sudo命令获取管理员权限后新建安装目录

```bash
sudo mkdir /opt/flato
```

**注意，在安装之前，一定要确认好目标目录的大小，这点经常会被忽略。请务必仔细检查，以避免不必要的麻烦。**

倘若检查结果没有问题，请执行以下命令完成安装：

```bash
./deploy-local.sh -d /opt/flato#如果想直接安装到当前目录，执行以下命令：#./deploy-local.sh -d ./
```

**注意：确保操作用户对-d指定的安装目录具有可写权限，否则安装将会出错。**

部署完成可看到如下信息：

```bash
flato has been successfully installed in: /opt/flatoPlease run these commands to start flato process:cd /opt/flato./start.sh 
```

然后把之前申请的证书和license文件从本地机器复制到该节点的安装目录下（需要**先退出用户登录在本地终端执行该命令**，执行完毕后再登录）：

```bash
#在本地解压证书文件#根据具体情况替换证书文件名字unzip 2019-10-31_06_43_59_allcerts.zip
```

解压后的2019-10-31_06_43_59_allcerts文件夹里包含了一个README文件，请先仔细阅读该文件，并按照文件内容进行操作。

```bash
#上传LICENSE文件#根据具体情况替换LICENSE文件的名字scp license.zip flato@node1:/opt/flato#解压license文件unzip xvf license.zip#解压出的license文件名可能不是LICENSE，需要重命名#根据实际情况替换LICENSE_20191031文件的名字mv LICENSE_20191031 LICENSE
```

最后，再执行以下命令，完成Flato节点的安装：

```bash
source ~/.bashrc
```

## 3.3 验证安装是否成功

在执行完步骤3.2后，需要验证一下节点是否已经正确安装。请执行以下命令做测试：

```bash
#/opt/flato为Flato的目标安装目录，可根据实际情况做修改cd /opt/flato/./flato --version
```

假如显示正确的版本信息，说明节点安装成功，示例如下：

```bash
$ ./flato --versionFlato Commercial Version: 0.1
```

如果出现了以下报错信息，说明openssl的动态链接库没有安装成功

```bash
error while loading shared libraries: libxxx. so: cannot open shared object file: No such file or directory
```

需要向用户目录下的`.bashrc`文件添加一行：

```bash
#添加一个环境变量LD_LIBRARY_PATH，根据实际情况修改/opt/flato路径echo 'export LD_LIBRARY_PATH=/opt/flato/tools/lib/' >> ~/.bashrc#导出环境变量source ~/.bashrc
```

在完成以上操作之后，再执行一次`./flato --version`，应该就可以输出正常的版本信息了。

至此，node1服务器上的Flato节点就算完成了。

# 第四章 检查、修改配置文件

在1.0.8版本及以后的安装包中的配置文件只包含了最精简化的配置，**安装包中的配置文件已经足够满足flato的正常使用**。若您是第一次使用flato，且希望更深入地使用flato时，可以查阅相关功能的使用手册从而知晓配置文件的修改方法；若您以前已经部署过flato，那么仍可以沿用原先的全量配置文件，只不过需要参考《配置变更》对一些配置上的变化进行确认。

**注意，以下操作都是在Flato的目标安装目录操作的，不是在原先未安装前的目录下操作。本例中，是在/opt/flato路径下检查、修改配置文件。**

安装包中的文件内容包括：

![](http://teambitiondoc.hyperchain.cn:8099/storage/012640d76a6a90aef09202ca9b19e1fb4070?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTYzNjYwNTkzNywiaWF0IjoxNjM2MDAxMTM3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzAxMjY0MGQ3NmE2YTkwYWVmMDkyMDJjYTliMTllMWZiNDA3MCJ9._ukG-fkZ1t6fq-1iA7VWkBGZAK1CrBhDSKeOQRmYdT0&download=image.png "")

## 4.1 检查LICENSE文件

由于LINCESE文件和Flato安装包不是一起打包分发的，所以在启动节点前，需要检查一下LICENSE文件是否已经更新到正确版本。

LICENSE文件位于Flato节点的根录下，文件名即LICENSE，如果不确定是否是最新版本，可以用原始的LICENSE文件再覆盖一遍。

```bash
#解压缩cd ~tar xvf LICENSE-20180701.tar.gz #解压出来后，LICENSE文件夹的名字可能是License-20180701#更新所有节点的LICENSE#根据实际情况修改License-20180701/LICENSE-abcdef和/opt/flato#拷贝命令的目标文件名，一定是LICENSEcp License-20180701/LICENSE-abcdef /opt/flato/LICENSE
```

**请依次检查4个节点的LICENSE文件。**

## 4.2 vi编辑器使用方法

下面的配置文件的编辑需要使用到vi文本编辑器，在此介绍vi的使用方法

1、使用vi命令加文件名对某个文件进行编辑，进入vi编辑文件的界面

```javascript
vi anyFile.txtna
```

2、按下i键进入编辑模式，方向键控制光标移动

3、编辑完成后，按下Esc键进入命令模式，输入:wq保存修改并退出vi

```javascript
:wq
```

4、若要放弃本次编辑，按下Esc键进入命令模式,输入:q!放弃修改并退出vi

```javascript
:q!
```

## 4.3 修改配置文件 

### **4.3.1 dynamic.toml **

编辑 `dynamic.toml`

```bash
vi configuration/dynamic.toml
```

其内容如下

```yaml
self = "node1"[port]jsonrpc     = 8081grpc        = 50011 [p2p]	[p2p.ip.self]        # domain 用于指定本地节点目前处在的网络域名称，比如本地节点目前        # 所处的域名称为“domain1”	    domain = "domain1"	    # addrs 用于指定本地节点网络可互通的网络域有哪些，并且指定了这些域        # 下的节点应该使用哪个 IP 地址来连自己（即本地节点），这个 IP 地址可能为本地        # 节点的 IP 地址，也可能是代理设备的地址        addrs = [             "domain1 127.0.0.1:50011",             "domain2 127.0.0.1:50011",             "domain3 127.0.0.1:50011",             "domain4 127.0.0.1:50011",         ][[namespace]]    name = "global"    start = true
```

- **修改port配置**

内容为：

```javascript
[port]jsonrpc     = 8081grpc        = 50011 
```

**因为我们选择单服务器单节点模式，实际上每个节点可以使用默认的port配置，但是为了介绍如何正确修改节点配置，这里还是区别一下各节点的端口，即1~4号节点分别使用为**`**xxxx1~xxxx4**`**号端口**

以2号节点为例，它的port内容如下：

```javascript
[port]jsonrpc     = 8082grpc        = 50012 # p2p
```

需要注意的是，在默认配置中除了1号节点不需要修改port配置，其他节点都要修改port配置。请依次配置剩余节点的port配置。

- **修改域配置**

以下是详细的配置说明：

```javascript
	[p2p.ip.self]        # 本节点所在域名的域名	    domain = "domain1"		# 本节点在不同的域中的地址		# 举例来说，如果节点2属于域`domain2`，那么节点2访问节点1时需要用节点1声明的在`domain2`域中对外暴露的地址，换句话说，节点2访问本节点时用的地址是`127.0.0.1:50012`。	    addrs = [	     "domain1 127.0.0.1:50012",	     "domain2 127.0.0.1:50012",	     "domain3 127.0.0.1:50012",	     "domain4 127.0.0.1:50012",	    ]
```

这里是配置是比较容易出错的地方，最简单的配置方式就是：

- 所有节点都在一个domain里：所有节点都在同一个内网环境，只要配置一个domain和该节点在这个domain里的IP地址

**请按照上述内容格式，依次更新剩余节点的域配置。**

**更复杂的网络环境下：**

在一些加入了类似Nginx代理的网络环境中，这个文件的配置极其容易出错，一般可以这样理解，服务器node1在domain1中有自己的`node1_domain1_ip`；但是在domain2中它的`node1_domain2_ip`，是它在domain2中`最内层的一个Nginx代理上，所分配的服务器node1转发地址`，domain2中其他的服务器node2、node3是通过连接最内层的Nginx上的`node1_domain2_ip`访问处于外部的node1服务器的。所以domain2中最内层Nginx上的`node1_domain2_ip`，就是node1服务器addr.toml中，该填的`domain2 node1_domain2_ip`地址。

- **修改host配置**

内容为：

```javascript
[p2p.ip.remote]hosts = [ "node1 127.0.0.1:50011", "node2 127.0.0.1:50012", "node3 127.0.0.1:50013", "node4 127.0.0.1:50014",  ]
```

配置规则很简单：`hostname ip_address:port`将所有的节点的节点名称和IP地址端口配置好即可（port为节点间通讯的端口）。

修改方法为：

- 将每行的`127.0.0.1`替换为4台服务器各自的IP地址

- 将每行的`5001x`端口换成每个Flato节点自己的grpc端口

**因为我们选择单服务器单节点模式，实际上每个节点可以使用默认的50011端口，但是为了介绍如何正确修改节点配置，这里还是将grpc端口定为**`**50011~50014**`

以服务器IP`10.10.10.1~10.10.10.4`为例，将hosts.toml文件修改为类似以下的内容：

```javascript
hosts = ["node1 10.10.10.1:50011","node2 10.10.10.2:50012","node3 10.10.10.3:50013","node4 10.10.10.4:50014"]
```

需要注意的是，4个节点的hosts配置都是一致的，请依次配置。

### 4.3.2 ns_dynamic.toml

编辑 `ns_dynamic.toml`

```bash
vi configuration/global/ns_dynamic.toml 
```

其内容如下：

```yaml
[consensus]algo = "RBFT"[self]n           = 4         # 运行时修改。表示所连vp节点的个数，该值在网络中有节点加入或退出时会实时变化。hostname    = "node1"   # 本地节点的hostnamenew         = false     # 运行时修改。新节点成功加入网络以后，该值会从true变为false。[[nodes]]hostname    = "node1"[[nodes]]hostname    = "node2"[[nodes]]hostname    = "node3"[[nodes]]hostname    = "node4"[p2p]        [p2p.ip.remote]               # hosts 用于指定本地节点启动后要向哪些节点发起连接，并且指定了通往               # 这些节点的可连通地址，这个地址可能是对端节点的 IP 地址，也可能是               # 代理设备的地址。               # 如果该列表里指定了本地节点自己的hostname和地址，则自动忽略该项。                hosts = [                 "node1 127.0.0.1:50011",                 "node2 127.0.0.1:50012",                 "node3 127.0.0.1:50013",                 "node4 127.0.0.1:50014",            ]
```

其中需要注意`[[nodes]]`配置，连接多少个VP节点，就加入多少个`[[nodes]]`部分：

```javascript
[[nodes]]  hostname = "node4"
```

**上面的**`**hostname**`**必须要与**`**dynamic.toml**`**文件中的host配置中保持一致；**

在`self`**部分需要注意的几个配置项以及配置解释**：

```javascript
[self]n         = 4           # 运行时修改。表示所连vp节点的个数，该值在有节点加入或退出时实时变化。hostname    = "node1"new         = false     # 运行时修改。新节点成功加入网络以后，该值会从true变为false。
```

**需要注意的是，本例中除了1号节点不需要修改ns_dynamic.toml，其他节点都要修改配置。请依次配置剩余节点的ns_dynamic.toml文件。**



## 4.4 修改配置文件ns_static.toml

在ns_static.toml的最上方有创世账户的默认配置，如下所示：

```javascript
[genesis][genesis.alloc]"000f1a7a08ccc48e5d30f80850cf1cf283aa3abd" = "1000000000""e93b92f1da08f925bdee44e91e7768380ae83307" = "1000000000""6201cb0448964ac597faf6fdf1f472edf2a22b89" = "1000000000""b18c8575e3284e79b92100025a31378feb8100d6" = "1000000000""856E2B9A5FA82FD1B031D1FF6863864DBAC7995D" = "1000000000""fbca6a7e9e29728773b270d3f00153c75d04e1ad" = "1000000000"
```

这些账户及其对应的余额会在区块链启动时被创建。**需要注意的是，作为默认账户，它们的私钥并不会对外暴露，因此请您自行创建创世账户，填入所有创世节点的配置文件，并妥善保管账户私钥。**



## 4.5 检查配置文件

假设服务器IP地址为`10.10.10.1~10.10.10.4`，各自使用的端口是`xxxx1~xxxx4`，以下是配置文件更新后的样例。

### 4.5.1 各节点dynamic.toml 

1号节点：

```javascript
self = "node1"[port]jsonrpc     = 8081grpc        = 50011[p2p]	[p2p.ip.self]	    domain = "domain1"	    addrs = [	     "domain1 10.10.10.1:50011",	    ][[namespace]]    name = "global"	start = true
```

2号节点：

```javascript
self = "node2"[port]jsonrpc     = 8082grpc        = 50012[p2p]	[p2p.ip.self]	    domain = "domain1"	    addrs = [	     "domain1 10.10.10.2:50012",	    ][[namespace]]    name = "global"	start = true
```

3号节点：

```javascript
self = "node3"[port]jsonrpc     = 8083grpc        = 50013[p2p]	[p2p.ip.self]	    domain = "domain1"	    addrs = [	     "domain1 10.10.10.3:50013",	    ][[namespace]]    name = "global"	start = true
```

4号节点：

```javascript
self = "node4"[port]jsonrpc     = 8084grpc        = 50014[p2p]	[p2p.ip.self]	    domain = "domain1"	    addrs = [	     "domain1 10.10.10.4:50014",	    ][[namespace]]    name = "global"	start = true
```

### 4.5.2 各节点ns_dynamic.toml 

1号节点：

```javascript
[consensus]algo = "RBFT"[self]n         = 4hostname    = "node1"new         = false[[nodes]]hostname    = "node1"[[nodes]]hostname    = "node2"[[nodes]]hostname    = "node3"[[nodes]]hostname    = "node4"[p2p]        [p2p.ip.remote]                # this node will connect to those peer, if here has self hostname, we will ignore it                hosts = [                 "node2 10.10.10.2:50012",                 "node3 10.10.10.3:50013",                 "node4 10.10.10.4:50014",            ]
```

2号节点：

```javascript
[consensus]algo = "RBFT"[self]n         = 4hostname    = "node2"new         = false[[nodes]]hostname    = "node1"[[nodes]]hostname    = "node2"[[nodes]]hostname    = "node3"[[nodes]]hostname    = "node4"[p2p]        [p2p.ip.remote]                # this node will connect to those peer, if here has self hostname, we will ignore it                hosts = [                 "node1 10.10.10.1:50011",                 "node3 10.10.10.3:50013",                 "node4 10.10.10.4:50014",            ]
```

3号节点：

```javascript
[consensus]algo = "RBFT"[self]n         = 4hostname    = "node3"new         = false[[nodes]]hostname    = "node1"[[nodes]]hostname    = "node2"[[nodes]]hostname    = "node3"[[nodes]]hostname    = "node4"[p2p]        [p2p.ip.remote]                # this node will connect to those peer, if here has self hostname, we will ignore it                hosts = [                 "node1 10.10.10.1:50011",                 "node2 10.10.10.2:50012",                 "node4 10.10.10.4:50014",            ]
```

4号节点：

```javascript
[consensus]algo = "RBFT"[self]n         = 4hostname    = "node4"new         = false[[nodes]]hostname    = "node1"[[nodes]]hostname    = "node2"[[nodes]]hostname    = "node3"[[nodes]]hostname    = "node4"[p2p]        [p2p.ip.remote]                # this node will connect to those peer, if here has self hostname, we will ignore it                hosts = [                 "node1 10.10.10.1:50011",                 "node2 10.10.10.2:50012",                 "node3 10.10.10.3:50013",            ]
```

## 4.6 检查证书配置

### 4.6.1 非分布式CA证书配置

flato在默认配置下都是以非分布式CA的方式进行启动。

在INFO或者OA上下载的V1.7+证书套件解压后会看到ca、flato、hyperchain三个目录，详细使用可见README.md。**注意下载时需要指明节点对应的节点名称（hostname），名称应该和稍后部署时填写的节点名称一致。**

其中flato目录里的证书套件用来部署flato，打开flato目录后可以看到一系列node目录，如下图所示。

![](http://teambitiondoc.hyperchain.cn:8099/storage/011wdc15e6755c5045267c28ebb6743a403a?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTYzNjYwNTkzNywiaWF0IjoxNjM2MDAxMTM3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzAxMXdkYzE1ZTY3NTVjNTA0NTI2N2MyOGViYjY3NDNhNDAzYSJ9.Sf71WT86VblDAF6rdnyU_M03B4CP4MAeMsGat2OD9TM&download=image.png "")

以节点1为例，部署时直接**将证书套件里node1目录下的CA、certs目录（如下图）放到./namespaces/global/certs/目录下**即可。将tls目录下的tlsca.ca 、tls_peer.cert、tls_peer.priv放到flato项目node1的./tls目录下即可。tls相关的配置在global.toml的p2p配置项下。

![](http://teambitiondoc.hyperchain.cn:8099/storage/011w7c53f321114635e0bad0f981d88ae965?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTYzNjYwNTkzNywiaWF0IjoxNjM2MDAxMTM3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzAxMXc3YzUzZjMyMTExNDYzNWUwYmFkMGY5ODFkODhhZTk2NSJ9.PPkhkNqzHK6Reu3YPIPeSl0fEQ3iAJVxkI5g_QY6JCY&download=image.png "")

注意事项：

- 如果发生找不到证书这类错误，请修改./configuration/global/ns_static.toml文件中的

[encryption.]配置项，修改为 `ca = "certs/CA"`

[encryption.ecert]配置项，修改为 `ecert = "certs/certs"`

- 如果节点启动报错**"the searched certificate configuration item does not match hostname : need hostname1, but hostname2"**此类的错误，请查看证书生成时是否有误。在INFO或者OA上申请SDKCERT,ECERT时，节点名称（域名）一栏需要填写每个节点对应的hostname，如下图：

![](http://teambitiondoc.hyperchain.cn:8099/storage/011vde077a93af963ac20b1d6ba4ba2db8d5?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTYzNjYwNTkzNywiaWF0IjoxNjM2MDAxMTM3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzAxMXZkZTA3N2E5M2FmOTYzYWMyMGIxZDZiYTRiYTJkYjhkNSJ9.2-HQ1GHngv3amzNDVAxjQ6irrwvQCpTz9EFBms3RC-k&download=image.png "")

### 4.6.2 分布式CA证书配置

目前分布式CA的证书能够通过certgen生成或INFO进行下载，证书放置路径和非分布式CA相同，但是需要修改./configuration/global/ns_static.toml文件中的

[distributedCA]配置项，修改为 `enable = true`

通过INFO下载分布式CA证书需要选择“分布式CA”选项：

![](http://teambitiondoc.hyperchain.cn:8099/storage/011v9ecbb85907fb4693915e8741ad99cea2?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTYzNjYwNTkzNywiaWF0IjoxNjM2MDAxMTM3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzAxMXY5ZWNiYjg1OTA3ZmI0NjkzOTE1ZTg3NDFhZDk5Y2VhMiJ9.9XJj4S12qexDWtQAMzE3XmF8jaSGGKLWCIJerW0vCQI&download=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-07-17%20%E4%B8%8B%E5%8D%888.07.28.png "")

通过certgen生成的方式需要借助于如下脚本：



下载完成后将其放到和可执行 certgen 二进制文件同一目录下，输入

```text
./gencert.sh
```

指令运行，按照提示输入相关内容即可完成证书的创建。

该shell脚本是通过调用certgen相关指令来完成创建证书的操作的，能够让操作者选择生成分布式CA或者非分布式CA的证书。无论是分布式CA还是非分布式CA都会选择是否生成国密自签证书和国密公私钥，需要说明的是，选择了生成国密自签证书会自动生成国密公私钥对，选择生成非国密自签证书会生成非国密公私钥对，两者需要配套。

对于分布式CA来说，该脚本默认生成4个CA，需要按照提示输入CA相关信息；并默认生成4个节点的证书，生成顺序为：

- node1需要生成node2.cert(root2颁发), node3.cert(root3颁发), node4.cert(root4颁发);

- node2需要生成node1.cert(root1颁发),node3.cert, node4.cert;

- node3需要生成node1.cert, node2.cert, node4.cert;

- node4需要生成node1.cert, node2.cert, node3.cert;

其中CA1与node1对应按照提示输入信息即可。

对于非分布式CA来说，默认生成一个CA，输入CA相关信息后会提示输入要生成的节点证书的数量，例如如果有5个节点需要生成证书，输入5，然后根据提示输入相关信息即可。

### 4.6.3 SOLO模式的证书说明

共识算法配置为solo的情况下启动flato时，flato单节点运行的模式称为solo模式。solo模式仅用于单节点功能的演示或者测试，不需要节点间的链接，因而我们不需要额外的证书配置。

当ns_dynamic.toml的[consensus.algo]配置为“SOLO”时，节点工作于solo模式。节点不需要任何证书的配置。

### 4.6.4不启用证书功能

当用户对区块链安全有较高要求时（例如有信息安全等级保护要求时）可能选择使用外部的硬件SSL VPN网关来保证准入控制和链路安全，这时可以手动关闭准入控制功能。方法是手动将ns_static.toml中的**[encryption.check.enable]**和**[encryption.check.enableT]**设置为false。这种情况下节点不需要配置任何证书即可启动。

```text
[encryption.check]enable     = false   #enable RCertenableT    = false  #enable TCert
```

这时节点关闭准入控制功能，但是仍然会启用链路加密。可以同将**[encryption.security. algo]**设置为pure关闭链路加密功能（默认为sm4加密保护）。

```text
[consensus]algo = "SOLO"
```

请**注意**这种情况下**请务必采取必要的外部措施保护区块链网络安全**。

# 第五章 启动节点

## 5.1 保存配置

在启动节点前，备份整个/opt/flato目录，主要是dynamic.toml和ns_dynamic.toml需要备份。

备份方法如下：

```bash
#根据实际情况修改/opt/flatocd /opt/flato/. ./tar zcvf ~/flato-backup.tar.gz flato
```

**请依次备份4个节点的Flato目标安装目录，本例中就是**`**/opt/flato**`**目录。**

## 5.2 启动节点

启动请再按照步骤`3.3`检查一次flato二进制程序能否正常执行。

```bash
#根据实际情况修改/opt/flatocd /opt/flato/./flato --version
```

检查完毕后，使用`start.sh`启动flato进程:

```bash
#根据实际情况修改/opt/flatocd /opt/flato./start.sh#或者如果上面命令失败，尝试下面这个命令#./flato start
```

**依次启动4台服务器上的Flato进程。**

## 5.3 查看日志

查看flato的日志，查看运行情况。

System级别日志的路径默认为：

`/opt/flato/system/logs`

Namespace级别日志的路径默认(以global为例)：

`/opt/flato/namespaces/global/data/logsls`

若Namespace级别日志显示如下信息，即表示节点正常加入共识网络，flato平台部署启动完成。

![](http://teambitiondoc.hyperchain.cn:8099/storage/011od45a6dc36dc1c4f1351155602c5fad69?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTYzNjYwNTkzNywiaWF0IjoxNjM2MDAxMTM3LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzAxMW9kNDVhNmRjMzZkYzFjNGYxMzUxMTU1NjAyYzVmYWQ2OSJ9.TiYp98DECRcMM1GwfE7ACZ-pYqIS7FfZrR6zazq7SPo&download=%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-12-16%2019.10.06.png "")

**请依次检查4台服务器上的Flato日志。**

## 5.4 停止节点

停止某个节点的flato，执行步骤如下：

```bash
cd /opt/flato./stop.sh#或者如果上面命令失败，尝试下面这个命令#./flato stop
```

## 5.5 重启节点

重启某个节点的flato，执行步骤如下：

```bash
cd /opt/flato./restart.sh#或者如果上面命令失败，尝试下面这个命令#./flato restart
```

## 5.6 失败恢复

若启动失败，需要使用5.1小节当中的备份进行失败恢复

将/opt/flato中的dynamic.toml和ns_dynamic.toml两个配置文件替换为备份中的相应配置文件

```javascript
tar xvf ~/flato-backup.tar.gz ~/cp ~/flato/configuration/dynamic.toml /opt/flato/configuration/dynamic.tomlcp ~/flato/configuration/global/ns_dynamic.toml /opt/flato/configuration/global/ns_dynamic.toml
```

