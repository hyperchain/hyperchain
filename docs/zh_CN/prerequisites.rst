准备工作
========

操作系统版本要求
----------------

以下图表说明了Hyperchain对于不同操作系统的版本要求。

**不同平台版本要求** 

+----------+--------------+------------+
| 操作系统 |  系统版本    | 系统架构   |
+==========+==============+============+
| RHEL     | 6 或更新     | amd64, 386 |
+----------+--------------+------------+
| CentOS   | 6 或更新     | amd64, 386 |
+----------+--------------+------------+
| SLES     | 11SP3 或更新 | amd64, 386 |
+----------+--------------+------------+
| Ubuntu   | 14.04 或更新 | amd64, 386 |
+----------+--------------+------------+
| macOS    | 10.8 或更新  | amd64, 386 |
+----------+--------------+------------+

安装Go语言开发环境
------------------

因为Hyperchain使用Go语言来实现它的各个组件，所以需要安装Go语言开发环境。

下载 Go
```````

Go为Mac OS X、Linux和Windows提供二进制发行版。如果您使用的是不同的操作系统，您可以下载Go源代码并从源代码安装。

在这里下载适用于您的平台的最新版本Go：\ `下载 <https://golang.org/dl>`__
- 请下载 ``1.7.x 或更新``

安装 Go
```````

请按照对应于您的平台的步骤来安装Go环境：\ `安装Go <https://golang.org/doc/install#install>`__\ ，推荐使用默认配置安装。

-  对于Mac OS X 和
   Linux操作系统，默认情况下Go会被安装到\ ``/usr/local/go/``\ ，并且将环境变量\ ``GOROOT``\ 设置为该路径\ ``/usr/local/go``.

.. code:: bash

    export GOROOT=/usr/local/go

-  同时，请添加路径 ``GOROOT/bin``
   到环境变量\ ``PATH``\ 中，可以使Go工具正常执行。

.. code:: bash

    export PATH=$PATH:$GOROOT/bin

设置 GOPATH
```````````
您的Go工作目录 (``GOPATH``)
是用来存储您的Go代码的地方，您必须要将他跟您的Go安装目录区分开
(``GOROOT``)。

以下命令是用了设置您的\ ``GOPATH``\ 环境变量的，您也可以参考Go官方文档，来获得更详细的内容:
https://golang.org/doc/code.html.

-  对于 Mac OS X 和 Linux 操作系统 将 ``GOPATH``
   环境变量设置为您的工作路径：

.. code:: bash

    export GOPATH=$HOME/go

-  同时添加路径 ``GOPATH/bin``
   到环境变量\ ``PATH``\ 中，可以使编译后的Go程序正常执行。

.. code:: bash

    export PATH=$PATH:$GOPATH/bin

-  由于我们将在Go中进行一系列编码，您可以将以下内容添加到您的\ ``~/.bashrc``\ 文件中：

.. code:: bash

    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export PATH=$PATH:$GOPATH/bin:$GOROOT/bin

检查Go安装结果
``````````````

创建和运行这里描述的hello.go应用:
https://golang.org/doc/install#testing.

如果您正确设置了Go运行环境，您应该能够从任何目录运行hello程序，并看到程序成功执行。

安装 Go vendor
--------------

Go
vendor是管理包及其依赖项的工具。此工具将依赖的包复制到项目的\ ``vendor``\ 目录中，并将其版本记录在名为\ ``vendor.json``\ 的文件中。

安装命令
````````

.. code:: bash

    go get -u github.com/kardianos/govendor

检查Go vendor安装结果
`````````````````````

为了要验证您的govendor安装正确，可以通过查看govendor版本信息来检验。

在命令提示符下，键入以下命令并确保您看到了govendor版本信息：

.. code:: bash

    $ govendor --version
    v1.0.9

更多信息
````````

您可以转到项目的主页了解更多细节。 - `Go
vendor <https://github.com/kardianos/govendor>`__

安装合约编译器(可选)
--------------------

Hyperchain
支持用\ `Solidity <https://solidity.readthedocs.org/en/latest/>`__\ 编写的智能合约，然后将它编译为字节码并部署到区块链中。

鉴于我们是用Solidity语言编写的合约，所以需要确保我们已经安装名为\ ``solc``\ 的合约编译器。

我们已经在源码中提供了一些平台的通用安装包，您可以直接使用他们来快速安装
``solc`` ，您也可以参考官方文档来完成安装 -
`安装Solidity <https://solidity.readthedocs.io/en/latest/installing-solidity.html#installing-solidity>`__.
