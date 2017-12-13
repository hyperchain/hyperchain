Transaction Flow
================

This document outlines the transactional mechanism from a transaction
initialized by client to final updated into the blockchain ledger. The
scenario includes the client that initiating the transaction and the
consensus peer A (also called validate peer, VP) directly connected to
it. The client interacts with the blockchain ledger by sending
transactions through the SDK (Java supported) with the VP-A; the VP-A
fully connected with other VP B, C and D... , while VP-A has two backup
nodes a1 and a2 (also called non-validate peer, NVP) . |image0|

**Assumption**

This flow assumes that a channel is set up and running. The application
user has registered and enrolled with the organization’s certificate
authority (CA) and received back necessary cryptographic material
(SDKCert) , which is used to authenticate to the blockchain network.

The smart contract (including the initial state and all the related
functions) has been deployed on the blockchain peers in the working
namespace.

**Client initiates a transaction**

What's happening? - Client is sending a transaction request (calling one
of the methods in the smart contract). The request targets VP-A, through
which it goes into blockchain network.

The client initializes a *HyperchainAPI* object by calling the interface
of the SDK. During initialization, the SDK requests the VP-A with the
SDKCert and the public key to obtain the TCert needed for initiating the
transaction. After that, client generates a transaction by calling the
*Transaction* interface of the SDK. The SDK firstly signs the
transaction with the private key specified by the client, then signs the
message with the private key corresponding to the TCert after the
transaction is encapsulated in the JSONRPC protocol. *HTTP/HTTPS*
"short" connection and *WebSocket* "long" connection is supported during
the SDK and representative peer. |image1|

**Peer accepts transaction & sends to the blockchain network**

VP-A performs TCert authentication as soon as it receives the
transaction. The peer only processes the request that passed the
validation of TCert. Then the API module of the peer will do the
following transaction verification:

(1) The transaction throughput has not exceed the configuration of rate
    limit;
(2) The transaction proposal is well formed (Verify the legality of the
    transaction field, like the legitimacy of the timestamp);
(3) It has not been submitted already in the past (replay-attack
    protection);
(4) The signature of transaction is valid (ECDH & SM2 supported).

After passing all the aforementioned verification, the transaction will
be submitted to the consensus module, and the consensus module will
broadcast it to all VPs in the entire network.

**Consensus of transaction: ordering, validating, writing**

The transaction goes through the three-phase protocol the Consensus
Algorithm (RBFT):

(1) *Pre-Prepare* The primary peer (master peer among all VPs) orders
    tansactions chronologically by channel, and creates block of
    transactions in a certain period of time (or a certain number). And
    then, primary broadcasts the block to all VPs. |image2|
(2) *Prepare* Replicas (another name of all VPs used in consensus) make
    confirmation and pre-execute the transactions in the block. Then
    they broadcast the result hash.
(3) *Commit* Replicas write the block, update in the blockchain ledger.
    |image3|

All the illegal transactions found after pre-executing will be recorded
into the database of illegal transactions instead of blockchain ledger.
The blockchain ledger is a ledger of blocks including all valid
transactions.

Furthermore, all VPs will push the block to all the respectively
connected NVPs after the block is successfully updated in blockchain
ledger.

**Transaction Receipt** SDK implements timely query in the result of
deploying transaction in the *Transaction* interface, that is,
transaction receipt. The configuration of the number of transactions and
packaging time (used in ordering) set in Blockchain network will affect
the latency of transaction.

.. |image0| image:: ../../images/tx_flow.png
.. |image1| image:: ../../images/get_tcert.png
.. |image2| image:: ../../images/txs_to_block.png
.. |image3| image:: ../../images/block_to_ledger.png
