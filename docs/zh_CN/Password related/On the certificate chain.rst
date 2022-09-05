.. _On-the-certificate-chain:

证书上链使用手册
^^^^^^^^^^^^^^^^^^^

1. 引言
===========

1.1 编写目的
----------------

本文档提供hyperchain中证书链上状态修改的操作指南，作为证书上链的使用说明。以下所述证书均为平台管理的证书，crl外部证书仅适用于本使用手册部分内容（2.4）。

2. 证书上链管理概述
===================

证书上链管理用于节点间同步变更证书的使用状态。如下图所示，证书生命周期包括四个状态：可用，冻结，吊销和过期。

证书是否过期由证书字段在生成证书时预先设定，已过期的证书不可进行任何操作。当可用证书通过revoke操作转换为吊销态后状态将无法恢复，而冻结状态与可用态之间可以随意转换。不可用状态下的证书将无法通过安全验证，因此在配置证书时需确认该证书在链上的状态，对此提供了check操作来方便检查。

|image0|

2.1 **证书吊销**
------------------

1. 节点证书的吊销：ecert/rcert

节点证书的吊销 **不提供单独的用户接口** ，但将作为节点删除的附加操作，即当执行删除节点的操作时，被删节点的证书也将自动转为吊销状态。该被删节点如需重新启动，应更换节点目录下的相关证书。

2. sdk证书的吊销

使用sdk接口发起一笔 **调用bvm合约的普通交易** ，完成证书的吊销操作。当发起操作的角色为管理员时，无需提供sdk证书私钥的签名；若为普通角色则需要通过验签证明自己为该证书私钥的所有者。

2.2 sdk证书冻结与恢复
------------------------------

1. sdk证书冻结

只能将目前处于可用状态的证书修改为冻结状态。同样地，与sdk证书吊销类似，使用sdk接口发起一笔 **调用bvm合约的普通交易** ，完成证书的吊销操作。当发起操作的角色为管理员时，无需提供sdk证书私钥的签名；若为普通角色则需要通过验签证明自己为该证书私钥的所有者。

2. sdk证书恢复

恢复冻结状态的证书同样需要发起一笔交易。具体步骤与sdk证书冻结类似，不再赘述。

2.3 证书替换
--------------------

1. 节点证书替换：ecert/rcert

实现节点证书的不停机替换包括以下两步。在正常执行本操作后，原证书将被注销，建议在执行成功后通过2.4的方式检查原证书状态来确保替换正常完成。

step1 **手动替换** 节点主机hyperchain目录( ./namespaces/global/certs/certs)下的全部文件

step2 通过 **rpc命令** 使节点的证书修改生效

**注：**

新文件可通过certgen工具手动生成

若step1中替换后的证书存在问题，将在step2的rpc命令返回中体现。

当仅完成step1时对节点实际使用证书并未产生影响，但若在此时节点发生重启，节点将直接使用替换后的证书同时原证书并未被吊销。

2. sdk证书替换

**手动替换** sdk证书并在sdk配置中修改指定的证书即可

2.4 证书状态检查
----------------------

使用sdk接口发起一笔 **调用bvm合约的普通交易** ，返回的回执包含指定证书的状态描述。发起交易时需将证书内容作为参数来指定待查询的证书。此功能可查询crl证书状态。

3. litesdk接口说明
=========================

3.1 sdk证书吊销接口
--------------------------

 ::

    /**
     * create revoke CertOperation to revoke cert.
     *
     * @param cert the der cert wait to revoke
     * @param priv the private key of this cert
     * @return {@link CertOperation.CertBuilder}
     */
    public CertOperation.CertBuilder revoke(String cert, String priv)

    //使用示例，若为管理员用户，priv字段可填写任意内容
    public void testCertOperationRevoke() throws Exception {
            String ecert = "-----BEGIN CERTIFICATE-----\n" +
                    "MIICODCCAeSgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV\n" +
                    "BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD\n" +
                    "VQQLEwAxDjAMBgNVBAMTBW5vZGUxMQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl\n" +
                    "cnQwIBcNMjAwNTIxMDQyNTQ0WhgPMjEyMDA0MjcwNTI1NDRaMHQxCTAHBgNVBAgT\n" +
                    "ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh\n" +
                    "dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTExCzAJBgNVBAYTAlpIMQ4wDAYD\n" +
                    "VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABDoBjgQsvY4xhyIy3aWh\n" +
                    "4HLOTTY6te1VbmZaH5EZnKzqjU1f436bVsfi9HLE3/MCeZD6ISe1U5giM5NuwF6T\n" +
                    "ZEOjaDBmMA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB\n" +
                    "BQUHAwEGAioDBgOBCwEwDwYDVR0TAQH/BAUwAwEB/zANBgNVHQ4EBgQEAQIDBDAM\n" +
                    "BgMqVgEEBWVjZXJ0MAoGCCqGSM49BAMCA0IAuVuDqguvjPPveimWruESBYqMJ1qq\n" +
                    "ryhXiMhlYwzH1FgUz0TcayuY+4KebRhFhb14ZDXBBPXcn9CYdtbbSxXTogE=\n" +
                    "-----END CERTIFICATE-----";
            String priv = "-----BEGIN EC PRIVATE KEY-----\n" +
                    "MHQCAQEEIFO8E/zYebPTI++gmHNYZEUetgn3DychVadgTUMIJX3VoAcGBSuBBAAK\n" +
                    "oUQDQgAEOgGOBCy9jjGHIjLdpaHgcs5NNjq17VVuZlofkRmcrOqNTV/jfptWx+L0\n" +
                    "csTf8wJ5kPohJ7VTmCIzk27AXpNkQw==\n" +
                    "-----END EC PRIVATE KEY-----";

            Account ac = accountService.fromAccountJson(accountJsons[5]);
            Transaction transaction = new Transaction.
                    BVMBuilder(ac.getAddress()).
                    invoke(new CertOperation.CertBuilder().revoke(
                            ecert, priv).build()).
                    build();
            transaction.sign(ac);
            ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
            Result result = Decoder.decodeBVM(receiptResponse.getRet());
            System.out.println(result.toString());
            Assert.assertTrue(result.isSuccess());
            Assert.assertEquals("", result.getErr());
       }


3.2 sdk证书冻结与恢复接口
---------------------------------

 ::

    /**
     * create freeze CertOperation to freeze cert.
     *
     * @param cert the der cert wait to check
     * @param priv the private key of this cert
     * @return {@link CertOperation.CertBuilder}
     */
     public CertOperation.CertBuilder freeze(String cert, String priv)

    /**
     * create unfreeze CertOperation to unfreeze cert.
     *
     * @param cert the der cert wait to check
     * @param priv the private key of this cert
     * @return {@link CertOperation.CertBuilder}
     */
     public CertOperation.CertBuilder unfreeze(String cert, String priv)

    //使用示例
    public void testCertOperationFreeze() throws Exception {
            String sdkcert = "-----BEGIN CERTIFICATE-----\n" +
                    "MIICFDCCAbqgAwIBAgIIbGmp7HEb95UwCgYIKoEcz1UBg3UwPTELMAkGA1UEBhMC\n" +
                    "Q04xEzARBgNVBAoTCkh5cGVyY2hhaW4xDjAMBgNVBAMTBW5vZGUxMQkwBwYDVQQq\n" +
                    "EwAwHhcNMjEwMzEwMDAwMDAwWhcNMjUwMzEwMDAwMDAwWjA/MQswCQYDVQQGEwJD\n" +
                    "TjEOMAwGA1UEChMFZmxhdG8xDjAMBgNVBAMTBW5vZGUxMRAwDgYDVQQqEwdzZGtj\n" +
                    "ZXJ0MFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAE1hoClj022lTxWSUCw0Ht4PT+dr8/\n" +
                    "n0BQLeuQVBCnZWKNntBg6cMyVSbMVtcyhAyB8s4+tvzS5bIOqYjLqdO18KOBpDCB\n" +
                    "oTAOBgNVHQ8BAf8EBAMCAe4wMQYDVR0lBCowKAYIKwYBBQUHAwIGCCsGAQUFBwMB\n" +
                    "BggrBgEFBQcDAwYIKwYBBQUHAwQwDAYDVR0TAQH/BAIwADAdBgNVHQ4EFgQUEo46\n" +
                    "euyltTBBzeqlUhbr7DhPVvowHwYDVR0jBBgwFoAUmrWTObRDvo/F/zj5lGV+tYEr\n" +
                    "LbswDgYDKlYBBAdzZGtjZXJ0MAoGCCqBHM9VAYN1A0gAMEUCIHnScuepuomkq2OT\n" +
                    "prJL44lxsSkc4Zhpq6c+IpX5cbmZAiEA6l2BMWHuDrVudJ2COYWo8E42mvn7lLPD\n" +
                    "mpMkfrWt5ek=\n" +
                    "-----END CERTIFICATE-----\n";
            String priv = "-----BEGIN EC PRIVATE KEY-----\n" +
                    "MHQCAQEEICKWeh1X4x1cZI+nfsAw5VXDgLPspN9vixkTlOTSllknoAcGBSuBBAAK\n" +
                    "oUQDQgAE1hoClj022lTxWSUCw0Ht4PT+dr8/n0BQLeuQVBCnZWKNntBg6cMyVSbM\n" +
                    "VtcyhAyB8s4+tvzS5bIOqYjLqdO18A==\n" +
                    "-----END EC PRIVATE KEY-----\n";

            Account ac = accountService.fromAccountJson(accountJsons[5]);
            Transaction transaction = new Transaction.
                    BVMBuilder(ac.getAddress()).
                    invoke(new CertOperation.CertBuilder().freeze(sdkcert, priv).build()).
                    build();
            transaction.sign(ac);
            ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();

            System.out.println(new String(ByteUtil.fromHex(receiptResponse.getRet())));
            Result result = Decoder.decodeBVM(receiptResponse.getRet());
            Assert.assertTrue(result.isSuccess());
            Assert.assertEquals("", result.getErr());

            transaction = new Transaction.
                    BVMBuilder(ac.getAddress()).
                    invoke(new CertOperation.CertBuilder().check(sdkcert.getBytes()).build()).
                    build();
            transaction.sign(ac);
            receiptResponse = contractService.invoke(transaction).send().polling();

            System.out.println(new String(ByteUtil.fromHex(receiptResponse.getRet())));
            result = Decoder.decodeBVM(receiptResponse.getRet());
            Assert.assertTrue(result.isSuccess());
            Assert.assertEquals("this cert is freezing", result.getErr());

            transaction = new Transaction.
                    BVMBuilder(ac.getAddress()).
                    invoke(new CertOperation.CertBuilder().unfreeze(sdkcert, priv).build()).
                    build();
            transaction.sign(ac);
            receiptResponse = contractService.invoke(transaction).send().polling();

            System.out.println(new String(ByteUtil.fromHex(receiptResponse.getRet())));
            result = Decoder.decodeBVM(receiptResponse.getRet());
            Assert.assertTrue(result.isSuccess());
            Assert.assertEquals("", result.getErr());
        }


3.3 节点证书替换接口(仅gosdk开放)
---------------------------------

 ::

    /*输入值：节点hostname
     *返回值：替换交易的hash值，可通过查询交易信息检查操作是否执行成功
     */
    func (rpc *RPC) ReplaceNodeCerts(hostname string) (string, StdError)

    //step2使用示例 (step1已完成)
    func TestRPC_ReplaceNodeCerts(t *testing.T) {
        rpc, _ = rpc.BindNodes(1)
        hash1, err := rpc.ReplaceNodeCerts("node1")
        assert.Nil(t, err)
        fmt.Println(hash1)
    }


3.4 证书状态检查接口
-----------------------

 ::

    /**
     * create check CertOperation to check cert.
     *
     * @param cert the der cert wait to check
     * @return {@link CertOperation.CertBuilder}
     */
     public CertOperation.CertBuilder check(byte[] cert)

    //使用示例
    public void testCertOperationCheck() throws Exception {
            String ecert = "-----BEGIN CERTIFICATE-----\n" +
                    "MIICSTCCAfWgAwIBAgIBATAKBggqhkjOPQQDAjB0MQkwBwYDVQQIEwAxCTAHBgNV\n" +
                    "BAcTADEJMAcGA1UECRMAMQkwBwYDVQQREwAxDjAMBgNVBAoTBWZsYXRvMQkwBwYD\n" +
                    "VQQLEwAxDjAMBgNVBAMTBW5vZGUyMQswCQYDVQQGEwJaSDEOMAwGA1UEKhMFZWNl\n" +
                    "cnQwIBcNMjAwNTIxMDU1MTE0WhgPMjEyMDA0MjcwNjUxMTRaMHQxCTAHBgNVBAgT\n" +
                    "ADEJMAcGA1UEBxMAMQkwBwYDVQQJEwAxCTAHBgNVBBETADEOMAwGA1UEChMFZmxh\n" +
                    "dG8xCTAHBgNVBAsTADEOMAwGA1UEAxMFbm9kZTExCzAJBgNVBAYTAlpIMQ4wDAYD\n" +
                    "VQQqEwVlY2VydDBWMBAGByqGSM49AgEGBSuBBAAKA0IABBI3ewNK21vHNOPG6U3X\n" +
                    "mKJohSNNz72QKDxUpRt0fCJHwaGYfSvY4cnqkbliclfckUTpCkFSRr4cqN6PURCF\n" +
                    "zkWjeTB3MA4GA1UdDwEB/wQEAwIChDAmBgNVHSUEHzAdBggrBgEFBQcDAgYIKwYB\n" +
                    "BQUHAwEGAioDBgOBCwEwDwYDVR0TAQH/BAUwAwEB/zANBgNVHQ4EBgQEAQIDBDAP\n" +
                    "BgNVHSMECDAGgAQBAgMEMAwGAypWAQQFZWNlcnQwCgYIKoZIzj0EAwIDQgB3Cfo8\n" +
                    "/Vdzzlz+MW+MIVuYQkcNkACY/yU/IXD1sHDGZQWcGKr4NR7FHJgsbjGpbUiCofw4\n" +
                    "4rK6biAEEAOcv1BQAA==\n" +
                    "-----END CERTIFICATE-----";

            Account ac = accountService.fromAccountJson(accountJsons[5]);
            Transaction transaction = new Transaction.
                    BVMBuilder(ac.getAddress()).
                    invoke(new CertOperation.CertBuilder().check(
                            ecert.getBytes()).build()).
                    build();
            transaction.sign(ac);
            ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();

            System.out.println(new String(ByteUtil.fromHex(receiptResponse.getRet())));
            Result result = Decoder.decodeBVM(receiptResponse.getRet());
            Assert.assertTrue(result.isSuccess());
            Assert.assertEquals("", result.getErr());
        }

.. |image0| image:: ../../images/cert on chain1.png
