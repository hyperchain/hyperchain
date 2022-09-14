.. _HVM-invocation-across-contracts:

HVM跨合约功能使用手册
^^^^^^^^^^^^^^^^^^^^^^

概念说明
===========

在区块链中，有些功能需要通过多个智能合约完成，在这种情况下，需要使用到合约的跨合约调用。在HVM中，用户可以通过hvm-sdk提供的接口获取到跨合约实例并进行调用。

HVM提供了两种跨合约调用实例：

- contract

- library

其中被跨合约的两个内置变量sender将变成发起跨合约调用的合约地址，origin则同发起跨合约调用的相同，都为交易发起者账户地址。

contract实例
----------------

调用contract实例时，会影响到该合约账本中的状态变量。即如果在跨合约调用中被调用contract实例合约对应的合约状态变量被修改了，那么在持久化阶段，被调用合约中的被修改的状态变量也将被持久化。

在不跨合约的场景下，正常调用或者合约部署是，合约默认为contract实例。

library实例
--------------

调用library实例时，被调用合约是以工具类的角色存在，对被调用合约的修改最终不会在账本中体现，即所有修改均会被抛弃。

使用说明
=========

在hvm-sdk的BaseContract中提供了内部类CrossCall，用于获取跨合约实例，CrossCall的字段属性如下表所示。

============ ======= ==================================================
名称         类型    描述
============ ======= ==================================================
crossAddress String  被调用的合约地址
isLibrary    boolean true表示获取library实例，false表示获取contract实例
============ ======= ==================================================

定义说明
----------

在介绍跨合约调用的流程前，我们先规定需要使用到的概念：

**合约A**
>>>>>>>>>>>>>>

 ::

     public class A extends BaseContract implements IA {
        public void funcA(){}
     }

合约A继承了BaseContract类，实现了IA中声明的方法，该合约方法中会调用合约B的方法。

 ::

     public interface IA extends BaseContractInterface {
        void funcA();
     }

IA继承了BaseContractInterface类。

**合约B**
>>>>>>>>>>>>>

 ::

     public class B extends BaseContract implements IB {
        public void funcB(){}
     }

合约B继承了BaseContract类，实现了IB中声明的方法。

 ::

     public interface IB extends BaseContractInterface {
        void funcB();
     }

IB继承了BaseContractInterface类。

注解说明
-----------

CrossCall需要配合相应的注解来使用。用户必须在合约声明CrossCall字段时，配合使用 `@Contract` 和 `@Library` 注解来声明实例类型，并且需要设置合约的地址。

如果我们要在合约A中调用合约B的方法，我们需要先在平台部署合约B，然后在合约A中声明一个带有注解的CrossCall字段，注解中的address即我们部署好的合约B地址，如下面的代码所示::

     public class A extends BaseContract implements IA {
        @Contract(address = "0x209a1a980946e899b2cb4fc0ecb2b921f64bd236")
        private CrossCall call = new CrossCall();
     }

跨合约调用
-----------

我们给CrossCall提供了一个获取被调用合约实例的方法 `getCrossContract`  ,若 `isLibrary=true` ，返回合约地址对应的library实例；若 `isLibrary=false` ，返回合约地址对应的contract实例。需要注意的是，该合约地址对应的合约主体类必须实现 **BaseContractInterface** 接口，可以使用对应 **合约接口类型变量** 进行接收。

 ::

     public final <T extends BaseContractInterface> T getCrossContract()

在我们合约A中声明了含有 `@contract` 的CrossCall字段以后，我们可以在合约A的方法中，通过 `getCrossContract` 获取合约B的contract实例，然后调用合约B的方法。

 ::

     public class A extends BaseContract implements IA {
        @Contract(address = "0x209a1a980946e899b2cb4fc0ecb2b921f64bd236")
        private CrossCall call = new CrossCall();

        public void funcA() {
            IB iB = call.getCrossContract();
            iB.funcB();
        }
     }

注意事项
============

下面是一些在跨合约调用中需要注意的方法，或者是建议的用法。

同全限定名类
--------------

在一个调用中涉及到的所有的合约中不能出现：使用全限定类名相同(即包名和类名都相同)，但是具体实现不同的类。

若出现，将会得到如下异常::

     cn.hyperchain.sdk.exception.RequestException: Deploy contract failed: DEPLOY_CONTRACT_FAILED:java.lang.RuntimeException: init library failed: found a duplicate class: org.example.bean.Man, please change it

跨合约调用链
-----------

目前的跨合约调用只支持一层跨合约调用。有以下跨合约调用场景：

1. 合约A中，调用了contract实例B的方法和contract实例C的方法。 **调用成功** 。

2. 合约A中，调用了contract实例B的方法，该实例B的方法中又跨合约调用了contract实例C的方法。 **调用失败** 。

合约生命周期钩子方法
---------------------

1. 在每次获取contract实例（library实例）时，均会触发contract实例（library实例）的onCreated钩子函数，即使该实例曾经已经被获取过。

2. 合约可以通过在 **onCreated** 钩子函数中，通过检查 **sender** 和 **origin** 地址，来进行权限控制，若想要终止对自己跨合约调用只需要在 **onCreated** 钩子函数中抛出 **RuntimeException** 即可。合约可以通过 **getSender()** 获取到本次对自己的直接调用者(用户或者合约)的地址，可以通过 **getOrigin()** 方法获取到调用链的起点(必然是用户)的地址。

3. 因为library实例的变更最终不会被持久化到账本，所以library实例的 **onPreCommit** 和 **onCommited** 钩子函数不会被触发。

4. **onCreated** 的执行顺序与合约执行顺序相同。 **onPreCommit** 和 **onCommited** 钩子方法的调用顺序与合约执行顺序相反。比如：有调用链A->B，那么 **onCreated** 的调用顺序为A->B，而 **onPreCommit** 和 **onCommited** 的调用顺序为B->A。

实例演示
=========

本小节将给出一个跨合约调用的实例，实例中包含两个合约：NumAdd合约和CrossCall合约。我们将通过跨合约调用的方式，在crossCall合约中调用addNum合约中的方法。合约的调用与部署都基于 `LiteSDK` 。

NumAdd合约
-------------

NumAdd合约代码如下。在合约部署阶段，通过 `onInit` 方法初始化 `num=100` 。合约提供了 `getNum` 方法获取num的值以及 `addNum` 方法增加num的值。

 ::

     public class NumAdd extends BaseContract implements INumAdd {
        @StoreField
        int num;

        @Override
        public void onInit() {
            num = 100;
        }

        @Override
        public int getNum() {
            return num;
        }

        @Override
        public int addNum(int v) {
            num += v;
            return num;
        }
     }

部署NumAdd合约
------------------

首先，我们部署打包好的NumAdd合约，并输出部署好的合约地址。

 ::

     public void deployNumAdd() throws IOException, RequestException {
        //1.部署合约
        InputStream is1 = FileUtil.readFileAsStream(jarPath1);
        DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(defaultURL).build();
        ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);

        ContractService contractService = ServiceManager.getContractService(providerManager);
        AccountService accountService = ServiceManager.getAccountService(providerManager);
        Account account = accountService.fromAccountJson(accountJson);

        Transaction transaction = new Transaction.HVMBuilder(account.getAddress()).deploy(is1).build();
        transaction.sign(account);

        ReceiptResponse receiptResponse = contractService.deploy(transaction).send().polling();
        String contractAddress = receiptResponse.getContractAddress();
        System.out.println("contract numAdd address: " + contractAddress);
     }

输出合约地址::

     contract numAdd address: 0xd40db0476049f1cf71c59a5ef754bd1c77d1cede

CrossCallContract
---------------------

在部署好NumAdd合约后，我们在CrossCallContract合约中编写调用addNum合约逻辑的代码。合约中声明了一个 `CrossCall` 类型的字段，使用 `@Contract` 注解标识这是一个 `Contract` 实例，注解中的address属性对应前面部署好的NumAdd合约的地址。

CrossCallContract合约包含了两个方法：

- crossCallGetNum：调用NumAdd合约的getNum方法，打印num的值。

- crossCallAddNum：调用NumAdd合约的getNum方法，打印num的值；调用NumAdd的addNum方法，使num的值加1.

 ::

     public class CrossCallContract extends BaseContract implements ICrossCallContract {
        @Contract(address = "0xd40db0476049f1cf71c59a5ef754bd1c77d1cede")
        private CrossCall contractCall = new CrossCall();

        public String crossCallGetNum() {
            INumAdd iNumAdd = contractCall.getCrossContract();
            int num = iNumAdd.getNum();
            return "crossCallGetNum: the num is " + num;
        }

        public String crossCallAddNum() {
            INumAdd iNumAdd = contractCall.getCrossContract();
            int num = iNumAdd.getNum();
            iNumAdd.addNum(1);
            return "crossCallAddNum: the num is " + num + ", and add 1";
        }
     }

调用CrossCallContract合约
---------------------------

CrossCallContract合约的部署与NumAdd合约的部署类似，这里不再赘述。部署好CrossCallContract合约后，我们通过直接调用的方式调用CrossCall合约。

 ::

     public void deployCrossCall() throws IOException, RequestException {
        //1.部署CrossCallContract合约
        ……

        //2.直接调用crossCallGetNum
        InvokeDirectlyParams invokeDirectlyParams1 = new InvokeDirectlyParams.ParamBuilder("crossCallGetNum").build();
        Transaction transaction1 = new Transaction.HVMBuilder(account.getAddress())
                .invokeDirectly(contractAddress, invokeDirectlyParams1)
                .build();
        transaction1.sign(account);
        ReceiptResponse receiptResponse1 = contractService
                .invoke(transaction1).send().polling();
        System.out.println(Decoder.decodeHVM(receiptResponse1.getRet(), String.class));

        //3.直接调用crossCallGetNum
        InvokeDirectlyParams invokeDirectlyParams2 = new InvokeDirectlyParams.ParamBuilder("crossCallAddNum").build();
        Transaction transaction2 = new Transaction.HVMBuilder(account.getAddress())
                .invokeDirectly(contractAddress, invokeDirectlyParams2)
                .build();
        transaction2.sign(account);
        ReceiptResponse receiptResponse2 = contractService
                .invoke(transaction2).send().polling();
        System.out.println(Decoder.decodeHVM(receiptResponse2.getRet(), String.class));

        //4.再次直接调用crossCallGetNum
        Transaction transaction3 = new Transaction.HVMBuilder(account.getAddress())
                .invokeDirectly(contractAddress, invokeDirectlyParams1)
                .build();
        transaction3.sign(account);
        ReceiptResponse receiptResponse3 = contractService
                .invoke(transaction3).send().polling();
        System.out.println(Decoder.decodeHVM(receiptResponse3.getRet(), String.class));
     }

上面的代码进行了三次调用：

1. 调用crossCallGetNum方法，获取num的值，打印结果为

 ::

    crossCallGetNum: the num is 100

2. 调用crossCallAddNum方法，使num的值加1，打印结果为

 ::

    crossCallAddNum: the num is 100, and add 1

3. 调用crossCallGetNum方法，获取num的值，打印结果为

 ::

    crossCallGetNum: the num is 101

通过上面的例子，我们可以发现被CrossCallContract合约调用的NumAdd合约将num的值被持久化记录到了账本上，符合contract实例的特点。有兴趣的读者可以在CrossCallContract合约中将CrossCall的 `@Contract` 注解类型改为 `@Library` ，观察区别。

下面是给出的跨合约调用demo，可以下载在本地体验。调用测试代码码时，请先部署NumAdd合约，然后在CrossCallContract合约的跨合约注解中，写入这个地址。接着执行 `mvn package` 命令，生成CrossCallContract合约的jar包。

**【源码包可参考HVM使用手册 - HVM合约Demo附件源码-hvm-manual-demo的crossCallDemo目录】**



