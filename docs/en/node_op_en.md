# Node Operation

## 1. Add Node

Adding nodes requires the configuration of the following three files:

- `peerconfig.toml`: Peer profile, used to configure connection information of logical peer.
- `hosts.toml`: Host profile, used to configure information of physical peer, including the mapping between the hostname of node and the IP address.
- `addr.toml`: Address profile, used to configure physical interconnection address information, including the mapping of domain name and IP address. This file configuration is required when the peer connects in reverse to local node.

#### Example1: On the basis of four VP nodes, add the fifth VP node

The new VP node needs to be configured with the following configuration files before it starts:

- **`peerconfig.toml`**

```toml

[[nodes]]
  hostname = "node1"
  id = 1
  static = true

[[nodes]]
  hostname = "node2"
  id = 2
  static = true

[[nodes]]
  hostname = "node3"
  id = 3
  static = true

[[nodes]]
  hostname = "node4"
  id = 4
  static = true

[[nodes]]
  hostname = "node5"
  id = 5
  static = true

[self]
  caconf = "config/namespace.toml"
  hostname = "node5"
  id = 5		# The node ID	
  n = 5		    # The number of VP to connect to
  new = true	# Whether this is a new node to attend or not
  org = false	# Whether this is a original node or not
  rec = false	# Whether this node needs to be reconnected
  vp = true	    # Whether this is a primary node or not
```

- **`hosts.toml`**

This file needs to configure **the physical addresses of all the nodes to be connected**. The hyperchain nodes communicate with each other through the hostname, so the hostname can be configured as an arbitrary node address.

```toml
hosts = [
"node1 127.0.0.1:50011",
"node2 127.0.0.1:50012",
"node3 127.0.0.1:50013",
"node4 127.0.0.1:50014",
"node5 127.0.0.1:50015"
]

```

- **`addr.toml`**

addr is declared in the form of a domain. For example, if two nodes with hostname of `node1` and `node5` belong to `domain1` and all other nodes belong to different domains, the configuration file should be configured as follows (this configuration file is `addr.toml` of `node5`):

```toml
addrs = [
"domain1 127.0.0.1:50015",
"domain2 192.168.100.20:50015",
"domain3 202.110.20.13:50015",
"domain4 127.0.0.1:50015",
]
domain = "domain1"
```

.. Note:: To reiterate, `addr.toml` is a configuration to let other peer know the node domain and nodes in different domains are interconnected by different network address, which allows nodes to connect between complex network segments.

#### Example2: On the basis of four VP nodes, add a NVP node
The new NVP node needs to be configured with the following configuration files before it starts:

- **`peerconfig.toml`**

```toml
[[nodes]]
  hostname = "node1"
  id = 1
  static = true

[[nodes]]
  hostname = "node2"
  id = 2
  static = true

[[nodes]]
  hostname = "node3"
  id = 3
  static = true

[[nodes]]
  hostname = "node4"
  id = 4
  static = true

[self]
  caconf = "config/namespace.toml"
  hostname = "node5"
  id = 0         # This value must be 0 for NVP node 
  n = 4		    # The number of VP to connect to
  new = true	# Whether this is a new node to attend or not
  org = false	# Whether this is a original node or not
  rec = false	# Whether this node needs to be reconnected
  vp = false	# Whether this is a primary node or not
```

- **`hosts.toml`**

```toml
hosts = [
"node1 127.0.0.1:50011",
"node2 127.0.0.1:50012",
"node3 127.0.0.1:50013",
"node4 127.0.0.1:50014",
"node5 127.0.0.1:50015"
]

```

- **`addr.toml`**

```toml
addrs = [
"domain1 127.0.0.1:50015",
"domain2 127.0.0.1:50015",
"domain3 127.0.0.1:50015",
"domain4 127.0.0.1:50015",
"domain5 127.0.0.1:50015"
]
domain = "domain5"
```



## 2. Delete Node

We divide deleting nodes into three cases:

1. VP disconnects from one VP;
2. VP proactively disconnects from one NVP;
3. NVP proactively disconnects from one VP;

The second and third results are the same, so that the NVP can no longer synchronize the VP data and the NVP no longer forwards the transaction to the VP.

In the following example, we assume that each node's JSON-RPC API service port mapping is as follows:

- Node 1：`8081`
- Node 2：`8082`
- Node 3：`8083`
- Node 4：`8084`
- Node 5：`8085`

#### Example1: VP disconnects from one VP
For example, there are currently five VP nodes and now delete VP node 5.

Firstly, get the hash of the VP node 5 to be deleted.

```bash
# Request
curl -X POST -d '{"jsonrpc":"2.0","method":"node_getNodeHash","params":[],"id":1, "namespace":"global"}' localhost:8085

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "55d3c05f2c24c232a47a1f1963ace172b21d3a2ec0ac83ea075da2d2427603bc"
}
```

Then, the request of deleting node is sent to VP node 1, 2, 3, 4 and 5 respectively.

```bash
# Request
curl -X POST -d '{"jsonrpc":"2.0","method":"node_deleteVP","params":[{"nodehash":"55d3c05f2c24c232a47a1f1963ace172b21d3a2ec0ac83ea075da2d2427603bc"}],"id":1, "namespace":"global"}' localhost:8081/8082/8083/8084/8085

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "successful request to delete vp node, hash 55d3c05f2c24c232a47a1f1963ace172b21d3a2ec0ac83ea075da2d2427603bc"
}

```

When you see the following logs on the terminal, the VP node is deleted successfully.

```bash
global::p2p 12:33:17.709 DELETE NODE 55d3c05f2c24c232a47a1f1963ace172b21d3a2ec0ac83ea075da2d2427603bc 
global::p2p 12:33:17.709 delete validate peer 5
```

#### Example2: VP disconnects from a NVP

For example, there are currently four VP nodes and one NVP node which is connected to VP node 1.

Firstly, get the hash of the NVP node.

```bash
# Request
curl -X POST -d '{"jsonrpc":"2.0","method":"node_getNodeHash","params":[],"id":1, "namespace":"global"}' localhost:8085

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad"
}
```

Then, send a request of deleting NVP to VP node 1.

```bash
# Request
curl -X POST -d '{"jsonrpc":"2.0","method":"node_deleteNVP","params":[{"nodehash":"4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad"}],"id":1, "namespace":"global"}' localhost:8081

# Response
{
  "jsonrpc": "2.0",
  "namespace": "global",
  "id": 1,
  "code": 0,
  "message": "SUCCESS",
  "result": "successful request to delete nvp node, hash 4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad"
}
```

The following logs are seen at the terminal of the VP node 1,

```bash
global::p2p 13:28:02.857 delete NVP peer, hash 4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad, vp pool size(4) nvp pool size(0) 
```

At the same time, the NVP node also prints the following log notes that the VP node 1 has been disconnected.

```bash
global::p2p 13:28:02.858 peers_pool.go:244 delete validate peer 1 
```

 The NVP node is deleted successfully.

#### Example3: NVP disconnects from a VP

The situation is similar to that of Example2, except that we send a request for deleting a node to the NVP node this time.

For example, there are currently four VP nodes and one NVP node which is connected to VP node 1.

Firstly, get the hash of the VP node 1.

```bash
# Request
curl -X POST -d '{"jsonrpc":"2.0","method":"node_getNodeHash","params":[],"id":1, "namespace":"global"}' localhost:8081

# Response
{
    "jsonrpc": "2.0",
    "namespace": "global",
    "id": 1,
    "code": 0,
    "message": "SUCCESS",
    "result": "fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a"
}
```

Then, send a request of deleting VP to NVP node.

```bash
# Request
curl -X POST -d '{"jsonrpc":"2.0","method":"node_deleteVP","params":[{"nodehash":"fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a"}],"id":1, "namespace":"global"}' localhost:8085

# Response
{
    "jsonrpc": "2.0",
    "namespace": "global",
    "id": 1,
    "code": 0,
    "message": "SUCCESS",
    "result": "successful request to delete vp node, hash fa34664ec14727c34943045bcaba9ef05d2c48e06d294c15effc900a5b4b663a"
}
```

The following logs are seen at the terminal of the NVP node,

```bash
global::p2p 13:47:17.744 delete validate peer 1 
```

At the same time, the VP node 1 also prints the following log,

```bash
global::p2p 13:47:17.744 delete NVP peer, hash 4886947d8191b62a1141dbc3250a0cc61a436ca28829f40cb5a690c7449825ad, vp pool size(4) nvp pool size(0)
```

 The VP node is deleted successfully.
