数据归档
^^^^^^^^^^^^^

功能概述
------------------
数据归档功能的主要职责是将一部分旧的线上数据移到线下转存。

一次完整的归档过程包括：

- filelog数据从线上到线下的迁移，包括区块数据、区块对应的日志数据、交易回执数据。
- leveldb数据从线上到线下的迁移，主要包括交易索引数据。

新创建的数据包括：

- snapshot.meta：存储本次归档的manifest内容；
- archive.meta：存储本次archive过程中迁移的数据的统计结果；
- archive.record：为保证本次archive流程的完整性记录文件。



使用说明
------------------
用户可以直接指定一个已经被提交的区块号作为参数，归档结果为： **[创世区块, 入参区块] 的数据将被归档至线下，入参区块将成为新的创世区块**。

考虑直接归档的流程较为耗时，直接归档的请求返回和归档流程是异步的，可以通过后文提到的查询归档结果的接口查询归档是否成功。

此外，需要保证归档的区块号小于或等于最新的一个checkpoint，checkpoint为整10的数。例如目前区块高度为125，则checkpoint为120，归档的区块号必须小于等于120。


归档数据
>>>>>>>>>>>>>>>>>>>>>>>>
调用JSONRPC接口::

    curl localhost:8081 POST --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"archive_archiveNoPredict",
    "params":[5],
    "id":1}'

该请求会返回一个长度为32的随机字符串，标识当前创世区块对应的世界状态数据库，从 ``namespaces/global/data/archive`` 文件夹中可以找到 ``SNAPSHOT_FileterID`` 为名称的归档数据库。



查询归档结果
>>>>>>>>>>>>>>>>>>>>>>>>
调用JSONRPC接口::

    curl localhost:8081 --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"archive_queryArchive",
    "params":["<filterId>"],
    "id":1}'

该接口用于查询指定的filterID对应的数据，从当前的链上状态来看其归档的状态。可以通过查询请求的返回值判断当前归档的状态。



查询归档列表
>>>>>>>>>>>>>>>>>>>>>
调用JSONRPC接口::

    curl localhost:8081 --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"archive_listSnapshot",
    "params":["<filterId>"],
    "id":1}'

该接口将返回目前归档过的所有记录，即snapshot.meta中所有manifest记录。


数据库变更
>>>>>>>>>>>>>>>>>
归档成功后的数据库结构如下：

|image1|

所有归档的数据在配置的archive目录下，每一个文件夹名为filterID，其中一个为0号区块的snapshot，其余的为每次归档的数据存放目录。

第一次归档，snapshot.meta文件中会多两条记录，其中一条为当前的归档请求对应的manifest文件，另一条是为配合0号区块的世界状态数据库而存在的manifest记录，但这条记录并不表示一次实际的归档操作。




注意事项
------------------
- 各个子文件夹为各次归档产生的文件，其中accountdb与statedb文件夹存储的是snapshot数据，如果要在此基础上进一步做归档，这两个文件夹不能删除。其余文件夹的删除对于线上功能不会有任何影响。即不可删除最后一次归档产生的snapshot文件夹中的accountdb与statedb。
- **archive.meta与snapshot.meta文件为系统级别重要文件，不可删除**。
- 如果想要恢复归档完的数据到线上，需要采用运维手段调用IPC接口。
- 请在归档前估算归档的数据量，并在磁盘中预留该部分的空间，因为归档过程中会发生多一倍的数据膨胀。线下数据生成后才会删除线上数据，释放磁盘空间。


.. |image1| image:: ../../images/data_archiving1.jpg