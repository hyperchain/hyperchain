Hyperchain Samples
==================

In this section, we will introduce some simple examples to use
Hyperchain.

HyperCli
--------

We recommend using ``HyperCli`` for administration.

``HyperCli`` is a CLI tool for Hyperchain administration, it has
various functions. We'll introduce its contract and transaction related
functions in the following steps.

To build HyperCli:

.. code:: bash

    cd $GOPATH/src/github.com/hyperchain/hyperchain/hypercli
    govendor build

You can run ``go build`` as well.

.. note:: By default, HyperCli sends message to localhost:8081,
          so we recommend you to run HyperCli on your Hyperchain node locally.
          Otherwise you need to specify HyperCli's ``--host`` and ``--port`` parameters 
          with Hyperchain node IP and JSON-RPC port for remote execution.

Sample Contract 1 - Set/Get Hash
--------------------------------

Here is a sample contract which implements setHash and getHash
functionality.

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

Compiling Contract
~~~~~~~~~~~~~~~~~~

You can get contract's bytecode with a simple CLI command if you've
installed ``solc``. Meanwhile, you can use the following bytecode which
is the compiled result of this contract if you've not installed the
solidity compiler.

**bytecode**

.. code:: bash

    0x606060405261015c806100126000396000f3606060405260e060020a60003504633cf5040a8114610029578063d7fa10071461007b575b610002565b34610002576100ca6004356000818152602081905260408120548190819015156101055750600091507f746865206b6579206973206e6f7420657869737400000000000000000000000090508161012a565b34610002576100ea6004356024356000828152602081905260408120548190156101315750600090507f746865206b657920657869737400000000000000000000000000000000000000610155565b604080519315158452602084019290925282820152519081900360600190f35b60408051921515835260208301919091528051918290030190f35b50505060008181526020819052604090205460019060c860020a665375636365737302905b9193909250565b50506000828152602081905260409020819055600160c860020a6653756363657373025b925092905056

Assuming that your contract file named ``sample1.sol``,
you can get the bytecode with the following command:

.. code:: bash

    solc --bin sample1.sol

Deploying Contract
~~~~~~~~~~~~~~~~~~

HyperCli provides a 'contract deploy' function, here's its parameters:

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

You can specify the contract's bytecode as the value of '--payload
option' to deploy this contract, for example:

.. code:: bash

    ./hypercli contract deploy --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x606060405261015c806100126000396000f3606060405260e060020a60003504633cf5040a8114610029578063d7fa10071461007b575b610002565b34610002576100ca6004356000818152602081905260408120548190819015156101055750600091507f746865206b6579206973206e6f7420657869737400000000000000000000000090508161012a565b34610002576100ea6004356024356000828152602081905260408120548190156101315750600090507f746865206b657920657869737400000000000000000000000000000000000000610155565b604080519315158452602084019290925282820152519081900360600190f35b60408051921515835260208301919091528051918290030190f35b50505060008181526020819052604090205460019060c860020a665375636365737302905b9193909250565b50506000828152602081905260409020819055600160c860020a6653756363657373025b925092905056

This command means HyperCli deploys the contract from the account
address 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd. Though HyperCli has
some default values for its parameters, you can also specify them
explicitly.

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xb1b7d4f083ac65679ddd31a9b864fc8ca1ec75eee2f7a46cca1b223eae94527c","vmType":"EVM","contractAddress":"0xbbe2b6412ccf633222374de8958f2acc76cda9c9","gasUsed":69660,"ret":"0x606060405260e060020a60003504633cf5040a8114610029578063d7fa10071461007b575b610002565b34610002576100ca6004356000818152602081905260408120548190819015156101055750600091507f746865206b6579206973206e6f7420657869737400000000000000000000000090508161012a565b34610002576100ea6004356024356000828152602081905260408120548190156101315750600090507f746865206b657920657869737400000000000000000000000000000000000000610155565b604080519315158452602084019290925282820152519081900360600190f35b60408051921515835260208301919091528051918290030190f35b50505060008181526020819052604090205460019060c860020a665375636365737302905b9193909250565b50506000828152602081905260409020819055600160c860020a6653756363657373025b925092905056","log":[]}}

You can get the contract address from the result, in this case, it's:

.. code:: bash

    0xbbe2b6412ccf633222374de8958f2acc76cda9c9

It will be used later to invoke contract's functions.

Invoking Contract
~~~~~~~~~~~~~~~~~

HyperCli also provides a 'contract invoke' function, here's its
parameters:

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

To invoke a contract function, you need to specify at least 2 parameters
in this case: 

- payload: payload of the function call 
- to: the contract address which you got before

We've provided some function calls' payloads, you can use them directly
for invoking.

setHash
^^^^^^^

Invoke setHash function to set key1 = value1, and here's its payload:

.. code:: bash

    0xd7fa10076b6579310000000000000000000000000000000000000000000000000000000076616c7565310000000000000000000000000000000000000000000000000000

Here's the contract address you got before:

.. code:: bash

    0xbbe2b6412ccf633222374de8958f2acc76cda9c9

Now you can run the command as follows:

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0xd7fa10076b6579310000000000000000000000000000000000000000000000000000000076616c7565310000000000000000000000000000000000000000000000000000 --to 0xbbe2b6412ccf633222374de8958f2acc76cda9c9

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xa28350777a964f5ab6f4ef355131c0c241388ac6e8548c191aa5b3b94af95571","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":20477,"ret":"0x00000000000000000000000000000000000000000000000000000000000000015375636365737300000000000000000000000000000000000000000000000000","log":[]}}

getHash
^^^^^^^

Invoke getHash function to get key1, and here's its payload:

.. code:: bash

    0x3cf5040a6b65793100000000000000000000000000000000000000000000000000000000

Here's the contract address you got before:

.. code:: bash

    0xbbe2b6412ccf633222374de8958f2acc76cda9c9

Now you can run the command as follows:

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x3cf5040a6b65793100000000000000000000000000000000000000000000000000000000 --to 0xbbe2b6412ccf633222374de8958f2acc76cda9c9

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x185dba5451ace5ffcf4c11d10968e1e4ed299eb78ca6ddda65539dfca2fc56df","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":523,"ret":"0x0000000000000000000000000000000000000000000000000000000000000001537563636573730000000000000000000000000000000000000000000000000076616c7565310000000000000000000000000000000000000000000000000000","log":[]}}

Sample Contract 2 - Simulate Bank
---------------------------------

Here is another sample contract which implements asset administration.

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

Compiling Contract
~~~~~~~~~~~~~~~~~~

You can get contract's bytecode with a simple CLI command if you've
installed ``solc``. Meanwhile, you can use the following bytecode which
is the compiled result of this contract if you've not installed the
solidity compiler.

**bytecode**

.. code:: bash

    0x606060405260405160608061020083395060c06040525160805160a05160018390556002829055600380547f01000000000000000000000000000000000000000000000000000000000000008084020460ff19909116179055600080546c0100000000000000000000000033810204600160a060020a03199091161790555050506101728061008e6000396000f3606060405260e060020a60003504635e5c06e2811461003f578063867904b41461005c57806393423e9c146100a8578063beabacc8146100d1575b610002565b346100025761013760043560046020526000908152604090205481565b34610002576101496004356024356000805433600160a060020a039081169116141561015d5750600160a060020a03821660009081526004602052604090208054820190556001610161565b3461000257610137600435600160a060020a038116600090815260046020526040902054919050565b3461000257610149600435602435604435600160a060020a0383166000908152600460205260408120548290106101675750600160a060020a0380841660009081526004602052604080822080548590039055918416815220805482019055600161016b565b60408051918252519081900360200190f35b604080519115158252519081900360200190f35b5060005b92915050565b5060005b939250505056

Assuming that your contract file named ``sample2.sol``, you can get the
bytecode with the following command:

.. code:: bash

    solc --bin sample2.sol

Deploying Contract
~~~~~~~~~~~~~~~~~~

As mentioned, HyperCli provides a 'contract deploy' function, here's its
parameters:

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

Thus you can specify the contract's bytecode as the value of '--payload
option', for example:

.. code:: bash

    ./hypercli contract deploy --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x606060405260405160608061020083395060c06040525160805160a05160018390556002829055600380547f01000000000000000000000000000000000000000000000000000000000000008084020460ff19909116179055600080546c0100000000000000000000000033810204600160a060020a03199091161790555050506101728061008e6000396000f3606060405260e060020a60003504635e5c06e2811461003f578063867904b41461005c57806393423e9c146100a8578063beabacc8146100d1575b610002565b346100025761013760043560046020526000908152604090205481565b34610002576101496004356024356000805433600160a060020a039081169116141561015d5750600160a060020a03821660009081526004602052604090208054820190556001610161565b3461000257610137600435600160a060020a038116600090815260046020526040902054919050565b3461000257610149600435602435604435600160a060020a0383166000908152600460205260408120548290106101675750600160a060020a0380841660009081526004602052604080822080548590039055918416815220805482019055600161016b565b60408051918252519081900360200190f35b604080519115158252519081900360200190f35b5060005b92915050565b5060005b939250505056

This command means HyperCli deploys the contract from the account
address 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd. Though HyperCli has
some default values for its parameters, you can also specify them
explicitly.

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x6790126ca4c072f53d1684dff9e080098db931358d8eca04c833373ae580ed9e","vmType":"EVM","contractAddress":"0xbbe2b6412ccf633222374de8958f2acc76cda9c9","gasUsed":109363,"ret":"0x606060405260e060020a60003504635e5c06e2811461003f578063867904b41461005c57806393423e9c146100a8578063beabacc8146100d1575b610002565b346100025761013760043560046020526000908152604090205481565b34610002576101496004356024356000805433600160a060020a039081169116141561015d5750600160a060020a03821660009081526004602052604090208054820190556001610161565b3461000257610137600435600160a060020a038116600090815260046020526040902054919050565b3461000257610149600435602435604435600160a060020a0383166000908152600460205260408120548290106101675750600160a060020a0380841660009081526004602052604080822080548590039055918416815220805482019055600161016b565b60408051918252519081900360200190f35b604080519115158252519081900360200190f35b5060005b92915050565b5060005b939250505056","log":[]}}

You can get the contract address from the result, in this case, it's:

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

It will be used later to invoke contract's functions.

Invoking Contract
~~~~~~~~~~~~~~~~~

As mentioned, HyperCli provides a 'contract invoke' function, here's its
parameters:

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

To invoke a contract function, you need to specify at least 2 parameters
in this case: 

- payload: payload of the function call 
- to: the contract address which you got before

We've provided some function calls' payloads, you can use them directly
for invoking.

Issue Asset
^^^^^^^^^^^

Issue asset for user 0x1234567 with 1000000000, and here's its payload:

.. code:: bash

    0x867904b40000000000000000000000000000000000000000000000000000000001234567000000000000000000000000000000000000000000000000000000003b9aca00

Here's the contract address you got before:

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

Now you can run the command as follows:

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x867904b40000000000000000000000000000000000000000000000000000000001234567000000000000000000000000000000000000000000000000000000003b9aca00 --to 0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x9f602fc0c4ac12d383cf765e2931978a0e5daaec666888e242485bd6314e17f9","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":20498,"ret":"0x0000000000000000000000000000000000000000000000000000000000000001","log":[]}}

Transfer Asset
^^^^^^^^^^^^^^

Transfer from user 0x1234567 to user 0x2345678 with 1 amount, and here's
its payload:

.. code:: bash

    0xbeabacc8000000000000000000000000000000000000000000000000000000000123456700000000000000000000000000000000000000000000000000000000023456780000000000000000000000000000000000000000000000000000000000000001

Here's the contract address you got before:

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

Now you can run the command as follows:

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0xbeabacc8000000000000000000000000000000000000000000000000000000000123456700000000000000000000000000000000000000000000000000000000023456780000000000000000000000000000000000000000000000000000000000000001 --to 0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x2d779df9db29541102e98ed9996263db42da6c13a2a6ac5c7ba9c606acecba28","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":25733,"ret":"0x0000000000000000000000000000000000000000000000000000000000000001","log":[]}}

Get Balance
^^^^^^^^^^^

Get balance of user 0x1234567, and here's its payload:

.. code:: bash

    0x93423e9c0000000000000000000000000000000000000000000000000000000001234567

Here's the contract address you got before:

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

Now you can run the command as follows:

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x93423e9c0000000000000000000000000000000000000000000000000000000001234567 --to 0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xcca7b19dde84bf174243398de4107ee0b783b6e243176e162a2239bae1f475f7","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":353,"ret":"0x000000000000000000000000000000000000000000000000000000003b9ac9ff","log":[]}}

Get balance of user 0x2345678, and here's its payload:

.. code:: bash

    0x93423e9c0000000000000000000000000000000000000000000000000000000002345678

Here's the contract address you got before:

.. code:: bash

    0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

Now you can run the command as follows:

.. code:: bash

    ./hypercli contract invoke --from 000f1a7a08ccc48e5d30f80850cf1cf283aa3abd --payload 0x93423e9c0000000000000000000000000000000000000000000000000000000002345678 --to 0x1e548137be17e1a11f0642c9e22dfda64e61fe6d

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0x04b82a4fcdadcf102d559b4eb6a29030f7ef29195f40a5f9b986021b48b48552","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":353,"ret":"0x0000000000000000000000000000000000000000000000000000000000000001","log":[]}}

Transaction Delivery
--------------------

HyperCli also provides transaction related functions, and here's its
parameters:

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

There are 3 sub-commands for HyperCli tx command, we'll introduce them
with some examples as follows.

Send Transaction
~~~~~~~~~~~~~~~~

The sub-command 'send' is used for sending normal transactions, it has
some parameters as follows:

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

For example, you can use this command to send transactions, it means
send 10 transactions in a row:

.. code:: bash

    ./hypercli tx send -c 10

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xefbc9f9b5048337fbaf64047ad5eff03c40c1c76991b0364686ba3620e2c5ea3","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":0,"ret":"0x0","log":[]}}

    ...
    ...

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":0,"ret":"0x0","log":[]}}

Please select one txHash in the results, it will be used later to
execute 'info' and 'receipt' sub-commands. In this case, we select this
txHash:

.. code:: bash

    0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d

Transaction Info
~~~~~~~~~~~~~~~~

The sub-command 'info' is used for getting a transaction's information,
it has some parameters as follows:

.. code:: bash

    ./hypercli tx info --help
    NAME:
       hypercli tx info - query the transaction info by hash

    USAGE:
       hypercli tx info [command options] [arguments...]

    OPTIONS:
       --hash value                 specify the tx hash used to query the detailed information
       --namespace value, -n value  specify the namespace to query transaction information (default: "global")

For example, you can use this command to get the transaction's
information with the txHash you got before:

.. code:: bash

    ./hypercli tx info --hash 0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d

You'll see these information if HyperCli command executed properly.

.. code:: bash

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","hash":"0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d","blockNumber":"0xf","blockHash":"0x2592e0f3e1f156effe93325a60b1190533be2053aa102eb182bd64f95b28080a","txIndex":"0x0","from":"0x000f1a7a08ccc48e5d30f80850cf1cf283aa3abd","to":"0x6201cb0448964ac597faf6fdf1f472edf2a22b89","amount":"0x8","timestamp":1512022032747286761,"nonce":5475154203949975728,"extra":"","executeTime":"0x5","payload":"0x0"}}

Transaction Receipt
~~~~~~~~~~~~~~~~~~~

The sub-command 'receipt' is used for getting a transaction's receipt,
it has some parameters as follows:

.. code:: bash

    $ ./hypercli tx receipt --help
    NAME:
       hypercli tx receipt - query the transaction receipt by hash

    USAGE:
       hypercli tx receipt [command options] [arguments...]

    OPTIONS:
       --hash value                 specify the tx hash used to query the transaction receipt
       --namespace value, -n value  specify the namespace to query transaction receipt (default: "global")

For example, you can use this command to get the transaction's receipt
with the txHash you got before:

.. code:: bash

    ./hypercli tx receipt --hash 0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d

You'll see these information if HyperCli command executed properly.

.. code:: json

    {"jsonrpc":"2.0","namespace":"global","id":1,"code":0,"message":"SUCCESS","result":{"version":"1.3","txHash":"0xc47b64ddad2be542bfdc5164d447317f5152142ac7961c88332b25f04b31783d","vmType":"EVM","contractAddress":"0x0000000000000000000000000000000000000000","gasUsed":0,"ret":"0x0","log":[]}}
