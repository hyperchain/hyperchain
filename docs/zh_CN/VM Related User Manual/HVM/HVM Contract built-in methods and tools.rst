.. _HVM-Contract-built-in-methods-and-tools:

HVM合约内置方法和工具方法使用手册
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. 合约内置变量
===============

目前合约的内置变量

- `sender` ：合约方法调用者的地址（当且仅当存在跨合约调用时值会与orgin不同）

- `origin` ：合约调用者的地址

- `txHash` ：当前交易的哈希

- `blockTimestamp` ：交易所在区块的时间戳

- `blockNumber` ：交易所在区块

- `contractAddress` ：合约地址

2. 合约内置方法
===============

合约内置方法是HVM合约实现类继承自BaseContract的方法，可在合约主类中通过this引用直接使用。

获取交易extra信息
--------------------

**getValueFromExtra**

获得当前交易的extra字段信息，其中extra记录了链上额外的存证信息，是一个json字符串。

 ::

     public String getValueFromExtra(String path) {}

+-------------+-----------------------------+--------------------------+
| 方法        | 参数                        | 返回值                   |
+=============+=============================+==========================+
| getVal      | path：对得                  | String类型               |
| ueFromExtra | 到的extra字段进行过滤的规则 | ，从extra字段得到的信息  |
+-------------+-----------------------------+--------------------------+

其中path是一系列由 `.` 分隔的key组成的字符串。

在key中可以包含一些特殊的通配符比如 `*` 和 `?` 。如果要访问数组的指定元素，使用下标作为key。要获取数组的长度或者要基于数组添加子path，可以使用 `#` 。

你可以对一个数组使用条件查询， `#[...]` 将返回第一个匹配元素， `#[...]#` 将返回所有匹配元素，其中查询条件支持比较运算符： `==` ，  `!=` ，  `<` ， `<=` ， `>` ， `>=` ，还有一些简单的模式匹配： `%` （like）和 `!%` （not like）。

注意： `.` 和 `?` 和 `*` 可以通过 `\\` 转义成为一般字符。

实例::

 {"name": {"first": "Tom", "last": "Anderson"},"age":37,"children": ["Sara","Alex","Jack"],"fav.movie": "Deer Hunter","friends": [{"first": "Dale", "last": "Murphy", "age": 44},{"first": "Roger", "last": "Craig", "age": 68},{"first": "Jane", "last": "Murphy", "age": 47}]}

+-------------------+-------------+-----------------------------------+
| Path              | Result      | 说明                              |
+===================+=============+===================================+
| “name.last”       | Anderson    | name的属性                        |
+-------------------+-------------+-----------------------------------+
| “age”             | 37          | age属性                           |
+-------------------+-------------+-----------------------------------+
| “children”        | [“Sara”,“Al | children属性                      |
|                   | ex”,“Jack”] |                                   |
+-------------------+-------------+-----------------------------------+
| “children.#”      | 3           | children数组的长度                |
+-------------------+-------------+-----------------------------------+
| “children.1”      | “Alex”      | children数组的下标为1的元素       |
+-------------------+-------------+-----------------------------------+
| c?ildren.0”       | “Sara”      | 匹配模式                          |
|                   |             | 为“c?ildren”的元素的下标为0的元素 |
+-------------------+-------------+-----------------------------------+
| “fav.movie”       | “Deer       | 转义.，“fav.movie”属性            |
|                   | Hunter”     |                                   |
+-------------------+-------------+-----------------------------------+
| “friends.#.first” | [           | friends数组内每个元素的first属性  |
|                   | “Dale”,“Rog |                                   |
|                   | er”,“Jane”] |                                   |
+-------------------+-------------+-----------------------------------+
| “friends.1.last”  | “Craig”     | friends数组的第2个元素的last属性  |
+-------------------+-------------+-----------------------------------+
| “friends.#[last=  | “Dale”      | friends数组中第一个满足last       |
| =”Murphy”].first” |             | 属性等于“Murphy”的元素的first属性 |
+-------------------+-------------+-----------------------------------+
| “friends.#[last== | [“Da        | friends数组中所有满足last         |
| ”Murphy”]#.first” | le”,“Jane”] | 属性等于“Murphy”的元素的first属性 |
+-------------------+-------------+-----------------------------------+
| “friends          | [“Craig     | friends数组中所有                 |
| .#[age>45]#.last” | ”,“Murphy”] | 满足age属性大于45的元素的last属性 |
+-------------------+-------------+-----------------------------------+
| “friends.#[       | “Murphy”    | friends数组中第一个满足：first属  |
| first%”D*”].last” |             | 性符合”D*“模式，的元素的last属性  |
+-------------------+-------------+-----------------------------------+
| “friends.#[f      | “Craig”     | f                                 |
| irst!%”D*”].last” |             | riends数组中第一个不满足：first属 |
|                   |             | 性符合”D*“模式，的元素的last属性  |
+-------------------+-------------+-----------------------------------+

**示例**::

     String extra = this.getValueFromExtra("friends.1.last");

合约事件推送
----------------

hvm为合约的开发者提供了合约事件推送的功能，即可以在合约的执行过程中，将事件推送给客户端。订阅了相关事件类型的客户端可以接收到该事件消息。

合约事件的推送
>>>>>>>>>>>>>>>>>

合约事件推送的功能由合约基础类BaseContract实现，提供的接口如下::

     public final void event(Object data, String name, String... topics);

**参数：**

- data：指定的合约推送数据，最后会转换json的数据的形式推送给客户端

- name：事件的名称，必须提供一个非空的字符串来作为标识。

- topics：用来事件的类型。用户订阅时，可以指定自己想要接收的topic来精确订阅相应的事件。通过topics来确定要接收哪些类型的事件。

**示例**::

     Map<String, String> map = new HashMap<String, String>();
     map.put("tom","99");
     map.put("bob","98");
     event(map, "testEvent", "topicA");

合约事件的订阅
>>>>>>>>>>>>>>>>

用户可以通过LiteSdk提供的API来订阅合约事件，合约事件必须通过MQParam来注册订阅队列。

MQParam对象的创建示例如下::

     ArrayList<String> array = new ArrayList<String>();
     array.add("SignedMQBlock");
     array.add("MQLog");
     array.add("MQException");
     String qname = "node1queue";
     MQParam mqParam = new MQParam.Builder().
            //队列订阅相关的参数
            msgTypes(array).     //表示要订阅的消息类型
            queueName(qname).    //表示队列名称
            // MQlog事件订阅需要的参数
            logFromBlock("1").       //表示需要推送log事件的起始区块号
            logToBlock("2").         //表示需要推送log事件的终止区块号
            logAddress("0x12...").   //表示log事件需要匹配的合约地址
            logTopics(new String[]{"topicA", "topicB"}).
            //表示log事件需要匹配的topic集合
            logTopics(new String[]{"topicC", "topicD"}).
            //可多次调用添加多个topic数组
            build();

详细的订阅规则请查看LiteSdk使用文档的 **第七章. MQ相关接口(MQService)** 以及Litedk的 **MQ使用手册**

获取内置变量
-------------

获取合约方法调用者地址
>>>>>>>>>>>>>>>>>>>>>

**getSender**

得到sender，合约方法调用者的地址（当且仅当存在跨合约调用时值会与orgin不同）

 ::

     public final String getSender() {}

========= ====== ==============================
方法      参数     返回值
========= ====== ==============================
getSender 无      sender，即合约方法调用者的地址
========= ====== ==============================

**示例**::

    String sender = this.getSender();

获取合约调用者地址
>>>>>>>>>>>>>>>>>>

**getOrigin**

得到origin，合约调用者的地址::

     public final String getOrigin() {}

========= ==== ==========================
方法      参数 返回值
========= ==== ==========================
getOrigin 无   origin，即合约调用者的地址
========= ==== ==========================

**示例**::

    String origin = this.getOrigin();

获取交易哈希
>>>>>>>>>>>>>

**getTxHash**

得到txHash，当前交易哈希

 ::

    public final String getTxHash() {}

========= ==== ========================
方法      参数 返回值
========= ==== ========================
getTxHash 无   txHash，即当前交易的hash
========= ==== ========================

**示例**::

     String txHash = this.getTxHash();

获取时间戳
>>>>>>>>>>>>>>

**getBlockTimestamp**

得到blockTimestamp，交易所在区块的时间戳

 ::

     public final long getBlockTimestamp() {}

================= ==== ======================================
方法              参数 返回值
================= ==== ======================================
getBlockTimestamp 无   blockTimestamp，即交易所在区块的时间戳
================= ==== ======================================

**示例**::

     long blockTimestamp = this.getBlockTimestamp();

获取区块号
>>>>>>>>>>>

**getBlockNumber**

得到blockNumber，交易所在的区块号

 ::

     public final long getBlockNumber() {}

============== ==== =============================
方法           参数 返回值
============== ==== =============================
getBlockNumber 无   blockNumber，即交易所在区块号
============== ==== =============================

**示例**::

 long blockNumber = this.getBlockNumber();

获取合约地址
>>>>>>>>>>>>>>

**getContractAddress**

得到contractAddress，合约地址

 ::

    public String getContractAddress() {}

================== ==== ===========================
方法               参数 返回值
================== ==== ===========================
getContractAddress 无   contractAddress，即合约地址
================== ==== ===========================

**示例**::

    String contractAddress = this.getContractAddress();

合约内置方法使用demo
-------------------------

**【源码包可参考HVM使用手册 - HVM合约Demo附件源码- hvm-manual-demo的contractMethodDemo目录】**



3. HVM合约工具方法
==================

概述
--------

本文档介绍HVM合约编写过程可能使用到的工具方法，将分为以下8类进行说明。

- ByteUtil

- CryproUtil

- HashUtil

- ObjectsUtil

- StringUtil

- Logger

- DIDUtil

- Hyperson

ByteUtil
----------

byte数组与Hex String转换
>>>>>>>>>>>>>>>>>>>>>>>>>>>>

**bytesToHex**

byte数组转换为十六进制String类型

 ::

     public static native String bytesToHex(byte[] input);

========== ========================= ==============
方法       参数                      返回值
========== ========================= ==============
bytesToHex input：需要转换的byte数组 十六进制String
========== ========================= ==============

**示例**::

     byte[] input = new byte[]{'a','b'};
     String hexString = ByteUtil.bytesToHex(byte[] input);

**fromHexString**

十六进制String类型转换为byte数组

 ::

     public static native byte[] fromHexString(String s);

============= =========================== ==========
方法          参数                        返回值
============= =========================== ==========
fromHexString s：需要转换的十六进制String byte[]类型
============= =========================== ==========

**示例**::

     String s1 = "0xabd43f";
     ByteUtil.fromHexString(s1);

     String s2 = "abd43f";
     byte[] bytes = ByteUtil.fromHexString(s2);
     //结果相同

byte数组和int转换
>>>>>>>>>>>>>>>>>>>>

**bytesToInt**

byte数组转换为int类型

 ::

    public static int bytesToInt(byte[] bytes) {}

========== ========================= ==============
方法       参数                      返回值
========== ========================= ==============
bytesToHex input：需要转换的byte数组 十六进制String
========== ========================= ==============

**示例**::

     byte[] bytes = new byte[]{'a','b','c','d'};
     int result = ByteUtil.bytesToInt(bytes);

**intToBytes**

int类型转换为byte数组

 ::

    public static byte[] intToBytes(int value) {}

========== ======================== ===========
方法       参数                     返回值
========== ======================== ===========
intToBytes value：需要转换的int类型 byte[] 类型
========== ======================== ===========

**示例**::

     int value = 48;
     byte[] intBytes = ByteUtil.intToBytes(value);

base64编码转byte数组
>>>>>>>>>>>>>>>>>>>>>

**fromBase64Str**

将base64编码的String转换为byte数组

 ::

     public native static byte[] fromBase64Str(String string);

============= ======================================== ===========
方法          参数                                     返回值
============= ======================================== ===========
fromBase64Str string：需要转换的base64编码的String类型 byte[] 类型
============= ======================================== ===========

**示例**::

    byte[] bytes = ByteUtil.fromBase64Str(string);

CryptoUtil
----------------

ECDSA账户验签
>>>>>>>>>>>>>>>>>

**verifySignature**

EC账户验签，通过公钥、原文和签名内容来验证签名内容。

 ::

     public static boolean verifySignature(byte[] addr, byte[] origin, byte[] signature) {}

============= ======================================== ===========
方法          参数                                     返回值
============= ======================================== ===========
fromBase64Str string：需要转换的base64编码的String类型 byte[] 类型
============= ======================================== ===========

**示例**::

     //以下代码使用了一些litesdk的工具类
     ECKey ecKey = new ECKey(new SecureRandom());
     byte[] address = ecKey.getAddress();
     byte[] origin = "hyperchain".getBytes();
     byte[] hash = HashUtil.sha3(origin);
     byte[] signature = ecKey.sign(hash).toByteArray();
     CryptoUtil.verifySignature(address, hash, signature);

SM2国密账户验签
>>>>>>>>>>>>>>>>>>

**verifySM2Signature**

SM账户验签，通过公钥、原文和签名内容来验证签名内容是否正确。

 ::

     public static boolean verifySM2Signature(byte[] pubKey, byte[] origin, byte[] signature) {}

+-----------+----------------------------------------+-----------------+
| 方法      | 参数                                   | 返回值          |
+===========+========================================+=================+
| verifySM2 | pubKey：byte[]                         | 布尔值，签名正  |
| Signature | 类型的公钥origin：byte[]类             | 确则返回true，  |
|           | 型的原文signature：byte[]类型的签名。  | 否则返回false。 |
+-----------+----------------------------------------+-----------------+

**示例**::

     //以下代码使用类litesdk的工具类
     AsymmetricCipherKeyPair keyPair = SM2Util.generateKeyPair();
     ECPublicKeyParameters ecPub;
     ecpub = (ECPublicKeyParameters) keyPair.getPublic();
     byte[] origin = "hyperchain".getBytes();
     byte[] publicKey = ecPub.getQ().getEncoded(false);
     byte[] signature = SM2Util.sign(keyPair, origin);
     CryptoUtil.verifySM2Signature(publicKey,origin,signature);

SM4加解密
>>>>>>>>>>>>>

**sm4Encrypt**

sm4加密::

     public static byte[] sm4Encrypt(byte[] key, byte[] src) {}

+------------+------------------------------------+-------------------+
| 方法       | 参数                               | 返回值            |
+============+====================================+===================+
| sm4Encrypt | key：16字节的                      | byte[]            |
|            | 对称加密密钥src：需要被加密的内容  | 类型被加密的密文  |
+------------+------------------------------------+-------------------+

**sm4Decrypt**

sm4解密::

     public static byte[] sm4Decrypt(byte[] key, byte[] src) {}

+------------+------------------------------------+-------------------+
| 方法       | 参数                               | 返回值            |
+============+====================================+===================+
| sm4Decrypt | key：16字节的                      | byte[]            |
|            | 对称加密密钥src：需要被解密的内容  | 类型被解密的原文  |
+------------+------------------------------------+-------------------+

**示例**::

     byte[] key = new byte[]{'e','h','r','s','e','h','r','s','e','h','r','s','e','h','r','s'};
     byte[] src = "hyperchain".getBytes();
     byte[] encryptBytes = CryptoUtil.sm4Encrypt(key, src);
     byte[] decryptBytes = CryptoUtil.sm4Decrypt(key, encryptBytes);
     ByteUtil.bytesToHex(src).equals(ByteUtil.bytesToHex(decryptBytes));


AES加解密
>>>>>>>>>>>>>

**aesEncrypt**

AES加密::

     public static byte[] aesEncrypt(byte[] key, byte[] src) {}

+------------+------------------------------------+-------------------+
| 方法       | 参数                               | 返回值            |
+============+====================================+===================+
| aesEncrypt | key：32字节的                      | byte[]            |
|            | 对称加密密钥src：需要被加密的内容  | 类型被加密的密文  |
+------------+------------------------------------+-------------------+

**aesDecrypt**

AES解密::

     public static byte[] aesDecrypt(byte[] key, byte[] src) {}

+------------+------------------------------------+-------------------+
| 方法       | 参数                               | 返回值            |
+============+====================================+===================+
| aesDecrypt | key：32字节的                      | byte[]            |
|            | 对称加密密钥src：需要被解密的内容  | 类型被解密的原文  |
+------------+------------------------------------+-------------------+

**示例**::

     keyStr = keyStr.substring(0, 32);
     byte[] key = keyStr.getBytes();
     byte[] src = "hyperchain".getBytes();
     byte[] encryptBytes = CryptoUtil.aesEncrypt(key, src);
     byte[] decryptBytes = CryptoUtil.aesDecrypt(key, encryptBytes);
     ByteUtil.bytesToHex(src).equals(ByteUtil.bytesToHex(decryptBytes));

tripleDES加解密
>>>>>>>>>>>>>>>>>>

**tripleDESEncrypt**

tripleDES 加密::

     public static byte[] tripleDESEncrypt(byte[] key, byte[] src) {}

+------------------+--------------------------------+-----------------+
| 方法             | 参数                           | 返回值          |
+==================+================================+=================+
| tripleDESEncrypt | key：24字节的对称              | byte[]          |
|                  | 加密密钥src：需要被加密的内容  | 类              |
|                  |                                | 型被加密的密文  |
+------------------+--------------------------------+-----------------+

**tripleDESDecrypt**

tripleDES 解密::

     public static byte[] tripleDESDecrypt(byte[] key, byte[] src) {}

+------------------+--------------------------------+-----------------+
| 方法             | 参数                           | 返回值          |
+==================+================================+=================+
| tripleDESDecrypt | key：24字节的对称              | byte[]          |
|                  | 加密密钥src：需要被解密的内容  | 类              |
|                  |                                | 型被解密的原文  |
+------------------+--------------------------------+-----------------+

**示例**::

     keyStr = keyStr.substring(0, 24);
     byte[] key = keyStr.getBytes();
     byte[] src = "hyperchain".getBytes();
     byte[] encryptBytes = CryptoUtil.tripleDESEncrypt(key, src);
     byte[] decryptBytes = CryptoUtil.tripleDESDecrypt(key, encryptBytes);
     ByteUtil.bytesToHex(src).equals(ByteUtil.bytesToHex(decryptBytes));

ecc加密
>>>>>>>>>>>>

**eccEncrypt**::

     public static byte[] eccEncrypt(byte[] publicKey, byte[] src) {

========== ================================== =======================
方法       参数                               返回值
========== ================================== =======================
eccEncrypt key：加密密钥src：需要被加密的内容 byte[] 类型被加密的原文
========== ================================== =======================

CryptoUtil使用demo
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

【源码包可参考HVM使用手册 - HVM合约Demo附件源码 -** hvm-manual-demo的cryptoCallDemo目录**】

HashUtil
------------

sha3-256哈希计算
>>>>>>>>>>>>>>>>>>>>

**sha3**

对输入的byte数组进行sha3-256哈希计算，返回散列值::

     public static byte[] sha3(byte[] input) {}

==== ========================= =================
方法 参数                      返回值
==== ========================= =================
sha3 input：需要哈希的byte数组 byte[] 类型散列值
==== ========================= =================

**示例**::

     byte[] input = new byte[]{'a','b','c','d'};
     byte[] hashResult = HashUtil.sha3(input);

ObjectsUtil
---------------

判断对象是否相等
>>>>>>>>>>>>>>>>>

**equals**

判断Object是否相等，返回布尔值::

     public static boolean equals(final Object x, final Object y) {

+-----------+----------------------------+----------------------------+
| 方法      | 参数                       | 返回值                     |
+===========+============================+============================+
| equals    | x：Java Object。y：Java    | 布尔值，x和y相等，         |
|           | Object                     | 返回true，否则返回false。  |
+-----------+----------------------------+----------------------------+

**示例**::

     String x = new String("test");
     Integer y = new Integet(0);
     boolean isEqual = ObjectsUtil.equals(x, y);

哈希计算
>>>>>>>>>>>>>

**hash**

计算多个Object的哈希值，返回多个Object的哈希值::

    public static int hash(final Object... values) {}

==== ============================== ==================
方法 参数                           返回值
==== ============================== ==================
hash values：0个或多个Java Object。 多个Object的哈希值
==== ============================== ==================

**示例**::

     String object1 = new String("test");
     Integer object2 = new Integet(0);
     int hashResult0 = ObjectsUtil.hash();
     int hashResult1 = ObjectsUtil.hash(object1);
     int hashResult2 = ObjectsUtil.hash(object1, object2);

StringUtil
--------------

检查String对象是否为空
>>>>>>>>>>>>>>>>>>>>>>>

**checkEmpty**

 ::

    public static boolean checkEmpty(String string) {}

+--------------+--------------------+---------------------------------+
| 方法         | 参数               | 返回值                          |
+==============+====================+=================================+
| checkEmpty   | string:            | 布尔值，String对象              |
|              | 需检查的String对象 | 为空则返回true，否则返回false。 |
+--------------+--------------------+---------------------------------+

**示例**::

     String string = new String();
     boolean isEmpty = StringUtil.checkEmpty(string);

Logger
----------

Logger类提供了打印对应classlog信息的功能。

获取logger
>>>>>>>>>>>>>>>

**getLogger**

 ::

     public static Logger getLogger(Class clazz) {}

========= ====================== ==========
方法      参数                   返回值
========= ====================== ==========
getLogger clazz: 对应的class对象 Logger对象
========= ====================== ==========

**示例**::

     public class InvokeTripleDES implements BaseInvoke<Boolean, ICrypto> {
        private Logger logger = Logger.getLogger(InvokeTripleDES.class);
     }

日志级别
>>>>>>>>>>>>

**critical级别的日志**::

    public void critical(Object message) {}

======== ===================================== ======
方法     参数                                  返回值
======== ===================================== ======
critical message: Object对象，需打印的日志信息 无
======== ===================================== ======

**示例**::

     logger.critical("logger critical message");

**err级别的日志**::

     public void err(Object message) {}

==== ===================================== ======
方法 参数                                  返回值
==== ===================================== ======
err  message: Object对象，需打印的日志信息 无
==== ===================================== ======

**示例**::

     logger.err("logger err message");

**warning级别的日志**::

     public void warning(Object message) {}

======= ===================================== ======
方法    参数                                  返回值
======= ===================================== ======
warning message: Object对象，需打印的日志信息 无
======= ===================================== ======

**示例**::

     logger.warning("logger warning message");

**notice级别的日志**::

     public void notice(Object message) {}

====== ===================================== ======
方法   参数                                  返回值
====== ===================================== ======
notice message: Object对象，需打印的日志信息 无
====== ===================================== ======

**示例**::

     logger.notice("logger notice message");

**info级别的日志**::

     public void info(Object message) {}

==== ===================================== ======
方法 参数                                  返回值
==== ===================================== ======
info message: Object对象，需打印的日志信息 无
==== ===================================== ======

**示例**::

     logger.info("logger info message");

**debug级别的日志**::

     public void debug(Object message) {}

===== ===================================== ======
方法  参数                                  返回值
===== ===================================== ======
debug message: Object对象，需打印的日志信息 无
===== ===================================== ======

**示例**::

     logger.debug("logger debug message");


DIDUtil
-----------

检查凭证是否有效
>>>>>>>>>>>>>>>>>>

**credentialIsValid**::

     public static native boolean credentialIsValid(String creID);

+------------+----------------+---------------------------------------+
| 方法       | 参数           | 返回值                                |
+============+================+=======================================+
| credent    | string:        | 布尔值，凭证被                        |
| ialIsValid | 需检查的凭证ID | 吊销或过期则返回false，否则返回true。 |
+------------+----------------+---------------------------------------+

**示例**::

     boolean isValid = DIDUtil.credentialIsValid(creID)

检查凭证是否吊销
>>>>>>>>>>>>>>>>>>

**credentialIsAbondoned**::

     public static native boolean credentialIsAbondoned(String creID);

+----------------+-----------------+-----------------------------------+
| 方法           | 参数            | 返回值                            |
+================+=================+===================================+
| credent        | string:         | 布尔值，凭证                      |
| ialIsAbondoned | 需检查的凭证ID  | 被吊销则返回true，否则返回false。 |
+----------------+-----------------+-----------------------------------+

**示例**::

     boolean isValid = DIDUtil.credentialIsAbondoned(creID)

Hyperson
------------

Hyperson提供了 `toHyperson` 方法将Java对象序列化成json字符串，以及 `fromHyperson` 方法将json字符串反序列化成Java对象。

在使用Hyperson提供的序列化和反序列化方法之前，首先要构造一个Hyperson对象，可通过构造方法以及HypersonBuilder来构建，示例如下::

     // 无参构造函数, 配置均按照默认值
     Hyperson hyperson = new Hyperson();

     // 有参构造函数，可传入相关配置
     Hyperson hyperson = new Hyperson(escapeHtml, serialzeNull, excludeAnnotationField, excludeClassField);

     // HypersonBuilder构建
     Hyperson hyperson = new Hyperson.HypersonBuilder()
            .disableEscapeHtml()
            .enableSerializeNull()
            .addExcludeAnnotationField(StoreField.class)
            .addExcludeClassField(PersonName.class)
            .create();

配置项说明如下：

- serializeNull 控制是否序列化对象中值为null的字段，设置为true，则序列化值为null的字段；设置为false，则忽略值为null的字段，默认为false。

- escapeHtml 控制是否转义html字符，如 `<` ，设置为true，则对html字符进行转义；设置为false，则不对html字符转义，默认为true。

- excludeAnnotationField 记录了注解类型，序列化和反序列化的过程中，将会忽略添加了这些注解的字段。即如果不想序列化对象中的某些字段，可以给相应字段添加注解，并将该注解类型设置到excludeAnnotationField中，那么添加了该注解的对象字段将不会被序列化。

- excludeClassField 记录了不被序列化及反序列化的字段类型。即如果不想序列化某一类型的字段，则可以将该类型设置到excludeClassField中，那么该类型的字段将不会被序列化。



序列化
>>>>>>>

**toHyperson**::

     public String toHyperson(Object object);

========== ====================== ================================
方法       参数                   返回值
========== ====================== ================================
toHyperson object: 需序列化的对象 字符串，对象序列化后的json字符串
========== ====================== ================================

**示例**::

     Hyperson hyperson = new Hyperson();
     Student student = new Student();
     String json = hyperson.toHyperson(student);

反序列化
>>>>>>>>>>>

**fromHyperson**::

     public <T> T fromHyperson(String hyperson, Type type);

+---------------+---------------------------------------+-------------+
| 方法          | 参数                                  | 返回值      |
+===============+=======================================+=============+
| fromHyperson  | hyperson:                             | 反          |
|               | 对象的json字符串type：对象的类型      | 序列化生成  |
|               |                                       | 的对象实例  |
+---------------+---------------------------------------+-------------+

**示例**::

     Hyperson hyperson = new Hyperson();

     String json = "test";
     Type type = String.class;
     String res = hyperson.fromHyperson(json, type);

     String mapJson = "{\"key\":\"value\"}";
     Type mapType = new ParameterizedTypeImpl(HashMap.class, new Type[]{String.class, String.class}, null);
     Map mapRes = hyperson.fromHyperson(mapJson, mapType);



**注意事项**

- 对象中不允许存在同名字段，即父类和子类不能存在同名字段

- 序列化Map类型的对象时，其key不能为复杂类型，可为八大基本类型或String类型

- 反序列化时，传入的类型参数必须是class类型对象或者实现了ParameterizedType接口的对象

- 反序列化时，如果没有传入相应的类型参数，则数组、集合对象默认反序列化为ArrayList；byte、short、int、long、float、double默认为Double；char、String默认为String；boolean默认为Boolean；其他对象反序列化为LinkedHashMap对象

- 反序列化时，当传入的类型参数是抽象类或接口时，如果其为继承或实现Collection的接口或抽象类类型，则SortedSet接口类型对应TreeSet类型，EnumSet类型对应EnumSet的子类，Set接口类型对应LinkedHashSet类型，Queue接口类型对应ArrayDeque类型，其他为ArrayList类型；如果其为继承或实现Map的接口或抽象类类型，则ConcurrentNavigableMap接口类型对应ConcurrentSkipListMap类型，ConcurrentMap接口类型对应ConcurrentHashMap类型，SortedMap接口类型对应TreeMap类型，其他为LinkedHashMap类型。如果不为Map、Collection以及Number，其他抽象类或接口会反序列化失败。

- 浮点数不允许为NaN或者inf(无穷值）

- Hyperson不支持对增加了注释的json字符串进行反序列化

