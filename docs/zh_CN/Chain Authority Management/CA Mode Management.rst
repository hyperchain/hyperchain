.. _CA-Mode-Management:

CA模式管理
^^^^^^^^^^^^^

引言
------

编写目的
>>>>>>>>>>>

本文档提供Hyperchain中ca模式管理的操作指南，作为ca模式管理与root ca管理的使用说明。

CA模式管理
------------

功能描述
>>>>>>>>>>>

ca模式管理用于管理ca模式。

默认情况下ca模式在启动时配置好无法更改，后续只能查询。但是对于特殊的，从2.7.0以前的版本升级到2.7.0时，配置的ca模式无法通过升级记录到链上，因此需要通过提案设置ca模式，设置后也无法更改。

ca模式管理包括如下操作：

1. 设置ca模式。从2.7.0以前的版本升级到2.7.0+，需要通过提案设置ca模式，如果为中心ca，设置完ca模式后还需要通过新增root ca的方式将所有使用的root ca上链。

2. 查询ca模式。查询ca模式可以通过提案的 `Direct` 操作直接进行查询。

当ca模式为中心ca（非分布式ca）即Center时，可通过 `RootCAContract` 合约管理中心ca模式下使用的root ca，root ca管理包括如下操作：

3. 新增root ca，即增加一个新的信任的root ca。

4. 查询root ca，即查询当前模式下信任的所有的root ca。

litesdk接口说明
>>>>>>>>>>>>>>>>>>>

ca模式接口说明
::::::::::::::::::::::::

litesdk提供了 `ProposalBuilder` 构造器用于构造提案的操作，在 `ProposalBuilder` 中提供了 `createForCAMode` 、 `directForCAMode` 、 `vote` 、 `cancel` 和 `execute` 方法分别用于创建ca模式类提案、直接执行ca模式类操作、提案投票、取消提案和执行提案的提案操作，其定义如下::

     public static class ProposalBuilder extends BuiltinOperationBuilder {

            /**
             * create creat ProposalOperation for ca to create ca mode proposal.
             *
             * @param opts ca_mode operations, i.e. setCAMode
             * @return {@link ProposalBuilder}
             */
            public ProposalBuilder createForCAMode(CAModeOperation... opts);

            /**
             * create direct ProposalOperation for ca to execute ca mode operation directly.
             *
             * @param opts ca_mode operations, i.e. getCAMode
             * @return {@link ProposalBuilder}
             */
            public ProposalBuilder directForCAMode(CAModeOperation... opts);

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

ca模式的操作分以下几种：

- SetCAMode，设置ca模式，即为当前ns设置ca模式。ca模式存在的情况下不能再次设置ca模式，即只有当老版本升级到2.7.0+的版本时，可通过这届操作设置ca模式，设置的ca模式需要与新版启动时设置的ca模式相同，否则当前ns将停止运行。

- GetCAMode，查询ca模式，即返回当前ns设置的ca模式。可通过 `ProposalContract` 的 `Direct` 操作直接查询，无需投票。

构造合约命名类操作 `CAModeOperation` 的构造器 `CAModeBuilder` 提供了 `setCAMode` 、 `getCAMode` 和 `build` 方法，其定义如下::

      public static class CAModeBuilder {
              /**
               * create CAModeOperation to set ca mode.
               * when has not set ca mode, can set ca mode.
               *
               * @param mode {@link CAMode}
               * @return {@link CAModeBuilder}
               */
              public CAModeBuilder setCAMode(CAMode mode);
      ​
              /**
               * create CAModeOperation to get ca mode.
               *
               * @return {@link CAModeBuilder}
               */
              public CAModeBuilder getCAMode();
        
              /**
               * return build CNSOperation.
               *
               * @return {@link CAModeOperation}
               */
              public CAModeOperation build();
      }

root ca接口说明
::::::::::::::::::::::::

`RootCAContract` 中提供的合约方法如下：

1. `AddRootCA` : AddRootCA方法接收一个参数，即新增的root.ca文件内容，用于新增root ca，当ca mode为center，即中心ca时，链级管理员（admin用户）可以新增root ca。

2. `GetRootCAs` : GetRootCAs方法不需要入参，用于查询链上所有的root ca，当ca mode为center时，返回链上所有当root ca。

构造 `RootCAContract` 操作的构造器 `RootCABuilder` 提供了 `addRootCA` 和 `getRootCAs` 方法，分别用于构造 `RootCAContract` 合约中的 `AddRootCA` 和 `GetRootCAs` 方法，其定义如下::

      public static class RootCABuilder extends BuiltinOperationBuilder{
              /**
               * create RootCAOperation to add root ca.
               * when ca mode is center, admin can add root ca.
               *
               * @param rootCA the root ca which will be add.
               * @return {@link RootCABuilder}
               */
              public RootCABuilder addRootCA(String rootCA);
      ​
              /**
               * create RootCAOperation to get root cas.
               * when ca mode is center, everyone can get root cas.
               *
               * @return {@link RootCABuilder}
               */
              public RootCABuilder getRootCAs();
      ​
      }

litesdk使用示例
>>>>>>>>>>>>>>>>>>>

ca模式使用示例
::::::::::::::::::

1. 设置ca模式::

     (旧版升级第一次设置才能成功)
     public void testCAModeOperation() throws RequestException {
        // new proposal create operation for ca mode
        BuiltinOperation opt = new ProposalOperation.ProposalBuilder().createForCAMode(
                new CAModeOperation.CAModeBuilder().setCAMode(CAMode.None).builder()
        ).build();

        // send transaction to create proposal
        invokeBVMContract(opt, accountService.fromAccountJson(accountJsons[0]));

        // get proposal
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

2. 通过 `Direct` 查询ca模式

 ::

     public void getCAMode() throws RequestException {
        Account ac = accountService.fromAccountJson(accountJson);
        Transaction transaction = new Transaction.
                BVMBuilder(ac.getAddress()).
                invoke(new ProposalOperation.ProposalBuilder().directForCAMode(new CAModeOperation.CAModeBuilder().getCAMode().builder()).build()).
                build();
        transaction.sign(ac);
        ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
        Result result = Decoder.decodeBVM(receiptResponse.getRet());
        System.out.println(result);
        System.out.println(result.getErr());
        System.out.println(result.getRet());
     }

root ca使用示例
::::::::::::::::::::

当ca模式为 `Center` 即中心ca时，可以对root ca进行管理。

1. 新增root ca ::

     public void setRootCA() throws RequestException {
        String rootCA = "-----BEGIN CERTIFICATE-----\n" +
                "MIICSTCCAfWgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV\n" +
                "BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD\n" +
                "VQQLEwAxDjAMBgNVBAMTBW5vZGUyMQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl\n" +
                "cnQwIBcNMjAwNTIxMDU1ODU2WhgPMjEyMDA0MjcwNjU4NTZaMHQxCTAHBgNVBAgT\n" +
                "ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh\n" +
                "dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTQxCzAJBgNVBAYTAlpIMQ4wDAYD\n" +
                "VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABBI3ewNK21vHNOPG6U3X\n" +
                "mKJohSNNz72QKDxUpRt0fCJHwaGYfSvY4cnqkbliclfckUTpCkFSRr4cqN6PURCF\n" +
                "zkWjeTB3MA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB\n" +
                "BQUHAwEGAioDBgOBCwEwDwYDVR0TAQH/BAUwAwEB/zANBgNVHQ4EBgQEAQIDBDAP\n" +
                "BgNVHSMECDAGgAQBAgMEMAwGAypWAQQFZWNlcnQwCgYIKoZIzj0EAwIDQgDJibFh\n" +
                "a1tZ3VhL3WIs36DqOS22aetvcn2dXHH9Pw5/s2XI70Mr3ow3RKqJmdmi0PsmLr+K\n" +
                "pCFkuMv2bHnkWuiZAQ==\n" +
                "-----END CERTIFICATE-----";
        String adminAccount = "";
        Account ac = accountService.fromAccountJson(adminAccount);
        Transaction transaction = new Transaction.
                BVMBuilder(ac.getAddress()).
                invoke(new RootCAOperation.RootCABuilder().addRootCA(rootCA).build()).
                build();
        transaction.sign(ac);
        ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
        Result result = Decoder.decodeBVM(receiptResponse.getRet());
        System.out.println(result);
        System.out.println(result.getErr());
        System.out.println(result.getRet());
     }

2. 查询root ca ::

     public void getRootCA() throws RequestException {
        Account ac = accountService.fromAccountJson(accountJson);
        Transaction transaction = new Transaction.
                BVMBuilder(ac.getAddress()).
        invoke(new RootCAOperation.RootCABuilder().getRootCAs().build()).
                        build();
        transaction.sign(ac);
        ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
        Result result = Decoder.decodeBVM(receiptResponse.getRet());
        System.out.println(result);
        System.out.println(result.getErr());
        System.out.println(result.getRet());
     }

