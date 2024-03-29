.. _NoxBFT-User-Manual:

NoxBFT使用手册
^^^^^^^^^^^^^^^^^^

1. 引言
============

1.1 编写目的
--------------

此文档描述Hyperchain中如何使用NoxBFT共识算法，使软件开发人员能清楚地了解NoxBFT共识，便于Hyperchain的开发工作。

1.2术语
-------------

+---------+---------+---------------------------------------------------+
| 英文    | 中文    | 含义                                              |
+=========+=========+===================================================+
| NoxBFT  | /       | Hyperchain新型拜占庭共识算法                      |
+---------+---------+---------------------------------------------------+
| h       | /       | NoxBFT共识算法的底层共识协议                      |
| otstuff |         |                                                   |
+---------+---------+---------------------------------------------------+
| mempool | 交易    | NoxBFT共识算法的交易池                            |
|         | 内存池  |                                                   |
+---------+---------+---------------------------------------------------+
| che     | 检查点  | Nox                                               |
| ckpoint |         | BFT共识算法的检查点协议，用于验证执行结果的正确性 |
+---------+---------+---------------------------------------------------+

2. 配置说明
=============

在 `configuration/<分区名>/ns_dynamic.toml` 配置文件中可以选择启用的算法类型为 **NoxBFT** ，在 `configuration/<分区名>/ns_static.toml` 配置文件中可以配置NoxBFT算法相关的设置参数

2.1 ns_dynamic.toml
-----------------------

通过更改 `consensus.algo` 配置项使用NoxBFT共识算法

 ::

 [consensus]
 algo = "NoxBFT" # Support RBFT or NoxBFT

注意：若非首次部署，请查看《线下共识切使用换手册》进行NoxBFT共识算法配置

2.2 ns_static.toml
-----------------------

2.2.1 hotstuff配置项
>>>>>>>>>>>>>>>>>>>>>>>

 ::

	[consensus.hotstuff]         # hotstuff configurations
	batch_time                    = "0.1s"  # Primary generate a proposal if there are pending requests, although batchsize isn't reached yet.
	max_block_size                = 500     # max number of one block generated by leader.
	proposer_type                 = "rotating_proposer" # proposer switching type, now support:
	                                                    # "fixed_proposer"
	                                                    # "rotating_proposer"
	contiguous_rounds             = 1 # how many rounds can one proposer continue leading in rotating proposer strategy.
	checkpoint_cycle              = 10 # how many round to trigger a new checkpoint cycle.
	checkpoint_interval           = 100 # how many rounds allowed that do not generate checkpoints
	checkpoint_mem_cache_size     = 2 # the maximum checkpoints with blocks can be cached in memory
	checkpoint_share_type         = "cross" # the checkpoint sharing type. (default: cross)
	                                        # nox (share checkpoints by NoxBFT messages)
	                                        # cross (share checkpoints by sending specific messages and broadcast it to all validators)
	pacemaker_initialTimeout      = "0.5s" # the base timeout value of Pacemaker.
	exponent_base                 = 2 # how much we increase interval every time
	max_exponent                  = 3 # maximum time interval which won't exceed max_exponent
	enforce_increasing_timestamps = false # keep the timestamp of the block is increasing.
	backward_round_limit          = 10 # how many blocks can be between ordered and checkpoint
	                                   # decide by checkpoint_mem_cache_size and checkpoint_cycle

- batch_time：区块打包等待时间，当从mempool中获取交易数目小于 `max_block_size` 时最多允许等待的时间。（默认值为 *0.1s* ）

- max_block_size：区块打包交易数量，每个区块允许的最大交易数量。（默认值为 *500* ）

- proposer_type：主节点选举模式，目前支持两种类型（默认值为 *rotating_proposer* )

  - fixed：固定主节点模式（仅有于测试，不建议使用）

  - rotating_proposer：轮换主节点

- contiguous_rounds：当 `proposer_type` 为 *rotating_proposer* 时，每个主节点连续当选的轮数。（默认值为 *1* ）

- checkpoint_cycle：检查点生成周期，每提交执行多少个区块会生成一次检查点。（默认值为 *10* ）

- checkpoint_interval：检查点生成间隔轮数，超过此间隔共识轮数未生成检查点时，会强制触发检查点的生成。（默认值为 *100* ）

- checkpoint_mem_cache_size：当检查点协议触发旧共识数据清理时，允许缓存多少个检查点中的区块（用于帮助落后节点同步，默认值为 *2* ）

- checkpoint_share_type：检查点分享模式，目前支持两种类型。（默认值为 *cross* ）

  - cross：采用节点间互相广播的方式分享检查点，建议在共识集群 **节点数量较少** 时使用

  - nox：采用主节点搜集的方式分享检查点，建议在共识集群 **节点数量较多** 时使用

- pacemaker_initialTimeout：每轮共识初始超时时间。（默认值为 *0.5s* ）

- exponent_base：连续超时时的增长指数的基数（超时时间 = 初始超时时间 * 基数 ^ 幂数，默认值为 *2* ，即每次连续超时，超时时间为之前的两倍 ）

- max_exponent：连续超时时间增长指数的最大幂数 （最大超时时间 = 初始超时时间 * 基数 * 最大幂数，默认值为 *3* ，即最大超时时间为4秒）

- enforce_increasing_timestamps：是否强制每轮共识提案具有递增的时间戳。（默认值为 *false* ）

- backward_round_limit：允许落后节点最大落后的轮数。（默认值为 *10* ）

2.2.2 mempool配置项目
>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

        [consensus.hotstuff.mempool]
        type               = "fifo" # memory pool mode: solo(no shared),
                                    # fifo(shared with first in first out sorting)
                                    # priced(shared with tx price sorting)
        pool_size          = 50000 # How many txs could the mempool stores in total
        commit_cache_size  = 3 # the maximum committed blocks with their transactions can be cached in memory


- type：交易内存池的类型，目前允许三种类型（默认值*fifo*)

  - solo：不与其他节点分享交易（仅用于测试，不推荐使用）

  - fifo：交易按先入先出顺序打包，支持节点间通过网络分享交易

  - priced：交易按交易GasPrice从高到低排序打包，支持节点间通过网络分享交易

- pool_size：交易池中最大允许的交易数量，`solo` 与 `fifo` 类型超过此数量时会拒绝新加入的交易； `priced` 类型超过此数量时，会剔除价格较低的交易并接收价格较高的新交易。（默认值为 *50000* )

- commit_cache_size：上链交易的缓存，允许缓存多少个区块中的交易。（默认值为 *3* ）



3.注意事项
==============

  需要注意集群中共识节点采用相同的配置，否则可能因为配置差异而成为拜占庭节点。