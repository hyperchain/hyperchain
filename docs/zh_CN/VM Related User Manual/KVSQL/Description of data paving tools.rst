.. _Description-of-data-paving-tools:

数据铺底工具说明
^^^^^^^^^^^^^^^^^^

> 一键链改数据铺底工具是为了将已有的mysql数据迁移到hyperchain平台。迁移过程需要mysql自带的 `mysqldump` 工具辅助

> 迁移前请确认当前使用的mysql语法特性hyperchain平台均能支持或能够修改后支持。

1. 导出数据
===============

1.1 使用工具导出数据
-----------------------

使用mysql的数据导出工具进行数据导出，需要导出 `.sql` 文件我们推荐使用 `mysqldump` 导出工具进行导出，当前datadump工具只针对特定内容的sql文件进行处理，即导出的文件需要满足一定的格式::

 #!/bin/bash
 # 需要导出的表名列表
 tables=(component slave_worker_info slow_log tables_priv time_zone user)

 for tableName in ${tables[@]}
 do
   mysqldump -uroot mysql --add-drop-table=false --add-locks=false --complete-insert --skip-comments --compact --extended-insert=false --no-create-db ${tableName} > ${tableName}.sql
 done

注意上述使用 `mysqldump` 时必须包含这些参数，否则工具可能无法正常工作。

1.2 语法检查
-----------------

导出数据后，需要检查导出文件中是否有不支持的语法，并进行修改。此步骤需要对比hyperchain支持的mysql语法，重点确认建表语句中是否包含不支持的语法，并进行修改。 例如，hyperchain中不支持手动设置字符集，因此建表语句::

 CREATE TABLE `table1` (
    `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
    `field1` text COLLATE utf8_unicode_ci NOT NULL COMMENT '字段1',
    `field2` varchar(128) COLLATE utf8_unicode_ci NOT NULL DEFAULT '' COMMENT '字段2',
    PRIMARY KEY (`id`)
 )

需要修改为::

 CREATE TABLE `table1` (
    `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
    `field1` text NOT NULL COMMENT '字段1',
    `field2` varchar(128) NOT NULL DEFAULT '' COMMENT '字段2',
    PRIMARY KEY (`id`)
 )

2. 启动hyperchain平台
--------------------------

推荐使用SOLO节点进行数据铺底，后续再将SOLO节点的数据迁移到正式的节点

3. 修改datadump配置文件
--------------------------

工具默认的目录如下::

    ├── account.json // 配置使用的账户
    ├── conf
    │   ├── hpc.toml // 修改此文件，参见go-sdk文档
    │   ├── certs
    │   │   ├── idcert.pfx
    │   │   ├── sdkcert.cert
    │   │   ├── sdkcert.priv
    │   │   ├── sdkcert_cfca.cert
    │   │   ├── sdkcert_cfca.priv
    │   │   ├── tls
    │   │   │   ├── tls_peer.cert
    │   │   │   ├── tls_peer.priv
    │   │   │   └── tlsca.ca
    │   │   ├── unique.priv
    │   │   └── unique.pub
    │   └── keystore
    │       └── 0xfc546753921c1d1bc2d444c5186a73ab5802a0b4
    ├── datadump


4. 运行命令，进行数据铺底
=========================

命令的详细使用方式可以通过命令查看::

 $ ./datadump --help

4.1 创建库
-----------------

创建库的命令如下::

 $ ./datadump createDatabase -c ./conf -a ./account.json

执行成功后会打印一个"库"地址

4.2 插入数据到库中
--------------------

命令如下::

 $ ./datadump mysql2hpc  -c ./conf -a ./account.json -d "4.1中生成的地址" -s "步骤1中导出的sql文件夹路径" -t 2

其中 `-t` 代表并发开启的线程数量，数字越大发送的速率越快（*注意平台最好关掉流控以提高执行效率*）,命令的执行会遍历文件夹下所有 以 `.sql` 结尾的文件，并执行里面的建表语句以及 `INSERT` 语句

检查命令以及hyperchain平台日志是否有报错，直到命令执行结束。

*注意：datadump默认在发送数据的时候不查询回执，即一笔交易不确认是否执行成功，因此过程中需确认平台是否产生error, 如需在工具端查询回执实时确认执行成功与否，需开启 `--checkRecept` 选项，但同时运行效率会大大降低。*

附：
======

datadump命令参数： 子命令:

1. createDatabase
--------------------------

在平台上创建一个"库"，返回一个"库"地址 参数：

- --sdkConfig value, -c value Required, sdk configgosdk的配置文件路径

- --accountFilePath value, -a value Required, account json file包含account的account json文件路径

- --accountPassword value, -p value account passwordaccount账户的密码（如果有，没有则不必指定）

2. mysql2hpc
-------------------

将sql文件数据转移到hyperchain平台 参数：

- --sdkConfig value, -c value Required, sdk configgosdk的配置文件路径

- --accountFilePath value, -a value Required, account json file包含account的account json文件路径

- --accountPassword value, -p value account passwordaccount账户的密码（如果有，没有则不必指定）

- --databaseAddress value, -d value databaseAddress of kvsql运行 `createDatabase` 得到的地址

- --slqDir value, -s value slqfile dir with datadump output使用mysqldump导出的sql文件的目录

- --maxConcurrent value, -t value maximum number of concurrent send (default: 2)发送的最大并发数

- --checkRecept, -r whether check recept of send tx是否校验发送交易的回执（开启后发送速率将降低，但每次发送后会检查是否执行成功）

- --insertWithDBName, -i whether insert statement with db name before table name like database.table插入语句中表名前是否包含库名



工具包附件（工具需要根据最新sdk重新打包一份 否则2.7.0链使用交易签名非法）：

 `附件 <https://upload.filoop.com/RTD-Hyperchain%2Fdata_tool_0_0_2.tar.gz>`_


