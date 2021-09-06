.. _account-life-manage:

账户生命周期管理
^^^^^^^^^^^^^^^^^^^^^^^^

账户生命周期管理
------------------

功能描述
>>>>>>>>>>>>>>>>>>>>>

账户生命周期管理提供账户注册和注销的功能，另外提供了查询账户状态的接口。（本文描述的账户指用户账户，非合约账户）。

账户注册
::::::::::::::::::::

账户注册接收两个参数，一个参数为注册的账户地址address，一个参数为账户对应的idcert，用于注册证书与账户的对应关系，只有链级管理员账户才有权限进行此操作。链级管理员的相关说明参考 :ref:`链级角色管理 <chain-role-manage>`

注册时使用的证书对应的地址需要为账户地址。如果用于注册的地址在注册前在链上已经存在（作为交易的from或to），则对账户地址没有要求。如果用于注册的地址在注册前在链上不存在，则需要满足，注册的地址以 `0xffffffffff` 开头。

账户注销
::::::::::::::::::::

账户注销接收两个参数，一个参数为要注销的账户地址address，一个参数为注销账户时使用的sdkcert，用于注销账户地址，只有链级管理员账户才有权限进行此操作。

注销时，被注销的账户需要为证书账户，即注册了证书与此账户的对应关系且没有被注销，使用的sdkcert需要与被注销的账户地址对应的证书是同一ca机构签发的。

查询账户状态
:::::::::::::::::::::::

用户可通过提供的查询账户状态的接口查询账户的状态。账户可能的状态如下：

- normal 即普通账户（没有注册证书与账户对应关系的账户）

- cert 即证书账户（注册了证书与账户对应关系且没有被注销的账户）

- abandon 即注销账户（注册了证书与账户对应关系且被注销的账户）

litesdk使用说明
>>>>>>>>>>>>>>>>>>>>>>>>>

账户生命周期管理接口
:::::::::::::::::::::::::

在litesdk中，提供了构造账户生命周期管理（注册和注销）操作的构造器 `AccountBuilder` ，其中提供了 `register` 和 `abandon` 方法，分别用于构造 `AccountContract` 合约中的 `Register` 和 `Abandon` 方法，其定义如下::

    public static class AccountBuilder extends BuiltinOperationBuilder {
         /**
         * create AccountBuilder to register.
         * @param address register address
         * @param cert register cert
         * @return {@link AccountBuilder}
         */
        public AccountBuilder register(String address, String cert);
  
         /**
         * create AccountBuilder to abandon.
         * @param address abandon address
         * @param sdkCert the used sdkCert to logout
         * @return {@link AccountBuilder}
         */
        public AccountBuilder abandon(String address, String sdkCert);
    }

账户生命周期管理的操作创建好之后，使用 `BVMBuilder` 提供的 `invoke` 方法构造bvm的交易体，使用 `build` 方法构造出交易 `transaction` ，并为交易设置 `txVersion` 并使用 `sign` 方法签名，得到最终可以发送执行的交易体。

账户状态查询接口
:::::::::::::::::::::::::::

在litesdk的AccountService中提供了getStatus接口，用于查询账户状态，其接口如下::

    public interface AccountService {
        Request<StatusResponse> getStatus(String address, int... nodeIds);
    }

`getStatus` 方法可以查询普通账户的状态，需要传一个普通 **账户地址** 为参数。

litesdk使用
>>>>>>>>>>>>>>>>>>>>>>

账户注册
::::::::::::::::::

账户注册方法接收两个参数，一个参数为注册的账户地址address，一个参数为账户对应的idcert，用于注册证书与账户的对应关系。。使用 `HashBuilder` 提供的 `set` 方法构造一个 `BuiltinOperation` ，然后使用 `BVMBuilder` 提供的 `invoke` 方法设置参数，使用 `build` 方法构造 `Transaction` ，然后使用 `ContractService` 提供的 `invoke` 方法构造请求，最后将请求发出拿到响应结果，其示例如下::

    public void testRegister() throws RequestException {
        String address = "0xffffffffffbf7e0de2f5a0dc1917f0552aa43d87";
        String cert = "-----BEGIN CERTIFICATE-----\n" +
                "MIICVjCCAgKgAwIBAgIIcy8/n1XOqQQwCgYIKoZIzj0EAwIwdDEJMAcGA1UECBMA\n" +
                "MQkwBwYDVQQHEwAxCTAHBgNVBAkTADEJMAcGA1UEERMAMQ4wDAYDVQQKEwVmbGF0\n" +
                "bzEJMAcGA1UECxMAMQ4wDAYDVQQDEwVub2RlMTELMAkGA1UEBhMCWkgxDjAMBgNV\n" +
                "BCoTBWVjZXJ0MB4XDTIwMTExMjAwMDAwMFoXDTIxMTAyMTAwMDAwMFowYTELMAkG\n" +
                "A1UEBhMCQ04xDjAMBgNVBAoTBWZsYXRvMTEwLwYDVQQDEyhmZmZmZmZmZmZmYmY3\n" +
                "ZTBkZTJmNWEwZGMxOTE3ZjA1NTJhYTQzZDg3MQ8wDQYDVQQqEwZpZGNlcnQwVjAQ\n" +
                "BgcqhkjOPQIBBgUrgQQACgNCAAQYF3xQqTY5Hr9f8I65BLCKOxuR9U+39HDqF6ba\n" +
                "/G2vTjGFDbOw/LXVIPk+GNrife1EDtvpBtQi2b9G0o+fxrzoo4GTMIGQMA4GA1Ud\n" +
                "DwEB/wQEAwIB7jAxBgNVHSUEKjAoBggrBgEFBQcDAgYIKwYBBQUHAwEGCCsGAQUF\n" +
                "BwMDBggrBgEFBQcDBDAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBTzl9gKHb19nd5m\n" +
                "rRnyavoaiQQrJzAPBgNVHSMECDAGgAQBAgMEMA0GAypWAQQGaWRjZXJ0MAoGCCqG\n" +
                "SM49BAMCA0IAon0Hym4rdLsZQnioh38SPrpQV66c9aWoBN4T9eYH8Nlouxi9C6Od\n" +
                "OqdJnRWkgUVw/kA+egZTzx0Bm/yF/VNgYAE=\n" +
                "-----END CERTIFICATE-----\n";
        Account ac = accountService.fromAccountJson(accountJsons[5]);
        Transaction transaction = new Transaction.
                BVMBuilder(ac.getAddress()).
                invoke(new AccountOperation.AccountBuilder().register(address, cert).build()).
                build();
        transaction.sign(ac);
        ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
        Result result = Decoder.decodeBVM(receiptResponse.getRet());
        System.out.println(result);
        Assert.assertTrue(result.isSuccess());
        Assert.assertEquals("", result.getErr());
    }


账户注销
::::::::::::::::

账户注销方法接收两个参数，一个参数为要注销的账户地址address，一个参数为注销账户时使用的sdkcert，用于注销账户地址。其示例如下::

    public void testAbandon() throws RequestException {
       String address = "0xffffffffffbf7e0de2f5a0dc1917f0552aa43d87";
       String sdkCert = "-----BEGIN CERTIFICATE-----\n" +
               "MIICVjCCAgKgAwIBAgIIQjE4PWfTGPAwCgYIKoZIzj0EAwIwdDEJMAcGA1UECBMA\n" +
               "MQkwBwYDVQQHEwAxCTAHBgNVBAkTADEJMAcGA1UEERMAMQ4wDAYDVQQKEwVmbGF0\n" +
               "bzEJMAcGA1UECxMAMQ4wDAYDVQQDEwVub2RlMTELMAkGA1UEBhMCWkgxDjAMBgNV\n" +
               "BCoTBWVjZXJ0MB4XDTIwMTAxNjAwMDAwMFoXDTIwMTAxNjAwMDAwMFowYjELMAkG\n" +
               "A1UEBhMCQ04xDjAMBgNVBAoTBWZsYXRvMTMwMQYDVQQDEyoweDk2MzkxNTIxNTBk\n" +
               "ZjkxMDVjMTRhZTM1M2M3YzdlNGQ1ZTU2YTAxYTMxDjAMBgNVBCoTBWVjZXJ0MFYw\n" +
               "EAYHKoZIzj0CAQYFK4EEAAoDQgAEial3WRUmVgLeB+Oi8R/FQDtpp4egSGnQ007x\n" +
               "M4uDHTIqlQmz6VAe4d2caMIXREecbYTkAK4HNR6y7A54ISc9pqOBkjCBjzAOBgNV\n" +
               "HQ8BAf8EBAMCAe4wMQYDVR0lBCowKAYIKwYBBQUHAwIGCCsGAQUFBwMBBggrBgEF\n" +
               "BQcDAwYIKwYBBQUHAwQwDAYDVR0TAQH/BAIwADAdBgNVHQ4EFgQU+7HuCW+CEqcP\n" +
               "UbcUJ2Ad5evjrIswDwYDVR0jBAgwBoAEAQIDBDAMBgMqVgEEBWVjZXJ0MAoGCCqG\n" +
               "SM49BAMCA0IA7aV3A20YOObn+H72ksXcUHx8PdC0z/rULhes2uFiINsqEPkGkaH9\n" +
               "HjBiP8uYn4YLtYVZ5pdmfoTHa7/CjVyOUwA=\n" +
               "-----END CERTIFICATE-----";
       Account ac = accountService.fromAccountJson(accountJsons[5]);
       Transaction transaction = new Transaction.
               BVMBuilder(ac.getAddress()).
               invoke(new AccountOperation.AccountBuilder().abandon(address, sdkCert).build()).
               build();
       transaction.sign(ac);
       ReceiptResponse receiptResponse = contractService.invoke(transaction).send().polling();
       Result result = Decoder.decodeBVM(receiptResponse.getRet());
       System.out.println(result);
       Assert.assertTrue(result.isSuccess());
       Assert.assertEquals("", result.getErr());
    }

查询账户状态
:::::::::::::::::::

查询账户状态接收一个参数，为账户地址，其示例如下::

    public void testGetStatus() throws RequestException {
        String address = "0x37a1100567bf7e0de2f5a0dc1917f0552aa43d88";
        Request<StatusResponse> balance = accountService.getStatus(address);
        StatusResponse send = balance.send();
        System.out.println(send.getStatus());
    }
