# Smart Contract

## 1. Introduction

> A smart contract is a piece of computer program deployed on the blockchain that automatically executes the terms. Smart contract can automatically execute predefined protocols based on outside input and complete the transfer of relevant status within the blockchain.

In the wide sense, smart contracts also include smart contract programming languages, compilers, virtual machines, events, state machines, fault tolerance mechanisms, and more. The most import components of smart contract is the smart contract programming language and its execution engine.

Smart contract virtual machines are generally sandboxed for security reasons, and the entire execution environment is completely isolated.Smart contracts executed inside virtual machines are not allowed to access to system resources such as network, file system, process threads, and so on. 
 
 Different smart contracts own different levels of security and richness of expression. The HyperVM, which is developed independently by the Hyperchain team, is a general-purpose intelligent contract engine designed to allow access by many different smart contract engines. Currently, HyperEVM is compatible with Ethereum's Solidity language and HyperJVM, a Java-enabled smart contract execution engine.


## 2. Smart contract execution engine HyperVM

HyperVM developed by Hyperchain is a pluggable smart contract engine generic framework that allows different smart contract execution engine embed. 

As shown in the following figure is the schematic diagram of HyperVM architecture, HyperVM architecture provides smart contract compiler, interpreter, executor and state management components and other related major components. Among them, Compiler provides smart contract compilation related functions, Interpreter and Executor provide smart contract interpretation and execution related functions, and State components help contract to operate blockchain ledger. Guard module provides smart contract security-related mechanisms.

![hypervm-architecture](./hypervm.png)

### 2.1 HyperEVM

To maximize the open source community's research and accumulation in smart contract technologies, enhance the reusability and compatibility of smart contracts. HyperEVM implementation uses a fully compliant Ethereum smart contract specification using Solidity as the smart contract development language and adopt the optimized Ethereum virtual machine EVM as the default backend execution engine.

The following figure shows the HyperEVM smart contract execution flow:

![hyperevm-flow](./hyperevm-flow_en.png)


HyperEVM executes a transaction, it returns an execution result. The system stores it in a variable called a transaction receipt, and the platform client can query the transaction result according to the current transaction hash. 

This process runs as follows:

1. HyperEVM received the upper transmission of the transaction, and do preliminary verification;
2. Determine the type of transaction, if it is the disposition of the contract, go to step 3, otherwise go to step 4;
3. HyperEVM create a new contract account to store the contract address and the compiled code;
4. HyperEVM verify transaction parameters and signature information, and call its execution engine to execute the corresponding smart contract byte code;
5. After the instruction is executed, HyperVM will determine if it is down normaly, skip Step 2 if not, otherwise go to Step 6;
6. Determine whether the shutdown state of HyperVM is normal, and if it is normal, stop execution; otherwise, go to Step 7;
7. Undo operation, the state should also rollback.


The instruction set execution module is the core of HyperEVM execution component.The execution of the instruction module has two implementations, namely bytecode-based execution and more complex and efficient Just-in-time compilation.

Bytecode implementation is relatively simple, HyperEVM virtual machine have an instruction execution unit. The instruction execution unit will always try to execute the instruction set. When the designated time is not completed, the virtual machine will interrupt the calculation logic and return a timeout error message to prevent the execution of malicious code in the smart contract.

JIT execution is relatively more complex, instant compilation also known as timely compilation, real-time compilation, is a form of dynamic compilation is a way to improve the efficiency of the program. In general, there are two ways to run a program: static compilation and dynamic transliteration. Statically compiled programs are all translated into machine code before execution, while literal translation is performed while translation is performed. The real-time compiler mixes both, compiling the source code sentence by sentence, but caches translated code to reduce performance penalties. Compare to the static compiled code, compiled code on the fly can handle delayed binding and enhance security. JIT execution model mainly includes the following steps:

1. All the information related to the smart contract is encapsulated in the contract object, and then find out whether the contract object stored and compiled by the hash of the code.There are four common status of the contract object, namely: unknown, compiled, ready to execute through JIT, error.
2. If the contract status is ready for execution through the JIT, HyperEVM selects the JIT executor to execute the contract. During execution, the virtual machine will further compile the compiled smart contract into a machine code and optimize the push and jump instructions.
3. If the contract status is unknown, HyperEVM first needs to check if the virtual machine is forcing the JIT to execute, and if so, it will be compiled sequentially and executed by the JTI instruction. Otherwise, open a separate thread to compile, the current program is still compiled by ordinary bytecode. Next time the virtual machine encounters the same encoded contract again during execution, the virtual machine directly selects the optimized contract. 
As a result of the optimization of the instruction set of such a contract, the efficiency of the execution and deployment of the contract can be greatly improved.


## 3. Usage of Smart contract

### 3.1 Smart Contract Based on the Solidity 

### Write a contract

The Solidity-based smart contract is similar to a JavaScript program and consists of a series of variables and related functions. Below is a smart contract that simulates simple accumulator functionality. We use this as an example to briefly introduce the basic components of a Solidity smart contract.


```js
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
```

Accumulator contract description:

* Solidity-based smart contracts begin with the keyword `contract`, similar to the keyword `class` in Java and other languages;
* The contract can have variables and functions inside, the sum is a simple variable uint32 type, Solidity smart contract also supports map and other complex collection types;
* The contract allows the definition of the implementation of the function `function` keyword definition;

Reference to [Solidiy official website] (https://solidity.readthedocs.io/en/develop/) for detailed specification of smart contracts based on the Solidity language.

### Compile the contract

Hyperchain's smart contracts can be compiled either with the official Solidity compiler or using the smart contract JSON-RPC interface provided by Hyperchain (this scenario requires installing the Solidity compiler `sloc` on the host where Hyperchain installed).

The command which call Hyperchain to compile the solidity smart contract as follows:

```js
curl -X POST --data 
'{
	"jsonrpc":"2.0", 
	"namespace":"global", 
	"method":"contract_compileContract", 
	"params":["contract_code"],
	"id":1
}'
```
The contract compilation call returned as follows:

```js
{  "jsonrpc": "2.0",  "namespace":"global",  "id": 1,  "code": 0,  "message": "SUCCESS",  "result": {    "abi": [      "[{\"constant\":false,\"inputs\":[{\"name\":\"num1\",\"type\":\"uint32\"},{\"name\":\"num2\",\"type\":\"uint32\"}],\"name\":\"add\",\"outputs\":[],\"payable\":false,\"type\":\"function\"},{\"constant\":false,\"inputs\":[],\"name\":\"getSum\",\"outputs\":[{\"name\":\"\",\"type\":\"uint32\"}],\"payable\":false,\"type\":\"function\"},{\"constant\":false,\"inputs\":[],\"name\":\"increment\",\"outputs\":[],\"payable\":false,\"type\":\"function\"}]"    ],    "bin": [      "0x60606040526000805463ffffffff1916815560ae908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d14605e578063d09de08a146084575b6002565b346002576000805460e060020a60243560043563ffffffff8416010181020463ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b34600257605c6000805460e060020a63ffffffff821660010181020463ffffffff1990911617905556"    ],    "types": [      "Accumulator"    ]  }}

```
The content corresponding to the field bin is the bytecode representation of the contract, and the bin will be used for subsequent deployment.

### Deploy the contract

Hyperchain deploy solidity contract command is as follows:

```js
curl localhost:8081 --data '{"jsonrpc":"2.0", "namespace":"global",  "method":"contract_deployContract", "params":[{"from":"0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc ","nonce":5373500328656597,"payload":"0x60606040526000805463ffffffff1916815560ae908190601e90396000f3606060405260e060020a60003504633ad14af381146030578063569c5f6d14605e578063d09de08a146084575b6002565b346002576000805460e060020a60243560043563ffffffff8416010181020463ffffffff199091161790555b005b3460025760005463ffffffff166040805163ffffffff9092168252519081900360200190f35b34600257605c6000805460e060020a63ffffffff821660010181020463ffffffff1990911617905556","signature":"0x388ad7cb71b1281eb5a0746fa8fe6fda006bd28571cbe69947ff0115ff8f3cd00bdf2f45748e0068e49803428999280dc69a71cc95a2305bd2abf813574bcea900","timestamp":1487771157166000000}],"id":"1"}'
```

Deploy contract execution is returned as follows, where the result field is the address of the contract in the blockchain, and the contract address needs to be specified for subsequent execution call to the contract.

```js

{	"jsonrpc": "2.0",	"namespace":"global",    "id": 1,    "code": 0,    "message": "SUCCESS",    "result": "0x406f89cb205e136411fd7f5befbf8383bbfdec5f6e8bcfe50b16dcff037d1d8a"}
```

### Call contract

The Hyperchain invocation command is as follows, where `payload` is the encoding result of the function in the invocation contract and its parameter value, and `to` is the address of the invoked contract.

```js
curl localhost:8081 --data 

'{
	"jsonrpc":"2.0", 
	"namespace":"global", 
	"method": "contract_invokeContract", 
	"params":[{			"from":"0x17d806c92fa941b4b7a8ffffc58fa2f297a3bffc",			"nonce":5019420501875693,			"payload":"0x3ad14af300000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000002",			"signature":"0xde467ec4c0bd9033bdc3b6faa43a8d3c5dcf393ed9f34ec1c1310b0859a0ecba15c5be4480a9ad2aaaea8416324cb54e769856775dd5407f9fd64f0467331c9301",			"simulate":false,			"timestamp":1487773188814000000,			"to":"0x313bbf563991dc4c1be9d98a058a26108adfcf81"			}],
	"id":"1"
}'
```

The contract call will immediately return the hash value of the transaction to the client, and later query the execution result of the specific transaction according to the hash value of the transaction.

```js
{	"jsonrpc":"2.0",	"namespace":"global",	"id":1,	"code":0,	"message":"SUCCESS",	"result":"0xd7a07fbc8ea43ace5c36c14b375ea1e1bc216366b09a6a3b08ed098995c08fde"}

```

The other contract operation methods and specifications of methods parameters are detailed in:
[Hyperchain API Documentation] ()



 

