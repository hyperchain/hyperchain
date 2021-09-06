.. _contract-auth-manage:

合约权限管理
^^^^^^^^^^^^^

合约权限管理，是基于链级角色，对账户进行链上的合约（也可以是转账交易的to）访问权限进行管理。

合约权限管理概述
------------------

合约权限管理是基于 :ref:`链级角色管理 <chain-role-manage>`，为账户分配不同的链级角色，然后使用 :ref:`链级配置管理 <chain-conf-manage>` 设置基于角色的交易拦截规则以及交易拦截开关。当开关打开时，设置了交易拦截规则后，交易在被执行之前会按照这个规则检验交易发送者是否有权限发送此交易。

拦截规则包含以下字段：

- allowAnyone : bool 是否允许任何人访问

- authorizedRoles : []string 允许访问的角色列表

- forbiddenRoles : []string 禁止访问的角色列表

- id : int 规则id

- name : string 规则名称

- to :  []string 规则限制的交易地址（可以是合约地址、或账户地址，目前支持通配符*）

- vm ：[]string 规则限制的虚拟机类型（目前支持的是evm 、 hvm、bvm三种，支持通配符*）

当同一交易地址被多个拦截规则匹配时，会按照 **id更小** 当规则进行校验。对于上述拦截规则中包含的字段，其优先级为forbiddenRoles > allowAnyone > authorizedRoles。对于没有拦截规则限制的交易地址，则不进行校验拦截。

例如，当某笔交易的to在多个规则中被添加了拦截规则时，会取出id更小的规则进行校验，先验证交易发起者是否含有被禁止的角色，有则校验不通过；再校验是否允许任何人访问，是则校验通过；最后校验交易发起者是否含有允许访问的角色列表，是则校验通过，否则校验不通过。 

litesdk使用示例
>>>>>>>>>>>>>>>>>>>>>>>>>

开启交易拦截过滤器，设置拦截规则，只有链级管理员(admin)才能访问hvm中的合约地址contractA，其中accountA具有链级管理员角色，而accouuntB没有，则使用accountA调用hvm的contractA合约成功，使用accountB调用hvm的contractA合约提示没有权限，其代码如下::

    public void testConfigOperation() throws RequestException {
        // setRules
        ArrayList<NsFilterRule> rules = new ArrayList<>();
        NsFilterRule rule = new NsFilterRule();
        List<String> authorizedRoles = new ArrayList<>();
        authorizedRoles.add("admin");
        rule.setAuthorizedRoles(authorizedRoles);
        List<String> vms = new ArrayList<>();
        vms.add("hvm");
        rule.setVm(vms);
        List<String> tos = new ArrayList<>();
        String contractA = "37a1100567bf7e0de2f5a0dc1917f0552aa43d88";
        tos.add(contractA);
        rules.add(rule);
        completeProposal(new ProposalOperation.ProposalBuilder().createForConfig(
                new ConfigOperation.ConfigBuilder().setFilterEnable(true).build(),
                new ConfigOperation.ConfigBuilder().setFilterRules(rules).build()
        ).build());

        // use account A, which has admin role, has permission to invoke contract A
        // use account B, which has not admin role, has not permission to invoke contract A
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