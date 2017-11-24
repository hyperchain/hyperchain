## Quick Start

If you haven't completed your [Preflight](./preflight.md) Checklist, do that first. This Quick Start tells you how to build Hyperchain from source code, and how to start up a Hyperchain cluster.

### Building from Source
**Create Your Clone**
Clone the repository to a directory of your `GOPATH` source path:
```bash
mkdir -p $GOPATH/src/github.com/hyperchain
cd $GOPATH/src/github.com/hyperchain
git clone https://github.com/hyperchain/hyperchain
```

**Building**
Please make sure you've installed Go tool properly, if you don't have it already, please see [Instructions](./Preflight.md).

To build Hyperchain:
```bash
cd $GOPATH/src/github.com/hyperchain/hyperchain
govendoer build
```
You can run `go build` as well.

### Start up Hyperchain
Since a Hyperchain cluster needs at least 4 nodes to establish a BFT system, we recommend starting up Hyperchain nodes in these modes:
- Local Mode - Local 4 Nodes
- Distributed Mode - Distributed 4 Nodes

#### Local Mode - Local 4 Nodes
We've provided a script named `local.sh` which starts all Hyperchain nodes locally.
```bash
cd $GOPATH/src/github.com/hyperchain/hyperchain/scripts
./local.sh
```

You'll see these information if all Hyperchain nodes start up properly.
```bash
$./local.sh
...
...
start up node 1 ... done
start up node 2 ... done
start up node 3 ... done
start up node 4 ... done
```

#### Distributed Mode - Distribute 4 Nodes
**Enable Password Less**
Since `server.sh` script prompts for a password when executing ssh operations, we recommend generating SSH keys on the deploy node and distribute the public key to each Hyperchain node.

1 . Generate the SSH keys, and leave the passphrase empty:
```bash
ssh-keygen

Generating public/private key pair.
Enter file in which to save the key (/home/hyperchain/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/hyperchain/.ssh/id_rsa.
Your public key has been saved in /home/hyperchain/.ssh/id_rsa.pub.
```

2 . Copy the key to each Hyperchain node, replacing `{username}` with the user name you created.

```bash
ssh-copy-id {username}@node1
ssh-copy-id {username}@node2
ssh-copy-id {username}@node3
ssh-copy-id {username}@node4
```

**Distribute Hyperchain**
We've provided a script named `server.sh` which distributes Hyperchain to all nodes and starts up them separately.

```bash
cd $GOPATH/src/github.com/hyperchain/hyperchain/scripts
./server.sh
```

You'll see these information if all Hyperchain nodes start up properly.
```bash
$./server.sh
...
...
start up node 1 ... done
start up node 2 ... done
start up node 3 ... done
start up node 4 ... done
```
