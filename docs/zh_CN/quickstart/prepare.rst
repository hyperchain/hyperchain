.. _prepare:

######################
安装准备
######################

这里假设4台服务器的IP分别为 `node1`、 `node2` 、 `node3` 和 `node4`，操作用户都是 `flato` 。 

**该部署文档是为模拟平台上线部署的场景而编写的，所有的操作都只有普通用户权限，因此不涉及任何sudo权限的操作。不推荐使用root用户部署平台，容易养成不好的操作习惯，也不利于发现操作步骤中的错误。**

（**注：“Flato”为趣链区块链平台新版英文名称**）

下载安装包
-------------

请根据您使用的操作系统下载相应的安装包。

**方式一：感兴趣的用户可通过客服微信下载正式版**



**方式二：登录免费试用版官网下载试用版：**

(试用版官网链接)

创建使用用户及文件权限
----------------------

创建平台部署所需的用户，例如创建如下用户::
 
 用户名：flato
 密码：flato

可用如下命令创建新用户::

 sudo useradd -m -d /home/flato -s /bin/bash -k /etc/skel flato
 sudo passwd flato

修改部署路径及数据存放路径的目录权限，例如 `/opt/flato` 及 `/data/hype` ::
 
 sudo chown -R flato: /data/flato	

上传安装包
------------

登录服务器前需要上传Flato安装包和另外的小工具。

以服务器地址 `node1` ，用户名 `flato` 为例，操作步骤如下::

 #上传Flato安装包
 #具体操作时将flato-install.tar.gz换成实际安装包名，将node1换成实际服务器IP地址
 scp flato-installer.tar.gz flato@node1:~
 #上传nt工具包
 #具体操作时将node1换成实际服务器IP地址
 scp nt-linux64.tar.gz flato@node1:~

登录操作用户
--------------

 #具体操作时将node1换成服务器IP地址
 ssh flato@node1
 Password:
 #输入登录密码

重复操作
-----------

请按照 **1.2~1.3** 中的步骤，再分别登录到 `node2~node4` 上，以继续之后的操作。

