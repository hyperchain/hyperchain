安全审计
^^^^^^^^^^^^^

其他审计使用手册：
-------------------

ELK审计使用手册
>>>>>>>>>>>>>>>>>>

:ref:`ELK审计使用手册 <ELK-Audit-Manual>`

Graylog审计使用手册
>>>>>>>>>>>>>>>>>>>>>>

:ref:`Graylog审计使用手册 <Graylog-Audit-Manual>`

[附录]审计内容列表
>>>>>>>>>>>>>>>>>>>

:ref:`[附录]审计内容列表 <Audit-list-Manual>`


功能概述
------------------

系统审计指对一个信息系统的运行状况进行检查与评价，以判断信息系统是否能够保证资产的安全、数据的完整以及有效率利用组织的资源并有效果地实现组织目标。

审计功能对区块链系统进行审计，对整个系统的运行过程进行数据采集记录，审计员可以根据这些审计记录进行分析以完成对区块链系统的审计。在趣链区块链平台中， **审计日志统一以审计事件的形式表示** ，审计事件主要为平台中各种的消息通信事件，例如外部的请求调用事件，各个模块之间的消息通信事件、账本数据变更事件、系统异常事件等。

审计后端
>>>>>>>>>>>>>>>>>>>

审计后端的功能是将平台产生的审计日志进行存储，目前平台支持的审计后端有三种类型：

- **文件** ：文件后端直接将审计日志以json的格式写入日志文件;
- **filelog** ：将审计日志写入filelog存储，支持存储加密;
- **webhook** ：将审计日志发送到外部。

目前，webhook方式对两个常用的分析平台做了适配：

- **ELK** ：将审计日志直接发送到elk（集群）的URL；
- **GrayLog** ：将审计日志直接发送到graylog（集群）的URL。


审计级别
>>>>>>>>>>>>>>>>
当前审计功能提供如下四种级别的审计:

- **None** ：不记录审计日志；
- **Metadata** ：只提供基础的事件信息，包括事件类型，时间戳，事件具体内容等；如果是外部请求事件，则还会包括ip，证书，请求url，响应码；
- **ResponseBody** ：对于api外部请求（如客户端jsonRPC调用）增加请求响应，对于内部事件为事件结果（对于能同步获取结果的事件），比如数据同步的时间消耗；
- **StorageObjec** ：最全记录方式，提供状态修改的变更。

用户可根据审计需求开启适当的审计级别。 **注意：审计级别越高则输出的内容越多，对平台的性能影响也越大。**


审计事件格式
>>>>>>>>>>>>>>>>>>>>

审计事件包含字段如下::

    {
        "source":"node1",      # 审计日志来源节点名称
        "namespace":"global",  # 产生审计日志的namespace
        "audit_type_id":400,   # 审计记录的类型ID，参照审计类型表
        "tags":[               
            "xxx"              # 审计记录的tag，可方便检索
        ],
        "risk_level":0,        # 审计日志等级
        "user":{               # 审计日志发起者，外部请求会填充此字段
            "ip":"127.0.0.1:50444","cert":"xxx"
        },
        "event_body":{         # 审计日志主题，包含审计级别request的主要内容		                
            "value":{          # value字段为对象类型，body为额外的key,value类型
                "replica_id":3,"view":2
            },
            "body":{
            "nodeHash":"c82a71a88c58540c62fc119e78306e7fdbe114d9b840c47ab564767cb1c706e2"
            }
        }，    
        "event_response":{       # 对应审计级别responsebody的主要内容
            "body":{
                "jsonrpc":"2.0","namespace":"global","id":1,"code":-32003,"message":"Invalid signature: tx hash 0x30693b679e8e5cb6fb61c3c4fed96616fb6aedcde37928d23de983dc2aec75e4"
            }
        }
    }


审计日志等级
>>>>>>>>>>>>>>>>>

目前将审计日志的级别划分为三种，级别包括 **危险(RISK)、警告(WARN)、普通(NORAMAL)** ，分别对应 `risk_level` 字段的数字为 **0、1、2** 。

- RISK：指系统出现的难以恢复异常或严重错误，会影响系统的正常运行，如写块错误；
- WARN：指系统出现的可恢复异常，不会影响系统的正常运行，如交易执行失败，一些异常行为如viewchange等；
- NORAMAL：一般系统事件，如正常请求。

审计日志标签
>>>>>>>>>>>>>>>>>>

日志的标签用于将审计事件归类，以便于在搜索审计事件的时候能快速定位以及区分审计事件，为 `Tags` 字段。

当前日志标签的包含有如下::

    ReceivedMessage = "ReceivedMessage" // 代表（从其他节点或模块）接收到的消息
    SentMessage     = "SentMessage" // 代表发出的消息

每个审计事件可以有多个标签。


审计事件清单
>>>>>>>>>>>>>>>>>>>

审计事件的详细清单请点击\ `此处 <https://upload.filoop.com/%E5%AE%A1%E8%AE%A1%E4%BA%8B%E4%BB%B620201111.xlsx>`_\


安装及初始化
---------------------

配置说明
>>>>>>>>>>>>>>>>>>>>>

安全审计功能是可选的（默认不开启），即可以通过节点开关配置来决定节点是否启用审计功能，相关配置位于 ``configuration/global/system.toml`` 中。

平台默认不开启安全审计功能， **如果您想启用该功能，请将下述配置复制到system.toml中，并做好相应配置** 。

 ::

    [audit]
	backend   = "graylog"      #审计后端类型，可选项包括"file", "filelog" ,"graylog"和"elk"，实际不建议选用前两者
	level     = "none"         #审计的级别，级别越高输出的内容越详细，从低到高的级别依次是：none, metadata, responsebody, storageobject，其中如果将审计级别配置为none，则代表不开启审计，平台的审计服务将不会启动 
	[audit.conn.pool]
    urls  = ["172.16.5.5:12202"]   #外部URL 可配置一个或者多个，根据实际使用的审计后端配置，格式为"IP+端口号"

Filelog说明
>>>>>>>>>>>>>>>>>>>
审计会使用到filelog的场景有两种：

- **将filelog作为审计后端** ：将filelog作为审计后端时，审计日志会存储在节点的指定filelog数据库中，支持filelog加密存储。 **实际场景中，不推荐使用file或者filelog作为审计后端。**
- **使用GrayLog或者ELK作为审计后端** ：当使用Graylog或者ELK作为审计后端时，平台会使用filelog作为审计消息发送失败时的临时存储。当配置的所有url的网络连接暂时不可用时，平台会将产生的审计日志临时写入filelog，当后续与审计后端的网络连接恢复成功，filelog中的审计日志会自动恢复到配置的审计后端。

Graylog安装及使用
>>>>>>>>>>>>>>>>>>>

1. 使用Graylog作为审计后端需将配置项 `backend` 设置为 `graylog` 。 **注意**：在平台向外部URL发送失败时会暂时使用filelog存储，因此使用GrayLog作为后端仍然需要配置filelog。
2. **使用Glaylog作为后端需要事先搭建好graylog平台，详情请参考：**\ `Graylog审计使用手册 <https://upload.filoop.com/Graylog%E5%AE%A1%E8%AE%A1%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C.docx>`_\。
3. 搭建好之后将graylog的URL配置在[audit.conn.pool]配置之下的 `urls` 配置项之中即可。


ELK安装及使用
>>>>>>>>>>>>>>>>>>>>>

1. 使用ELK作为审计后端需将配置项 `backend` 设置为 `elk` 。 **注意**：在平台向外部URL发送失败时会暂时使用filelog存储，因此使用ELK作为后端仍然需要配置filelog。
2. **使用ELK作为后端需要事先搭建好ELK平台，详情请参考：** \ `ELK审计使用手册 <https://upload.filoop.com/ELK%E5%AE%A1%E8%AE%A1%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C.docx>`_\。
3. 搭建好之后，将filebeat的URL配置在 ``[audit.conn.pool]`` 配置之下的 `urls` 配置项之中，然后在filebeat的配置文件中配置上Elasticsearch(集群)的URL即可。

