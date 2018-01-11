智能合约
========

1. 智能合约简介
---------------

.. Note::

    智能合约是指部署在区块链上的一段可以自动执行条款的计算机程序。智能合约能够根据外界输入信息自动执行预先定义好的协议并完成区块链内部相关状态的转移。

广泛意义上的智能合约还包括智能合约编程语言、编译器、虚拟机、事件、状态机、容错机制等。其中对智能合约影响较大的是智能合约编程语言以及其执行引擎。智能合约虚拟机一般为了安全起见是作为沙箱被封装起来，整个执行环境完全被隔离。虚拟机内部执行的智能合约不允许接触网络、文件系统、进程线程等系统资源。不同智能合约的安全性等级、表达的丰富性有所不同，Hyperchain系统自主研发智能合约的执行引擎HyperVM是一种通用智能合约引擎设计，允许多种不同智能合约引擎接入。目前的实现了兼容Ethereum的Solidity语言的HyperEVM和支持Java语言的智能合约引擎HyperJVM.

2. 智能合约引擎HyperVM
----------------------

HyperVM
是Hyperchain自主研发的可插拔智能合约引擎通用框架，允许不同智能合约执行引擎接入。如下图所示是HyperVM的架构示意图，HyperVM的架构图提供了智能合约编译、执行等相关主要组件。其中，Compiler提供了智能合约编译相关功能，Interpreter和Executor则提供了智能合约解释以及执行相关功能，State组件赋予智能合约操作区块链账本的相关功能。Guard模块提供智能合约安全保障相关机制。

.. image:: ../../images/hypervm.png

2.1 HyperEVM
~~~~~~~~~~~~

为了最大程度利用开源社区在智能合约技术方面的研究和积累，提高智能合约的可重用性以及兼容性。HyperEVM的实现采用了完全兼容Ethereum的智能合约规范，使用Solidity作为智能合约开发语言，底层使用了优化了的Ethereum虚拟机EVM。
如下图所示为HyperEVM智能合约执行流程图:

.. figure:: ../../images/hyperevm-flow.png
   :alt: hyperevm-flow

   hyperevm-flow

HyperEVM执行一次交易之后会返回一个执行结果，系统将其保存在被称为交易回执的变量中，之后平台客户端可以根据本次的交易哈希进行交易结果的查询。HyperEVM的执行流程如下：

1. HyperEVM接收到上层传递的transaction，并进行初步的验证；
2. 判断transaction类型，如果是部署合约则执行3，否则执行4；
3. HyperEVM新建一个合约账户来存储合约地址以及合约编译之后的代码；
4. HyperEVM解析transaction中的交易参数等信息，并调用其执行引擎执行相应的智能合约字节码；
5. 指令执行完成之后，HyperEVM会判断其是否停机，否的话跳转步骤2，否则执行步骤6;
6. 判断HyperEVM的停机状态是否正常，正常则结束执行，否则执行步骤7；
7. 进行Undo操作，状态回滚到本次交易执行之前。

执行指令集模块是HyperEVM执行模块的核心，指令的执行模块有两种实现，分别是基于字节码的执行以及更加复杂高效的即时编译（Just-in-time
compilation）。
字节码执行的方式比较简单，HyperEVM实现的虚拟机会有指令执行单元。该指令执行单元会一直尝试执行指令集，当指定时间未执行完成，虚拟机会中断计算逻辑，返回超时错误信息，以此防止智能合约中的恶意代码执行。

JIT方式的执行相对复杂，即时编译也称为及时编译、实时编译，是动态编译的一种形式，是一种提高程序运行效率的方法。通常，程序有两种运行方式：静态编译与动态直译。静态编译的程序在执行前全部被翻译为机器码，而直译执行的则是边翻译边执行。即时编译器则混合了这二者，一句一句编译源代码，但是会将翻译过的代码缓存起来以降低性能损耗。相对于静态编译代码，即时编译的代码可以处理延迟绑定并增强安全性。JIT模式执行智能合约主要包含以下步骤：

1. 将所有同智能合约相关的信息封装在合约对象中，然后会通过该代码的哈希值去查找该合约对象是否已经存储编译。合约对象有四个常见状态，即：合约未知，合约已编译，合约准备好通过JIT执行，合约错误。
2. 如果合约状态是合约准备好通过JIT执行，则HyperEVM会选择JIT执行器来执行该合约。执行过程中虚拟机将会对编译好的智能合约进一步编译成机器码并对push、jump等指令进行深度优化。
3. 如果合约状态是合约未知的情况下，HyperEVM首先需要检查虚拟机是否强制JIT执行，如果是则顺序编译并通过JTI的指令进行执行。否则，开启单独线程进行编译，当前程序仍然通过普通的字节码编译。下次虚拟机执行过程中再次遇到相同编码的合约时，虚拟机会直接选择经过优化的合约。这样合约的指令集由于经过了优化，该合约的执行和部署的效率能够获得较大的提高。

3. 智能合约使用
---------------

3.1 基于Solidity智能合约案例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

编写合约
~~~~~~~~

基于Solidity的智能合约同基于JS的程序类似，由一系列变量和相关函数组成。如下所示为模拟简单累加器功能的智能合约。我们以此为例简单介绍基于Solidity智能合约的基本组成部分。

.. code:: js

    contract Accumulator{    
        uint32 sum = 0;   
        function increment(){
            sum = sum + 1;     
        }    

        function getSum() returns(uint32){
            return sum;    
        }   

        function add(uint32 num1,uint32 num2) {
            sum = sum+num1+num2;     
        }
    }

Accumulator合约说明：

-  基于solidity的智能合约以关键字\ ``contract``\ 开头，类似Java等语言中的class关键字；
-  合约内部可以有变量和函数，上述sum
   为uint32类型的简单变量，Solidity智能合约还支持map等集合类型；
-  合约允许定义执行函数以\ ``function``\ 关键字定义；

基于Solidity语言编写智能合约的详细规范参考\ `Solidiy官方网站 <https://solidity.readthedocs.io/en/develop/>`__

编译合约
~~~~~~~~

Hyperchain的智能合约的编译既可以采用Solidity官方编译器编译，也可以使用Hyperchain提供的智能合约部署JSON-RPC的接口进行编译（这种场景需要在安装Hyperchain的宿主机安装Solidity编译器sloc）。

调用Hyperchain编译Solidity智能合约命令如下：

.. code:: js

    curl -X POST --data
    '{
        "jsonrpc":"2.0",
        "namespace":"global",
        "method":"contract_compileContract",
        "params":["contract_code"],
        "id":1
    }'

合约编译接口调用的返回如下：

.. code:: js

    {
      "jsonrpc": "2.0",
      "namespace":"global",
      "id": 1,
      "code": 0,
      "message": "SUCCESS",
      "result": {
        "abi": [
          "[{\"constant\":false,\"inputs\":[{\"name\":\"num1\",\"type\":\"uint32\"},{\"name\":\"num2\",\"type\":\"uint32\"}],\"name\":\"add\",\"outputs\":[],\"payable\":false,\"type\":\"function\"},{\"constant\":false,\"inputs\":[],\"name\":\"getSum\",\"outputs\":[{\"name\":\"\",\"type\":\"uint32\"}],\"payable\":false,\"type\":\"function\"},{\"constant\":false,\"inputs\":[],\"name\":\"increment\",\"outputs\":[],\"payable\":false,\"type\":\"function\"}]"
        ],
        "bin": [
          "0x60606040526000805463ffffffff1916815560ae908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d14605e578063d09de08a146084575b6002565b346002576000805460e060020a60243560043563ffffffff8416010181020463ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b34600257605c6000805460e060020a63ffffffff821660010181020463ffffffff1990911617905556"
        ],
        "types": [
          "Accumulator"
        ]
      }
    }

其中字段bin对应的内容为该合约的字节码表示，该bin内容供后续部署使用。

部署合约
~~~~~~~~

Hyperchain部署Solidity命令如下：

.. code:: js

    curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global",  "method":"contract_deployContract", "params":[{
    "from":"0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc ",
    "nonce":5373500328656597,
    "payload":"0x60606040526000805463ffffffff1916815560ae908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d14605e578063d09de08a146084575b6002565b346002576000805460e060020a60243560043563ffffffff8416010181020463ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b34600257605c6000805460e060020a63ffffffff821660010181020463ffffffff1990911617905556",
    "signature":"0x388ad7cb71b1281eb5a0746fa8fe6fda006bd28571cbe69947ff0115ff8f3cd00bdf2f45748e0068e49803428999280dc69a71cc95a2305bd2abf813574bcea900",
    "timestamp":1487771157166000000
    }],"id":"1"}'

部署合约返回如下，其中result字段内容为该合约在区块链中的地址，后期对该合约的调用需要指定该合约地址。

.. code:: js


    {
        "jsonrpc": "2.0",
        "namespace":"global",
        "id": 1,
        "code": 0,
        "message": "SUCCESS",
        "result": "0x406f89cb205e136411fd7f5befbf8383bbfdec5f6e8bcfe50b16dcff037d1d8a"
    }

调用合约
~~~~~~~~

Hyperchain调用命令如下，其中payload为调用合约中函数以及其参数值的编码结果，to为所调用合约的地址。

.. code:: js

    curl localhost:8081 --data

    '{
        "jsonrpc":"2.0",
        "namespace":"global",
        "method": "contract_invokeContract",
        "params":[{
                "from":"0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",
                "nonce":5019420501875693,
                "payload":"0x3ad14af300000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000002",
                "signature":"0xde467ec4c0bd9033bdc3b6faa43a8d3c5dcf393ed9f34ec1c1310b0859a0ecba15c5be4480a9ad2aaaea8416324cb54e769856775dd5407f9fd64f0467331c9301",
                "simulate":false,
                "timestamp":1487773188814000000,
                "to":"0x313bbf563991dc4c1be9d98a058a26108adfcf81"
                }],
        "id":"1"
    }'

合约调用会立即给客户端返回该交易的哈希值，后期可以根据该交易的哈希值查询具体交易的执行结果。

.. code:: js

    {
        "jsonrpc":"2.0",
        "namespace":"global",
        "id":1,
        "code":0,
        "message":"SUCCESS",
        "result":"0xd7a07fbc8ea43ace5c36c14b375ea1e1bc216366b09a6a3b08ed098995c08fde"
    }

