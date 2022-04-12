.. _contract-naming-service:

合约命名服务CNS
^^^^^^^^^^^^^^^^^^^^^^^^

合约命名
------------------

合约命名，为一个合约地址绑定一个别名。目前一个合约地址只能有一个合约命名，一个合约命名只能绑定一个合约地址。其操作步骤如下：

1. 由一个链级管理员发送交易，执行bvm合约，创建一个合约命名类型的提案；

2. 如果提案的阈值大于1，需要其他的链级管理员发送交易，执行bvm合约，对这个提案投票，当收集到足够的同意票数后提案进入等待执行状态；

3. 创建这个提案的链级管理员发送交易，执行bvm合约，执行这个投票通过的提案，完成合约地址与合约命名的绑定。

合约地址与合约命名绑定之后，可以通过合约命名执行合约，也可以根据合约命名查询相应数据。

创建合约命名类型的提案
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

使用LiteSDK发送交易，执行bvm合约，创建合约命名类型的提案时：

1. 使用 `CNSBuilder` 提供的 `setCName` 方法构造合约命名操作的内容，使用 `build` 方法构造合约命名操作 `CNSOperation` ；

2. 使用 `ProposalBuilder` 提供的 `createForCNS` 方法构造创建提案操作的内容，使用 `build` 方法构造提案操作 `ProposalOperation` ；

3. 使用 `BVMBuilder` 提供的 `invoke` 方法构造执行bvm合约的交易内容，使用 `build` 方法构造交易 `Transaction` ；

4. 使用 `Transaction` 提供的 `sign` 方法，用链级管理员账户私钥对交易签名；

5. 使用 `ContractService` 提供的 `invoke` 方法构造执行合约的交易请求，再通过请求中的 `send` 方法发送请求，使用 `polling` 获取交易回执。

最后使用 `Decoder` 提供的 `decodeBVM` 方法解析回执即可拿到交易执行结果。

例如，链级管理员A需要创建一个提案，为合约地址 `0x1234567890123456789012345678901234567890` 绑定一个合约命名 `Bank` ，通过LiteSDK发送交易创建合约命名类型提案的示例如下::

    Account ac = accountService.fromAccountJson(adminA);
    Transaction transaction = new Transaction.
            BVMBuilder(ac.getAddress()).
            invoke(new ProposalOperation.
                    ProposalBuilder().
                    createForCNS(new CNSOperation.
                            CNSBuilder().
                            setCName("0x1234567890123456789012345678901234567890", "Bank").
                            build()).
                    build()).
            build();
    transaction.sign(ac);
    ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
    Result result = Decoder.decodeBVM(receiptResponse.getRet());
    System.out.println(result);

对合约命名类型的提案进行投票
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

当提案阈值大于1时，需要其他的链级管理员投票同意，通过后提案进入等待执行状态。

使用LiteSDK发送交易，执行bvm合约，对提案投票时：

1. 使用 `ProposalBuilder` 提供的 `vote` 方法构造提案投票操作的内容，使用 `build` 方法构造提案操作 `ProposalOperation` ；

2. 使用 `BVMBuilder` 提供 `invoke` 方法构造执行bvm合约的交易内容，使用 `build` 方法构造交易 `Transaction` ；

3. 使用 `Transaction` 提供的 `sign` 方法，用链级管理员账户私钥对交易签名；

4. 使用 `ContractService` 提供的 `invoke` 方法构造执行合约的交易请求，再通过请求中的 `send` 方法发送请求，使用 `polling` 获取交易回执。

例如，链级管理员B对上述创建的提案投赞同票（假设上述提案的id是1），通过LiteSDK发送交易对提案进行投票对示例如下::

    Account ac = accountService.fromAccountJson(adminB);
    Transaction transaction = new Transaction.
            BVMBuilder(ac.getAddress()).
            invoke(new ProposalOperation.ProposalBuilder().
                    // 赞同票
                    vote(1, true).
                    build()).
            build();
    transaction.sign(ac);
    ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
    Result result = Decoder.decodeBVM(receiptResponse.getRet());
    System.out.println(result);

执行合约命名类提案
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

当提案投票通过后，处于等待执行状态时，提案创建者可发送交易，执行bvm合约，执行提案。

1. 使用 `ProposalBuilder` 提供的 `execute` 方法构造执行提案操作的内容，使用 `build` 方法构造提案操作 `ProposalOperation` ；

2. 使用 `BVMBuilder` 提供的 `invoke` 方法构造执行bvm合约的交易内容，使用 `build` 方法构造交易 `Transaction` ；

3. 使用 `Transaction` 提供的 `sign` 方法，用链级管理员账户私钥对交易签名；

4. 使用 `ContractService` 提供的 `invoke` 方法构造执行合约的交易请求，再通过请求中的 `send` 方法发送请求，使用 `polling` 获取交易回执。

例如，链级管理员A发送交易执行提案，通过LiteSDK发送交易对提案进行投票对示例如下::

    Account ac = accountService.fromAccountJson(adminA);
    Transaction transaction = new Transaction.
            BVMBuilder(ac.getAddress()).
            invoke(new ProposalOperation.ProposalBuilder().
                    execute(1).
                    build()).
            build();
    transaction.sign(ac);
    ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
    Result result = Decoder.decodeBVM(receiptResponse.getRet());
    System.out.println(result);


根据合约命名执行合约
-----------------------------

根据合约命名执行合约时，使用 `Builder` 提供的 `invokeByName` 方法构造相应的交易内容，发送交易，执行合约。

例如，evm合约 `0x123456789012345678901234567890123456789` 的合约命名为 `Bank` ，通过合约命名执行这个合约的示例如下::

    Account account = accountService.genAccount(Algo.SMAES, PASSWORD);
    Transaction transaction = new Transaction.EVMBuilder(account.getAddress()).
                    invoke("", "TestBytes32(bytes32)", abi, params1).
                    contractName("Bank").
                    build();
    transaction4.sign(account);
    ReceiptResponse receiptResponse = contractService.maintain(transaction4).send().polling();
    System.out.println(receiptResponse.getRet());

LiteSDK中合约命名相关接口
------------------------------

根据合约地址查询合约命名（getNamebyAddress）
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

参数：

- address 合约地址

- nodeIds 请求向哪些节点发送

::

    Request<NameResponse> getNameByAddress(String address, int... nodeIds);

拿到 `NameResponse` 后，通过 `getName` 方法拿到合约命名。 `getName` 方法返回的是一个字符串。

根据合约命名查询合约地址（getAddressByName）
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

参数：

- name 合约命名

- nodeIds 请求向哪些节点发送

::

    Request<AddressResponse> getAddressByName(String name, int... nodeIds);

拿到 `AddressResponse` 后，通过 `getAddress` 方法拿到合约地址。 `getAddress` 方法返回的是一个字符串。

查询所有合约地址到合约命名的映射关系（getAllCNS）
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

参数：

- nodeIds 请求向哪些节点发送

::

    Request<AllCNSResponse> getAllCNS(int... nodeIds);

拿到 `AllCNSResponse` 后，通过 `getAllCNS` 方法拿到所以的合约地址到合约命名的映射关系。 `getAllCNS` 方法返回的是key为合约地址，value为合约命名的map。

例如：evm合约 `0x1234567890123456789012345678901234567890`。
​