.. _node_install:

###################
安装节点
###################

首先以node1服务器为例，完成以下的安装步骤。

备份数据
-------------

在做安装操作之前，需要先检查目标目录是否有数据，如果不是首次安装，请先备份一下历史数据。

安装节点
-------------

以下步骤以安装node1上的Flato为例

首先解压安装包

::
	 
 #回到用户主目录，解压安装包
 cd
 #根据实际情况修改flato-install.tar.gz
 tar xvf flato-installer.tar.gz
 #根据实际情况修改flato-abcdef
 cd flato-abcdef

假设目标安装目录是 `/opt/flato` , 请先对照操作步骤中的 "检查服务器配置" 确定文件系统检查结果，再次确认目标目录的大小满足需求。

::

    df -h

若安装目录尚不存在，且登陆用户为非root用户，则需要使用sudo命令获取管理员权限后新建安装目录

::

    sudo mkdir /opt/flato
 
**注意，在安装之前，一定要确认好目标目录的大小，这点经常会被忽略。请务必仔细检查，以避免不必要的麻烦。**

倘若检查结果没有问题，请执行以下命令完成安装::

 ./deploy-local.sh -d /opt/flato
 #如果想直接安装到当前目录，执行以下命令：
 #./deploy-local.sh -d ./

**注意：确保操作用户对-d指定的安装目录具有可写权限，否则安装将会出错。**

部署完成可看到如下信息::

 flato has been successfully installed in: /opt/flato

 Please run these commands to start flato process:
 cd /opt/flato
 ./start.sh 

然后把之前申请的证书和license文件从本地机器复制到该节点的安装目录下（需要 **先退出用户登录在本地终端执行该命令** ，执行完毕后再登录）::

 #在本地解压证书文件
 #根据具体情况替换证书文件名字
 unzip 2019-10-31_06_43_59_allcerts.zip

解压后的2019-10-31_06_43_59_allcerts文件夹里包含了一个README文件，请先仔细阅读该文件，并按照文件内容进行操作。

::

    #上传LICENSE文件
    #根据具体情况替换LICENSE文件的名字
    scp license.zip flato@node1:/opt/falto
    #解压license文件
    unzip xvf license.zip
    #解压出的license文件名可能不是LICENSE，需要重命名
    #根据实际情况替换LICENSE_20191031文件的名字
    mv LICENSE_20191031 LICENSE

最后，再执行以下命令，完成Flato节点的安装::

 source ~/.bashrc

验证安装是否成功
--------------------

在执行完步骤3.2后，需要验证一下节点是否已经正确安装。请执行以下命令做测试::

 #/opt/flato为Flato的目标安装目录，可根据实际情况做修改
 cd /opt/flato/
 ./flato --version

假如显示正确的版本信息，说明节点安装成功，示例如下::

 $ ./flato --version
 Flato Commercial Version: 0.1

如果出现了以下报错信息，说明openssl的动态链接库没有安装成功

::

    error while loading shared libraries: libxxx. so: cannot open shared object file: No such file or directory

需要向用户目录下的`.bashrc`文件添加一行::

 #添加一个环境变量LD_LIBRARY_PATH，根据实际情况修改/opt/flato路径
 echo 'export LD_LIBRARY_PATH=/opt/flato/tools/lib/' >> ~/.bashrc
 #导出环境变量
 source ~/.bashrc

在完成以上操作之后，再执行一次 `./flato --version` ，应该就可以输出正常的版本信息了。

至此，node1服务器上的Flato节点就算完成了。