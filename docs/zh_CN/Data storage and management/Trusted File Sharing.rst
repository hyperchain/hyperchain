.. _Trusted-File-Sharing:

可信文件共享
^^^^^^^^^^^^^

功能概述
------------------
平台通过自研可信文件共享功能，通过链上存证、链下传输的文件分离存储模型，实现了文件可信存储、安全共享与高效查询。

- 支持GB级别图片、音频、视频等各类文件上链存储，通过文件证明链上共识留存，保证文件内容无法篡改，真实可信；
- 支持节点白名单和用户白名单，用户可自定义授权存储节点与用户下载权限，保障文件数据安全；
- 支持主动推送文件和被动请求文件双共享模式，用户可按需查询、下载文件；
- 支持IPFS和原生文件夹双存储模式。


安装及初始化
------------------
**可信文件共享强依赖于索引数据库，因此在使用前请先完成索引数据库的安装部署**，详情请参考 :ref:`数据索引 <data-index>`。

可信文件共享是可选的，即可以通过节点开关配置来决定节点是否启用文件共享功能，相关配置位于 ``configuration/global/ns_static.toml`` 中。

平台默认不开启文件共享功能，如果您想启用该功能，请将下述配置复制到ns_static.toml中，并做好相应配置。

- ``executor.filemgr.enable``：是否启用可信文件共享功能(true, false)；
- ``executor.filemgr.file_system_mode``：文件系统类型，可选项为 origin 和 ipfs，前者为本地文件系统；
- ``executor.filemgr.ipfs_url``：若采用IPFS作为文件系统，需配置IPFS的URL；
- ``executor.filemgr.data_path``：若采用本地文件系统，需配置本地文件系统路径。

示例如下::

    [executor.filemgr]
    enable           = true
    file_system_mode = "origin"
    ipfs_url         = "localhost:5001"
    data_path        = "namespaces/global/data/filemgr"



索引数据库安装部署
>>>>>>>>>>>>>>>>>>>>>>>>>>

具体步骤请参考 :ref:`数据索引 <data-index>`。


文件系统连接
>>>>>>>>>>>>>>>>>>>
**原生文件夹模式** (试用版建议选择)

无需额外安装部署，只需确保有一个磁盘空间足够的文件夹。

在 ``global/ns_static.toml`` 配置文件中，executor.filemgr模块的file_system_mode设置为origin，data_path设置为文件存放目录即可。

**ipfs集群模式**

步骤一：部署ipfs集群

ipfs请至其官网进行下载安装：

https://github.com/ipfs/go-ipfs#install

步骤二：配置ipfs服务端口

在 ``global/ns_static.toml`` 配置文件中，executor.filemgr模块的file_system_mode设置为ipfs，data_path设置为ipfs索引存放目录（不可删除），ipfs_url设置为ipfs服务端口。



使用说明
------------------
链上大文件系统对外提供四个接口，分别是上传、下载、推送以及更新文件信息。

- 上传：同一个用户对同一个文件只能上传一次，不同用户可以分别对同一个文件上传一次。节点允许上传的并发操作。上传时，用户需要将文件的信息与文件一起发送给节点，并指定本次上传需要将文件分享给哪些节点。
- 下载：用户可以指定要下载的是哪个账户上传的哪个文件。发送的参数与上传功能一致。不同用户向不同节点的任何下载操作均可以并发。
- 推送：用于在完成上传后，可以指定让某节点将文件推送到其他节点.
- 更新：对于更新文件信息类型的操作，用户同样发送一笔标准交易。只有上传者自己可以文件信息，可以更新的内容包括文件名、文件描述、白名单列表。

采用原生文件夹origin模式时，文件最终将保存在节点配置的文件目录下，采用ipfs模式时，文件将存储在用户配置的ipfs集群中.

**可信文件共享的功能涉及到文件传输，不建议直接通过JSON-RPC接口调用，建议通过SDK使用，详情请参考** https://github.com/hyperchain/javasdk/blob/master/docs/hyperchain_litesdk_document.md



注意事项
-------------------
1. 可信文件共享功能主要消耗节点的磁盘和带宽资源，在使用该功能时，建议提高节点的磁盘和带宽资源
2. 节点对上传的文件大小没有限制，支持GB及以上的文件上传，建议文件大小在100M以下，响应时间较短
3. 当节点需要同时进行文件共享与其他交易业务时，建议打开文件传输限流开关，并配置文件限流参数。当其他业务交易频繁时，建议文件传输带宽限制在总带宽的50%以下，避免文件传输占用过多系统带宽，导致节点消息传播阻塞。

附件资源
-----------

`go-ipfs_v0.4.23_linux-amd64.tar <https://upload.filoop.com/RTD-Hyperchain%2Fgo-ipfs_v0.4.23_linux-amd64.tar>`_
`go-ipfs_v0.4.23_darwin-amd64.tar <https://upload.filoop.com/RTD-Hyperchain%2Fgo-ipfs_v0.4.23_darwin-amd64.tar>`_