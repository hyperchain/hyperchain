Hyperchain 使用示例
===================

在这一节中，我们将会用一些例子，来介绍一下Hyperchain平台的简单使用方法。

HyperCli
--------

我们推荐使用\ ``HyperCli``\ 作为您的管理工具。

``HyperCli``
是一个基于Hyperchain平台开发的命令行工具，方便管理员操作Hyperchain平台。
``HyperCli``\ 提供了丰富的命令行接口，下面我们会介绍它的智能合约、交易处理等相关功能。

编译HyperCli:

.. code:: bash

    cd $GOPATH/src/github.com/hyperchain/hyperchain/hypercli
    govendor build

您也可以执行 ``go build``\ 来编译。

.. note:: 默认情况下，HyperCli是给localhost:8081发送消息的，
          所以我们建议您在Hyperchain节点服务器上运行HyperCli。
          如果您希望在本地执行HyperCli，那么就需要指定 ``--host``\ 和 ``--port``\ 选项
          来设置远程Hyperchain节点的地址和JSON-RPC端口。

合约样例1 - Set/Get Hash
------------------------

这里是一个实现setHash和getHash功能的智能合约。

.. code:: javascript

    contract Anchor{
        mapping(bytes32 => bytes32) hashMap;

        function setHash(bytes32 key,bytes32 value) returns(bool,bytes32){
            if(hashMap[key] != 0x0){
                return (false,"the key exist");
            }   
            hashMap[key] = value;
            return (true,"Success");
        }   

        function getHash(bytes32 key) returns(bool,bytes32,bytes32){
            if(hashMap[key] == 0x0){
                return (false,"the key is not exist",0x0);
            }   
            return (true,"Success",hashMap[key]);
        }   
    }

编译合约
~~~~~~~~

如果您已经安装了\ ``solc``\ ，您只需通过一个简单的命令就能得到合约的字节码。如果您没有安装编译器，也可以直接使用以下的字节码，该字节码是本合约的编译结果。

**字节码**

.. code:: bash

    0x606060405261015c806100126000396000f3606060405260e060020a60003504633cf5040a8114610029578063d7fa10071461007b575b610002565b34610002576100ca6004356000818152602081905260408120548190819015156101055750600091507f746865206b6579206973206e6f7420657869737400000000000000000000000090508161012a565b34610002576100ea6004356024356000828152602081905260408120548190156101315750600090507f746865206b657920657869737400000000000000000000000000000000000000610155565b604080519315158452602084019290925282820152519081900360600190f35b60408051921515835260208301919091528051918290030190f35b50505060008181526020819052604090205460019060c860020a665375636365737302905b9193909250565b50506000828152602081905260409020819055600160c860020a6653756363657373025b925092905056

假设您的智能合约文件名为\ ``sample1.sol``,
您可以通过以下命令获得合约字节码：

.. code:: bash

    solc --bin sample1.sol

部署合约
~~~~~~~~

HyperCli提供了一个合约部署的功能，以下是该功能的参数：

.. code:: bash

    $ ./hypercli contract deploy --help

    NAME:
       hypercli contract deploy - Deploy a contract

    USAGE:
       hypercli contract deploy [command options] [arguments...]

    OPTIONS:
       --namespace value, -n value  specify the namespace, default to global (default: "global")
       --from value, -f value       specify the account (default: "000f1a7a08ccc48e5d30f80850cf1cf283aa3abd")
       --payload value, -p value    specify the contract payload
       --extra value, -e value      specify the extra information
       --simulate, -s               simulate execute or not, default to false
       --directory value, -d value  specify the contract file directory

您可以将合约的字节码作为\ ``--payload``
选项的值，使用以下的命令来部署合约：

.. code:: bash

    ./hypercli contract deploy --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x606060405261015c806100126000396000f3606060405260e060020a60003504633cf5040a8114610029578063d7fa10071461007b575b610002565b34610002576100ca6004356000818152602081905260408120548190819015156101055750600091507f746865206b6579206973206e6f7420657869737400000000000000000000000090508161012a565b34610002576100ea6004356024356000828152602081905260408120548190156101315750600090507f746865206b657920657869737400000000000000000000000000000000000000610155565b604080519315158452602084019290925282820152519081900360600190f35b60408051921515835260208301919091528051918290030190f35b50505060008181526020819052604090205460019060c860020a665375636365737302905b9193909250565b50506000828152602081905260409020819055600160c860020a6653756363657373025b925092905056

该命令的意思是HyperCli使用地址为000f1a7a08ccc48e5d30f80850cf1cf283aa3abd的账户来部署合约。

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xb1b7d4f083ac65679ddd31a9b864fc8ca1ec75eee2f7a46cca1b223eae94527c","vmType":"EVM","contractAddress":"0xbbe2b6412ccf633222374de8958f2acc76cda9c9","gasUsed":69660,"ret":"0x606060405260e060020a60003504633cf5040a8114610029578063d7fa10071461007b575b610002565b34610002576100ca6004356000818152602081905260408120548190819015156101055750600091507f746865206b6579206973206e6f7420657869737400000000000000000000000090508161012a565b34610002576100ea6004356024356000828152602081905260408120548190156101315750600090507f746865206b657920657869737400000000000000000000000000000000000000610155565b604080519315158452602084019290925282820152519081900360600190f35b60408051921515835260208301919091528051918290030190f35b50505060008181526020819052604090205460019060c860020a665375636365737302905b9193909250565b50506000828152602081905260409020819055600160c860020a6653756363657373025b925092905056","log":[]}}

从结果中可以得到部署后的合约地址，在这个例子中，合约地址为：

.. code:: bash

    0xbbe2b6412ccf633222374de8958f2acc76cda9c9

之后它将会被用于合约调用的操作中。

调用合约
~~~~~~~~

HyperCli提供了一个合约调用的功能，以下是该功能的参数：

.. code:: bash

    $ ./hypercli contract invoke --help

    NAME:
       hypercli contract invoke - Invoke a contract

    USAGE:
       hypercli contract invoke [command options] [arguments...]

    OPTIONS:
       --namespace value, -n value  specify the namespace, default to global (default: "global")
       --from value, -f value       specify the account (default: "000f1a7a08ccc48e5d30f80850cf1cf283aa3abd")
       --payload value, -p value    specify the contract payload
       --to value, -t value         specify the contract address
       --extra value, -e value      specify the extra information
       --simulate, -s               simulate execute or not, default to false
       --args value, -a value       specify the args of invoke contract

在本例中，您至少需要指定两个参数的值，才能调用合约中的函数： 

- payload选项: 函数调用的字节码
- to选项: 合约地址

我们提供了一些函数调用的字节码，您可以直接使用它们。

setHash
^^^^^^^

调用setHash函数，设置key1 = value1, 以下是该调用的字节码：

.. code:: bash

    0xd7fa10076b6579310000000000000000000000000000000000000000000000000000000076616c7565310000000000000000000000000000000000000000000000000000

以下是该合约的地址：

.. code:: bash

    0xbbe2b6412ccf633222374de8958f2acc76cda9c9

您可以通过以下命令调用合约：

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0xd7fa10076b6579310000000000000000000000000000000000000000000000000000000076616c7565310000000000000000000000000000000000000000000000000000 --to 0xbbe2b6412ccf633222374de8958f2acc76cda9c9

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xa28350777a964f5ab6f4ef355131c0c241388ac6e8548c191aa5b3b94af95571","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":20477,"ret":"0x00000000000000000000000000000000000000000000000000000000000000015375636365737300000000000000000000000000000000000000000000000000","log":[]}}

getHash
^^^^^^^

调用getHash函数，获得key1对应的value值， 以下是该调用的字节码：

.. code:: bash

    0x3cf5040a6b65793100000000000000000000000000000000000000000000000000000000

以下是该合约的地址：

.. code:: bash

    0xbbe2b6412ccf633222374de8958f2acc76cda9c9

您可以通过以下命令调用合约：

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x3cf5040a6b65793100000000000000000000000000000000000000000000000000000000 --to 0xbbe2b6412ccf633222374de8958f2acc76cda9c9

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x185dba5451ace5ffcf4c11d10968e1e4ed299eb78ca6ddda65539dfca2fc56df","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":523,"ret":"0x0000000000000000000000000000000000000000000000000000000000000001537563636573730000000000000000000000000000000000000000000000000076616c7565310000000000000000000000000000000000000000000000000000","log":[]}}

合约样例2 - Simulate Bank
-------------------------

这里是另一个实现了资产管理的合约示例。

.. code:: javascript

    contract SimulateBank{
        address owner;
        bytes32 bankName;
        uint bankNum;
        bool isInvalid;
        mapping(address => uint) public accounts;
        function SimulateBank( bytes32 _bankName,uint _bankNum,bool _isInvalid){
            bankName = _bankName;
            bankNum = _bankNum;
            isInvalid = _isInvalid;
            owner = msg.sender;
        }   
        function issue(address addr,uint number) returns (bool){
            if(msg.sender==owner){
                accounts[addr] = accounts[addr] + number;
                return true;
            }   
            return false;
        }   
        function transfer(address addr1,address addr2,uint amount) returns (bool){
            if(accounts[addr1] >= amount){
                accounts[addr1] = accounts[addr1] - amount;
                accounts[addr2] = accounts[addr2] + amount;
                return true;
            }   
            return false;
        }   
        function getAccountBalance(address addr) returns(uint){
            return accounts[addr];
        }   
    }

编译合约
~~~~~~~~

如果您已经安装了\ ``solc``\ ，您只需通过一个简单的命令就能得到合约的字节码。如果您没有安装编译器，也可以直接使用以下的字节码，该字节码是本合约的编译结果。

**字节码**

.. code:: bash

    0x606060405260405160608061020083395060c06040525160805160a05160018390556002829055600380547f01000000000000000000000000000000000000000000000000000000000000008084020460ff19909116179055600080546c0100000000000000000000000033810204600160a060020a03199091161790555050506101728061008e6000396000f3606060405260e060020a60003504635e5c06e2811461003f578063867904b41461005c57806393423e9c146100a8578063beabacc8146100d1575b610002565b346100025761013760043560046020526000908152604090205481565b34610002576101496004356024356000805433600160a060020a039081169116141561015d5750600160a060020a03821660009081526004602052604090208054820190556001610161565b3461000257610137600435600160a060020a038116600090815260046020526040902054919050565b3461000257610149600435602435604435600160a060020a0383166000908152600460205260408120548290106101675750600160a060020a0380841660009081526004602052604080822080548590039055918416815220805482019055600161016b565b60408051918252519081900360200190f35b604080519115158252519081900360200190f35b5060005b92915050565b5060005b939250505056

假设您的智能合约文件名为\ ``sample2.sol``,
您可以通过以下命令获得合约字节码：

.. code:: bash

    solc --bin sample2.sol

.. 部署合约-1:

部署合约
~~~~~~~~

HyperCli提供了一个合约部署的功能，以下是该功能的参数：

.. code:: bash

    $ ./hypercli contract deploy --help

    NAME:
       hypercli contract deploy - Deploy a contract

    USAGE:
       hypercli contract deploy [command options] [arguments...]

    OPTIONS:
       --namespace value, -n value  specify the namespace, default to global (default: "global")
       --from value, -f value       specify the account (default: "000f1a7a08ccc48e5d30f80850cf1cf283aa3abd")
       --payload value, -p value    specify the contract payload
       --extra value, -e value      specify the extra information
       --simulate, -s               simulate execute or not, default to false
       --directory value, -d value  specify the contract file directory

您可以将合约的字节码作为\ ``--payload``
选项的值，使用以下的命令来部署合约：

.. code:: bash

    ./hypercli contract deploy --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x606060405260405160608061020083395060c06040525160805160a05160018390556002829055600380547f01000000000000000000000000000000000000000000000000000000000000008084020460ff19909116179055600080546c0100000000000000000000000033810204600160a060020a03199091161790555050506101728061008e6000396000f3606060405260e060020a60003504635e5c06e2811461003f578063867904b41461005c57806393423e9c146100a8578063beabacc8146100d1575b610002565b346100025761013760043560046020526000908152604090205481565b34610002576101496004356024356000805433600160a060020a039081169116141561015d5750600160a060020a03821660009081526004602052604090208054820190556001610161565b3461000257610137600435600160a060020a038116600090815260046020526040902054919050565b3461000257610149600435602435604435600160a060020a0383166000908152600460205260408120548290106101675750600160a060020a0380841660009081526004602052604080822080548590039055918416815220805482019055600161016b565b60408051918252519081900360200190f35b604080519115158252519081900360200190f35b5060005b92915050565b5060005b939250505056

该命令的意思是HyperCli使用地址为000f1a7a08ccc48e5d30f80850cf1cf283aa3abd的账户来部署合约。

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x6790126ca4c072f53d1684dff9e080098db931358d8eca04c833373ae580ed9e","vmType":"EVM","contractAddress":"0xbbe2b6412ccf633222374de8958f2acc76cda9c9","gasUsed":109363,"ret":"0x606060405260e060020a60003504635e5c06e2811461003f578063867904b41461005c57806393423e9c146100a8578063beabacc8146100d1575b610002565b346100025761013760043560046020526000908152604090205481565b34610002576101496004356024356000805433600160a060020a039081169116141561015d5750600160a060020a03821660009081526004602052604090208054820190556001610161565b3461000257610137600435600160a060020a038116600090815260046020526040902054919050565b3461000257610149600435602435604435600160a060020a0383166000908152600460205260408120548290106101675750600160a060020a0380841660009081526004602052604080822080548590039055918416815220805482019055600161016b565b60408051918252519081900360200190f35b604080519115158252519081900360200190f35b5060005b92915050565b5060005b939250505056","log":[]}}

从结果中可以得到部署后的合约地址，在这个例子中，合约地址为：

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

之后它将会被用于合约调用的操作中。

.. 调用合约-1:

调用合约
~~~~~~~~

HyperCli提供了一个合约调用的功能，以下是该功能的参数：

.. code:: bash

    $ ./hypercli contract invoke --help

    NAME:
       hypercli contract invoke - Invoke a contract

    USAGE:
       hypercli contract invoke [command options] [arguments...]

    OPTIONS:
       --namespace value, -n value  specify the namespace, default to global (default: "global")
       --from value, -f value       specify the account (default: "000f1a7a08ccc48e5d30f80850cf1cf283aa3abd")
       --payload value, -p value    specify the contract payload
       --to value, -t value         specify the contract address
       --extra value, -e value      specify the extra information
       --simulate, -s               simulate execute or not, default to false
       --args value, -a value       specify the args of invoke contract

在本例中，您至少需要指定两个参数的值，才能调用合约中的函数：

- payload选项: 函数调用的字节码 
- to选项: 合约地址

我们提供了一些函数调用的字节码，您可以直接使用它们。

发布资产
^^^^^^^^

调用issue函数，给账户0x1234567发布资产1,000,000,000，以下是该调用的字节码：

.. code:: bash

    0x867904b40000000000000000000000000000000000000000000000000000000001234567000000000000000000000000000000000000000000000000000000003b9aca00

以下是该合约的地址：

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

您可以通过以下命令调用合约：

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x867904b40000000000000000000000000000000000000000000000000000000001234567000000000000000000000000000000000000000000000000000000003b9aca00 --to 0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x9f602fc0c4ac12d383cf765e2931978a0e5daaec666888e242485bd6314e17f9","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":20498,"ret":"0x0000000000000000000000000000000000000000000000000000000000000001","log":[]}}

交易资产
^^^^^^^^

调用transfer函数，从账户0x1234567向账户0x2345678转移1个资产，以下是该调用的字节码：

.. code:: bash

    0xbeabacc8000000000000000000000000000000000000000000000000000000000123456700000000000000000000000000000000000000000000000000000000023456780000000000000000000000000000000000000000000000000000000000000001

以下是该合约的地址：

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

您可以通过以下命令调用合约：

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0xbeabacc8000000000000000000000000000000000000000000000000000000000123456700000000000000000000000000000000000000000000000000000000023456780000000000000000000000000000000000000000000000000000000000000001 --to 0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x2d779df9db29541102e98ed9996263db42da6c13a2a6ac5c7ba9c606acecba28","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":25733,"ret":"0x0000000000000000000000000000000000000000000000000000000000000001","log":[]}}

查询资产
^^^^^^^^

调用getAccountBalance函数，查询账户0x1234567的资产，以下是该调用的字节码：

.. code:: bash

    0x93423e9c0000000000000000000000000000000000000000000000000000000001234567

以下是该合约的地址：

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

您可以通过以下命令调用合约：

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x93423e9c0000000000000000000000000000000000000000000000000000000001234567 --to 0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xcca7b19dde84bf174243398de4107ee0b783b6e243176e162a2239bae1f475f7","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":353,"ret":"0x000000000000000000000000000000000000000000000000000000003b9ac9ff","log":[]}}

调用getAccountBalance函数，查询账户0x2345678的资产，以下是该调用的字节码：

.. code:: bash

    0x93423e9c0000000000000000000000000000000000000000000000000000000002345678

以下是该合约的地址：

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

您可以通过以下命令调用合约：

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x93423e9c0000000000000000000000000000000000000000000000000000000002345678 --to 0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x04b82a4fcdadcf102d559b4eb6a29030f7ef29195f40a5f9b986021b48b48552","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":353,"ret":"0x0000000000000000000000000000000000000000000000000000000000000001","log":[]}}

交易处理
--------

HyperCli提供了交易处理相关的功能，以下是该功能的参数：

.. code:: bash

    $ ./hypercli tx --help
    NAME:
       hypercli tx - transaction related commands

    USAGE:
       hypercli tx command [command options] [arguments...]

    COMMANDS:
         send     send normal transactions
         info     query the transaction info by hash
         receipt  query the transaction receipt by hash

    OPTIONS:
       --help, -h  show help

可以看到tx命令有三个子命令，下面我们将介绍这三个命令。

发送交易
~~~~~~~~

send命令可以用来发送普通的交易，以下是它的参数：

.. code:: bash

    $ ./hypercli tx send --help
    NAME:
       hypercli tx send - send normal transactions

    USAGE:
       hypercli tx send [command options] [arguments...]

    OPTIONS:
       --count value, -c value      send how many transactions (default: 1)
       --namespace value, -n value  specify the namespace to send transactions to (default: "global")
       --from value, -f value       specify the account (default: "000f1a7a08ccc48e5d30f80850cf1cf283aa3abd")
       --to value, -t value         specify the contract address
       --password value, -p value   specify the password used to generate signature (default: "123")
       --amount value, -a value     specify the amount to transfer (default: 0)
       --extra value, -e value      specify the extra information
       --snapshot value, -s value   specify the snapshot ID
       --simulate                   simulate execute or not

您可以用以下命令发送交易，它的意思是连续发送10笔交易：

.. code:: bash

    ./hypercli tx send -c 10

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xefbc9f9b5048337fbaf64047ad5eff03c40c1c76991b0364686ba3620e2c5ea3","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":0,"ret":"0x0","log":[]}}

    ...
    ...

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":0,"ret":"0x0","log":[]}}

选取结果中的一条txHash，它之后将被用于查看交易的信息和回执。在这个例子中，我们选取这一个txHash：

.. code:: bash

    0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d

交易信息
~~~~~~~~

info命令可以用来获取交易的信息，以下是它的参数：

.. code:: bash

    ./hypercli tx info --help
    NAME:
       hypercli tx info - query the transaction info by hash

    USAGE:
       hypercli tx info [command options] [arguments...]

    OPTIONS:
       --hash value                 specify the tx hash used to query the detailed information
       --namespace value, -n value  specify the namespace to query transaction information (default: "global")

您可以用以下命令获取交易信息，它的意思是获取某一笔指定txHash的交易信息：

.. code:: bash

    ./hypercli tx info --hash 0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d

如果命令执行正确，您将看到以下输出结果：

.. code:: bash

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","hash":"0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d","blockNumber":"0xf","blockHash":"0x2592e0f3e1f156effe93325a60b1190533be2053aa102eb182bd64f95b28080a","txIndex":"0x0","from":"0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd","to":"0x6201cb0448964ac597faf6fdf1f472edf2a22b89","amount":"0x8","timestamp":1512022032747286761,"nonce":5475154203949975728,"extra":"","executeTime":"0x5","payload":"0x0"}}

交易回执
~~~~~~~~

receipt命令可以用来获取交易的回执，以下是它的参数：

.. code:: bash

    $ ./hypercli tx receipt --help
    NAME:
       hypercli tx receipt - query the transaction receipt by hash

    USAGE:
       hypercli tx receipt [command options] [arguments...]

    OPTIONS:
       --hash value                 specify the tx hash used to query the transaction receipt
       --namespace value, -n value  specify the namespace to query transaction receipt (default: "global")

您可以用以下命令获取交易回执，它的意思是获取某一笔指定txHash的交易回执：

.. code:: bash

    ./hypercli tx receipt --hash 0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d

如果命令执行正确，您将看到以下输出结果：

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":0,"ret":"0x0","log":[]}}
