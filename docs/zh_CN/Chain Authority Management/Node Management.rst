.. _node-manage:

节点增删管理
^^^^^^^^^^^^^

节点管理
------------------

节点管理是基于 :ref:`链级角色管理 <chain-role-manage>` 及 :ref:`链级配置管理 <chain-conf-manage>`，链级管理员通过提案投票的方式进行节点管理。

节点管理包括增加节点和删除节点。增加节点分为组网和加入共识两步。删除节点时直接从共识删除节点即可。共识部分删除节点之后，会向eventhub发送一个删除节点的事件， `eventhub` 会处理这个事件，调用 `p2p` 断开与此节点的逻辑连接。

节点的操作分为以下几种：

1. AddNode，增加建立逻辑连接的节点，即将节点加到hosts中（此时没有加入共识）；

2. AddVP，增加VP节点，即将节点加入共识；

3. RemoveVP，删除共识VP节点，同时断开此节点与对应namespace中其他节点建立的逻辑连接，如果节点没有加入其他namespace，则将节点停掉。

增加节点
>>>>>>>>>>>>>>>>>>>>>>>>>

依赖分布式ca增加节点
::::::::::::::::::::::::

依赖分布式ca增加节点组网和加入共识这两个流程是分开进行的，分别使用两个提案，提案投票通过后在执行的时候分别完成组网交换证书和加入共识的操作。

1. 组网

新节点要加入某个已有的区块链网络，需要先与网络中的各个节点建立逻辑连接，这一步称之为组网。

- 创建提案（自动发交易）

新节点启动后与网络中已有节点建立物理连接，之后在建立逻辑连接之前，向已有节点发送准入请求，已有节点收到后会用当前节点的公私钥构造交易，自动发送创建提案的交易。

- 投票（手动发交易）

管理员收到创建提案的消息推送后对当前提案进行投票。

- 执行提案（自动发交易

发送创建提案交易的节点检测到提案投票通过后，会自动发送执行提案的交易。


2. 加入共识

新节点与网络中的各个节点建立好逻辑连接之后，此时节点还没有加入共识，如果需要加入共识，需要由管理员发送创建提案的交易，请求加入共识。其他管理员收到创建提案的消息推送后对当前提案进行投票。当投票通过后再由创建提案的管理员发送执行提案的交易，在执行提案的过程中，将新节点加入共识。

加入共识的所有操作都需要管理员手动进行。


不依赖分布ca增加节点
:::::::::::::::::::::::::::

不依赖分布式ca增加节点表示新加入的节点事先在通过其他方式已经与已有的节点交换了证书，可以直接启动新节点组网，这时只需要通过一个提案完成加入共识的流程即可。由于新节点不是通过线上的方式与其他节点组网，因此账本上的节点状态没有加入新节点，在创建提案将新节点加入共识的时候，需要在加入共识的操作前加上将节点加入hosts的操作，这些操作可以在一个提案里完成，其操作流程如下：

1. 新节点拿到证书后，启动新节点，新节点与其他节点建立连接

2. 使用管理账户发送创建提案的交易，提案中包含两个操作：

   1. 新节点加入Hosts

   2. 新节点加入VSet，即加入共识

3. 其他管理员对此提案进行投票

4. 投票通过后，提案创建者发送交易执行提案，在提案执行的过程中完成新节点状态维护到账本以及加入共识的操作。


删除节点
>>>>>>>>>>>>>>>>>>>>>>>

共识删除节点
::::::::::::::::::::::::

网络中已有节点要退出网络时，首先是退出共识，即从共识中删除节点。一旦从共识中删除节点，节点要再次加入共识需要重新走一遍加入共识的流程。

从共识中删除节点，首先需要一个管理员发起创建提案的交易，这个提案的内容是从共识中删除节点。其他管理员收到创建提案的消息推送后对当前提案进行投票。当投票通过后再由创建提案的管理员发送执行提案的交易，在执行提案的过程中，从共识中删除节点。

当共识删除节点完成之后，会向 `eventhub` 抛出一个 `UpdateRotingTableEvent` ， `eventhub` 收到之后会根据事件的内容，调用网络层与此节点断开连接的操作。

共识删除节点的所有操作都需要管理员手动进行。


litesdk接口说明
>>>>>>>>>>>>>>>>>>>>>>>>>>>

节点管理接口
:::::::::::::::::::::::::::

litesdk提供了 `ProposalBuilder` 构造器用于构造提案的操作 `ProposalOperation` ，在 `ProposalBuilder` 中提供了 `createForNode` 、 `vote` 、 `cancel` 和 `execute` 方法分别用于创建节点类(即节点管理)提案、提案投票、取消提案和执行提案的提案操作，其定义如下::

    public static class ProposalBuilder extends BuiltinOperationBuilder {
        
        /**
         * create creat ProposalOperation for node to create node proposal.
         *
         * @param opts node operations
         * @return {@link ProposalBuilder}
         */
        public ProposalBuilder createForNode(NodeOperation... opts);

        /**
         * create vote ProposalOperation to vote proposal.
         *
         * @param proposalID proposal id
         * @param vote       vote value, true means agree; false means refuse
         * @return {@link ProposalBuilder}
         */
        public ProposalBuilder vote(int proposalID, boolean vote);

        /**
         * create cancel ProposalOperation to cancel proposal.
         *
         * @param proposalID proposal id
         * @return {@link ProposalBuilder}
         */
        public ProposalBuilder cancel(int proposalID);

        /**
         * create execute ProposalOperation to cancel proposal.
         *
         * @param proposalID proposal id
         * @return {@link ProposalBuilder}
         */
        public ProposalBuilder execute(int proposalID);
    }

在创建节点类提案时，根据提案内容中需要对节点管理的操作，接收对应的节点类操作即可。litesdk为节点类操作 `NodeOperation` 提供了构造器 `NodeBuilder` ，构造器中提供了 `addNode` 、 `addVP` 、 `removeVP` 以及 `build` 方法，其定义如下::

    public static class NodeBuilder {
  
        /**
         * create NodeBuilder to add node with give params.
         *
         * @param pub       public key of new node
         * @param hostname  host name of new node
         * @param role      node role
         * @param namespace namespace
         * @return {@link NodeBuilder}
         */
        public NodeBuilder addNode(byte[] pub, String hostname, String role, String namespace);
  
        /**
         * create NodeBuilder to add vp.
         *
         * @param hostname  host name of new node
         * @param namespace namespace the new node will add
         * @return {@link NodeBuilder}
         */
        public NodeBuilder addVP(String hostname, String namespace);
  
        /**
         * create NodeBuilder to remove vp.
         *
         * @param hostname  host name of remove node
         * @param namespace namespace the node will be removed
         * @return {@link NodeBuilder}
         */
        public NodeBuilder removeVP(String hostname, String namespace);
  
        /**
         * return build NodeOperation.
         *
         * @return {@link NodeOperation}
         */
        public NodeOperation build();
    }

节点管理的操作构造好后，用ProposalBuild构造器构造提案相关的操作，创建好之后，使用 `BVMBuilder` 提供的 `invoke` 方法构造bvm的交易体，使用 `build` 方法构造出交易 `transaction` ，并为交易设置 `txVersion` 并使用 `sign` 方法签名，得到最终可以发送执行的交易体。

节点查询接口
::::::::::::::::::::::

1. 查询连接的节点信息（getHosts）

参数：

- role 节点角色（目前只支持查询vp节点）

- nodeIds 请求向哪些节点发送

::

    Request<HostsResponse> getHosts(String role, int... nodeIds);

拿到 `HostsResponse` 后，通过 `getHosts` 方法拿到节点信息。 `getHosts` 方法返回的是key为节点名，value为节点公钥的map。


2. 查询参与共识的节点信息（getVSet）

参数：

- nodeIds 请求向哪些节点发送

::

    Request<VSetResponse> getVSet(int... nodeIds);

拿到 `VSetResponse` 后，通过 `getVSet` 拿到共识的节点信息。 `getVSet` 方法返回的是所有参与共识的节点列表。


操作流程
-------------------------

启动节点并初始化
>>>>>>>>>>>>>>>>>>>>>>>>>

目前只有rbft共识算法支持动态增删节点。因此，需要预先启动至少四个节点。由于预先启动的节点的证书是通过线下颁发的，且作为创世节点，不是通过提案投票的形式，因此在节点启动完成后需要通过提案完成节点初始化的流程。

另外，在新节点加入时，老节点通过节点的公私钥发送交易创建提案。也需要预先拿到每个节点的地址（可使用certgen获取节点公钥地址），为节点地址授予 `nodeOfVP` 角色，以便有权创建执行提案（此角色的权重为0，即创建或投票的操作有权进行，但是不占权重，需要其他 `admin` 角色的管理员投票才有权重）。

节点初始化
:::::::::::::::::::::::::

对于节点类型的提案与权限类型的提案相似，都支持列表操作（只是目前增删节点一次只支持增删一个节点）。节点类型的提案在创建时用node表示。

节点初始化流程：

1. 创建一个节点类型的提案，提案中包含将初始节点加入hosts和vset的操作。加入hosts使用AddNode，加入vset使用AddVP。每一个节点加入vset之前要先将其加入hosts。

2. 对提案进行投票

3. 创建提案的用户执行提案

节点账户授权（可选）
:::::::::::::::::::::::::::

当依赖分布式ca进行增加节点时，有节点自动发交易创建提案对操作，因此需要为节点账户授权。

节点账户授权流程如下：

1. 创建一个权限类型的提案，提案中包含给所有节点账户（节点账户可根据节点公钥即ecert获得）授予nodeOfVP角色的操作。

2. 对提案进行投票

3. 创建提案的用户执行提案

litesdk初始化使用示例
:::::::::::::::::::::::::::::

预先启动了四个节点，四个节点启动后已达成共识，然后进行初始化。四个节点的hostname分别为node1、node2、node3、node4，四个节点的公钥(ecert)文件分别命名为node1.cert、node2.cert、node3.cert、node4.cert，四个节点的公钥地址分别为address1、address2、address3、address4。使用litesdk完成四个节点的初始化并为节点账户授权，其代码如下::

    public void testInit() throws RequestException {
        // init node
        String role = "nodeOfVP";
        List<NodeOperation> nodeOpts = new ArrayList<>();
        for (int i = 1; i < 5; i++) {
            InputStream inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("node" + i + ".cert");
            byte[] resource = FileUtil.readFileAsBytes(inputStream);
            nodeOpts.add(new NodeOperation.NodeBuilder().addNode(resource, "node" + i, "vp", "global").build());
            nodeOpts.add(new NodeOperation.NodeBuilder().addVP("node" + i, "global").build());
        }
        completeProposal(new ProposalOperation.ProposalBuilder().createForNode(nodeOpts.toArray(new NodeOperation[nodeOpts.size()])).build());

        // init node account
        completeProposal(new ProposalOperation.ProposalBuilder().createForPermission(
                new PermissionOperation.PermissionBuilder().grant(role, address1).build(),
                new PermissionOperation.PermissionBuilder().grant(role, address2).build(),
                new PermissionOperation.PermissionBuilder().grant(role, address3).build(),
                new PermissionOperation.PermissionBuilder().grant(role, address3).build(),
                new PermissionOperation.PermissionBuilder().grant(role, address4).build()
        ).build());
    }

    public String completeProposal(BuiltinOperation opt) throws RequestException {
        // create
        invokeBVMContract(opt, accountService.fromAccountJson(accountJsons[0]));

        Request<ProposalResponse> proposal = configService.getProposal();
        ProposalResponse proposalResponse = proposal.send();
        ProposalResponse.Proposal prop = proposalResponse.getProposal();

        // vote
        for (int i = 1; i < 6; i++) {
            invokeBVMContract(new ProposalOperation.ProposalBuilder().vote(prop.getId(), true).build(), accountService.fromAccountJson(accountJsons[i]));
        }

        // execute
        Result result = invokeBVMContract(new ProposalOperation.ProposalBuilder().execute(prop.getId()).build(), accountService.fromAccountJson(accountJsons[0]));
        Assert.assertEquals("", result.getErr());

        System.out.println(result.getRet());
        List<OperationResult> resultList = Decoder.decodeBVMResult(result.getRet());
        for (OperationResult or : resultList) {
            Assert.assertEquals(SuccessCode.getCode(), or.getCode());
            Assert.assertEquals(SuccessCode.getCode(), or.getCode());
        }
        if (resultList.size() > 0) {
            return resultList.get(0).getMsg();
        }
        return null;
    }

    public Result invokeBVMContract(BuiltinOperation opt, Account acc) throws RequestException {
        Transaction transaction = new Transaction.
                BVMBuilder(acc.getAddress()).
                invoke(opt).
                build();
        transaction.sign(acc);

        ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
        Result result = Decoder.decodeBVM(receiptResponse.getRet());
        System.out.println(result);
        return result;
    }

新增节点
>>>>>>>>>>>>>>>>>>

依赖分布式ca新增节点
::::::::::::::::::::

1. 组网

   1. 启动新节点

   2. 管理员对新节点加入对提案进行投票

   3. 等待投票通过后进行证书交换组网成功

2. 加入共识

   1. 管理员创建节点类型的提案，提案中包含将新节点加入共识的操作，即AddVP

   2. 管理员投票

   3. 管理员执行提案

   4. 等待达成共识

不依赖分布式ca新增节点
::::::::::::::::::::::::

1. 启动新节点

2. 管理员创建节点类型的提案，提案中包含将新节点加入hosts的操作（AddNode）和加入共识的操作（AddVP）

3. 管理员投票

4. 管理员执行提案

5. 等待达成共识

litesdk使用示例
:::::::::::::::::::::::

依赖分布式ca新增节点node5（新增节点之前完成了节点初始化和节点账户初始化）到名为global的namespace中，使用litesdk完成分布式ca新增节点，其代码如下::

    public void testAdd() throws RequestException {

        // start node5, and node5 already connect other nodes

        // use admin account vote for node5's addNode proposal
        Request<ProposalResponse> proposal = configService.getProposal();
        ProposalResponse proposalResponse = proposal.send();
        ProposalResponse.Proposal prop = proposalResponse.getProposal();
        for (int i=0; i< prop.getThreshold(); i++){
            invokeBVMContract(new ProposalOperation.ProposalBuilder().vote(prop.getId(), true).build(), accountService.fromAccountJson(accountJsons[i]));
        }

        // wait node5's addNode proposal execute

        // create、vote and execute proposal for node5 addVP
        completeProposal(new ProposalOperation.ProposalBuilder().createForNode(new NodeOperation.NodeBuilder().addVP("node5","global").build()).build());
    }

    public String completeProposal(BuiltinOperation opt) throws RequestException {
        // create
        invokeBVMContract(opt, accountService.fromAccountJson(accountJsons[0]));

        Request<ProposalResponse> proposal = configService.getProposal();
        ProposalResponse proposalResponse = proposal.send();
        ProposalResponse.Proposal prop = proposalResponse.getProposal();

        // vote
        for (int i = 1; i < 6; i++) {
            invokeBVMContract(new ProposalOperation.ProposalBuilder().vote(prop.getId(), true).build(), accountService.fromAccountJson(accountJsons[i]));
        }

        // execute
        Result result = invokeBVMContract(new ProposalOperation.ProposalBuilder().execute(prop.getId()).build(), accountService.fromAccountJson(accountJsons[0]));
        Assert.assertEquals("", result.getErr());

        System.out.println(result.getRet());
        List<OperationResult> resultList = Decoder.decodeBVMResult(result.getRet());
        for (OperationResult or : resultList) {
            Assert.assertEquals(SuccessCode.getCode(), or.getCode());
            Assert.assertEquals(SuccessCode.getCode(), or.getCode());
        }
        if (resultList.size() > 0) {
            return resultList.get(0).getMsg();
        }
        return null;
    }

    public Result invokeBVMContract(BuiltinOperation opt, Account acc) throws RequestException {
        Transaction transaction = new Transaction.
                BVMBuilder(acc.getAddress()).
                invoke(opt).
                build();
        transaction.sign(acc);

        ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
        Result result = Decoder.decodeBVM(receiptResponse.getRet());
        System.out.println(result);
        return result;
    }

不依赖分布式ca新增节点node5到名为global的namespace中（新增节点之前完成了节点初始化和节点账户初始化），使用litesdk完成非分布式ca新增节点，node5的公钥(ecert)文件名为node5.cert，其代码如下::

    public void testAdd() throws RequestException {

        // start node5, and node5 already connect other nodes

        // create、vote and execute proposal for node5 addVP
        InputStream inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("node5.cert");
        byte[] pub = FileUtil.readFileAsBytes(inputStream);
        completeProposal(new ProposalOperation.ProposalBuilder().createForNode(
                new NodeOperation.NodeBuilder().addNode(pub, "node5","vp", "global").build(),
                new NodeOperation.NodeBuilder().addVP("node5","global").build()
        ).build());
    }

    public String completeProposal(BuiltinOperation opt) throws RequestException {
        // create
        invokeBVMContract(opt, accountService.fromAccountJson(accountJsons[0]));

        Request<ProposalResponse> proposal = configService.getProposal();
        ProposalResponse proposalResponse = proposal.send();
        ProposalResponse.Proposal prop = proposalResponse.getProposal();

        // vote
        for (int i = 1; i < 6; i++) {
            invokeBVMContract(new ProposalOperation.ProposalBuilder().vote(prop.getId(), true).build(), accountService.fromAccountJson(accountJsons[i]));
        }

        // execute
        Result result = invokeBVMContract(new ProposalOperation.ProposalBuilder().execute(prop.getId()).build(), accountService.fromAccountJson(accountJsons[0]));
        Assert.assertEquals("", result.getErr());

        System.out.println(result.getRet());
        List<OperationResult> resultList = Decoder.decodeBVMResult(result.getRet());
        for (OperationResult or : resultList) {
            Assert.assertEquals(SuccessCode.getCode(), or.getCode());
            Assert.assertEquals(SuccessCode.getCode(), or.getCode());
        }
        if (resultList.size() > 0) {
            return resultList.get(0).getMsg();
        }
        return null;
    }

    public Result invokeBVMContract(BuiltinOperation opt, Account acc) throws RequestException {
        Transaction transaction = new Transaction.
                BVMBuilder(acc.getAddress()).
                invoke(opt).
                build();
        transaction.sign(acc);

        ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
        Result result = Decoder.decodeBVM(receiptResponse.getRet());
        System.out.println(result);
        return result;
    }


删除节点
>>>>>>>>>>>>>>>>>>>>>

删除节点流程
:::::::::::::::::::::

1. 管理员创建节点类型的提案，提案中包含删除节点的操作

2. 管理员投票

3. 管理员执行提案

4. 等待达成共识

litesdk使用示例
::::::::::::::::::::::

从名为global的namespace中删除vp节点node5，其代码如下::

    public void testRemoveVP() throws RequestException {
        BuiltinOperation opt = new ProposalOperation.ProposalBuilder().createForNode(new NodeOperation.NodeBuilder().removeVP("node5", "global").build()).build();
        // create proposal for removeVP node5
        invokeBVMContract(opt, accountService.fromAccountJson(accountJsons[0]));

        Request<ProposalResponse> proposal = configService.getProposal();
        ProposalResponse proposalResponse = proposal.send();
        ProposalResponse.Proposal prop = proposalResponse.getProposal();

        // vote
        for (int i = 1; i < 6; i++) {
            invokeBVMContract(new ProposalOperation.ProposalBuilder().vote(prop.getId(), true).build(), accountService.fromAccountJson(accountJsons[i]));
        }

        // execute
        Result result = invokeBVMContract(new ProposalOperation.ProposalBuilder().execute(prop.getId()).build(), accountService.fromAccountJson(accountJsons[0]));
        Assert.assertEquals("", result.getErr());

        System.out.println(result.getRet());
        List<OperationResult> resultList = Decoder.decodeBVMResult(result.getRet());
        for (OperationResult or : resultList) {
            Assert.assertEquals(SuccessCode.getCode(), or.getCode());
            Assert.assertEquals(SuccessCode.getCode(), or.getCode());
            System.out.println(or);
        }
    }

    public Result invokeBVMContract(BuiltinOperation opt, Account acc) throws RequestException {
        Transaction transaction = new Transaction.
                BVMBuilder(acc.getAddress()).
                invoke(opt).
                build();
        transaction.sign(acc);

        ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
        Result result = Decoder.decodeBVM(receiptResponse.getRet());
        System.out.println(result);
        return result;
    }