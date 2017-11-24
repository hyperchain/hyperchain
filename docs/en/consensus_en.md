# consensus

## 1. Overview

Consensus mechanism is the foundation of blockchain consistency, it ensures that all consensus nodes (or say validating peer, VP) execute transactions in the same order and then write into exactly the same ledgers. Accounting nodes(or say non-validating peer, NVP) which connect to one or more VP(s) can only synchronize ledger information from its connected VP(s), so NVP don't participate in consensus while NVP can forward transactions to VP(s) received from client.

Hyperchain supports the pluggable consensus mechanism and provides different consensus algorithms for different scenarios of the blockchain. Current version has implemented the improved algorithm of [PBFT][1]: Robust Byzantine Fault Tolerance (RBFT), the idea of this algorithm comes from many classic papers (especially [Aardvark][2]). Hyperchain will continue to support other consensus algorithms such as RAFT later.

After received transcations from clients, the API layer parses out the transactions and forwards to the consensus module. Consensus module receives and stores the transactions into local transaction pool (TxPool). TxPool takes the role of caching transactions and packaging blocks so it is implemented as a sub-module of consensus module. In addition, consensus module needs to maintain a consensus database to store some variables required by the algorithm for autonomous recovery after the system is crashed. For example, the RBFT algorithm needs to maintain consensus information such as View, PrePrepare, Prepare, and Commit.

![](../../images/consensus.png)



## 2. RBFT related parameters

In a consensus network of N nodes (N> = 4), RBFT can tolerate at most f Byzantine faults which:

​                                                                          $$f=\lfloor \frac{N-1}{3} \rfloor$$

The number of nodes that can guarantee consensus is:

​                                                                  $$quorum=\lceil \frac{N+f+1}{2} \rceil$$



## 3. RBFT normal case

The normal case of RBFT ensures that each consensus node in blockchain processes the transactions from the client in the same order. RBFT requires at least 3f + 1 nodes to tolerate f Byzantine falult which is the same as PBFT. The figure below is the consensus flow under the minimum number of cluster nodes, where N = 4 and f = 1. Primary1 is the master node which is dynamically elected by the consensus node, and is responsible for sorting and packing the transactions sent from the client. Replica2, 3 and 4 are backup nodes. All Replica nodes execute the transaction with the same logic and are able to participate in the election of new primary node when the primary node fails.

### Process of normal case

The consensus of RBFT retains PBFT's original three-phase submission flow (PrePrepare, Prepare, Commit) and inserts important transaction validation session which not only guarantees the consensus on the transaction execution sequence but also gurantees the consensus on block validation results.

![](../../images/normal.png)

RBFT inserts important transaction validation session into native PBFT normal case operations. Primary will validate block immidiately after packing the transactions, and then include the validation result into the PrePrepare message for the whole network broadcast, so PrePrepare message contains both the ordered Transaction information and block validation result. After receiving a PrePrepare message from primary, the backup nodes check the legitimacy of the message. After passing the legality check, backup node broadcasts Prepare message which indicates that the backup node agrees with primary's sorting result. The backup node starts to validate the batch after receiving (quorum-1) Prepare messages and compares the validation result with the validation result of primary's. If consistent, backup broadcasts Commit message which indicates that the backup node agrees with the validation result of primary's, otherwise, directly triggers ViewChange which indicates that the current node discovers primary's abnormal behavior. RBFT normal case operation is divided into the following steps:

1. **Transaction forward:** Client sen transactions to any node (consensus nodes or accounting nodes) in the blockchain. Accounting nodes need to forward the transactions received from clients to its connected consensus nodes and the consensus nodes broadcast transactions received from clients or accounting nodes to all other consensus nodes, so the transaction pool for all consensus nodes maintains a complete list of transactions;
2. **PrePrepare**: Primary packs transactions according to the policies below: User can customize the batch timeout and the packed batch size according to demand. Primary triggers the package event when it collects more than batchsize transactions during the batch timeout or it dosen't collect batchsize transactions when batch timeout happens. Primary packs the transactions into blocks according to the received chronological order, then validates and computes the execution result, and finally writes the ordered transaction information together with the validation result into the PrePrepare message to be broadcast to all consensus nodes which starts the three-phase processing flow;
3. **Prepare:** After receiving the PrePrepare message from the primary, backup node first checks the legitimacy of the message(such as current view and block number information). If the check passed, backup nodes broadcast Prepare message to all consensus node;
4. **Commit:** After receiving the (quorum-1) Prepare message and the corresponding PrePrepare message, the backup node validates the batch and compares the validation result with the validation result of primary which is written in the PrePrepare message. If consistent, the backup node broadcasts Commit message which indicates that backup node agrees with the validation result of primary, otherwise, directly triggers the ViewChange event which indicates that the current node discovers primary's abnormal behavior;
5. **Write Block:** All consensus nodes write the execution result to the local ledger after receiving quorum Commit messages.

By adding a validation mechanism in the consensus module, Hyperchain ensures that every backup node participates in checking all primay's ordering results, so backup can discovery primary's Byzantine behavior as soon as possible which improves the stability of the system.

### Checkpoint

Consensus nodes need to periodically clean up some useless message caches in order to prevent unlimit message caching during operation. RBFT collectes garbage by introducing checkpoint mechanism in the PBFT algorithm and fixedly set the checkpoint size K to 10. The node reaches a checkpoint after writing into an integer multiple of K and broadcasts the checkpoint information. After receiving the same checkpoint information from other quorum-1 nodes, replica reaches a stable checkpoint, then replica can clean up some of the message cache whose message number is less than checkpoint index.

### Transaction pool(txpool)

Transaction pool(txpool) is the transaction cache place of consensus node. The existence of txpool on the one hand limits the client's sending frequence, on the other hand reduces the bandwidth pressure of primary. Firstly, by limiting the size of the transaction pool, consensus node can refuse transactions from clients after the transaction pool reaches its limit size, so users can maximize utilization without abnormalities by setting the transaction cache size for a reasonable assessment of machine performance. Secondly, the consensus node stores the transactions received from the client into its own transaction pool and then broadcasts the transactions to other consensus nodes to ensure that all consensus nodes maintain a complete transaction list. After primary packed transactions, it only needs to put the transaction hash list into the PrePrepare message for broadcasting instead of put the complete transaction list into PrePrepare for broadcasting, which greatly reduces the pressure of the egress bandwidth of primary. If the backup node finds that some transactions are missing before validation, it needs only fetch the missing entries from primary rather than fetching all the transactions in the block.



## 4. RBFT ViewChange

The ViewChange mechanism of RBFT solves the problem that the primary node may become a Byzantine node. In the RBFT algorithm, nodes participating in consensus can be divided into Primary node and Replica nodes according to roles. The most important function of the Primary node is to package the received transactions according to a specific strategy, order the transactions, and have all the nodes execute in this order. However, if the Primary node crashes, goes wrong, or is hacked (that is, it becomes a Byzantine node), the Replica nodes need to discover the abnormality of the Primary node in time and elect a new Primary node. This is a problem that all BFT algorithms must solve in order to achieve stability.

### view

In RBFT, the concept of view has been introduced as same as PBFT. The view is changed each time a new Primary node is elected. At present, RBFT chooses the Primary node by rotation, and the view increases monotonically from zero. The current view and the total number of nodes N determines the Primary node id:

​                                                                 $$PrimaryId = (view + 1) \bmod N$$

### Byzantine behavior that can be detected

Currently, there are mainly two types of Primary‘s Byzantine behavior that RBFT can detect：

1. The Primary node stops working and sending no message；
2. The Primary node sends wrong messages.

For scenario 1, being detected could be guaranteed by the nullRequest mechanism. A properly behaved Primary node will send nullRequest messages to all Replica nodes periodically to maintain normal connection when no transaction occurs. If the Replica node does not receive a nullRequest messages within the specified time, the ViewChange process is triggered to elect a new Primary node.

For scenario 2, Replica nodes would check the messages sent from the Primary node, such as the verification result contained in the PrePrepare message, which is mentioned in the previous section. The Replica node will directly initiate the ViewChange process to elect a new Primary node if the messages fail to pass the verification.

In addition, RBFT provides a configurable option called ViewChangePeriod. Users can set this option according to their needs. Each time a certain number of blocks are written, the network would take a proactive ViewChange process to rotate the Primary node. This can alleviate the additional pressure on the primary node as a package node. And secondly, all the nodes participating in the consensus can take some packaging work to ensure fairness.

### Process of ViewChange

![](../../images/viewchange.png)

In the above figure, Primary 1 is a Byzantine node and the network need to take ViewChange process. The ViewChange process in RBFT is as follows

1. Replica nodes broadcast a ViewChange message to the entire network after detecting an abnormal behavior of Primary node (without receiving a nullRequest message on time) or after receiving a ViewChange message from other f + 1 nodes, and change their view from v to v + 1;
2. In the new view, after receiving N-f ViewChange messages, Primary node calculates the checkpoint where Primary node would start executing from in the new view and the transactions to be processed next, according to the received ViewChange message, then encapsulates them into the NewView message and broadcasts the message. Finally Primary node initiates the VcReset;
3. After receiving the NewView message, Replica nodes validate the message. If the message passes validation, Replica nodes initiate the VcReset. If it does not pass validation, Replica nodes send ViewChange message to start another round of ViewChange;
4. After finishing VcReset, all nodes broadcast FinishVcReset to the whole network;
5. After each node receives N-f FinishVcReset messages, it starts to process the transactions after the determined checkpoint and finishes the entire ViewChange process.

Because the communication between the consensus module and the execution module is asynchronous and execution module may have some useless validation cache after ViewChange, the consensus module needs to inform the execution module to clear this useless cache before the end of ViewChange. The RBFT proactively notifies the execution module through the VcReset event to clear the cache. The node can finish ViewChange only after clearing the cache.



## 5. RBFT Recovery

During the operation of the blockchain network, the execution speed of some nodes may lag behind that of most nodes due to network jittering, sudden power failure, disk failure and the like. In this scenario, these nodes need to be able to recover automatically to continue participating in subsequent consensus processes. In order to solve this kind of data recovery problem, the RBFT algorithm provides a mechanism for automatic recovery of dynamic data (recovery). Node updates its storage status by actively retrieving information such as the view of all nodes in the existing consensus network and the latest block information, and finally synchronizes to the latest status of the entire system. When the node is going to start up, restart or the node falls behind, it will automatically enter recovery and synchronize to the latest state of the entire system.

### Process of Recovery

![](../../images/recovery.png)

In the above figure, replica 4 is a backward node and needs to be recovered. This node's automatic recovery process in RBFT is as follows:

1. At the beginning, replica 4 broadcasts the NegotiateView message to retrieve the current view of the active nodes;
2. The remaining three nodes send NegotiateViewResponse back to replica 4, returning the current view;
3. Replica 4 updates its own view after it receives quorum NegotiateViewResponse messages;
4. Replica 4 broadcasts the RecoveryInit message to the remaining nodes to notify them that replica 4 needs to be recovered, and requests the checkpoint information and the latest block information of the remaining nodes;
5. After receiving the RecoveryInit message, the active node sends a RecoveryResponse to return its own checkpoint information and latest block information to the Replica 4;
6. After Replica 4 receives quorum RecoveryResponse messages, it tries to find the highest checkpoint among these responses, and then updates its status to this checkpoint point;
7. Replica 4 requests PQC data after the checkpoint from the active node , and finally synchronize to the latest status of the entire network



## 6. RBFT Node Management

In consortium blockchain, the dynamic addition and deletion of members is required due to the expansion of the consortium or the withdrawal of some members, but the traditional PBFT algorithm does not support it. To make it easier to control addition and deletion of members, RBFT adds the function to dynamically add and remove nodes without shutting down the cluster.

### Process of Adding nodes

![](../../images/node_management.png)

In the above figure,  replica 5 is the node to be added. The process of dynamically adding this node is as follows:

1. Newly added node replica 5 initiates connections to all existing nodes by reading the configuration file information. After confirming that all nodes are connected successfully, replica 5 updates its own routing table and initiates recovery;
2. After receiving a connection request from replica 5, the existing node(including node 1, node 2, node 3 and node 4) confirms that replica 5 is allowed to join, and then broadcasts an AddNode message to the entire network, indicating that it agrees replica 5 to join the consensus network;
3. When an existing node receives N AddNode messages (N is the total number of nodes in the current blockchain consensus network), it updates its own routing table and then starts to respond to the replica 5's consensus message request (before this, All the consensus message from replica 5 would not be processed);
4. After Replica 5 finishs recovery, it broadcasts ReadyForN requests to existing nodes across the network;
5. After receiving the ReadyForN request, the existing node recalculates N and view after replica 5 joins, and then encapsulates the PQC message into AgreeUpdateN message and broadcast it to the whole network;
6. There would be a new primary node after Replica 5 joins, and now it's still node 1. After receiving N-f AgreeUpdateN messages, node 1 sends the UpdateN message as the new primary node;
7. All nodes in the network check the correctness of the UpdateN message after receiving it, and proceed to VCReset if there is no problem;
8. After completing VCReset, each node broadcasts FinishUpdate message to the whole network;
9. After receiving N-f FinishUpdate messages, all nodes process the subsequent requests and complete the adding node process.



[1]: http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.84.6725&amp;amp;rep=rep1&amp;amp;type=pdf
[2]: https://www.usenix.org/legacy/event/nsdi09/tech/full_papers/clement/clement.pdf



