第一章 前言
==========

1.1 版本声明
-------------

本文档著作权归趣链科技单独所有，未经趣链科技事先书面许可，任何主体不得以任何形式复制、修改、抄袭、传播全部或部分本文档内容。

1.2 服务声明
-------------

本文档意在向客户介绍趣链科技区块链底层平台的运维说明。您所购买的趣链科技的产品、服务的种类、服务标准等应由您与趣链科技之间的商业合同约定，
除非双方另有约定，否则，趣链科技对本文档内容不做任何明示或模式的承诺或保证。

第三章 配置管理
===============

3.1 配置文件说明
----------------

> 配置文件中各配置项的含义及默认值请参考附录。

3.1.1 global.toml
^^^^^^^^^^^^^^^^^^^^

在该配置文件中记录了一些 **全局的、无法动态修改的** 配置项，包括输出日志等级、输出日志文件的路径、RPC配置、流量控制参数等::

 1. title = "flato global configurations"
 2. commercialVersion = "2.0"
 3. #
 4. #
 5. # Logger section
 6. # config log's level by module
 7. # CRITICAL ERROR WARNING NOTICE INFO DEBUG
 8. # high <------------- log level -------> low
 9. #
 10. #

 11. [log]
 12. dump_file           = true # dump the log file or not
 13. dump_interval       = "24h"  # Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h". such as "300ms", "2h45m".
 14. log_dir             = "./logs"
 15. log_level           = "NOTICE" # default loglevel for all modules which can be override by module level log setting
 16. file_format         = "[%{module}][%{level:.5s}] %{time:15:04:05.000} %{shortfile} %{message}"
 17. console_format      = "%{color}[%{module}][%{level:.5s}] %{time:15:04:05.000} %{shortfile} %{message} %{color:reset}"
 18. max_log_size        = "200mb"  # "mb", "kb"
 19. check_size_interval = "2m"
 20. [log.module] #set log level by module
 21. p2p         = "NOTICE"
 22. consensus   = "INFO"
 23. core        = "NOTICE"
 24. flatodb     = "NOTICE"
 25. eventhub    = "NOTICE"
 26. jsonrpc     = "NOTICE"
 27. hypernet    = "NOTICE"
 28. audit       = "NOTICE"

 29. [audit]
 30. backend				= "file" # "file" or "filelog" or graylog" or "elk"
 31. level         		= "none" # none, metadata, responsebody, storageobject
 32. queue_size          = 4096
 33. rewrite_interval    = "2s"
 34. update_conn_interval = "2s"
 35. [audit.conn.pool]
 36. #urls             = ["172.16.3.1:12201", "172.16.3.2:12201", "172.16.3.3:12201", "172.16.3.4:12201"]  #graylog tcp conn urls
 37. urls             = ["172.16.5.5:12202"]    #filebeat tcp conn url
 38. init_cap         = 1
 39. max_cap          = 5
 40. idle_timeout     = "60s"
 41. dial_timeout     = "3s"
 42. [audit.file]               #filelog config
 43. encrypt = false
 44. type="filelog"
 45. [audit.file.filelog]
 46. path = "system/auditlog/filelog/"
 47. compression = "snappy" #zlib pure snappy
 48. max_log_file_size = "100mb" # "mb", "kb"
 49. data_version = 1 # default to be 1, means disk storage struct version
 50. index_enable = true
 51. [audit.file.filelog.cache]
 52. enable = true
 53. max_cache_size = 100 #mb
 54. cache_expired_time = 48 #hour
 55. cache_entry_num = 20

 56. [audit.file.filelog.handler_cache]
 57. handler_cache_num = 100
 58. handler_cache_evict_time = 1 # *time.Second

 59. #
 60. #
 61. #  JSONRPC section
 62. #
 63. #
 64. [http]
 65. # allowedOrigins should be a comma-separated list of allowed origin URLs
 66. # to allow connections with any origin, pass "*".
 67. allowedOrigins=["*"]
 68. # if true, it will enable secure http connection(https).
 69. security    = false
 70. # if true, use http/2, otherwise use http/1.1.
 71. # WARN: if version_2 is true, option sercurity must be true, otherwise use default http/1.1 without https.
 72. http_2   = false
 73. #Path to tlsca for rpc
 74. tlsCA                   = "./tls/tlsca.ca"
 75. #Path to tlscert for rpc
 76. tlsCert                 = "./tls/tls_peer.cert"
 77. #Path to private key of tlscert for rpc
 78. tlsCertPriv             = "./tls/tls_peer.priv"
 79. [http.request]
 80. max_content_length = "100kb" # default 100kB
 81. read_timeout = "1000s"      # default 1000s

 82. #
 83. #
 84. # P2P section
 85. #
 86. #
 87. [p2p]
 88. transport               = "grpc" # we now only support "grpc"
 89. compress                = false  # enable compression and decompression of network message

 90. # network mode, enum: direct, relay, discover. direct is default mode.
 91. # discover mode not require full hosts.toml.
 92. # relay mode not require direct node.
 93. mode                    = "direct"

 94. retrytime               = "3s"
 95. ipc                     = "hpc_1.ipc"
 96. enableTLS               = true
 97. tlsCA                   = "./tls/tlsca.ca"
 98. tlsServerHostOverride   = "hyperchain.cn"
 99. tlsCert                 = "./tls/tls_peer.cert"
 100. tlsCertPriv             = "./tls/tls_peer.priv"

 101. # connection control configurations
 102. # keepalive configuration
 103. # [1,5,10] means minLimit=1, threshold=5 maxLimit=20
 104. keepAliveDuration = [1,5,20]

 105. # pending configuration
 106. # [5,20,100] means minLimit=1, threshold=20 maxLimit=100
 107. pendingDuration = [5,20,100]

 108. # send keepalive message every keepAliveInterval
 109. keepAliveInterval = "5s"

 110. # stream will be closed if not receive keepalive message for keepAliveTimeout
 111. keepAliveTimeout = "3m"

 112. [p2p.grpc]
 113. maxRecvMessageSize = "50mb" # default 50mb
 114. maxSendMessageSize = "50mb" # default 50mb

 115. #
 116. #
 117. # flow control configurations section
 118. #
 119. #
 120. [flow.control.ratelimit]
 121. enable            = true
 122. fileReceivePeak   = 100
 123. fileReceiveRate	  = "8ms"
 124. fileSendPeak	  = 100
 125. fileSendRate      = "8ms"

 126. [flow.control.bandwidth]
 127. enable            = false
 128. outgoingBandwidth = "500Mb" # NOTE: The unit is Mb/s NOT MB/s


其中log模块表示日志相关的配置选项，您可以通过修改log.dump_file来控制是否将输出日志重定向至日志文件中， **我们推荐您开启日志重定向** 。

您可以根据实际申请开放的端口号进行port模块的配置，其中grpc端口是节点间通信的端口号，注意要与addr.toml中的端口号对应；jsonrpc端口是外部应用向hyperchain平台发送请求使用的端口号。

您可以通过修改flow.control.ratelimit.enable和flow.control.bandwidth的值来控制是否开启节点流量控制和带宽限制，建议根据测试的tps进行流控设置，详细的流控配置可参考3.2节内容。

3.1.2 dynamic.toml
^^^^^^^^^^^^^^^^^^^^^

该配置文件包含了一些 **全局的，可运行时动态修改的配置项::

 1. self = "node1"``

 2. #
 3. #
 4. # key ports section
 5. #
 6. #
 7. [port]
 8. jsonrpc     = 8081
 9. grpc        = 50011 # p2p

 10. #
 11. #
 12. # p2p system config
 13. # 1. define the remote peer's hostname and its IP address
 14. # 2. define self address list under different domain
 15. #
 16. #
 17. [p2p]
 18. [p2p.ip.remote]
 19. # this node will connect to those peer, if here has self hostname, we will ignore it
 20. hosts = [``
 21. "node1 127.0.0.1:50011",
 22. "node2 127.0.0.1:50012",
 23. "node3 127.0.0.1:50013",
 24. "node4 127.0.0.1:50014",
 25. ]

 26. [p2p.ip.self]
 27. domain = "domain1"

 28. # addr is (domain,endpoint) pair, those items defined the ip address:port which
 29. # other domains' host how connect to self
 30. addrs = [
 31. "domain1 127.0.0.1:50011",
 32. "domain2 127.0.0.1:50011",
 33. "domain3 127.0.0.1:50011",
 34. "domain4 127.0.0.1:50011",
 35. ]
 36. #这里配置时候需要注意,配置的是其他节点访问本节点时，使用的本节点的IP地址，举个例子，如果节点2属于域 **domain2** ，那么节点2访问节点1时需要用节点1声明的在 **domain2** 域中对外暴露的地址，换句话说，节点2访问本节点时用的地址是 127.0.0.1:50012。

 37. [[namespace]]
 38. name = "global"
 39. start = true

您可以根据实际申请开放的端口号进行port模块的配置，其中grpc端口是节点间通信的端口号，注意要与下方 [p2p.ip.remote.hosts] 中的端口号对应；jsonrpc端口是外部应用向Flato平台发送请求使用的端口号。

**domain的配置是比较容易出错的地方，最简单的配置方式就是**：

- 所有节点都在一个domain里：所有节点都在同一个内网环境，只要配置一个domain和该节点在这个domain里的IP地址即可。

namespace模块指定了namespace的根目录路径以及节点启动时默认参与的namespace名称， **我们建议每个节点都要默认启动global这个namespace** 。

3.1.3 ns_dynamic.toml
>>>>>>>>>>>>>>>>>>>>>>>>

该配置文件中记录了 **namespace级别的可动态修改的配置项** ，包括当前节点的启动方式、启动身份、区块链网络节点数目以及每个节点的网络配置信息。
您在使用之前必须确保所有的网络配置正确。节点启动的时候会 **检查该配置文件的可用性** ，比如 `nodes` 列表中不能有重复的hostname、 `self.n`
必须等于 `nodes` 列表项目数，平台通过检查网络配置文件的可用性，可以让应用开发者及时发现配置异常。

 ::

 1. [consensus]
 2. algo = "RBFT"
 3. [consensus.set]
 4. set_size       = 25    # How many transactions should the node broadcast at once
 5. [consensus.pool]
 6. batch_size       = 500    # How many txs should the primary pack before sending pre-prepare
 7. pool_size        = 50000  # How many txs could the txPool stores in total

 8. [self]
 9. n         = 4           # 运行时修改。表示所连vp节点的个数，该值在节点运行过程中会实时变化。
 10. hostname    = "node1"   # 运行时修改，仅限于CVP节点。对于cvp来说，该值会发生变化，仅在cvp节点升级为vp的时候，这里的hostname会被替换为要升级vp的hostname。
 11. new         = false     # 运行时修改。新节点成功加入网络以后，该值会变为false。
 12. # the value can only be vp、nvp and cvp, case-insensitive
 13. type        = "vp"	    # 候选项为vp/nvp/cvp
 14. vp          = true      # （过时的无效配置）

 15. #[[cvps]]				# 运行时修改。cvps在节点运行过程中实时变化。
 16. #hostname 	= "cvp1"

 17. #[[cvps]]
 18. #hostname 	= "cvp2"

 19. #[[nvps]]				# 运行时修改。nvps数组在节点运行过程中实时变化。
 20. #hostname	= "nvp1"

 21. #[[nvps]]
 22. #hostname	= "nvp2"

 23. [[nodes]]				# 运行时修改。nodes数组在节点运行过程中实时变化。
 24. hostname    = "node1"
 25. score       = 10

 26. [[nodes]]
 27. hostname    = "node2"
 28. score       = 10

 29. [[nodes]]
 30. hostname    = "node3"
 31. score       = 10

 32. [[nodes]]
 33. hostname    = "node4"
 34. score       = 1

可查询附录了解更多配置项信息。

3.1.4 ns_static.toml
>>>>>>>>>>>>>>>>>>>>>>

该配置文件中记录了所有namespace级别的配置项，包括 **共识算法配置、加密证书的配置、数据库相关配置、日志等级配置** 等等，这些配置都是 **无法在运行中修改** 的，各个配置项的释义在注释以及文末的附录中给出。

1. #
2. #
3. #
4. #
5. #
6. #
7. #
8. #
9. title = "namespace configurations"

10. [genesis]
11. [genesis.alloc]
12. "000f1a7a08ccc48e5d30f80850cf1cf283aa3abd" = "1000000000"
13. "e93b92f1da08f925bdee44e91e7768380ae83307" = "1000000000"
14. "6201cb0448964ac597faf6fdf1f472edf2a22b89" = "1000000000"
15. "b18c8575e3284e79b92100025a31378feb8100d6" = "1000000000"
16. "856E2B9A5FA82FD1B031D1FF6863864DBAC7995D" = "1000000000"
17. "fbca6a7e9e29728773b270d3f00153c75d04e1ad" = "1000000000"

18. #
19.  consensus algorithm configuration section
20. #
21. # 1. choose which consensus algorithm to use
22. # 2. define the algorithm related configurations
23. #
24. #
25. [consensus]
26. [consensus.set]
27. # set_size       = 25    # How many transactions should the node broadcast at once
28. set_timeout    = "0.1s"# Node broadcasts transactions if there are cached transactions, although set_size isn't reached yet

29. [consensus.pool]
30. # batch_size       = 500    # How many txs should the primary pack before sending pre-prepare
31. # pool_size        = 50000  # How many txs could the txPool stores in total
32. check_interval  = "3m"  # interval of the check loop
33. tolerance_time  = "5m"  # the max tolerance time, if time.Now - timestamp > toleranceTime, send event to consensus
34. batch_mem_limit  = false # If Limit the batch memory size or not
35. batch_max_mem    = "10mb" # The maximum memory size of one batch

36. [consensus.rbft]         # rbft configurations
37. k                = 10    # After how many blocks a checkpoint will be sent
38. vc_period        = 0     # After how many checkpoint periods( Blocks = 10 * vcperiod ) the primary gets cycled automatically. ( Set 0 to disable )

39. [consensus.rbft.timeout]
40. sync_state        = "1s"  # How long to wait quorum sync state response
41. sync_interval     = "10s"  # How long to restart sync state process
42. recovery          = "10s" # How long to wait before recovery finished
43. first_request     = "30s" # How long to wait before first request should come
44. batch             = "0.5s"# Primary send a pre-prepare if there are pending requests, although batchsize isn't reached yet,
45. request           = "6s"  # How long may a request(transaction batch) take between reception and execution, must be greater than the batch timeout
46. validate          = "1s"  # How long may a validate (transaction batch) process will take by local Validation
47. null_request      = "9s"  # Primary send it to inform aliveness, must be greater than request timeout
48. viewchange        = "8s"  # How long may a view change take
49. resend_viewchange = "10s" # How long to wait for a view change quorum before resending (the same) view change
50. clean_viewchange  = "60s" # How long to clean out-of-data view change message
51. fetch_checkpoint  = "5s"  # How long to wait for config change checkpoint quorum before fetch checkpoint take

52. [consensus.raft]        # RAFT configurations
53. snap_count        = 20 # How many entries should trigger a snapshot
54. catchup_count = 20 # when doing log compaction, keep some entries in memory for slow followers to catchup

55. [consensus.raft.timeout]
56. batch        = "0.5s"  # Node make a batch if there are pending requests, although batchsize isn't reached yet
57. set          = "0.1s" # Node broadcasts transactions if there are cached transactions, although set_size isn't reached yet
58. fetch  = "3s" # How long to fetch a missing batch
59. negotiate = "6s" # How long to wait for quorum responses after send negotiate

60. [consensus.raft.dir]
61. snap = "data/raft/snap" # snapshot dir
62. wal  = "data/raft/wal"  # wal dir

63. [consensus.solo.timeout]
64. batch             = "0.5s"# Node send a batch if there are pending requests, although batchsize isn't reached yet
65. set               = "0.1s"# Node broadcasts transactions if there are cached transactions, although set_size isn't reached yet


66. [encryption]
67. [encryption.TEE]
68. TEEPath = "http://nexus.hyperchain.cn/repository/arch/sgx/enclave.sign.so"
69. DataEncrypt = false

70. [encryption.root]
71. ca    = "certs/CA"

72. [encryption.ecert]
73. ecert  = "certs/certs"

74. [encryption.bcert]
75. # path of the back-up permission certs
76. listDir = "certs/bcerts"

77. [encryption.check]
78. enable     = true   #enable RCert
79. enableT    = false  #enable TCert

80. [distributedCA]
81. enable = false  #true is aco mode

82. [encryption.security]
83. algo   = "sm4"   # Selective symmetric encryption algorithm (pure,3des,aes or sm4)


84. #
85. #
86. # system upgrade configuration
87. #
88. #

89. [self-government.upgrade]
90. archive_path = "upgrade_archive"
91. archive_prefix = "upgrade_"
92. fetch_remote = true
93. fetch_path = "http://127.0.0.1:8000/hyperchain-deploy.tar.gz"

94. [[self-government.nodes]]
95. hostname    = "node1"
96. address     = "14f14e4e316ed13c41eedd4759d79d07aac775ac"

97. [[self-government.nodes]]
98. hostname    = "node2"
99. address     = "916a3c39f7c0f579c9df5b9931af49a899c5bb4a"

100. [[self-government.nodes]]
101. hostname    = "node3"
102. address     = "be79466f637b4c18de25f0312c5961af66fb2d3d"

103. [[self-government.nodes]]
104. hostname    = "node4"
105. address     = "e593460a7c31f9984deae6529a57229285bd0370"

106. #
107. #
108. # radar configurations
109. #
110. #

111. [radar-params]
112. viewable_db_type = "mysql"
113. mysql_connURL  = "root:root@tcp(127.0.0.1:3306)/%s?charset=utf8"
114. #
115. #
116. # executor configuration section
117. # config log's level by module
118. # CRITICAL ERROR WARNING NOTICE INFO DEBUG
119. # high <------------- log level -------> low
120. #
121. #
122. [log]
123. dump_file           = true # dump the log file or not
124. dump_interval       = "24h"  # Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h". such as "300ms", "2h45m".
125. log_dir             = "data/logs"
126. log_level           = "NOTICE" # default loglevel for all modules which can be override by module level log setting
127. file_format         = "[%{module}][%{level:.5s}] %{time:15:04:05.000} %{shortfile} %{message}"
128. console_format      = "%{color}[%{module}][%{level:.5s}] %{time:15:04:05.000} %{shortfile} %{message} %{color:reset}"
129. max_log_size        = "200mb"  # "mb", "kb"
130. check_size_interval = "2m"

131. [log.module] #set log level by module
132. p2p         = "NOTICE"
133. consensus   = "INFO"
134. flatodb     = "NOTICE"
135. eventhub    = "NOTICE"
136. executor    = "NOTICE"
137. execmgr     = "NOTICE"
138. syncmgr     = "NOTICE"
139. filemgr     = "NOTICE"
140. buckettree  = "NOTICE"
141. radar       = "NOTICE"
142. mq          = "INFO"
143. private     = "INFO"
144. midware     = "NOTICE"
145. dispatch    = "NOTICE"
146. node        = "NOTICE"
147. api         = "NOTICE"
148. nvp         = "NOTICE"

149. [rpc.qps.flowCtrl]
150. enable   = true
151. capacity = 100
152. limit    = 2000 # qps

153. [duplicate]
154. # tx generated in the past tx_active_time is legal
155. tx_active_time    = "24h"
156. # The transaction's timestamp can only be greater than the current time by up to tx_drift_time
157. tx_drift_time     = "5m"

158. [duplicate.bloomfilter] # bloom filter used in tx duplication checking
159. # each bloomfilter's size
160. bloombit          = 100000000
161. # Bloom filter's total memory limit
162. max_mem  = "100mb"

163. #[[service]]
164. #    service_name = "Radar"
165. #[[service]]
166. #    service_name = "MQ"

167. [mq.broker]
168. type = "rabbit"
169. #type = "kafka"

170. [mq.rabbit]
171. url = "amqp://guest:guest@127.0.0.1:5672/"

172. [mq.kafka]
173. urls				= ["localhost:9092"]
174. writerBatchSize		= 100
175. writerBatchBytes	= 52428800
176. writerBatchTimeout 	= "0.1s"
177. writeTimeout 		= "10s"
178. rebalanceInterval 	= "10s"
179. idleConnTimeout 	= "10s"

180. [mq.kafka.partitionNums]
181. "localhost:9092" = 1

182. [mq.kafka.replicaFactor]
183. "localhost:9092" = 1


184. #
185. #
186. # private protection configuration section
187. #
188. #
189. [private]
190. cache_size    = 500

191. [private.timeout]
192. sync_data     = "3s"
193. query_data    = "3s"
194. fetch_data    = "3s"
195. check         = "1h"

196. [private.ttl]
197. name = "time"
198. [private.ttl.time]
199. timeout = "24h"
200. [private.ttl.block]
201. number = 5

202. [executor]
203. [executor.syncer]
204. 
205. max_block_fetch         = 50

206. [executor.archive]
207. archive_root  = "data/archive/"

208. [executor.nvp]
209. exitflag = false
210. sync_mode = "block" #another mode is journal #新增

211. [executor.sync_chain]
212. priority = "block"
213. sync_journal_receipt = true
214. hs_interval = "5s"
215. hs_resend_times = 5
216. batch_size = 100
217. state_sync_interval = "2s"
218. fetcher_resend = 5
219. mode_negotiate_interval = "5s"

220. [executor.filemgr]
221. enable = false
222. deadline = "600s"
223. clean_interval = "60s"
224. file_system_mode = "ipfs" #origin, ipfs
225. data_path = "namespaces/global/data/filemgr"
226. ipfs_url="localhost:5001"
227. hs_interval = "30s"
228. batch_size = 100
229. transfer_interval = "30s"

230. [database]
231. public_path = "data/public/"
232. private_path = "data/private/"
233. [database.state]
234. # 1. Encryption is a resource consuming functionality and will somewhat slow the process down.
235. # 2. Something unexpected can happen if this field is changed more than once (e.g. switch-off -> start -> swicth-on -> restart).
236. # 3. Make sure the whole cluster is running with the same encryption config, or the data transfer (sync-chain)
237. #    between nodes will fail.
238. encrypt = false
239. type="multicache"
240. [database.state.multicache]
241. #maximum memory occupation of tables
242. persist_goroutine_num = 100
243. underlyint_num = 10
244. memory_limit = "500mb"
245. data_path = "statedb/"
246. [database.state.multicache.persist_db]
247. type="leveldb" #multidb, memdb or leveldb
248. [database.state.multicache.persist_db.multidb] # work iff type="multidb"
249. db_amount_limit = 32
250. db_paths = [
251. "namespaces/global/data/public/statedb/persist/multidb-0",
252. "namespaces/global/data/public/statedb/persist/multidb-8",
253. "namespaces/global/data/public/statedb/persist/multidb-16",
254. "namespaces/global/data/public/statedb/persist/multidb-24"
255. ]
256. [database.state.multicache.persist_db.leveldb]
257. block_cache_capacity      = "8mb" # "mb", "kb"
258. block_size                = "4kb" # "mb", "kb"
259. write_buffer              = "4mb" # "mb", "kb"
260. write_l0_pause_trigger    = 12
261. write_l0_slowdown_trigger = 8
262. # the level db file size (default is 2mb, v1.2 is 8mb)
263. compaction_table_size     = "8mb"
264. [database.state.multicache.persist_db.tikv]
265. pd_addrs = ["172.16.5.4:2371"]
266. [database.state.multicache.temp_db]
267. type="leveldb"
268. [database.state.multicache.temp_db.leveldb]
269. block_cache_capacity      = "8mb" # "mb", "kb"
270. block_size                = "4kb" # "mb", "kb"
271. write_buffer              = "4mb" # "mb", "kb"
272. write_l0_pause_trigger    = 12
273. write_l0_slowdown_trigger = 8
274. # the level db file size (default is 2mb, v1.2 is 8mb)
275. compaction_table_size     = "8mb"
276.    [database.account]
277.        encrypt = false
278.        type="multicache"
279.        [database.account.multicache]
280.            #maximum memory occupation of tables
281.			persist_goroutine_num = 5
282.			underlyint_num = 1
283.            memory_limit = "50mb"
284.            data_path = "accountdb/"
285.            [database.account.multicache.persist_db]
286.                type="leveldb"
287.                [database.account.multicache.persist_db.leveldb]
288.                    block_cache_capacity      = "8mb" # "mb", "kb"
289.                    block_size                = "4kb" # "mb", "kb"
290.                    write_buffer              = "4mb" # "mb", "kb"
291.                    write_l0_pause_trigger    = 12
292.                    write_l0_slowdown_trigger = 8
293.                    # the level db file size (default is 2mb, v1.2 is 8mb)
294.                    compaction_table_size     = "8mb"
295.                [database.account.multicache.persist_db.tikv]
296.                    pd_addrs = ["172.16.5.4:2371"]

297.    [database.chain]
298.        encrypt = false
299.        type="multicache"
300.        database.chain.multicache]
301.            #maximum memory occupation of tables
302.			persist_goroutine_num = 5
303.			underlyint_num = 1
304.			memory_limit = "50mb"
305.            data_path = "chaindb/"
306.            [database.chain.multicache.persist_db]
307.               type="leveldb"
308.                [database.chain.multicache.persist_db.leveldb]
309.                    block_cache_capacity      = "8mb" # "mb", "kb"
310.                    block_size                = "4kb" # "mb", "kb"
311.                    write_buffer              = "4mb" # "mb", "kb"
312.                    write_l0_pause_trigger    = 12
313.                    write_l0_slowdown_trigger = 8
314.                    # the level db file size (default is 2mb, v1.2 is 8mb)
315.                    compaction_table_size     = "8mb"
316.                [database.chain.multicache.persist_db.tikv]
317.                    pd_addrs = ["172.16.5.4:2371"]
318.    [database.block]
319.        encrypt = false
320.        type="filelog"
321.        [database.block.filelog]
322.            path = "blockdb/filelog/"
323.            compression = "snappy" #zlib pure snappy
324.            max_log_file_size = "100mb" # "mb", "kb"
325.            data_version = 1 # default to be 1, means disk storage struct version
326.            index_enable = true

327.            [database.block.filelog.cache]
328.                enable = true
329.                max_cache_size = 100 #mb
330.                cache_expired_time = 48 #hour
331.                cache_entry_num = 20

332.            [database.block.filelog.handler_cache]
333.                handler_cache_num = 100
334.                handler_cache_evict_time = 1 # *time.Second
335.    [database.journal]
336.        encrypt = false
337.        type="filelog"
338.        [database.journal.filelog]
339.            path = "journaldb/filelog/"
340.            compression = "snappy" #zlib pure snappy
341.            max_log_file_size = "100mb" # "mb", "kb"
342.            data_version = 1 # default to be 1, means disk storage struct version
343.            index_enable = true

344.            [database.journal.filelog.cache]
345.                enable = true
346.                max_cache_size = 100 #mb
347.                cache_expired_time = 48 #hour
348.                cache_entry_num = 20

349.            [database.journal.filelog.handler_cache]
350.                handler_cache_num = 100
351.                handler_cache_evict_time = 1 # *time.Second

352.    [database.receipt]
353.        encrypt = false
354.        type="filelog"
355.        [database.receipt.filelog]
356.            path = "receiptdb/filelog/"
357.            compression = "snappy" #zlib pure snappy
358.            max_log_file_size = "100mb" # "mb", "kb"
359.            data_version = 1 # default to be 1, means disk storage struct version
360.            index_enable = true

361.            [database.receipt.filelog.cache]
362.                enable = true
363.                max_cache_size = 100 #mb
364.                cache_expired_time = 48 #hour
365.                cache_entry_num = 20

366.            [database.receipt.filelog.handler_cache]
367.                handler_cache_num = 100
368.                handler_cache_evict_time = 1 # *time.Second

369.    [database.invalidtx]
370.        encrypt = false
371.        type="leveldb"
372.        [database.invalidtx.leveldb]
373.                path="invalidtx/leveldb/"
374.				logpath="invalidtx/log"
375.                block_cache_capacity      = "8mb" # "mb", "kb"
376.                block_size                = "4kb" # "mb", "kb"
377.                write_buffer              = "4mb" # "mb", "kb"
378.                write_l0_pause_trigger    = 12
379.                write_l0_slowdown_trigger = 8
380.                # the level db file size (default is 2mb, v1.2 is 8mb)
381.                compaction_table_size     = "8mb"

382.    [database.consensus]
383.        encrypt = false
384.        type="leveldb"
385.        [database.consensus.leveldb]
386.                path="consensusdb/leveldb/"
387.				logpath="consensusdb/log"
388.                block_cache_capacity      = "8mb" # "mb", "kb"
389.                block_size                = "4kb" # "mb", "kb"
390.                write_buffer              = "4mb" # "mb", "kb"
391.                write_l0_pause_trigger    = 12
392.                write_l0_slowdown_trigger = 8
393.                # the level db file size (default is 2mb, v1.2 is 8mb)
394.                compaction_table_size     = "8mb"

395.    [database.camanager]
396.        encrypt = false
397.        type="leveldb"
398.        [database.camanager.leveldb]
399.                path="cadb/leveldb/"
400.				logpath="cadb/log"
401.                block_cache_capacity      = "8mb" # "mb", "kb"
402.                block_size                = "4kb" # "mb", "kb"
403.                write_buffer              = "4mb" # "mb", "kb"
404.                write_l0_pause_trigger    = 12
405.                write_l0_slowdown_trigger = 8
406.                # the level db file size (default is 2mb, v1.2 is 8mb)
407.                compaction_table_size     = "8mb"

408.    [database.radar]
409.        encrypt = false
410.        type="leveldb"
411.        [database.radar.leveldb]
412.                path="radardb/leveldb/"
413.				logpath="radardb/log"
414.                block_cache_capacity      = "8mb" # "mb", "kb"
415.                block_size                = "4kb" # "mb", "kb"
416.                write_buffer              = "4mb" # "mb", "kb"
417.                write_l0_pause_trigger    = 12
418.                write_l0_slowdown_trigger = 8
419.                # the level db file size (default is 2mb, v1.2 is 8mb)
420.                compaction_table_size     = "8mb"

421.    [database.mq]
422.        encrypt = false
423.        type="leveldb"
424.        [database.mq.leveldb]
425.                path="mqdb/leveldb/"
426.				logpath="mqdb/log"
427.                block_cache_capacity      = "8mb" # "mb", "kb"
428.                block_size                = "4kb" # "mb", "kb"
429.                write_buffer              = "4mb" # "mb", "kb"
430.                write_l0_pause_trigger    = 12
431.                write_l0_slowdown_trigger = 8
432.                # the level db file size (default is 2mb, v1.2 is 8mb)
433.                compaction_table_size     = "8mb"

434.	[database.minifile]
435.		consensus = "minifile/consensus"
436.		sync = "minifile/sync"
437.	  bloom = "minifile/bloom"
438.		nvp = "minifile/nvp"

439.    [database.indexdb]
440.        [database.indexdb.layer1]
441.            enable = false
442.            dbType = "mongodb"
443.       [database.indexdb.tempdb]
444.            path = "indexdb/tempdb/leveldb/"
445.       [database.indexdb.layer2]
446.            # Defines for which fields to create layer2 index, optional value including:
447.            #   1 - indicate field named block write time;
448.            #   2 - indicate field named transaction from;
449.            #   3 - indicate field named transaction to;
450.            #   4 - indicate field named transaction hash;
451.            # For example:
452.            #      active = [] - means dont create any layer2 index;
453.            #      active = [1] - means create layer2 index for block write time;
454.            #      active = [1, 2] - means create layer2 index for block write time and transaction from;
455.            # This config item works only when database.indexdb.layer1.enable is true.
456.            active = []
457.        [database.indexdb.mongodb]
458.            # if you should set username and password, please use
459.            # mongodb://username:password@127.0.0.1:27017?w=1&journal=true,
460.            # for example: "mongodb://flatoUser:123456@127.0.0.1:27017?w=1&journal=true"
461.            uri = "mongodb://127.0.0.1:27017/?w=1&journal=true"
462.            limit = 5000
463.            tlsEnable = false
464.            tlsCA = "certs/mongodb_ca.pem"
465.            tlsCertKey = "certs/mongodb_client_cert.pem"
466.	[cvp.backup]
467.    	path = "data/cvp"

468.	[send.args.extra.check]
469.		enable = false
470.		url    = "https://filoop.com/api/v1/safe/text"

3.2 节点流控配置（投产推荐）
----------------------------------------------------------------------------------------------------

节点流控主要分为节点所在服务器的流控配置以及平台级别的流控配置。

3.2.1 服务器流控配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
一般来说，由于平台的tps比较稳定，流量根据tps也处于稳定状态，但是例如节点宕机落后和增节点等特殊情况下的recovery行为会向其他节点区块，为了避免公网共享带宽环境下不影响其他应用程序的带宽占用情况，可根据需要限制节点带宽。推荐以下三种服务器流控方法：

3.2.1.1 网络中心统一调控
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

可由各机构的网络中心进行调控配置，由于各家机构策略不一，不做详述，可咨询各家机构的网络运维工作人员。

3.2.1.2 TC（Traffic Control）服务器端限流配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

对于应用服务器来说，报文分组从输入网卡（入口）接收进来，经过路由的查找，以确定是发给本机的，还是需要转发的，如果是转发的，则会从输出网卡（出口）发出，网络流量的控制通常发生在输出网卡处。一般说来，由于我们无法控制自己网络之外的设备，入口处的流量控制相对较难，因此我们这里处理的流量控制一般指出口处的流量控制。

tc.sh配置文件详解：

1. tc qdisc del dev eth0 root 2> /dev/null > /dev/null
2. # 删除原有的tc规则
3. tc qdisc add dev eth0 root handle 1: htb
4. # 为网卡eth0创建htb根队列
5. tc class add dev  eth0 parent 1: classid 1:1 htb rate  100mbit ceil 100mbit
6. # 为根队列创建子队列1:1分配带宽100M
7. tc class add dev  eth0 parent 1:1 classid 1:10 htb rate  10mbit ceil 10mbit
8. # 为1:1队列创建子队列1:10分配带宽10M
9. tc qdisc add dev  eth0 parent 1:10 sfq perturb 10
10. # 防止一个段内的ip占用整个宽带
11. tc filter add dev eth0 protocol ip parent 1: prio 1 u32 match ip dst 10.200.0.0/16 flowid 1:1
12. # 为跟队列添加优先级为1的过滤器使得发往10.200.xxx.xxx的包转到1:1队列
13. tc filter add dev eth0 protocol ip parent 1: prio 2 u32 match ip dst 0.0.0.0/0  flowid 1:10
14. # 为跟队列添加优先级为2过滤器使得发往所有ip的包都转到1:10队列

需要注意的是，tc.sh的相关命令都需要root权限。

3.2.1.3 Nginx转发限流配置
----------------------------------------------------------------------------------------------------------------

由于各家机构对节点服务器带宽要求不尽相同，可能需要nginx做统一的转发限流。我们目前提供nginx安装包及三个脚本：start.sh、stop.sh、reload.sh，可通过这三个脚本满足基本nginx配置需求。

假设A机构需要使用nginx做限流转发的话则为如下情况：

1. 机构A(hyperchain-1):
2. node1:172.16.0.1(内),port:50011
3. node2:172.16.0.2(内),port:50011
4. nginx:172.16.0.3(内),115.200.10.1(外)
5. 机构B(hyperchain-2):
6. node3:10.200.10.1(内),118.180.10.1(外),port:50011
7. node4:10.200.10.2(内),118.180.10.2(外),port:50011

则可编辑nginx/conf/nginx.conf如下：

1. stream {
2.    server {
3.        #node1
4.        listen               10001;
5.        proxy_pass           172.16.0.1:50011;
6.    }
7.    server {
8.        #node1-node3
9.        listen               10002;
10.        proxy_pass           118.180.10.1:50011;
11.        proxy_upload_rate    500k;
12.    }
13.    server {
14.        #node1-node4
15.        listen               10003;
16.        proxy_pass           118.180.10.2:50011;
17.        proxy_upload_rate    500k;
18.    }
19.   server {
20.        #node2
21.        listen               10004;
22.        proxy_pass           172.16.0.2:50011;
23.    }
24.    server {
25.        #node2-node3
26.        listen               10005;
27.        proxy_pass           118.180.10.1:50011;
28.        proxy_upload_rate    500k;
29.    }
30.    server {
31.        #node2-node4
32.        listen               10006;
33.        proxy_pass           118.180.10.2:50011;
34.        proxy_upload_rate    500k;
35.    }
36. }

需要注意的是，如果通过nginx节点间已经建立好长连接，这时如果修改poxy_upload_rate将不能通过reload.sh生效。所以，务必配置好nginx启动后，再启动flato平台。

3.2.2 平台流控配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

平台级别的流控主要有外部请求限流和平台带宽限流两部分，前者表示该节点可处理的来自客户端的突发的最大HTTP请求数，限制的是来自客户端的压力，一定程度上可防止系统遭受DDoS攻击。后者表示节点间通信的网络带宽限流，与《3.2.1 服务器流控配置》一样可以实现网络带宽限流，但区别在于，本节所述的网络带宽限流会根据配置文件里配的最大网络带宽来计算flato协议消息最大大小，从而达到带宽限流目的。

3.2.2.1 外部请求限流
--------------------------------------------------------------------------------------------------------

外部请求限流表示节点可处理的突发HTTP请求数是多少，该配置项可在配置文件ns_static.toml 中进行配置：

1. [rpc.qps.flowCtrl]
2. enable   = true # 是否开启外部请求限流
3. capacity = 100  # 令牌桶容量
4. limit    = 2000 # 稳定状态下节点的最大qps限制

说明：rpc.qps.flowCtrl.limit 代表节点稳定状态下的最大qps，如上文配置，2000即表示令牌桶每0.5ms恢复一个令牌。rpc.qps.flowCtrl.capacity 代表令牌桶容量，capacity+limit 可以认为是节点允许的突发流量最大值。如上文配置，在该配置下，节点同一时间内可处理的突发HTTP请求数为 2000 + 100 = 2100。一般来说，设置节点最大突发流量数只需要配置 rpc.qps.flowCtrl.limit 即可。

3.2.2.2 节点带宽限流
------------------------------------------------------------------------------------------------------------------

节点带宽限流限制的是节点出口带宽的最大大小，该配置项可在配置文件 global.toml 中进行配置，该配置默认关闭。需要说明的是，节点实际出口带宽与交易大小和并发处理交易数（TPS）有强关联，开启该配置可能导致节点实际出口带宽值超过配置设置的最大出口带宽时容易进入viewchange状态拒收新交易，但节点最终总能恢复到正常状态并接受处理新交易：

1. [flow.control.bandwidth]
2. enable            = false
3. outgoingBandwidth = "500Mb" # NOTE: The unit is Mb/s NOT MB/s

3.3 性能相关配置
--------------------------------------------------------------------------------------------------------------

下文列出了影响系统性能的若干配置项，若您期望达到最佳性能效果或遇到性能下降的情况，请参考以下配置说明：

3.3.1 共识模块
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`consensus.set.set_size`

节点单次广播的交易数量，默认值为25，建议区间【10,500】，极端情况set_size=1时, 性能下降20%左右。

`consensus.set.batch_size`

主节点单次打包的交易数量上限，默认值为500，建议区间【250,8000】，且宜大不宜小，极端情况batch_size=[1-16]时，性能下降95%左右。

`consensus.rbft.k`

RBFT共识算法的checkpoint间隔（以区块为单位），默认值为10，建议区间【8,32】。

在不同场景下的最佳性能配置：
------------------------------------------------------------------------------------------------------------

- 在普通转账场景下，当batch_size=【6000,8000】，set_size=【200,500】，可获得最佳性能，该配置比默认配置下的最高TPS提升10%左右；

- 在合约存证场景下，当batch_size=【200,300】，set_size=【32,64】，可获得最佳性能，该配置比默认配置下的最高TPS提升10%左右。

3.3.2 网络模块
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`flow.control.bandwidth`

限制带宽上限，默认值为500Mb/s。如果实际所需带宽<bandwidth配置，则性能无影响；如果实际所需带宽>bandwidth配置，则性能降低50%左右，因共识消息可能阻塞，节点极易进入syncChain或viewChange的状态，导致集群拒收交易，从而严重影响TPS。因此`flow.control.bandwidth`需要根据实际所需带宽进行配置。

3.3.3 执行与存储模块
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`database.indexdb.layer1.enable`

是否开启索引数据库，默认关闭。对于普通转账/extra存证场景，开启索引数据库后是否对系统性能产生影响，由磁盘性能决定，ssd几乎不会有性能影响，本地独享hdd会有30%左右的性能下降，共享存储hdd可能会有80%的性能下降；对于合约存证场景，由于实际业务场景的TPS都在1000以内，到不了磁盘瓶颈，故暂无影响。

`duplicate.tx_drift_time`

SDK端和平台端服务器之间的时间差值，用以保证平台对交易是否过期判断的准确性，默认值为5min。在默认配置进行持续时间为5分钟的压测，性能会下降20%左右，所以如果要进行短时间高TPS压测，务必同步服务器时间，并将tx_drift_time改为1s~5s。注意，不能设0s，因为即使同步了时间，服务器间始终会有时差，导致交易拒收。

第四章 节点管理
======================================================================================

目前flato的增删节点需要动态的通过创建提案、投票、执行的形式进行，只有投票通过之后才能执行提案进行实际的增删节点的操作。相应的，账户对提案投票的权限也通过提案来进行权限管理。为了简化操作，提供了rockit工具快速进行节点管理。

4.1 rockit工具获取
--------------------------------------------------------------------------------------------

如果您已通过其他方式获取安装包请忽略此步骤。

【内部】登录OA：[__https://moffi.hyperchain.cn/__](https://moffi.hyperchain.cn/)

- 点击签发->平台组件->组件列表->rockit->下载，选择适用于您平台的rockit版本下载（**目前暂无法通过该方式获取rockit安装包，请联系相关人员**）

为了便于使用rockit工具，建议将rockit放到 `/usr/local/bin` 或者 `$GOPATH/bin` 目录下。如果没有，也可以将其当做一个普通的二进制文件使用 `./rockit [cmd]` 进行操作，此时必须确保目录文件夹下含有rockit二进制文件。

4.2 初始化
------------------------------------------------------------------------

初始化的操作只需操作一遍就好。

4.2.1  rockit工作目录初始化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在使用rockit时，需要使用一个独立的工作目录。**所有rockit操作都需要在工作目录下执行，不可以在工作目录之外或者是工作目录的子目录中，否则会产生异常** 。可以通过以下命令初始化出一个工作目录：

1. # 创建test空目录
2. mkdir test && cp rockit test && cd test
3. # 初始化目录
4. rockit dir

该命令会将rockit使用过程中需要使用的配置文件及文件目录进行初始化，得到的目录结构如下

1. test
2. |_publickey   #存放各节点公钥
3. |_keystore    #存放管理员账户私钥
4. |_config      # gosdk配置文件
5. | |_hpc.toml  # 通过gosdk连接节点服务器的配置文件

`publickey` 文件夹下放的是节点的公钥

`keystore` 文件夹下放的是管理员账户私钥（默认管理员为genesis账户）

hpc.toml文件详解

1. text
2. title = "GoSDK configuratoin file"

3. namespace = "global"

4. #发送重新连接请求间隔(/ms)
5. reConnectTime = 10000

8. [jsonRPC]
9.    # local environment
10.    nodes = ["localhost","localhost","localhost","localhost"]

11.    # JsonRpc connect port
12.    # local ports
13.    ports = ["8081", "8082", "8083", "8084"]

14. [webSocket]
15.    # webSocket connect port
16.    ports = ["10001", "10002", "10003", "10004"]

17. [polling]
18.     #重发次数
19.     resendTime = 10
20.     #第一次轮训时间间隔 unit /ms
21.     firstPollingInterval = 100
22.     #发送一次,第一次轮训的次数
23.     firstPollingTimes = 10
24.     #第二次轮训时间间隔 unit /ms
25.     secondPollingInterval = 1000
26.     #发送一次,第二次轮训的次数
27.     secondPollingTimes = 10

28. [privacy]
29.     #send Tcert during the request or not
30.     sendTcert = false
31.     #if sendTcert is true , you should add follow path.
32.     #the paths followed are relative to conf root path
33.     sdkcertPath = "certs/sdkcert.cert"
34.     sdkcertPrivPath = "certs/sdkcert.priv"
35.     uniquePubPath = "certs/unique.pub"
36.     uniquePrivPath = "certs/unique.priv"
37.     cfca = false

38. [security]
39.    #Use Https
40.    https = false
41.    #If https is true, you shoule add follow properties
42.    #the paths followed are relative to conf root path
43.    tlsca = "certs/tls/tlsca.ca"
44.    tlspeerCert = "certs/tls/tls_peer.cert"
45.    tlspeerPriv = "certs/tls/tls_peer.priv"

46. [log]
47.    #设置日志输出门槛
48.    #"CRITICAL","ERROR","WARNING","NOTICE","INFO","DEBUG",
49.    log_level = "ERROR"
50.    #存放日志文件夹
51.    log_dir = "./logs"

52. [transport]
53.    	# MaxIdleConns controls the maximum number of idle (keep-alive)
54. 	  # connections across all hosts. Zero means no limit.
55.     maxIdleConns = 0
56. 	  # MaxIdleConnsPerHost, if non-zero, controls the maximum idle
57. 	  # (keep-alive) connections to keep per-host. If zero,
58. 	  # DefaultMaxIdleConnsPerHost is used.
59. 	  maxIdleConnsPerHost = 10

60. [tx]
61.    # if it is use for hyperchain, please use 1.0 to replace default
62.    # if use for flato, please use 2.2 to replace default
63.    version = "2.2"

4.2.2 节点初始化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

目前只有rbft共识算法支持动态增删节点。因此，需要预先启动至少四个节点。由于预先启动的节点的证书是通过线下颁发的，且作为创世节点，不是通过提案投票的形式，因此在几个节点启动完成后需要通过提案完成初始化的流程。

初始化时包括了节点初始化、管理员账户初始化以及提案投票阈值初始化。

工作目录初始化之后，可以使用`rockit` 的`init` 初始化节点信息、管理员账户及提案投票阈值。`init` 命令的说明如下：

1. crete proposal and init node account

2. Usage:
3.  rockit init [flags]

4. Examples:
5. rockit init --ns global --nodePub "hello1 hello2 hello3 hello4" --nodes  "node1 node2 node3 node4"

6. Flags:
7.       --admins string     grant admin role to given account, split by space. for example:0x00000000000001
8.   -h, --help              help for init
9.       --nodes string      init hostname of nodes,default node1-node4 (default "node1 node2 node3 node4")
10.      --nodesPub string   public key of nodes, in publickey file, input public key file name, defaule node1-node4 (default "node1 node2 node3 node4")
11.      --threshold int     proposal threshold, default 1 (default 1)

12. Global Flags:
13.      --ns string   ns of nodes,default global (default "global")

其中，通过 `--admins` 知道要初始化的新的管理员账户地址，用空格分割，默认值为空；通过 `--ns` 知道初始化的namespace，默认为 `gloabl` ；通过 `--nodes` 指定初始化的节点名，用空格分割，默认值为 `node1 node2 node3 node4`  ；通过 `--nodesPub` 指定与 `nodes` 对应的节点的公钥文件名（放在publickey文件夹下，命名方式为 `文件名+.cert` ），默认值为 `node1 node2 node3 node4` ，与 `nodes` 的默认值对应；通过 `--threshold` 指定初始化的新的提案投票阈值，默认值为1.

节点启动后，将所有genesis账户（默认的管理员账户）的私钥放到 `keystore` 目录下（默认提案投票阈值为所有的管理员总数），将每个节点的公钥按照放到 `publickey` 目录下，如果启动的是四个节点，每个节点的hostname分别为 `node1` 、 `node2` 、 `node3` 、 `node4` ,且每个节点的公钥也都放在了 `publickey` 目录下，分别命名为 `node1.cert` 、 `node2.cert` 、 `node3.cert` 、  `node4.cert` ，可使用如下命令初始化节点：

14. text
15. # 使用默认值初始化，即节点名为node1-node4，不初始化新的admin账户，提案投票阈值初始化为1,namespace为global
16. rockit init
17. # 使用指定的值初始化
18. #rockit init --nodes "node1 node2 node3 node4 node5" --publickey "node1 node2 node3 node4 node5" --admins "0x9202d80df4c6d658290bc0c18fc2ddeb08735c8c0x0eb804bf69adb78d19555db1f869f26ccc2c0cfb" --threshold 2

4.3 添加节点
--------------------------------------------------------------------------------------------------------------

通过 `rockit` 的 `addvp` 实现动态新增节点。 `addvp` 命令说明如下：

1. text
2. add vp node

3. Usage:
4.   rockit addvp [flags]

5. Examples:
6. rockit addvp

7. Flags:
8.       --cfca             not use decentralized ca,default false
9.   -h, --help             help for addvp
10.       --node string      hostname of new node,default node5 (default "node5")
11.       --nodePub string   public key's absolute path of new node (default "node5")

12. Global Flags:
13.      --ns string   ns of nodes,default global (default "global")

通过 `--cfca` 指定是否 **不使用分布式CA** ，默认为false；通过 `--node` 指定新增节点的hostname，默认值为 `node5` ；通过 `--nodePub` 指定新增节点的公钥（如果不使用分布式CA新增节点，则需要指定，使用分布式CA新增节点则不需要指定），默认值为 `node5` 

注：使用 `addvp` 命名新增节点时需要保证 `keystore` 目录下的管理员账户私钥的总数大于等于提案投票阈值。

4.3.1 非分布式CA新增VP节点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

使用非分布式CA新增节点时，新加入的节点除了端口等配置文件配置好以外，还需要事先拿到证书，以便与其他节点建立逻辑连接。当这些都准备好后，将新节点启动，当新节点启动后与其他节点连接成功后，打印如下日志后，可使用 `addvp` 命令添加节点。

|image0|

使用 `rockit` 的 `addvp` ，非分布式CA方式添加节点如下：

1. text
2. # 添加的新节点为默认的node5，新节点的公钥也放在publickey目录下，命令为node5.cert
3. rockit addvp --cfca true

执行完成后等待新节点加入共识、同步区块。

4.3.2 分布式CA新增VP节点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

使用分布式CA新增节点时，新加入的节点准备好后，将新节点启动，在新节点与其他节点建立好物理连接之后，其他节点会有新的区块生成，这时可使用 `rockit` 的 `addvp` ，分布式CA方式添加节点，示例如下：

1. text
2. # 添加的新节点为默认的node5
3. rockit addvp

执行完成后等待新节点加入共识、同步区块。

4.4 删除节点
-------------------------------------------------------------------------------------------

通过 `rockit` 的 `removevp` 实现动态删除节点，起说明如下：

1. text
2. remove vp node

3. Usage:
4.  rockit rmvp [flags]

5. Examples:
6. rockit rmvp --node node5 --ns global

7. Flags:
8.   -h, --help          help for rmvp
9.       --node string   name of the deleted node ,default node5 (default "node5")

10. Global Flags:
11.     --ns string   ns of nodes,default global (default "global")

通过 `--node` 指定要删除的vp节点的hostname，默认值为 `node5` ，其使用示例如下：

12. text
13. # 删除节点node5
14. rockit rmvp

执行完成后等待节点退出共识，如果节点退出这个namespace后没有处于其他的namespace中，节点将断开连接。

第五章 证书管理 
======================================================================

5.1 证书体系
------------------------------------------------------------------------------------

5.1.1 证书体系介绍
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

按照PKI系统的规范，证书按照在证书链中的位置，可以被分为最终实体证书、中间证书、根证书（我们简称为rootCA）三种。中间证书和根证书都可以签发证书，而最终实体证书不能继续签发证书。在证书链中，相邻的两个证书是签发和被签发的关系，因此可以相对地称二者为父证书和子证书。验证子证书的有效性时需要用到对应的父证书。

验证一个子证书的有效性可以粗略的认为分成以下步骤： **验证证书内容、验证证书签名、查询是否被吊销** 。验证证书签名是一个验签的过程，flato使用父证书的公钥验证该证书的签名是否有效，查询是否被吊销则是通过查询吊销列表（一个黑名单）完成。我们更需要重点强调的是对证书内容的验证。该验证证书内容的步骤中，flato除了验证基本的过期时间、签发结构和被签发主体的身份等内容，还会验证和区块链有关的相应信息，这属于flato对证书的特有要求：

1. **证书用途** 。证书中会有相应字段规定证书的用处，根据证书的功能可划分为节点证书和SDK证书。顾名思义，节点证书配置在节点上用于节点身份的验证，而SDK证书( **sdkcert** )则配置在平台SDK上以确定SDK合法身份。节点证书包括：**ecert** 和 **rcert** ，其中VP节点将配置ecert，而NVP节点则配置rcert。

2. **该证书所属节点的hostname** 。证书是和节点绑定的，因为证书中写入了hostname的信息，因此node1的证书拷贝到node2是不能正常工作的。

证书能够被验证通过有个关键的前提条件，即节点能获取到其父证书并且承认该父证书的有效性。如果不能获取到父证书，那么内容验证或者签名验证都无从说起。

部署和运维人员应该在相应的目录中放置必需的CA证书，在flato中我们称该目录为 **可信CA列表** ，也就是说部署人员应该将所有认可的、有效的、必需的CA证书加入到可信CA列表中。启动后，当有外来证书需要被验证时，flato会从可信CA列表中搜索证书并尝试构建证书链，如果构建成功则能够进一步完成上述的三部验证，否则验证失败。可信CA列表的路径配置在ns_static.toml的encryption.root.ca中。

|image1|

encryption.ecert.ecert配置了节点的证书目录。

5.1.2 概念及用途
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**1 namespace级别**

每个节点都有配置命名空间，不同的命名空间之间是物理隔离的，但可以处于同一个区块链网络内。所有在一个命名空间中的节点都处于同一条业务链上，不同的命名空间处于不同的业务链上，就比如不同的数据库一样。以下介绍namespace级别的相关证书，它们主要实现节点的准入控制并放置在cert目录下：

- ROOTCA（节点根目录/namespaces/global/certs/CA）根部证书，用于节点证书的分发

ROOTCA参与以下几类具体节点证书的生成、验证及吊销，是所有证书的根证书。区块链中可能存在多个ROOTCA，一个ROOTCA只能验证由自己颁发的证书的合法性。

- ECERT（节点根目录/namespaces/global/certs/certs） 节点准入证书，用于证明该节点为VP节点，可参与共识验证

- RCERT（节点根目录/namespaces/global/certs/certs）  节点角色证书，用于证明该节点为NVP节点，不参与共识验证，仅参与记账

持有ECERT或RCERT的节点和SDK才能访问区块链网络。运行中的节点会定期检查其他节点的证书合法性。

- SDKCERT（sdk目录/certs，不同sdk会有不同） 客户端准入证书，用于证明SDK的合法性

非法的SDK将无法向节点发出请求。

**2 链级别**

除上述证书外，flato还设置了跨namespace的节点级证书，主要用于节点间ssl通信，放置在tls目录下。

- TLSCA（certs/tls）  安全传输层协议CA证书，用于TLSCERT的分发

- TLSCERT（certs/tls）节点安全传输层协议证书，用于传输层

在传输网络传输过程中需要验证传输层安全协议证书的安全性，验证通过即可以进行正常网络通信，反之则无法进行网络通信。

5.2 certgen使用说明
--------------------------------------------------------------------------------------------------

certgen作为flato证书管理的配套工具，用来生成和管理相关的CA证书和数字证书。certgen主要包括证书签发，公私钥生成，证书检查等功能。

5.2.1 certgen安装
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**法一：源码安装**

step1  源码下载

git clone git@git.hyperchain.cn:innovation/certgen.git $GOPATH/src/git.hyperchain.cn/innovation/certgen

step2 编译安装

cd $GOAPTH/src/git.hyperchain.cn/innovation/certgen

go build

注意：使用 go1.13.x 版本

**法二：使用预编译版本**

【公司外部】登录飞洛：[__https://filoop.com/console/issue__](https://filoop.com/console/issue)

- 登录账号->控制台->资源下载->证书签发-填写信息->选择对应版本下载

> 注意：飞洛账号需要经过实名认证、账号审核过后才可看到资源下载，如果相关问题可联系飞洛客服

【公司内部】登录OA：[__https://moffi.hyperchain.cn/__](https://moffi.hyperchain.cn/)

- 点击签发->平台组件->组件列表->certgen->下载，选择适用于您平台的版本下载

> 注意：普通用户使用的时候如果无法放到path搜索目录，则下面的命令用 `./certgen` 运行

完整的certgen使用指南参考链接（certgen使用手册v2）：

[__http://thoughts.hyperchain.cn:8099/workspaces/5b6c3babbe825b41b446a307/docs/5fc78a4b9e5cf00001f95fcd__](http://thoughts.hyperchain.cn:8099/workspaces/5b6c3babbe825b41b446a307/docs/5fc78a4b9e5cf00001f95fcd)

**检验certgen是否安装成功**

``$ certgen``

|image2|

5.2.2 生成证书
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**1** **根证书签发**

签发根证书(自签证书)，命令如下：

``$ certgen gs --cn=CommonName --org=Org ./root.ca ./root.priv``

运行上述命令，会在指定路径生成root.ca、root.priv两个文件，即根证书文件。需要通过 `--cn` 指定根证书的name， `--org` 指定根证书的组织。

默认情况下生成的是secp156k1曲线的证书，可以通过 `--c` flag来指定曲线类型，可选的类型有p256、secp256k1、sm2这三种。p256又可以叫做secp256r1等，但在该flag中统一使用p256代表这类椭圆曲线。另外 `--from` 和 `--to` flag用于指定证书的有效期限。下同。

例如签发国密根证书（自签），命令如下：

``$ certgen gs --c=sm2 --cn=CommonName --o=Org ./root_gm.ca ./root_gm.priv``

运行上述命令，在指定路径产生root_gm.ca、root_gm.priv两个文件，即国密根证书和对应私钥。

**2 子证书签发（子密钥不存在）**

flato平台使用的子证书根据用途不同分为两种类型，分别是ECert和SDKCert。通过 `-ct` 可以指定子证书的类型为上述两种之一（类型名称不区分大小写）。子证书也可以不拥有类型，只需要不指定 `-ct` 即可。

ECERT：

``$ certgen gc --cn=node --o=flato --isca=y --ct=ecert root.ca root.priv node.cert node.priv``

需要通过 `--cn` 指定子证书的name， `--o` 指定子证书的组织， `--isca=y` 表示是ca证书。

第一个参数为根证书存储路径，第二个参数为根证书的私钥存储路径，第三个参数为节点的证书存储路径，第四个参数为节点的私钥存储路径。

SDKCERT：

``$ certgen gc --cn=node --org=flato --isCA=n --ct=sdkcert root.ca root.priv node.cert node.priv``

需要通过 `--n` 指定子证书的name， `--org` 指定子证书的组织， `--isca=n` 表示是非ca证书。

同时可以使用 `--c` flag来指定椭圆曲线， `--c` 可选的曲线类型有 p256、secp256k1、sm2共三种。

> 注意：国密子证书只能由国密父证书生成，密钥是p256或者secp256k1的父证书，可以生成secp256k1或者p256类型的子证书。同样适用于下面的子证书签发命令（gc）。

**2 生成公私钥对**

生成一对公私钥，该指令需两个参数:

``$ certgen gk ./key.priv ./key.pub``

第一个参数表示要生成的密钥对的私钥存储路径，第二个参数表示要生成的密钥对的公钥存储路径。

使用flag `--c` 可以指定生成公私钥对的用到的椭圆曲线类型。

3 **子证书签发(子密钥已存在)**

此方式需要各个节点的公钥，通过公钥生成子证书时不需要特别指明曲线类型。

flato平台使用的子证书根据用途不同分为两种类型，分别是ECert和SDKCert。通过 `-t` 可以指定子证书的类型为上述两种之一（类型名称不区分大小写）。子证书也可以不拥有类型，只需要不指定 `-t` 即可。

ECERT：

``$ certgen gc --cn=node --org=flato --isca=y --ct=ecert root.ca root.priv key.pub node.cert``

需要通过 `--cn` 指定子证书的name， `--org` 指定子证书的组织， `--isca=y` 表示是ca证书。

第一个参数为根证书存储路径，第二个参数为根证书的私钥存储路径，第三个参数为节点的公钥存储路径，第四个参数为节点的证书存储路径。

使用 `--from` 和 `--to` 指定子证书的有效期。

SDKCERT：

``$ certgen gc --cn=node --org=flato --isca=n --ct=sdkcert root.ca root.priv key.pub node.cert``

需要通过 `--cn` 指定子证书的name， `--org` 指定子证书的组织， `--isca=n` 表示是非ca证书。

**3 Tls证书签发**

生成根CA:

``$ certgen gs --c sm2 --from 2020-12-4 --to 2030-12-4 ./tls_root.ca ./tls_root.priv --cn flato``

需要通过 `--cn` 指定根证书的name， `--org` 指定根证书所属的组织。

通过 `--c` 指定该自签名证书的曲线类型， `--from` 和 `--to` 指定了证书的有效日期和时间。

生成tls证书:

``$ certgen gc --isca=n --from 2020-12-04 --to 2030-12-04 --c sm2  ./tls_root.ca ./tls_root.priv ./tls_peer1.cert  --cn flato``

通过 `--isca=n` 表示该tls子证书为非ca证书， `--cn` 指定该证书的name， `--org` 指定该子证书所属的组织。

通过 `--c` 指定该tls证书的曲线类型， `--from` 和 `--to` 指定了证书的有效日期和时间。

5.2.3 检查证书
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

检查子证书是否由该CA证书签发：

1. $ certgen cc ./root ./sub
2. root cert path: pathToCA
3. sub cert path: pathToCert

5.3 CA证书签发管理方案
---------------------------------------------------------------------------------------------

5.3.1 CA生成及保管（certgen）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **非分布式CA**

该模式下仅需要维护一套CA和对应的证书。

``# 签发ROOTCA``

``$ certgen gs --cn=node --org=flato -isca=y root.ca root.priv``

- **分布式CA**

该模式下可维护任意多套CA、Cert、私钥。所以需要为每个CA签发ROOTCA。

5.3.2 节点cert签发（certgen）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **非分布式CA**

创世的四个VP节点需要使用以上CA通过certgen的如下命令生成各自节点的一套证书：

1. # 签发ECert:
2. $ certgen gc --cn=node --org=flato --isca=n --ct=ecert
3. ./root.ca ./ root.priv ./ecert.cert ./ecert.priv
4. # 签发SDKCert:
5. $ certgen gc --cn=node --org=flato --isca=n --ct=sdkcert
6. ./root.ca ./root.priv ./sdkcert.cert ./sdkcert.priv

需要通过 `--cn` 指定子证书的name， `--org` 指定子证书的组织， `--isca` 表示是否是ca证书。同时可以使用 `--c` flag来指定椭圆曲线。

新增证书签发命令：(根据自定义公钥进行子证书签发)

1. #签发ECert:
2. $ certgen gc --cn=node --org=flato --isca=n --ct=ecert ./parent.cert ./parent.priv ./subcert.pub ./subcert.cert
3. #签发SDKCert:
4. $ certgen gc --cn=node --org=flato --isca=n --ct=sdkcert ./parent.cert ./parent.priv ./subcert.pub ./subcert.cert

注：subcert.pub必须事前生成。

- **分布式CA**

对分布式CA来说，每个CA都需要向节点颁发证书。

假设有四个节点为node1、node2、node3、node4, 那么node1的certs目录应当含有一个私钥和node2、node3、node4为其颁发证书，证书中的信息为node1节点的信息。node2给node1颁发证书，意为root2.ca对node1的公钥进行签发的证书。

这里假设为node1生成由node2颁发的证书：

``# 签发ecert``

``$ certgen gc --cn=subcert --org=flato -isca=n --ct=ecert ./parent.cert ./parent.priv ./subcert.pub ./subcert.cert``

那么此时 `--cn` 需要指定为node1的CommonName，第一个参数为node2的根证书，第二个参数为node2的私钥存储路径，第三个参数为node1的公钥存储路径，第四个参数为node2给node1颁发的证书存储路径。

分布式CA下，需要生成的相应子证书如下：

node1需要为node2、node3、node4颁发子证书；

node2需要为node1、node3、node4颁发子证书；

node3需要为node1、node2、node4颁发子证书；

node4需要为node1、node2、node3颁发子证书。

生成sdkcert:

``#签发SDKCert:``

``$ certgen gc --n=node --org=flato --isca=n -ct=sdkcert ./parent.cert ./parent.priv ./subcert.pub ./subcert.cert``

注：subcert.pub必须事前生成。

5.3.3 VP节点的cert文件配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**非分布式CA**

非分布式CA即原有的中心化CA，在该模式下仅需要维护一套CA和对应的证书。采用的策略是启动时从配置项读入，内存维护相应证书和CA，没有运行中持久化的需要。对于一个VP节点主要包括两个子目录：CA、certs。

- CA目录：保存CA证书和CA私钥（root.ca和root.priv）。所有节点的CA目录下内容应该完全一致，使用同一个CA进行认证

- certs目录：保存由节点保存CA所颁发的一套证书，至少包含三个文件，节点私钥（key.priv）、节点ECERT证书（node1.cert）、节点SDKCERT（sdkcert.cert）

- tls目录：保存安全传输层协议证书，包含tlsCA（tlsca.ca、tlsca.priv）及其生成的tls子证书（tls_peer.cert、tls_peer.priv）

|image3|

**分布式CA**

分布式CA可维护任意多套CA、Cert、私钥。与非分布式的目录结构相同，包括CA目录和certs目录。根据节点为flato启动节点和后续加入节点两种类型，证书的配置有所区别：

- 1. 启动节点

在规范化部署的四节点启动时，节点1的目录内容如下所示（其余三个节点配置类似）：

- CA目录：保存CA证书和CA私钥。四个节点的CA目录下内容应该完全一致。其中root1为node1的CA，root2为node2的CA，root3为node3的CA，root4为node4的CA。

- certs目录：保存节点私钥（key.priv）、由其他节点CA所颁发的一套证书。至少包含三个文件，node2节点CA（root2）颁发给node1的ECERT证书（node2.cert）、node3节点CA（root3）颁发给node1的ECERT证书（node3.cert）、node4节点CA（root4）颁发给node1的ECERT证书（node4.cert）。

|image4|

- 2. 新加入节点

如果有新节点要加入flato，无需再配置证书，但需要保证节点目录下有CA和certs目录（空目录）。

5.3.4 ns_static.toml文件相关配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**非分布式和分布式CA配置项**

- [distributedCA]配置项

- 非分布式CA：修改为 `enable = true`

- 分布式CA：修改为 `enable = false`

**证书目录配置项**

- [encryption.root]配置项，修改为 `ca = "certs/CA"`

- [encryption.ecert]配置项，修改为 `ecert = "certs/certs"`

5.3.5 sdk文件相关配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

gosdk/conf/hpc.toml文件配置

如需使用sdkcert进行交互验证： `sendTcert = true` ，否则置为 `false`

第六章 IPC命令
===============================================================================

在hyperchain 的运行目录下，有一个 `hpc_1.ipc` （取决于配置文件中的配置）文件。使用如下命令进入交互式命令模式：

``./hyperchain -s --ipc=hpc_1.ipc``

将会进入交互式命令行模式。关于IPC命令使用说明可以详见help。本章将主要介绍IPC命令常用的几个命令。

6.1 网络连接管理
-------------------------------------------------------------------------------------

你可以使用 `network` 命令进行物理网络连接的管理。

6.1.1 列出连接主机
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

可以查询得到当前节点的主机名（Self），已连接节点数（Connected Node Count），以及对应的连接的各个节点的主机名和套接字。

|image5|

6.1.2 更新连接主机IP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当节点配置文件里配错了对端节点的IP地址，可以在本地节点不停机重启的情况下通过IPC命令更新所要连的节点IP。步骤主要分为两步，首先，通过 close 子命令断开连接，然后，再通过 connect 子命令指定新IP进行连接。

network close <hostname>：关闭到某个节点的连接。

network connect <hostname> <ip:port> -f：向某个指定节点建立连接。

|image6|

6.1.3 检测通信延迟
------------------------------------------------------------------------------------------------------

network ping <hostname>：测试到某个节点的网络延迟。

|image7|

6.2 日志级别修改
----------------------------------------------------------------------------------------------------

ipc命令也支持日志级别修改，修改的日志级别立即生效：

参数为： logger setlevel <namespace> <module> <log level>

|image8|

6.3 LICENSE信息查询
--------------------------------------------------------------------------------------------------------

通过ipc命令可以获取到本机ip，LICENSE绑定ip以及LICENSE的过期时间。

|image9|

6.4 对外服务管理
-----------------------------------------------------------------------------------

通过ipc命令可以启动、关闭或者重启hyperchain的对外HTTP JSON-RPC服务，比较实用的一个功能是在节点不重启的情况下，通过 `service` 命令修改该服务监听的端口号。

启动JSON-RPC服务：service http start <port>

关闭JSON-RPC服务：service http stop

重启JSON-RPC服务：service http restart

|image10|

6.5 非交互式命令
----------------------------------------------------------------------------------------------------

上述命令也支持非交互方式进行，请使用如下命令进行：

``./hyperchain -s --ipc=hpc_1.ipc --nit --cmd="network lis``

第七章 日志说明
==================================================================================================

7.1 日志配置
---------------------------------------------------------------------------------

Flato日志主要分为系统日志和NS（NameSpace）日志，这两者相互独立，互不影响。

- 系统日志：相关配置位于 `global.toml` 文件中，主要包括系统级别的main、metrics、jsonrpc、nsmgr、audit、txgen、hypernet、config等模块；

- NS日志：相关配置位于 `ns_static.toml` 文件中，主要包括每个namespace下的config、p2p、consensus、flatodb、eventhub等模块。

系统日志和NS日志的配置内容相同，位于各自配置文件的 `[log]` 和 `[log.module]` 配置项中。其中 `[log]` 是日志通用配置， `[log.module]` 配置各模块的日志级别。

通用日志配置项如下：

- `dump_file` ：是否输出日志文件，推荐开启

- `dump_interval` ：日志文件生成时间间隔，避免单个日志文件过大，默认为24h

- `log_dir` ：系统级别的日志文件路径

- `log_level` ：系统级别的日志级别，可被module下的模块设置覆盖，目前主要包括 `DEBUG` 、 `INFO` 、 `NOTICE` 、 `WARNING` 、 `ERROR` 、 `CRITICAL`

- `max_log_size` ：系统级别日志文件的最大大小，默认为200MB

- `check_size_interval` ：检查系统级别日志文件大小的时间间隔

日志配置示例如下：

1. [log]
2. dump_file           = true
3. dump_interval       = "24h"
4. log_dir             = "./logs"
5. log_level           = "NOTICE"
6. max_log_size        = "200mb"
7. check_size_interval = "2m"

8. [log.module]
9. p2p         = "NOTICE"
10. consensus   = "INFO"
11. core        = "NOTICE"
12. flatodb     = "NOTICE"
13. eventhub    = "NOTICE"
14. jsonrpc     = "NOTICE"
15. hypernet    = "NOTICE"

7.2 日志格式
----------------------------------------------------------------------------------

Flato的所有模块都制定了统一的日志格式，方便用户通过日志查看系统状态。

日志记录格式如下：

``#日志格式``

``log_level [time] [module_name] file_name:line_number content``

``#日志示例``

``NOTI [2020-08-21T16:18:03.824] [consensus] flato-rbft.git@v0.2.8/exec.go:183 ======== Replica 4 finished recovery, view=1/height=0.``

各字段含义如下：

- `log_level` : 日志级别，目前主要包括 `DEBUG` 、 `INFO` 、 `NOTICE` 、 `WARNING` 、 `ERROR` 、 `CRITICAL` ，其中在发生极严重错误时会输出 `CRITICAL`

- `time` : 日志输出时间，精确到毫秒

- `module_name` ：模块名称，如共识模块为 `consensus` 、网络模块为 `P2P` （逻辑连接层）和 `hypernet` （物理连接层）

- `file_name` ：文件名称

- `line_number` ：所在行数 

- `content` ：日志记录内容

7.3 常见日志说明
---------------------------------------------------------------------------

7.3.1 Flato版本号
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Flato节点启动的第一条日志会打印出版本号信息

``NOTI [2020-08-26T14:36:41.096] [main] flato/main.go:52 Flato Version:``

``release-1.0.0-20200826-3cfedd0ef``

该日志表示当前Flato的版本是release-1.0.0版本，打包二进制时间是2020年08月26日，二进制的提交号是3cfedd0ef

7.3.2 License信息
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``NOTI [2020-08-26T14:13:54.807] [main] license/license_id.go:127 license to 趣链科技, exp data 2020-12-31``

该日志标志着License检查成功，同时打印出了License的过期时间

7.3.3 Flato服务启动日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``NOTI [2020-08-26T14:13:54.765] [main] flato/main.go:83 Flato server starting...``

该日志标识着Flato服务开始启动

7.3.4 网络连接成功日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``NOTI [2020-08-26T14:13:54.767] [hypernet] flato-transport-grpc.git@v0.1.10/hypernet.go:448 successfully establish stream with node2``

该日志标志着成功连接到了hotstname为node2的节点

7.3.5 分区启动日志

Flato运行过程中一个节点可能参与到多个分区之中，其中启动一个分区的相关日志如下：

``NOTI [2020-08-26T14:13:54.802] [namespace] namespace/namespace.go:462 Try to start namespace global``

该日志标志着开始启动global分区

``NOTI [2020-08-26T14:13:57.348] [namespace] namespace/namespace.go:518 Namespace global start successfully``

该日志标志着global分区启动成功

7.3.6 共识世代检查日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Flato共识节点运行过程中，每发生一次节点增删，都会进行一次世代变更（epoch change），因此节点启动后需要进行世代检查，相关日志如下：

``INFO [2020-08-26T14:14:01.352] [consensus] flato-rbft.git@v0.2.8/epoch_mgr.go:67 ======== Replica 4 start epoch check, epoch=0``

该日志标识着节点尝试进行世代检查，且本节点的世代号为0

``INFO [2020-08-26T14:14:13.359] [consensus] flato-rbft.git@v0.2.8/exec.go:354 ======== Replica 4 finished epoch check, N=4/epoch=0/height=0/view=0``

该日志标志着世代检查成功，当前共识节点总数N为4个，世代号epoch为0，区块高度height为0，视图值view为0

7.3.7 数据恢复成功日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Flato共识节点启动或者发生异常后，都会进行数据恢复流程。

``NOTI [2020-08-26T14:14:16.014] [consensus] flato-rbft.git@v0.2.8/exec.go:183 ======== Replica 4 finished recovery, view=1/height=0.``

``NOTI [2020-08-26T14:14:16.014] [consensus] flato-rbft.git@v0.2.8/exec.go:184``

  +==============================================+
  |                                              |
  |            RBFT Recovery Finished            |
  |                                              |
  +==============================================+

该日志标志着数据恢复完成，当前视图值view为1，区块高度height为0

7.3.8 区块共识完成日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``NOTI [2020-08-26T14:14:55.188] [consensus] flato-rbft.git@v0.2.8/rbft_impl.go:1243 ======== Replica 4 Call execute, view=1/seqNo=1``

该日志标志着共识模块完成了1号区块的共识，开始执行1号区块

7.3.9 区块执行完成日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``NOTI [2020-08-26T14:14:55.189] [executor] executor/validator.go:191 commit 1``

该日志标志着1号区块执行完成

7.3.10 区块提交完成日志
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``NOTI [2020-08-26T14:14:55.195] [executor] executor/commitor.go:255 Block number 1``

``NOTI [2020-08-26T14:14:55.195] [executor] executor/commitor.go:256 Block hash b04662998b2c97ea84a1cb8bbc61bc667e3a262065abbe87c1045be76b8296f6``

该日志标志着1号区块提交写块完成，并且打印出了1号区块的哈希值

第八章 异常处理
================================================================================================================

8.1 网络异常
------------------------------------------------------------------------------------------------

8.1.1 端口占用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

grpc端口被占用时，节点无法启动。

jsonrpc端口被占用时，节点可以启动，但是在调用时会出现Connection reset by peer。

在容器里部署节点1后，其默认的rpc端口8081被宿主机抢占，这时向节点1发送jsonrpc请求，结果在命令行显示Recv failure：Connection reset by peer的报错

**处理方式：** lsof -i：8081查看端口号占用情况，kill掉占用端口的进程，重启节点

|image11|

8.1.2节点网络异常
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

集群中节点网络断开或开启防火墙，会导致节点间连接失败，查询节点状态，返回状态为timeout，表明本节点与该节点之间的网络连接断开。

**处理方式**：恢复网络或关闭防火墙后重启该节点。

`注意` ：出现该情况有可能是该节点未开放相应端口，请务必确认所有节点之间的网络通信正常，防止因为防火墙等问题导致节点间不能通信。

8.2 共识异常
--------------------------------------------------------------------------------------------------------------

8.2.1 重启后相同区块高度的节点数不超过规定数目
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

所有节点重新启动后，拥有相同区块高度的节点数不超过算法规定的数值。当N为4，F为1时，该值为3。

``quorum = int(math.Ceil(float64(N+F+1) / float64(2)))``

出现这种情况，即区块链网络中拜占庭节点的数量已经超过了算法容错的数目。

**处理方式**：等待超过quorum个节点正常启动后再发送交易进行共识写块。

8.2.2 Viewchange异常
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息：** Replica 2 received viewChange from replica 3

当在所有节点上看到这条消息，且消息都是来自相同的节点例如3号节点时， 说明节点3发生了异常情况触发了ViewChange，无需理会3号节点的行为，当节点3发送10次ViewChange都不成功时会自动触发recovery，最终达成一致。

**处理方式**：自动恢复

如果在在所有节点都看到类似消息而且不断出现时

**处理方式**：重启所有节点

8.2.3 Duplicate transaction异常
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息：** Duplicate transaction in addTxs with hash XXX

如果偶尔出现以上信息，是正常情况，可能是客户端在长时间未收到交易回执后重发交易导致的。

**处理方式**：自动恢复。

8.2.4 system is too busy异常
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息：** system is too busy

如果在SDK客户端出现这种报错，则说明客户端发送交易的频率过高，导致平台不能及时处理，交易缓存池达到了上限。

**处理方式**：SDK停止发送交易，等待平台处理一段时间后再发送交易。

8.3 数据异常
----------------------------------------------------------------------------------------

8.3.1 第一个区块merkle root不一致
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息：** mappedMerkleRoot and calculated root not match

- 首次启动集群，若集群中的节点的genesis账户不一致，则在产生第一个区块时，报错merkle root不一致；

- 新增节点到集群中，若新增节点的genesis账户和集群中节点不一致，则新节点同步数据产生第一个区块时，报错merkle root不一致；

**处理方式：** 保持集群节点间的genesis账户一模一样后，重启节点

8.3.2 数据同步失败 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息**：syncchain failed

当节点发现自身数据落后时，就会主动向其他节点请求数据，这个过程叫syncchain；syncchain同步过程由于节点网络问题或对方节点处于归档状态，会导致本次syncchain失败，失败后会尝试向其他节点重试，直到syncchain成功。

**处理方式：** 自动恢复

8.3.3 同一高度，重复commit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息：**

|image12|

节点到达checkpoint高度或发生一笔配置交易时，就会自动进行commit操作，但是由于某些外部原因删除共识数据库，并且重启节点，重启后的节点会对最近checkpoint高度或最后一笔配置交易进行commit，就会出现上述报错。

**处理方式：** 自动恢复

8.4 句柄值异常
--------------------------------------------------------------------------------------------------

8.4.1 系统句柄配置过低
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息**：to many open files

在节点运行过程中，写数据，特别是进行归档操作时，节点异常，节点日志报错to many open files，原因是归档过程中leveldb的句柄打开数量会急剧增多，flato上层限制总的系统文件句柄数不能小于16384。

**处理方式：** 修改服务器系统的句柄值，将句柄值改为65535后，重启节点

8.5 配置异常
--------------------------------------------------------------------------------------------------------------

8.5.1 配置异常
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息**：new namespace global failed: config is illegal

在节点启动时，回检查配置项，遇到配置项修改不合法时，启动时抛出异常，提醒自行检查，一般配置项都有取值建议可参考

|image13|

8.5.2 lock文件异常
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**报错信息：** Please make sure that you have deleted the lock file before restart Hyperchain.

在节点遇到上述任何异常，包括license过期、配置项异常、句柄值异常等，都会在根目录生成lock文件以防止守护进程启动异常的节点；修复方法为解决异常后，删除lock文件后重启启动节点

|image14|

第九章  产品激活、续期
==============================================================================================================

9.1 激活
--------------------------------------------------------------------------------

使用Hyperchain需要拥有本产品的激活码。本产品的激活码为名为LICENSE的文件。激活本产品只需将LICENSE文件放置在产品根目录下即可。

9.2 续期
--------------------------------------------------------------------------------

当您购买的Hyperchain产品即将到期时，您可以重新向本公司购买新的激活码，在不停止hyperchain服务的情况下动态替换旧LICENSE文件即可。

9.3 查询
---------------------------------------------------------------------------------------

您可以通过 `hypercli server license` 或者 `hyperchain -L` 或者ipcshell的 `license` 命令来查看LICENSE文件的过期时间以及IP信息。

# 联系我们

您可以通过以下联系方式联系我们：

联系电话: 0571-81180102, 0571-81180103

Email: support@hyperchain.cn

我们的工作时间是：9:00 AM to 5:00 PM PST (Monday-Friday, except Holidays)

**请您在联系我们之前，将以下信息附上：**

``姓名`` 、 ``公司`` 、 ``联系电话`` 、 ``电子邮箱`` 、 ``操作系统及版本号`` 、 ``产品版本号`` 、 ``问题概述`` 、 ``问题描述``


附录
================================================================================

配置项说明
---------------------------------------------------------------------------------------------------

global.toml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

文件描述：全局配置项，一般使用默认值。

- **log.dump_interval** :产生新的日志文件的时间间隔，避免单个日志文件过大。
- **log.log_dir** :系统级别日志文件路径。
- **log.log_level** :系统级别的日志级别，可被module下的模块设置覆盖。
- **log.file_format** :系统级别日志输出到文件的格式。
- **log.console_format** : 系统级别日志输出到控制台的格式。
- **log.max_log_size** : 系统级别日志文件的最大大小。

- **log. check_size_interval** : 检查系统级别日志文件大小的时间间隔。                                                                                               
- **log.module** : 各个指定模块的日志级别。                                                                                                     
- **audit.backend** : 后端类型，目前有四种后端可选项，分别是"file", "filelog" ,"graylog"和"elk"。                                                            
- **audit.level** : 审计的级别，级别越高输出的内容越详细，从低到高的级别依次是：none, metadata, responsebody, storageobject，其中如果将审计级别配置为none，则代表不开启审计，平台的审计服务将不会启动。
- **audit.queue_size** : 审计事件队列的默认大小，用于缓存当前产生的审计事件，一般无需修改。                                                                            

- **audit.rewrite_interval** : 从临时filelog尝试恢复日志到webhook的时间间隔。
- **audit.update_conn_interval** : 检查连接状态事件间隔（如果使用webhook方式）,如果检查发现连接不可用，则会更新连接。
- **[audit.conn.pool]** : 里面包含的配置项均是在backend配置为"graylog"或者"elk"才生效，为连接池配置，连接池的作用是当一个连接失效后会自动将消息发送到配置的其他节点。

- **urls** : 外部的url, 可配置一个或者多个，根据实际使用的审计平台配置，格式为“IP+端口号”。
- **init_cap** : 连接池初始容量。
- **max_cap** : 连接池最大容量，最好不小于url的个数。
- **idle_timeout** : 连接空闲超时时间，连接空闲超过这个事件后将会被关闭。
- **dial_timeout** : 建立连接超时时间。
- **[audit.file]** : filelog配置项，详见filelog配置项说明。
- **http.tlsCertPriv** : rpc 客户端私钥文件路径。
- **http.max_content_length** : http请求大小上限。
- **http.read_timeout** : http请求超时时间。
- **p2p.transport** : 表示启用哪种协议通信的网络，目前仅支持grpc。                                                                                       
- **p2p.compress** : 是否对网络报文进行压缩。       
- **p2p.mode** : 网络类型，可选"direct","relay","discover"。
- **p2p.retrytime** : p2p 重连尝试时间间隔。 
- **p2p.ipc** : 系统产生ipc通信管道文件路径。
- **p2p.enableTLS** : 是否启用TLS。
- **p2p.tlsCA** : TLS CA证书路径。
- **p2p.tlsServerHostOverride** : TLS 服务域名。
- **p2p.tlsCert** : TLS 客户端证书路径。    
- **p2p.tlsCertPriv** : TLS 客户端私钥文件路径。                     
- **p2p.keepAliveDuration** : keep alive 时间间隔。 
- **p2p.keepAliveFailTimes** : keep alive 失败终止次数。
- **p2p.pendingDuration** : pending 重连时间间隔。
- **p2p.grpc.maxRecvMessageSize** : grpc接收消息大小上限。
- **p2p.grpc.maxSendMessageSize** : grpc发送消息大小上限。
- **flow.control.ratelimit.enable** : 是否启用大文件流量控制。
- **flow.control.ratelimit.fileReceivePeak** : 初始化时大文件的接收流量上限，默认为100MB。
- **flow.control.ratelimit.fileReceiveRate** : 接收大文件的流量速率，默认为1MB/8ms。
- **flow.control.ratelimit.fileSendPeak** : 初始化时大文件的发送流量上限，默认为100MB。
- **flow.control.ratelimit.fileSendRate** : 发送大文件的流量速率，默认为1MB/8ms。
- **flow.control.bandwidth.enable** : 是否开启带宽流控。
- **flow.control.bandwidth.outgoingBandwidth** : 带宽限制，默认限制为500Mb/s。

dynamic.toml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **self** : 本节点的hostname。
- **port** : 本节点服务的端口。
- **p2p.ip.remote** : 与本节点相连的所有节点的hostname以及ip和grpc端口。
- **p2p.ip.self.domain** : 本节点所在的域名。
- **p2p.ip.self.addrs**  : 本节点在其他域中使用的ip和端口。
- **namspace.name** : namespace名称。
- **namespace.start** : 是否启用该namespace。

ns_dynamic.toml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **consensus.algo** : 当前采用的共识算法。
- **consensus.set.set_size** : 节点一次性广播的交易数目。
- **consensus.pool.batch_size** : 主节点一次性打包的交易数量上限，即区块的交易数量上限。
- **consensus.pool.pool_size** : 交易内存池总大小。
- **self.n** : 与本节点相连的vp节点数目。
- **self.hostname** : 本节点的主机名，与dynamic.toml中的self对应。
- **self.new** : 本节点是否为新增节点，若节点类型为nvp或cvp，则该字段无效。
- **self.type** : 节点类型，可选nvp/vp/cvp。
- **nodes** : nodes的数目应当为n。
- **nodes.hostname** : 与本节点相连的vp节点的主机名。
- **nodes.score** : 与本节点相连的vp节点的网络状况。

ns_static.toml
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **genesis.alloc** : 用于新建创世区块，其中key为用户帐号地址，要求地址长度为20个16进制字符长度，value为用户所拥有的余额，要求为10进制值。
- **consensus.set.set_timeout** : 在ns_dynamic.toml中配置了set_size,实际上只要发生了超时，也会广播这批交易。                         
- **consensus.pool.check_interval** : 检查交易池中的交易是否长时间未被处理的时间间隔。
- **consensus.pool.tolerance_time** : 交易池中的交易等待处理的最长时间间隔，若超时，则会进行重发。
- **consensus.pool.batch_mem_limit** : 是否开启对主节点打包batch的大小限制。
- **consensus.pool.batch_max_mem** : 主节点打包batch的大小限制。
- **consensus.rbft.k** : rbft检查点间隔，以区块为单位。
- **consensus.rbft.vc_period** : rbft算法视图变更的周期。
- **consensus.rbft.timeout.sync_state** : 进行世界状态同步时，等待quorum回复的时间上限。
- **consensus.rbft.timeout.sync_interval** : 进行世界状态同步的频率。
- **consensus.rbft.timeout.recovery** : 等待recovery过程结束的最长等待时间。
- **consensus.rbft.timeout.first_request** : 节点启动后的最长准备时间。
- **consensus.rbft.timeout.batch** : 主节点打包的时间间隔。
- **consensus.rbft.timeout.request** : 节点接收、执行区块的最长时间，其值必须大于timeout.batch。
- **consensus.rbft.timeout.validate** : 节点进行validate最长的时间。
- **consensus.rbft.timeout.null_request** : 主节点发送给其它节点的心跳消息。
- **consensus.rbft.timeout.viewchange** : 进行viewchange的最长时间。
- **consensus.rbft.timeout.resend_viewchange** : 重新发送view change的间隔。
- **consensus.rbft.timeout.clean_viewchange** : view change消息的过期清理的时间间隔。
- **consensus.rbft.timeout.epoch_check** : 进行epoch check的时间间隔。
- **consensus.rbft.timeout.update** : 增删节点时，更新共识参数的最长时间。
- **encryption.TEE.TEEPath** : sgx路径。
- **encryption.TEE.DataEncrypt** : 是否开启区块数据的TEE加密。
- **encryption.root.ca** : Root CA目录。
- **encryption.ecert.ecert** : Enrollment Certcate，准入证书，用于控制VP和NVP节点进入Hyperchain中。
- **encryption.check.enable** : 是否开启RCert。
- **encryption.check.enableT** : 是否开启TCert。
- **distributedCA.enable** : 是否开启分布式CA。
- **encryption.security** : 会话密钥协商加密算法，可选pure,3des,aes or sm4。
- **radar-params.viewable_db_type** : radar数据库类型。
- **radar-params.mysql_connURL** : radar数据库URL。
- **log.dump_file** : 是否输出日志文件。
- **log.dump_interval** : 产生新的日志文件的时间间隔。
- **log.log_dir** : 输出日志文件路径。
- **log.log_level** : namespace级别的默认日志级别，可被module下的模块设置覆盖。
- **log.file_format** : 日志输出到文件的格式。
- **log.console_format** : 日志输出到控制台的格式。
- **log.max_log_size** : 最大的日志大小，达到该大小后自动进行日志切分。
- **log. check_size_interval** : 检查系统级别日志文件大小的时间间隔。
- **radar-params.viewable_db_type** : radar 数据库类型。
- **radar-params.mysql_connURL** : radar mysql url。
- **[log]** : 日志相关的配置同global.toml。
- **rpc.qps.flowCtrl.enable** : 是否开启rpc流控。
- **rpc.qps.flowCtrl.capacity** : 令牌桶容量。
- **rpc.qps.flowCtrl.limit** : qps限流值。
- **duplicate.tx_active_time** : 合法交易的时间，即在过去这段时间之内产生的交易才能通过验证。
- **duplicate.tx_drift_time** : 若交易的时间戳大于节点当前时间，那么超过的时间不能大于这个值。
- **duplicate.bloomfilter.bloombit** : 布隆过滤器大小。
- **duplicate.bloomfilter.max_mem** : 布隆过滤器的总内存占用上限。
- **mq.broker.type** : 启动的mq实现类型。
- **mq.rabbit.url** : rabbitmq地址。
- **mq.kafka.urls** : kafka集群地址。
- **mq.kafka.partitionNums** : 每个kafka实例下partition的数量。
- **mq.kafka.replicaFactor** : 每个kafka实例在zookeeper下的备份数量。
- **mq.kafka.writerBatchSize** : 平台一次推送请求到kafka最多包含的消息数量。
- **mq.kafka.writerBatchBytes** : 平台一次推送请求到kafka最多包含的消息大小。
- **mq.kafka.writerBatchTimeout** : 平台一次推送请求到kafka最多包含的最大等待时间。
- **mq.kafka.writeTimeout** : kafka writer超时时间。
- **mq.kafka.rebalanceInterval** : 刷新partion的间隔。
- **mq.kafka.idleConnTimeout** : 每个连接最大闲置时间。
- **executor.syncer.max_block_fetch** : 在区块同步过程中，一次性拉取的区块数目上限。
- **executor.archive.archive_root** : 归档数据所在目录。
- **executor.nvp.sync_mode** : nvp数据同步的方式，即拉取区块还是拉取journal。
- **executor.sync_chain.priority** : vp节点数据同步的优先级，即拉取尽可能多的区块还是尽可能少的区块，候选项为block和snapshot。
- **executor.sync_chain.sync_journal_receipt** : 是否同步journal数据与回执数据。
- **executor.sync_chain.hs_interval** : 握手是数据同步建立连接的过程，该字段控制了若握手失败，则下次继续握手的时间间隔。
- **executor.sync_chain.hs_resend_times** : 同上，该字段为握手重试的次数上限。
- **executor.sync_chain.batch_size** : 节点间的数据传输是分批次进行的，每个批次都包含了batch_size数量的固定大小（默认1MB）的数据包。
- **executor.sync_chain.state_sync_interval** : 若数据提供方在state_sync_interval时间内未收到数据接收方的任何响应，则终止本次数据发送。
- **executor.sync_chain.fetcher_resend** : 拉取数据失败后重试次数的上限。
- **executor.sync_chain.mode_negotiate_interval** : 协商数据同步模式的重试间隔。
- **executor.filemgr.enable** : 是否启用大文件存储功能，若启用，则须保证索引数据库同样启用。
- **executor.filemgr.deadline** : 临时文件的过期时间。
- **executor.filemgr.clean_interval** : 扫描临时目录的时间间隔。
- **executor.filemgr.file_system_mode** : 文件系统类型，ipfs或origin，后者表示本地文件系统。
- **executor.filemgr.data_path** : 数据目录。
- **executor.filemgr.ipfs_url** : ipfs服务的url，若采用ipfs作为文件系统。
- **executor.filemgr.hs_interval** : 同executor.sync_chain.hs_interval。
- **executor.filemgr.batch_size** : 同executor.sync_chain.batch_size。
- **executor.filemgr.transfer_interval** : 每一批次的数据包传输时间的上限，即若batch_size配置为100，本字段配置为30s，则需要保证在30s内完成100个包（即100M数据）的传输。
- **service.service_name** : 是否开启服务。
- **database.public_path** : 区块链数据库目录。
- **database.state** : 状态数据/账本数据库相关配置。
- **database.state.encrypt** : 是否开启账本加密，详细说明参见账本加密使用手册。
- **database.state.type** : 账本数据库，仅支持multicache。
- **database.state.multicache.persist_goroutine_num** : memdb异步刷入底层数据库时的并发度。
- **database.state.multicache.underlying_num** : 若采用leveldb作为底层数据库，则该值为leveldb的数量。
- **database.state.multicache.memory_limit** : memdb的最大内存占用量。
- **database.state.multicache.data_path** : 相对于database.public_path的数据目录。
- **database.state.multicache.persist_db.type** : 底层持久化数据库，目前可选leveldb,multidb,memdb(仅限测试环境)。
- **database.state.multicache.persist_db.multidb** : 若底层数据库配置为multidb，则下面的配置将会被启用。
- **database.state.multicache.persist_db.multidb.db_amount_limit** : 数据库数量上限。
- **database.state.multicache.persist_db.multidb.db_paths** : 每个数据库的路径，路径数量需要小于上限值。
- **database.state.multicache.persist_db.leveldb** : 若底层数据库采用leveldb，则下面的配置 将会被启用。
- **database.state.multicache.persist_db.leveldb.block_cache_capacity** : leveldb sstable中datablock的缓存大小。
- **database.state.multicache.persist_db.leveldb.block_size** : leveldb sstable中datablock的大小。
- **database.state.multicache.persist_db.leveldb.write_buffer** : leveldb的写缓存大小，超出该部分的写入将会触发dump或compaction。
- **database.state.multicache.persist_db.leveldb.write_l0_pause_trigger** : 若0层sstable文件超过该数量，则暂停写入操作等待后台compaction完成。
- **database.state.multicache.persist_db.leveldb.write_l0_slowdown_trigger** : 若0层sstable文件超过该数量，则降低写入操作等待后台compaction完成。
- **database.account** : 账户数据库的相关配置，同state数据库。
- **database.chain** : 链数据库的相关配置，同state数据库。
- **database.block** : 区块数据库相关配置。
- **database.block.encrypt** : 弃用，若想开启区块数据加密，请参考账本加密使用手册。
- **database.block.type** : 区块数据库类型，仅支持filelog。
- **database.block.filelog.path** : 数据目录。
- **database.block.filelog.compression** : 压缩算法，可选snappy/zlib/pure。
- **database.block.filelog.max_log_file_size** : 单个数据文件的大小上限，超出将会写入新的文件。
- **database.block.filelog.index_enable** : 是否开启数据索引。
- **database.block.filelog.cache.enable** : 是否开启缓存。
- **database.block.filelog.cache.max_cache_size** : 缓存大小。
- **database.block.filelog.cache_expired_time** : 缓存过期时间。
- **database.block.filelog.cache_entry_num** : 可缓存的区块数目。
- **database.block.filelog.handler_cache.handler_cache_num** : 缓存的文件句柄数。
- **database.block.filelog.handler_cache.handler_cache.evict_time** : 文件句柄淘汰时间，单位为秒。
- **database.journal** : journal数据库配置，同上。
- **database.receipt** : 回执数据库配置，同上。
- **database.invalidtx** : 非法交易数据库，默认使用leveldb，配置在上面已经介绍过。
- **database.consensus** : 共识数据库，默认使用leveldb，配置在上面已经介绍过。
- **database.camanager** : CAManager数据库，默认使用leveldb，配置在上面已经介绍过。
- **database.radar** : Radar数据库，默认使用leveldb，配置在上面已经介绍过。
- **database.mq** : MQ数据库，默认使用leveldb，配置在上面已经介绍过。
- **database.minifile.consensus** : 共识minifile路径。
- **database.minifile.sync** : 数据同步过程minifile路径。
- **database.minifile.bloom** : 布隆过滤器minifile路径。
- **database.minifile.nvp** : nvp minifile路径。
- **database.indexdb.layer1.enable** : 是否启用索引数据库。
- **database.indexdb.layer1.dbType** : 索引数据库类型，目前仅支持mongodb。
- **database.indexdb.tempdb.path** : 索引数据库临时数据库路径。
- **database.indexdb.layer2.active** : 需要开启二级索引的数据，1 - 区块写入时间2 - 交易的from字段3 - 交易的to字段4 - 交易哈希。
- **database.indexdb.mongodb** : mongodb服务的相关配。



.. |image0| image:: ../../images/Operations.png
.. |image1| image:: ../../images/image1.png
.. |image2| image:: ../../images/image2.png
.. |image3| image:: ../../images/image3.png
.. |image4| image:: ../../images/image4.png
.. |image5| image:: ../../images/image5.png
.. |image6| image:: ../../images/image6.png
.. |image7| image:: ../../images/image7.png
.. |image8| image:: ../../images/image8.png
.. |image9| image:: ../../images/image9.png
.. |image10| image:: ../../images/image10.png
.. |image11| image:: ../../images/image11.png
.. |image12| image:: ../../images/image12.png
.. |image13| image:: ../../images/image13.png
.. |image14| image:: ../../images/image14.png

