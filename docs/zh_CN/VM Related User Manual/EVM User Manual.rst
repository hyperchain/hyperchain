.. _EVM-User-Manual:

EVM使用手册
^^^^^^^^^^^^^

1.使用方式
-------------

evm的基本使用方式大致与以太坊相同, 基本语法参见solidity官方文档

`https://docs.soliditylang.org/en/v0.8.15/ <https://docs.soliditylang.org/en/v0.8.15/>`_

2.与以太坊差异
---------------

2.1 blockhash
>>>>>>>>>>>>>>>>>>>>>

以太坊中获取指定区块的区块哈希可以使用 `blockhash(uint blockNumber) returns (bytes32)` 方法来获取前256个区块号中任意区块号的hash，但是由于flato平台存在区块归档功能，与此功能冲突，因此不提供此功能。

2.2 跨合约调用
>>>>>>>>>>>>>>>>>>

在以太坊evm中，跨合约调用时如果跨合约调用的方法失败，跨合约调用的方法会返回false，并不会导致当前调用的直接失败， 而在flato-evm中交易的执行会失败。

跨合约调用demo::

     pragma solidity >=0.7.0 <0.9.0;

     contract Base {
        uint public test_count=1;
        function test() public {
            test_count+=1;
        }
     }


假若base合约部署后的地址为 `0xca35b7d915458ef540ade6068dfe2f44e8fa733c`

 ::

     pragma solidity >=0.7.0 <0.9.0;

     contract CrossCall{
        address addr = 0xca35b7d915458ef540ade6068dfe2f44e8fa733c;
        function delegatecall() public {
            addr.delegatecall(abi.encode(bytes4(keccak256("test()"))));
        }
     }

说明：evm的使用同样支持create2的方式部署发合约，此操作失败时的行为与上文描述一致。

2.3 Gas问题
>>>>>>>>>>>>>>

为保证兼容性，evm升级时，对应gas计费方式一般不会升级的，因此会与以太坊标准的Gas消耗有所差异

2.4 调用预编译合约
>>>>>>>>>>>>>>>>>>>

**2.4.1 调用方式**
:::::::::::::::::::

我们有两种调用方式

1. 通过抽象合约调用

 ::

     // 合约中取得hash示例
     pragma solidity ^0.4.10;
     contract TxHashAccessor {
        function getHash() returns(bytes32);
     }
     TxHashAccessor hasher = TxHashAccessor(0x00000000000000000000000000000000000000fa);
     bytes32 txhash = hasher.getHash();

2. 通过call指令调用

 ::

     address txHashGetter = 0x00000000000000000000000000000000000000fa;
     (bool success, bytes memory returnData) = txHashGetter.call(payload);
     bytes32 txHash = bytesToBytes32(returnData,0);

**2.4.2 新增预编译合约**
:::::::::::::::::::::::::::

当前在兼容以太坊预编译合约的前提下新增了一个预编译合约，如下：

+----------------------------+---------------------------------+------+
| 合约名称                   | 合约功能                        | 合约 |
|                            |                                 | 地址 |
+============================+=================================+======+
| ecrecover                  | 与以太                          | 0x1  |
|                            | 坊一致，请查阅solidity官方文档  |      |
+----------------------------+---------------------------------+------+
| sha256hash                 |                                 | 0x2  |
+----------------------------+---------------------------------+------+
| ripemd160hash              |                                 | 0x3  |
+----------------------------+---------------------------------+------+
| memCpy                     |                                 | 0x4  |
+----------------------------+---------------------------------+------+
| bigModExp                  |                                 | 0x5  |
+----------------------------+---------------------------------+------+
| bn256Add                   |                                 | 0x6  |
+----------------------------+---------------------------------+------+
| bn256ScalarMul             |                                 | 0x7  |
+----------------------------+---------------------------------+------+
| bn256Pairing               |                                 | 0x8  |
+----------------------------+---------------------------------+------+
| getTransactionHash         | 获取当前交易的txHash            | 0xfa |
| （新增合约）               |                                 |      |
+----------------------------+---------------------------------+------+

使用方式参考2.4.1

3. 合约升级规范
=================

对于已部署到区块链上的智能合约做版本升级需要用到合约升级的功能。合约编码者要注意一个正确的新版合约需要满足以下所有的升级规范，若不符合以下规范而进行合约代码升级的话，在之后的合约调用过程中，会出现**变量内容读取失败、变量内容读取异常、虚拟机执行失败**等情况，造成合约中存储的数据与变量名无法对应的情况。出现这种情况，可能会造成合约中某些**数据永久无法恢复**。因此合约编码者若需要做合约升级，请务必阅读以下升级规范。

注意，不规范的新版合约在升级过程中是不会报错的，即使在造成了数据混乱的情况下，在之后的调用过程中，虚拟机也有可能是不会报错的，即调用者感知错误比较困难。

3.1 变量定义
-----------------

3.1.1 新增变量定义
>>>>>>>>>>>>>>>>>>>>>>>>

新版合约若需要新增变量定义，注意一定要在旧版合约变量定义的基础上，在尾部追加新定义。示例::

 // 旧版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint    var1;
        string   var2;
     }

若旧版合约源码如上所示，在合约中定义了两个类型为uint和string的变量，若新版合约想要新增类型为bytes32类型的变量var3, 正确的定义方式为::

     // 正确的新版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint    var1;
        string   var2;
        byte32  var3;      // 将新增的变量定义追加在旧合约变量定义的尾部
     }

而以下这种新增变量定义的行为均是错误的::

     // 错误的新版合约
     pragma solidity ^0.4.4;
     contract Demo {
     uint    var1;
     byte32  var3;  // 将新增的变量定义插入在旧合约变量定义的中间
     string   var2;
     }

3.1.2 删除变量定义
:::::::::::::::::::

新版合约若需要删除部分在旧合约中定义的变量，需要注意的是只能删除在尾部定义的变量。示例::

     // 旧版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint    var1;
        string   var2;
     }

若旧版合约源码如上所示，在合约中定义了两个类型为uint和string的变量，若新版合版合约试图删除变量var2的定义，这种行为是容许的。正确示例::

     // 正确的新版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint    var1;
        // string   var2;   // 删除了定义在“尾部”的变量
     }

若新版合约试图删除变量var1的定义，这种行为是错误的。错误示例::

     // 错误的新版合约
     pragma solidity ^0.4.4;
     contract Demo {
        // uint    var1;     // 删除了定义在“非尾部”的变量
        string   var2;
     }

即合约编码者想要在新版合约中删除部分旧变量的定义，当且仅当删除的这些旧变量全部是定义在尾部的才是合法的。

3.1.3 修改变量定义
>>>>>>>>>>>>>>>>>>>>

更改变量定义的变量名是允许的，更改变量的类型是不被允许的。

 ::

     // 旧版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint    var1;
        string   var2;
     }

修改变量名的示例如下，这种行为是合法的::

     // 正确的新版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint    var3;  // 将变量名由var1改为了var3, 合法
        string   var4;  // 将变量名由var2改为了var4, 合法
     }

修改变量的类型的示例如下，这种行为是错误的::

     // 错误的新版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint8     var1;  // 将变量var1的类型改为uint8, 不合法
        bytes32   var2;  // 将变量var2的类型改为bytes32, 不合法
     }

3.1.4 更改变量定义顺序
>>>>>>>>>>>>>>>>>>>>>>

修改变量定义的顺序是不被允许的。

以下有个错误示例，合约编码者在新版合约中将旧版合约定义的var1,var2调换了定义顺序::

     // 旧版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint    var1;
        string   var2;
     }

     // 错误的新版合约
     pragma solidity ^0.4.4;
     contract Demo {
        string    var2;
        uint     var1;
     }

3.2 变量声明
---------------

3.2.1 新增变量声明
>>>>>>>>>>>>>>>>>>>>>

变量声明包括例如结构体的声明，枚举类型的声明等。新增变量声明是允许的，且允许声明在合约的任意位置。示例如下::

     // 旧版合约
     pragma solidity ^0.4.4;
     contract Demo {
        uint    var1;
        string   var2;
     }

以下几种新增定义方式都是合法的。

 ::

     // 正确的新版合约
     pragma solidity ^0.4.4;
     contract Demo {
        // 将结构体User声明在合约首部，合法
        struct User {
          bytes32      ID;
          uint         balance;
        }
        uint    var1;
        string   var2;
        // 将枚举类型UserType声明在合约尾部，合法
        enum UserType{ STUDENT, TEACHER, STUFF }
     }

3.2.2 删除变量声明
>>>>>>>>>>>>>>>>>>>

若在新版合约中删除旧版合约中未使用的变量声明，这种行为是合法的；若在新版合约中删除旧版合约正在使用的变量声明，这种行为是错误的。

 ::

     // 旧版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
     }

若在新版合约中删除未使用的变量声明ClassType, 这种行为是合法的

 ::

     // 正确的新版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        // enum ClassType {MATH, ENGLISH, CHINESE}  // 删除未使用的enum类型声明，合法
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
     }

若在新版合约中删除正在使用的变量声明UserType, 这种行为是错误的::

     // 错误的新版合约
     contract Demo {
     // enum UserType {STUDENT, TEACHER, STUFF}   // 删除正在使用的enum类型声明
                                             // 非法
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
     }

3.2.3 修改变量声明
>>>>>>>>>>>>>>>>>>>>

修改已有的变量声明是错误的::

     // 旧版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
     }

错误示例::

     // 错误的新版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER}  // 删除了UserType中的STUFF枚举项，非法
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            // string     id;             // 删除了User结构体中的id字段，非法
            UserType    t;
            ClassType    c;             // 新增了类型为ClassType的c字段，非法
        }
        User[] users;
     }

3.2.4 更改变量声明顺序
>>>>>>>>>>>>>>>>>>>>>>>>

更改变量声明的顺序是合法的示例如下

 ::

     // 旧版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
     }

     // 正确的新版合约
     contract Demo {
        // 调换了User结构体，ClassType，UserTyep枚举类型的声明位置，合法
        struct User {
            string     id;
            UserType   t;
        }
        enum ClassType {MATH, ENGLISH, CHINESE}
        enum UserType {STUDENT, TEACHER, STUFF}
        User[] users;
     }

3.3 函数定义
--------------

3.3.1 新增函数定义
>>>>>>>>>>>>>>>>>>>>

所有新增函数定义的行为都是合法的。

示例::

     // 旧版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
        function AddStudent(string id) {
            users.push(User(id, UserType.STUDENT));
        }
     }

     // 正确的新版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
        function AddStudent(string id) {
            users.push(User(id, UserType.STUDENT));
        }
        // 新增AddTeacher函数定义，合法
        function AddTeacher (string id) {
            users.push(User(id, UserType.TEACHER));
        }
     }

3.3.2 删除函数定义
>>>>>>>>>>>>>>>>>>>>

所有删除函数定义的行为都是合法的。

示例::

     // 旧版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
        function AddStudent(string id) {
            users.push(User(id, UserType.STUDENT));
        }
     }

     在新版合约中删除了AddStudent函数, 合法

     // 正确的新版合约
     contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
     // 删除了函数AddStudent, 合法
     // function AddStudent(string id) {
     //         users.push(User(id, UserType.STUDENT));
     // }
     }

3.3.3 修改函数定义
>>>>>>>>>>>>>>>>>>>>

所有修改函数定义的行为都是合法的

示例::

    // 旧版合约
    contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
        function AddStudent(string id) {
            users.push(User(id, UserType.STUDENT));
        }
    }

修改了AddStudent函数的定义，合法

 ::

    // 正确的新版合约
    contract Demo {
        enum UserType {STUDENT, TEACHER, STUFF}
        enum ClassType {MATH, ENGLISH, CHINESE}
        struct User {
            string     id;
            UserType   t;
        }
        User[] users;
        uint  userCnt;     // 在变量定义尾巴追加定义uint类型的变量userCnt，合法
        function AddStudent(string id) {
            users.push(User(id, UserType.STUDENT));
            userCnt += 1;     // 更改函数逻辑，合法
        }
    }

3.3.4 更改函数定义顺序
>>>>>>>>>>>>>>>>>>>>>>>>>

所有更改函数定义顺序的行为都是合法的

示例::

    // 旧版合约
    contract Demo {
        struct User {
            string     id;
            UserType   t;
        }
        enum ClassType {MATH, ENGLISH, CHINESE}
        enum UserType {STUDENT, TEACHER, STUFF}
        User[] users;
        function AddStudent(string id) {
            users.push(User(id, UserType.STUDENT));
        }
        function AddTeacher(string id) {
            users.push(User(id, UserType.TEACHER));
        }
    }

    // 新版合约
    contract Demo {
        struct User {
            string     id;
            UserType   t;
        }
        enum ClassType {MATH, ENGLISH, CHINESE}
        enum UserType {STUDENT, TEACHER, STUFF}
        User[] users;
        // 更改AddTeacher与AddStudent两个函数的定义顺序，合法
        function AddTeacher(string id) {
            users.push(User(id, UserType.TEACHER));
        }
        function AddStudent(string id) {
            users.push(User(id, UserType.STUDENT));
        }
    }



4. 编译器最高版本对应
------------------------

=========== ========
flato       solidity
=========== ========
1.0.0~1.0.5 0.5.x
1.0.6+      0.8.x
=========== ========

