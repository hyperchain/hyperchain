.. _Zhang-to-encrypt:

账本加密使用手册
^^^^^^^^^^^^^^^^^^

1. 引言
===========

1.1 编写目的
---------------

账本加密是Hyperchain目前为用户提供的隐私保护的方案，在数据库层面对业务数据进行加密保护，确保数据写入磁盘后的安全性和隐私性。

1.2 核心特性
--------------

目前账本加密主要分为对filelog和multicache(leveldb)类型的数据库进行加密。**filelog存储区块（block）、日志（journal）、回执（receipt）数据，multicache主要存储账本（state）、账户（account）、链级元数据（meta）、Did以及Credential数据。**

在当前版本，hyperchain仅支持软件层面加密。

2. 使用方式
=============

在使用本功能前，需要 **注意以下几点：**

    - 加密功能不允许修改，即一旦打开或关闭了加密功能并启动节点，不允许再次变更配置；

    - 要求区块链集群中所有节点拥有相同的加密配置，否则涉及到数据同步时可能会出现数据不一致的问题；

    - 打开加密会造成额外的性能损失和CPU开销。

2.1 filelog加密
-----------------

在 `ns_static.toml` 配置文件中，加入如下字段，即可打开filelog加密功能::

    [encryption.TEE]
        DataEncrypt = true

2.2 multicache加密
----------------------

在 `ns_static.toml` 配置文件中，加入如下字段，即可打开对应数据库的加密功能::

    [database.state]
        encrypt = true
    [database.account]
        encrypt = true
    [database.chain]
        encrypt = true
    [database.meta]
        encrypt = true
    [database.did]
        encrypt = true
    [database.credential]
        encrypt = true


