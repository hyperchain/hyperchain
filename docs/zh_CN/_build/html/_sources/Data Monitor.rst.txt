数据监控
^^^^^^^^^^^^^

功能概述
------------------
Metrics数据监控平台提供一站式的数据可视化监控服务，实现数据的自动化采集及展示，帮助用户轻松了解底层平台运行情况，及时识别并处理异常。

数据监控平台由数据生成层Hyperchain、数据监控层Prometheus以及数据展示层Grafana三部组成。Hyperchain负责生成需要监控的数据并推送到Prometheus服务器中；Prometheus负责存储Hyperchian推送出来的数据，并对外提供读取接口；Grafana：负责从Prometheus服务器中读取数据，并在web上进行展示。

平台现已支持100+业务及系统层面的监控指标，指标开关灵活可配，具体可参考附录中的数据监控指标清单。

安装及初始化
------------------
为了能够完整的使用并展示数据监控的数据，除了需要应用层（即趣链区块链平台）适配并暴露一些监控数据之外，还需要下载并使用额外的两个组件：Prometheus（用于记录监控数据）以及Grafana（用于展示监控数据）。

安装Prometheus
>>>>>>>>>>>>>>>>>>>>>

**Prometheus**

Prometheus是一种时序数据库（TSDB, time-series database），用于记录平台推送出去的监控数据。

详细介绍请参考：https://prometheus.io/docs/introduction/overview/

安装下载请参考：https://prometheus.io/download/（建议安装最新版本）

Prometheus安装完成之后，需要配置好需要监听的区块链节点的地址与端口号，其配置文件默认为： ``prometheus.yml``

**node_exporter**

如果用户希望监控系统资源的使用情况（例如CPU、磁盘、网络等），我们推荐使用Prometheus官方的工具node_exporter进行监控。

安装使用教程请参考：https://prometheus.io/docs/guides/node-exporter/


安装Grafana
>>>>>>>>>>>>>>>>>>>>>
Grafana是我们推荐使用的数据监控展示工具。

安装使用教程请参考：https://prometheus.io/docs/visualization/grafana/

相关配置
>>>>>>>>>>>>>>>>>>>>>>>>>>
数据监控功能是可选的，即可以通过节点开关配置来决定节点是否启用数据监控，相关配置位于configuration/debug.toml中。

平台默认不开启数据监控功能， **如果您想启用该功能，请将下述配置复制到debug.toml中，并做好相应配置**。

- ``metrics.enable``：是否开启监控服务(true, false)；
- ``metrics.enable_expensive``：是否开启资源消耗较大的监控服务，**实际不建议开启，需谨慎选择**；
- ``metrics.port``：监控服务端口。

::

    [metrics]
    enable           = true
    enable_expensive = false
    # visit "http://127.0.0.1:'port'" to get metrics data
    port             = 9001


参考教程
>>>>>>>>>>>>>>>>>>>>>>

搭建简单的Linux服务监控：\ `参考教程 <https://blog.csdn.net/xuqide77/article/details/107850386?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-6.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-6.control>`_\


数据指标
-----------------------

指标说明 
>>>>>>>>>>>>>>>>>>>>>>

数据指标按用途不同分为外部指标和内部指标；按对平台性能影响程度又分为normal级别与expensive级别。其中，normal级别的指标不影响平台的运行， **expensive级别的指标打开后会降低平台整体的性能，因此需要用户谨慎选择开启。**

**有关于系统资源的使用情况，建议使用Prometheus官方推荐的node_exporter工具进行监控**。


系统相关指标
>>>>>>>>>>>>>>>>>>>>>>>>

所有指标以"flato_namespace"（“flato”为趣链区块链平台新版英文简称）为前缀，例如在global这namespace下所有下述指标都要加上前缀"flato_global"（“flato”为趣链区块链平台新版英文简称）。

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - execMgr_commit_txs
   - counter
   - normal
   - 此次启动后新增的交易数量
   - 可以通过rate/irate命令计算出交易TPS
 * - execMgr_total_blocks
   - gauge
   - normal
   - 当前链高度
   - 可以通过rate/irate命令计算出区块BPS
 * - rbft_batch_to_commit_duration
   - histogram
   - normal
   - 共识区块处理时间（rbft从打包到提交的时间）
   - 由于主从节点间有时差，因此以主节点的数据为主
 * - execMgr_validate_tx_time
   - histogram
   - expensive
   - 单笔交易执行时间
   - 交易级别的平均处理时间
 * - execMgr_commit_time
   - histogram
   - normal
   - 写入区块处理时间（写块时间，包括区块组成、写入等等）
   - 区块级别的平均写入时间

共识相关指标
>>>>>>>>>>>>>>>>>>>>>>>>
所有指标以"flato_namespace"（“flato”为趣链区块链平台新版英文简称）为前缀，例如在global这个Namespace下所有下述指标都要加上前缀"flato_global"（“flato”为趣链区块链平台新版英文简称）。

RBFT
::::::::::::::::

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - rbft_ID
   - gauge
   - normal
   - rbft节点ID
   - 
 * - rbft_version
   - gauge
   - normal
   - rbft共识协议版本号
   - 
 * - rbft_epoch
   - gauge
   - normal
   - rbft当前共识所处的epoch
   - 
 * - rbft_view
   - gauge
   - normal
   - rbft当前共识所处的view
   - 
 * - rbft_cluster_size
   - gauge
   - normal
   - rbft当前共识节点的个数
   - 
 * - rbft_quorum_size
   - gauge
   - normal
   - rbft当前quorum大小
   - 
 * - rbft_status_normal
   - gauge
   - normal
   - rbft是否处于normal状态on：1；off：0
   - 
 * - rbft_status_conf_change
   - gauge
   - normal
   - rbft是否处于配置区块状态on：2；off：0
   - 
 * - rbft_status_viewchange
   - gauge
   - normal
   - rbft是否处于vc状态on：3；off：0
   - 
 * - rbft_status_recovery
   - gauge
   - normal
   - rbft是否处于recovery状态on：4；off：0
   - 
 * - rbft_status_state_update
   - gauge
   - normal
   - rbft是否处于state update状态on：5；off：0
   - 
 * - rbft_status_pool_full
   - gauge
   - normal
   - rbft是否处于pool full状态on：6；off：0
   - 
 * - rbft_status_pending
   - gauge
   - normal
   - rbft是否处于pending状态on：7；off：0
   - 
 * - rbft_committed_block_number
   - counter
   - normal
   - rbft提交的区块数
   - 
 * - rbft_committed_config_block_number
   - counter
   - normal
   - rbft提交的配置区块数
   - 
 * - rbft_committed_empty_block_number
   - counter
   - normal
   - rbft提交的空块数（由vc导致的）
   - 
 * - rbft_committed_txs
   - counter
   - normal
   - rbft提交的交易个数
   - 
 * - rbft_txs_per_block
   - histogram
   - normal
   - rbft提交的每个区块的交易个数
   - 
 * - rbft_batch_persist_duration
   - histogram
   - normal
   - rbft从打包到提交的时间
   - 
 * - rbft_batche_number
   - gauge
   - normal
   - rbft当前缓存的batch个数
   - 
 * - rbft_outstanding_batche_number
   - gauge
   - normal
   - rbft当前正在共识的batch个数，无负载时=0
   - 
 * - rbft_state_update_times
   - counter
   - normal
   - rbft触发的StateUpdate的次数
   - 
 * - rbft_cache_batch_number
   - gauge
   - normal
   - rbft当前缓存的已打包但是不能共识的batch个数，仅主节点有该值无负载时=0
   - 
 * - rbft_fetch_missing_txs_times
   - counter
   - normal
   - rbft向主节点索取缺失交易的次数
   - 
 * - rbft_fetch_request_batch_times
   - counter
   - normal
   - rbft向其他节点索取batch的次数
   - 
 * - txset_incoming_txs
   - counter
   - normal
   - txset模块接收到的来自API的交易（这里包括了NVP转发过来的）
   - 
 * - txset_pending_txs
   - counter
   - normal
   - txset模块等待共识的交易个数
   - 
 * - rbft_incoming_local_tx_sets
   - counter
   - normal
   - rbft接收到的本地生成的txSet的个数
   - 
 * - rbft_incoming_remote_tx_sets
   - counter
   - normal
   - rbft接收到的其他VP节点转发过来的txSet的个数
   - 
 * - rbft_incoming_local_txs
   - counter
   - normal
   - rbft接收到的来自API的交易（这里包括了NVP转发过来的）
   - 
 * - rbft_incoming_remote_txs
   - counter
   - normal
   - rbft接收到的其他VP节点转发过来的交易
   - 
 * - rbft_rejected_local_txs
   - counter
   - normal
   - rbft拒收的来自API的交易
   - 
 * - rbft_rejected_remote_txs
   - counter
   - normal
   - rbft拒收的其他VP节点转发过来的交易
   - 

注意：

- 由于交易进入consensus模块之后需要经由txSet模块打包成一个set才能进入rbft模块，因此有txset_incoming_txs >= rbft_incoming_local_txs

txpool
:::::::::::::::::::::

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - txpool_incoming_txs
   - counter
   - normal
   - txpool接收到的交易总数
   - 
 * - txpool_duplicate_txs
   - counter
   - normal
   - txpool中检测到的重复交易
   - 
 * - txpool_nonBatched_txs
   - gauge 
   - normal
   - 当前交易池中未打包的交易个数，无负载时=0
   - 
 * - txpool_batched_txs
   - gauge 
   - normal
   - 当前交易池中已打包的交易个数
   - 
 * - txpool_batches
   - gauge 
   - normal
   - 当前交易池中的batch个数，无负载时<20（共识缓存最多20个batch）有负载时从节点<50，主节点理论上无上限
   - 

**注意：**

- txpool接收的交易是经由rbft模块传递下来的，因此有

txpool_incoming_txs = rbft_incoming_local_txs + rbft_incoming_remote_txs - rbft_reject_txs


存储相关指标
>>>>>>>>>>>>>>>>>>>

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - db_accountdb_batchCommitTime
   - histogram
   - normal
   - 写入accountdb的返回时间
   - 
 * - db_statedb_batchCommitTime
   - histogram
   - normal
   - 写入statedb的返回时间
   - 
 * - db_metadb_batchCommitTime
   - histogram
   - normal
   - 写入metadb的返回时间
   - 
 * - db_chaindb_batchCommitTime
   - histogram
   - normal
   - 写入chaindb的返回时间
   - 
 * - db_blockdb_batchCommitTime
   - histogram
   - normal
   - 写入blockdb的返回时间
   - 
 * - db_journaldb_batchCommitTime
   - histogram
   - normal
   - 写入journaldb的返回时间
   - 
 * - db_receiptdb_batchCommitTime
   - histogram
   - normal
   - 写入receiptdb的返回时间
   - 
 * - db_indexdb_batchCommitTime
   - histogram
   - normal
   - 写入indexdb的返回时间
   - 
 * - db_dbtype_multicache_memSize
   - gauge
   - normal
   - 多级缓存内存占用大小
   - 
 * - db_dbtype_multicache_persist_time
   - histogram
   - normal
   - 多级缓存持久化一个区块数据至底层数据库所需时间
   - 
 * - db_dbtype_multicache_walPersist_time
   - histogram
   - normal
   - 多级缓存写一个seqNo对应的wal的耗时
   - 
 * - db_dbtype_multicache_cache_get
   - gauge
   - normal
   - multicache尝试从自身的读缓存中读取数据的次数
   - 
 * - db_dbtype_multicache_cache_set
   - gauge
   - normal
   - multicache向自身的读缓存中插入数据的次数
   - 
 * - db_dbtype_multicache_cache_hit
   - gauge
   - normal
   - multicache缓存命中的次数
   - 
 * - db_dbtype_leveldb_compaction_occurrence
   - gauge
   - normal
   - 底层leveldb compaction的次数
   - 
 * - db_dbtype_leveldb_size
   - gauge
   - normal
   - 底层leveldb的数据量大小
   - 
 * - db_dbtype_filelog_fdNumber
   - gauge
   - normal
   - filelog中处于open状态的句柄数
   - 
 * - db_dbtype_filelog_readTime
   - histogram
   - expensive
   - filelog读取一个元素的耗时
   - 
 * - db_dbtype_filelog_fsyncTime
   - histogram
   - normal
   - filelog在一个log文件写完后，做一次fsync的耗时
   - 

执行相关指标
>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - bloomFilter_memSize
   - gauge
   - normal
   - 布隆过滤器占用的内存大小
   - 
 * - bloomFilter_lookCounter
   - counter
   - normal
   - 布隆过滤器查询次数
   -  有「source」标签说明调用查询的模块，分为“namespace表示共识去重”、“executor表示执行层去重”、“API表示接口层去重”
 * - bloomFilter_lookExistCounter
   - counter
   - normal
   - 布隆过滤器中，查询到存在的交易数量，即会穿透布隆过滤器进行db查询的次数
   - 有「source」标签说明调用查询的模块，分为“namespace表示共识去重”、“executor表示执行层去重”、“API表示接口层去重”
 * - execMgr_validate_bloom_readValidDBTime
   - histogram
   - normal
   - 布隆过滤器发现交易可能重复后，查询合法交易的时间
   - 
 * - execMgr_validate_bloom_readInvalidDBTime
   - histogram
   - normal
   - 布隆过滤器发现交易可能重复后，查询非法交易的时间（会先查询合法交易，再查询非法交易，查询次数可能比合法交易少一些）
   - 
 * - execMgr_validate_tx_time
   - histogram
   - expensive
   - 单笔交易执行时间
   - 
 * - execMgr_total_blocks
   - gauge
   - normal
   - 当前链高度
   - 
 * - execMgr_total_txs
   - gauge
   - normal
   - 区块链所有交易总量
   - 
 * - execMgr_online_txs
   - gauge
   - normal
   - 区块链归档点后所有交易总量
   - 
 * - execMgr_commit_txs
   - counter
   - normal
   - 此次启动后新增的交易数量
   - 
 * - execMgr_commit_time
   - histogram
   - normal
   - 写块时间，包括区块组成、写入等等
   - 
 * - execMgr_commit_writeDB_time
   - histogram
   - normal
   - 区块batch写入数据库的时间
   - 


网络相关指标
>>>>>>>>>>>>>>>>>>>>>>>>>>

P2P
:::::::::::::::

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - grpc_stream_request_send_total
   - counter
   - normal
   - grpc发送的流请求数目
   - 
 * - grpc_stream_request_received_total
   - counter
   - normal
   - grpc收到的流请求数目
   - 
 * - grpc_stream_request_completed_total
   - counter
   - normal
   - grpc完成的流请求的数目
   - 
 * - grpc_stream_message_send_total
   - counter
   - normal
   - grpc发送的流消息的数目
   - 
 * - grpc_stream_message_recv_total
   - counter
   - normal
   - grpc收到的流消息的数目
   - 
 * - grpc_conn_opened_total
   - counter
   - normal
   - grpc连接打开数目。打开数目减去关闭数目便是目前活跃的连接数。
   - 
 * - grpc_conn_closed_total
   - counter
   - normal
   - grpc连接关闭数目。打开数目减去关闭数目便是目前活跃的连接数。
   - 
 * - grpc_stream_opened_total
   - counter
   - normal
   - grpc流打开数目。打开数目减去关闭数目便是目前活跃的流数。
   - 
 * - grpc_stream_closed_total
   - counter
   - normal
   - grpc流关闭数目。打开数目减去关闭数目便是目前活跃的流数。
   - 
 * - msg_dropped_count_total
   - counter
   - normal
   - 某个节点在某个message channel弃的消息的数目
   - 
 * - grpc_stream_message_send_time_microseconds
   - histogram
   - expensive
   - grpc发送一条消息所需要的时间
   - 
 * - logic_conn_opened_count_total
   - counter
   - normal
   - 逻辑连接打开数目。打开数目减去关闭数目便是目前活跃的连接数。
   - 
 * - logic_conn_closed_count_total
   - counter
   - normal
   - 逻辑连接关闭数目。打开数目减去关闭数目便是目前活跃的连接数。
   - 
 * - grpc_network_receive_bytes_total
   - counter
   - normal
   - grpc网络接收到的消息大小总量
   - 
 * - grpc_network_send_bytes_total
   - counter
   - normal
   - grpc网络发送的消息大小总量
   - 

消息分发
::::::::::::::::

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - dispatcher_%s_writeMsg_ch_size_usage
   - gauge
   - normal
   - 消息分发器发送消息通道对应节点的模块消息数量
   - 
 * - dispatcher_%s_writeMsg_ch_mem_usage
   - gauge
   - normal
   - 消息分发器发送消息通道对应节点的模块消息占用内存的大小
   - 
 * - dispatcher_%s_readMsg_ch_size_usage
   - gauge
   - normal
   - 消息分发器接收消息通道对应节点的模块消息数量
   - 
 * - dispatcher_%s_readMsg_ch_mem_usage
   - gauge
   - normal
   - 消息分发器接收消息通道对应节点的模块消息占用内存的大小
   - 

API相关指标
>>>>>>>>>>>>>>>>>>>>>

.. list-table::
 :widths: 25 25 25 25
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
 * - jsonrpc_request_received_total
   - counter
   - normal
   - 接收的jsonrpc请求个数
 * - jsonrpc_new_tx_request_success_total
   - counter
   - normal
   - 成功处理的发送交易相关请求的个数
 * - jsonrpc_new_tx_request_error_total
   - counter
   - normal
   - 发送交易相关请求处理失败，不同失败原因的请求个数
 * - jsonrpc_received_request_bytes_total
   - counter
   - normal
   - 请求总流量统计单位byte
 * - jsonrpc_sent_response_bytes_total
   - counter
   - normal
   - 响应总流量统计单位byte
 * - jsonrpc_new_tx_request_consensus_abnormal_error_total
   - counter
   - normal
   - 由于共识状态异常原因导致请求失败的个数

密码相关指标
>>>>>>>>>>>>>>>>>>>>

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - verify_tube_signature_in_total
   - counter
   - normal
   - 输入验签管道的总签名数
   - 
 * - verify_tube_signature_out_total
   - counter
   - normal
   - 输出验签管道的总签名数
   - 
 * - vt_liner_duration_cycle
   - Histogram
   - normal
   - 完成批量验证两段子操作的时间
   - 
 * - vt_logic_duration_cycle
   - Histogram
   - normal
   - 验签中每段操作的时间
   - 
 * - vt_hash_duration_cycle
   - Histogram
   - normal
   - 完成哈希的时间
   - 
 * - verify_tube_batch_size
   - Gauge
   - normal
   - 每批次的签名数量
   - 
 * - crypto_verify_channel_length
   - Gauge
   - normal
   - 每个阶段的缓冲池大小
   - 

Go相关指标
>>>>>>>>>>>>>>>>>>

.. list-table::
 :widths: 20 20 20 20 20
 :header-rows: 1

 * - 名称
   - 类型
   - 级别
   - 描述
   - 备注
 * - go_goroutines
   - gauge
   - normal
   - 
   - 
 * - go_threads
   - gauge
   - normal
   - 
   - 
 * - go_gc_duration_seconds
   - summary
   - normal
   - 
   - 
 * - go_info
   - gauge
   - normal
   - 
   - 
 * - alloc_bytes
   - gauge
   - normal
   - 
   - 
 * - alloc_bytes_total
   - counter
   - normal
   - 
   - 
 * - sys_bytes
   - gauge
   - normal
   - 
   - 
 * - lookups_total
   - counter
   - normal
   - 
   - 
 * - mallocs_total
   - counter
   - normal
   - 
   - 
 * - frees_total
   - counter
   - normal
   - 
   - 
 * - heap_alloc_bytes
   - gauge
   - normal
   - 
   - 
 * - heap_sys_bytes
   - gauge
   - normal
   - 
   - 
 * - heap_idle_bytes
   - gauge
   - normal
   - 
   - 
 * - heap_inuse_bytes
   - gauge
   - normal
   - 
   - 
 * - heap_released_bytes
   - gauge
   - normal
   - 
   - 
 * - heap_objects
   - gauge
   - normal
   - 
   - 
 * - stack_inuse_bytes
   - gauge
   - normal
   - 
   - 
 * - stack_sys_bytes
   - gauge
   - normal
   - 
   - 
 * - mspan_inuse_bytes
   - gauge
   - normal
   - 
   - 
 * - mspan_sys_bytes
   - gauge
   - normal
   - 
   - 
 * - mcache_inuse_bytes
   - gauge
   - normal
   - 
   - 
 * - mcache_sys_bytes
   - gauge
   - normal
   - 
   - 
 * - buck_hash_sys_bytes
   - gauge
   - normal
   - 
   - 
 * - gc_sys_bytes
   - gauge
   - normal
   - 
   - 
 * - other_sys_bytes
   - gauge
   - normal
   - 
   - 
 * - next_gc_bytes
   - gauge
   - normal
   - 
   - 
 * - last_gc_time_seconds
   - gauge
   - normal
   - 
   - 
 * - gc_cpu_fraction
   - gauge
   - normal
   - 
   - 



