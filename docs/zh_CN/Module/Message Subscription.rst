.. _Message-Subscription:

消息订阅MQ
^^^^^^^^^^^^^^^^^

功能描述
---------------------

MQ介绍
>>>>>>>>>>>>>>>>>>>>>>>>>>
平台MQ服务底层依赖 RabbitMQ（KafkaMQ）服务，用户需要独立与平台启动一个rabbitmq（kafkamq）服务器，我们将rabbitmq（kafkamq）所在的服务器称为rabbitmq-broker（kafkamq-broker），平台提供的MQ服务相当于一个生产者客户端，负责将平台消息推送到rabbitmq-broker（kafkamq-broker）。
消息的消费者作为一个消费者客户端，与rabbitmq-broker（kafkamq-broker）建立连接，等待从rabbitmq-broker（kafkamq-broker）推送出来的消息。

**RabbitMQ术语说明**

.. list-table:: 
 :widths: 30 70
 :header-rows: 1

 * - 术语名称
   - 解释说明
 * - channel
   - 与rabbitmq-broker建立连接后，可以通过connection创建channel，一个channel对象相当于一个连接会话，是客户端（生产者，消费者）直接用来推送或消费消息的媒介
 * - queue
   - 由客户端创建，通过绑定路由键routingkey来决定感兴趣的消息类型，相应的消息会在rabbitmq-broker内部被分发到queue中等待被消费  
 * - routingkey
   - 路由键，标识一种消息
 * - exchanger
   - rabbitmq-broker内部直接接收生产者消息的组件，通过名称标识，exchanger负责将消息按照routingkey分发到相应的queue中 

**KafkaMQ术语说明**

.. list-table:: 
 :widths: 30 70
 :header-rows: 1

 * - 术语名称
   - 解释说明
 * - topic
   - 逻辑概念，消息的类别，同一topic的消息可以认为是同一类，在kafka集群中会被作为一类存储。每一条生产者推送的消息，只可以属于一个topic
 * - partition
   - partition是一个物理概念，是kafka集群真正存储数据的地方。一个topic可以包含多个partition，该topic下的消息可以在partition之间做负载均衡。一个partition上存储的消息是有序的
 * - replica
   - 这是kafka集群内部的概念，可以认为replica是partition的备份

使用步骤
>>>>>>>>>>>>>>>>>>>>>>>>>>>

1. 独立启动mq服务，一般来讲，只需要一个节点连接mq服务端。需要连接mq服务端的节点只需要在节点配置中启用mq服务，并配置mq服务的地址即可。剩下的操作都可以来API接口进行；
2. 用户向平台注册自己的消息队列queue，指明queue感兴趣的路由键集合；平台会创建队列同时启动服务，将queue的名称与交换机名称exchanger返回给用户；
3. 用户正常使用平台发送交易，当有routingkey相应的事件产生时，消息会被MQ服务自动推送至broker，用户可以通过命令行或管理页面查询；
4. 消费者消费消息；
5. 用户可以查询当前节点上注册过的所有的queue名称；
6. 用户可以删除queue，但需要提供注册时平台返回的exchanger。

安装及初始化
-----------------------------

配置说明
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

- ``service.service_name``：节点开启的外部服务。

开启MQ服务的示例如下::

    [[service]]
        service_name = "MQ"

- ``mq.broker``：MQ的Broker类型，可选项为 ``rabbit`` 和 ``kafka``；

示例如下::

    [mq.broker]
    type = "rabbit"

- ``mq.rabbit.url``：RabbitMQ地址。

示例如下::

    [mq.rabbit]
    url = "amqp://guest:guest@localhost:5672/"

- ``mq.kafka.urls``：Kafka地址列表；
- ``mq.kafka.partitionNums``：每个kafka实例下分区(partition)的数量；
- ``mq.kafka.replicaFactor``：每个kafka实例在zookeeper下的备份数量。

示例如下::

    [mq.kafka]
    urls = ["localhost:9092"]

    [mq.kafka.partitionNums]
    "localhost:9092" = 1

    [mq.kafka.replicaFactor]
    "localhost:9092" = 1

安装RabbitMQ
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

**下载安装**

1. RabbitMQ官方下载地址： http://www.rabbitmq.com/download.html （如未安装Erlang，请首先安装Erlang）
2. 通过命令行工具直接安装（brew或yum等）

**启动MQ服务**

1. 当前窗口启动： `rabbitmq-server`
2. 如需开启插件，在启动前设置： `rabbitmq-plugins enable rabbitmq_management`
3. 浏览器打开管理界面： http://localhost:15672/
4. 默认用户名和密码：guest,guest

**使用docker容器启动**

启动带管理界面的MQ::

    docker run -d --name myrabbitmq -p 5672:5672 -p 15672:15672 docker.io/rabbitmq:3-management

启动不带管理界面的MQ::

    docker run -d --name myrabbitmq -p 5672:5672 -p 15672:15672 docker.io/rabbitmq:latest


安装KafkaMQ
>>>>>>>>>>>>>>>>>>>

请参考官方文档：
https://kafka.apachecn.org/quickstart.html


使用说明
---------------------------

完整操作流程
>>>>>>>>>>>>>>>>>>>>>>>>>>>>

1. 调用informNormal，通知节点与rabbitmqMQ建立连接（kafka可跳过）
2. 注册队列
3. 正常发送交易，产生消息
4. 消费者等待rabbitmq-broker推送的消息
5. 当queue不再使用时，删除queue


JSONRPC接口
>>>>>>>>>>>>>>>>>>>>>>>>>>

MQ服务对外提供的API有六种，分别用于注册queue、删除queue、查询指定节点注册的所有queue、通知平台rabbitmq-broker正常工作、查询当前节点最新的exchangerName，以及删除指定名称的exchanger。

**MQ服务接口详情请参考** https://docs.hyperchain.cn/document/detail?type=1&id=14


注册queue
:::::::::::::::::::

平台接口名称为 **mq_register** ，参数为：RegisterMeta结构，MQ服务将创建与发送目标节点绑定的exchanger。

目前Signature字段暂时为预留字段，平台不会对Signature字段做处理。

目前可选的routingKey字段包括：

::

    - MQBlock
        - 区块事件，包含区块号、区块哈希、交易列表等
    - MQLog
        - 合约内部定义的抛出事件，包含合约地址、抛出事件的具体内容等
    - MQException
        - 共识状态异常事件，包含viewChange（主节点切换）、systemBusy（节点打包失败）、recovery（节点数据恢复）、stateUpdate（节点状态更新）、confChange（节点增删）等
    - MQFileInfo
        - 可信文件共享相关事件，包含文件内容、文件哈希等
    - SignedMQBlock
    - SignedMQLog
    - SignedMQException
    - SignedMQFileInfo

其中以 "Signed" 字段开头routingKey，表示该事件推送时，需要附带节点的ECert签名，具体的签名方式为将事件做json序列化，然后用SHA3算法做哈希，再用Ecert签名。

::

    curl localhost:8081 --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"mq_register",
    "params":[{
        "addresses":[],
    "toBlock":"",
    "signature":"010492bf5d92d97bfbf6ab67eb6b5ceadee91a73a2e33876a339b3d25e23c9cd2322104eef8a9191bab5728f401adcd95a00f7e3f802bb374598f47593fc82bb0bd23045022047204b8b763f7dab117e530a06c0d9ac6ef2a0b77a1179be46f75ca0efcc450c02210098e9836dc9bbf0d8a39206f7c9bdc3148e3805e79e58d98df3cd0cbe2e42aff6",
    "isVerbose":true,
    "topics":[],
    "routingKeys":["MQBlock",
    "MQLog",
    "MQException"],
    "fromBlock":"",
    "queueName":"gw_node3queue",
    "from":"B2CC762775FC1AA7486AA2FCF0D7885D4EEE4DA2"}],
    "id":1}'

返回值示例::

    {
        "jsonrpc":"2.0",
        "namespace":"global",
        "id":1,
        "code":0,
        "message":"SUCCESS",
        "result":"{\"queueName\":\"node1queue1\",
        \"exchangerName\":\"global_fa34664e_1529920070165345052\"}"
    }

其中的result字段内容为：队列名称(queueName)，exchanger名称(exchangerName)；如果发生异常，这两个返回值为空字符串。

在使用此接口注册订阅了带签名的事件时，需要注意： **平台务必开启Ecert，否则注册不成功**。

删除queue
::::::::::::::::::::

平台接口名称为 **mq_unRegister** ，参数为：队列名称，exchanger名称，账户地址，签名（目前只用到前三个）。

::

    curl localhost:8081 --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"mq_unRegister",
    "params":["gw_node4queue",
    "global_34d29974_1533616771290750181",
    "B2CC762775FC1AA7486AA2FCF0D7885D4EEE4DA2",
    "010492bf5d92d97bfbf6ab67eb6b5ceadee91a73a2e33876a339b3d25e23c9cd2322104eef8a9191bab5728f401adcd95a00f7e3f802bb374598f47593fc82bb0bd2304502205c7a9e27263fb39ba5e3c6ec1d1fde6bff113577087579a7c23a9c167f93ceaf022100a197f0b514be1dd78ddc480af4372425a26f5b20832b83707e77ae58e1d4994e"],
    "id":1}'

返回值示例::

    {"jsonrpc":"2.0",
    "namespace":"global",
    "id":1,
    "code":0,
    "message":"SUCCESS",
    "result":"{
    "count":1,
    "success":true,
    "error":null}"}

其中result字段内容为：队列中还未消费掉的消息数目(count)，是否删除成功(success)，以及过程中产生的error信息(error，正常情况下为"null")，如果exchangerName与注册queue时的返回值不匹配，会删除失败。

查询所有的queueName
:::::::::::::::::::::::::::::

平台接口名称为 **mq_getAllQueueNames** ，不需要参数。

::

    curl localhost:8081 --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"mq_getAllQueueNames",
    "params":[],
    "id":1
    }'

返回值示例::

    {
    "jsonrpc":"2.0",
    "namespace":"global",
    "id":1,
    "code":0,
    "message":"SUCCESS",
    "result":["node1queue1"]}

通知节点rabbitmq-broker正常工作
::::::::::::::::::::::::::::::::::::

**注意：该接口仅再以rabbitmq作为mq服务时生效**

平台接口为 **mq_informNormal** ，不需要参数。

::

    curl localhost:8081 --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"mq_informNormal",
    "params":[""],
    "id":1}'

返回值示例::

    {
        "jsonrpc":"2.0",
        "namespace":"global",
        "id":1,
        "code":0,
        "message":"SUCCESS",
        "result":"{
        "success":true,
        "error":null
        }"
    }

查询节点的exchangerName
:::::::::::::::::::::::::::::

**注意：该接口仅再以rabbitmq作为mq服务时生效**

平台接口为 **mq_getExchangerName** ，不需要参数。

::

    curl localhost:8081 --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"mq_getExchangerName",
    "params":[],
    "id":1}'

返回值示例::

    {
        "jsonrpc":"2.0",
        "namespace":"global",
        "id":1,
        "code":0,
        "message":"SUCCESS",
        "result":"global_fa34664e_1530265355500005853"
    }

删除exchanger
:::::::::::::::::::::

**注意：该接口仅再以rabbitmq作为mq服务时生效**

平台接口为 **mq_deleteExchanger** ，参数为exchanger名称。

::

    curl localhost:8081 --data '{
    "jsonrpc":"2.0",
    "namespace":"global",
    "method":"mq_deleteExchanger",
    "params":["global_34d29974_1532417825700513378"],
    "id":1
    }'

返回值示例::

    {
        "jsonrpc":"2.0",
        "namespace":"global",
        "id":1,
        "code":0,
        "message":"SUCCESS",
        "result":"{
        "success":true,
        "error":null}"
    }


SDK接口
>>>>>>>>>>>>>>>>>>>

订阅
::::::::::::::::

**方法名称**：registerQueue

**方法说明**：注册queue队列，只有调用了registerQueue，服务才能够正常工作

**输入参数**：

- RegisterMeta结构体

**返回值**：

- exchanger名称
- queue名称


**调用示例**::

    public String registerQueue(int id){
      ...
      ArrayList<String> array = new ArrayList<String>();
      array.add("SignedMQBlock");
      array.add("MQLog");
      array.add("MQException");
      String qname = "node1queue";
      MQParam blockInfoParam = new MQParam.Builder().
                //队列订阅相关的参数
                regRoutingKeys(array).  //表示queue绑定的routingKey集合
                regQueueName(qname).    //表示队列名称，已经使用过的队列名称，没有手动删除前，不可以重复使用
                // MQBlock事件订阅需要的参数
                setVerbose(false).      //表示推送区块时是否需要推送交易列表，true表示推送交易列表
                // MQlog事件订阅需要的参数
                fromBlock("1").         //表示需要推送log事件的起始区块号
                toBlock("2").           //表示需要推送log事件的终止区块号
                addAddress("0x1258b70e2d620805a5351d8553ca76606a0c6fc5").   //表示log事件需要匹配的合约地址
                addAddress("0xea22068f8ef04f6c4b5b73e55b51c4c758c7276e").   //可多次调用添加多个合约地址
                addTopics(new String[]{"topicA", "topicB"}).    //表示log事件需要匹配的topic集合
                addTopics(new String[]{"topicC", "topicD"}).    //可多次调用添加多个topic数组
				// 是否需要以延迟模式推送，若为true，则可更大程度上保证消息的准确性和顺序性，但消息会在平台checkpoint点之后才会被推送
                .delayMode(false)
				//请求发起人自身信息
                setFrom(TEST_FROM).     //表示请求发起者的账户地址，用于权限控制
                build(TEST_ACCOUNT_JSON, TEST_PASSWD);  //表示请求者以RegisterMeta结构体为内容，使用自己的私钥做sm2签名，格式为16进制字符串
      String result = hyperchain.registerQueue(blockInfoParam, SPEC_ID);
      return result;
    }

    /** 输出结果为：
    {
        "queueName": "node1queue",
        "exchangerName": "global_fa34664e_1529591668938108811"
    }**/

**MQ中事件Demo**

以订阅了SignedMQBlock为例，在rabbitmq-broker中收到的事件内容为::

    {
	    "timestamp": 1564475540778317000,
	    "type": "SignedMQBlock_2dca544db",
        "body": {
            "type": "SignedMQBlock_2dca544db",
            "name": "BlockInfo",
            "version": "1.5",
            "number": "1",
            "hash": "0x892d956394af2604ed47567d7999c837a8955444f42d0e6095d5d764cba20c95",
            "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
            "writeTime": 1564475540751617000,
            "avgTime": 44,
            "txcounts": 1,
            "merkleRoot": "0xb746f69f4c87eb4635f4458d06b1e929385f6664292d60ae3efa305728266ead",
            "txs": []
        },
	    "signature": "304502203b1dcc44c1415aa8f1be35a59f5588b5406965f859c333dd65d3d2ab0b65ad9002210082cdfb9f14abdfa4cbcaf66db1b1886c1440e27e1c93592abc3b51aff1e44638",
	    "cert": "2d2d2d2d2d424547494e2043455254494649434154452d2d2d2d2d0a4d4949422b6a43434161476741774942416749424154414b42676771686b6a4f50515144416a424d4d524d77455159445651514b457770496558426c636d4e6f0a59576c754d52597746415944565151444577316f6558426c636d4e6f59576c754c6d4e754d5241774467594456515171457764455a585a6c624739774d5173770a435159445651514745774a6153444167467730784f4441334d7a41774e4445324e546861474138794d5445344d4463774e6a41314d5459314f466f77544445540a4d4245474131554543684d4b53486c775a584a6a61474670626a45574d4251474131554541784d4e61486c775a584a6a614746706269356a626a45514d4134470a413155454b684d48524756325a5778766344454c4d416b474131554542684d43576b67775754415442676371686b6a4f5051494242676771686b6a4f50514d420a42774e43414154564c482f426a3753324e6c457677446d3049656e32773976667474304a306c6664704a3030772b3475386f56536e7265365057525a545948730a5243765066394f4c55547041574e7052694c517a706732714571627a6f3349776344414f42674e56485138424166384542414d43416751774a6759445652306c0a42423877485159494b7759424251554841774947434373474151554642774d424267497141775944675173424d41384741315564457745422f7751464d414d420a41663877445159445652304f424159454241454341775177466759444b6c59424241396f6558426c59326868615735665a574e6c636e5177436759494b6f5a490a7a6a304541774944527741775241496744384c394732765739726f56426f7a2f633445362f54702f57636763717a796c796f30587971474e706234434943775a0a554d346c424b33474c2f69617767593661425a4c2f6945446b66734b37794b6a69395a66457459750a2d2d2d2d2d454e442043455254494649434154452d2d2d2d2d0a"
    }

解订阅
:::::::::::::::

**方法名称**：unregisterQueue

**方法说明**：删除指定名字的queue，如果queue不存在或者给出了不匹配的excahngerName，都将删除失败

**输入参数**：

- queue名称(string)
- exchanger名称(string)
- 发起者地址(string)
- 签名(string)
- 节点ID(int)

**返回值**：

- queue删除时仍持有的消息数目
- 是否删除成功
- 过程中的错误信息

**调用示例**::

    public String unRegisterQueue(int id) {
      String qname = "node1queue";
      String exchName = "global_34d29974_1533616771290750181";
      String signature = new UnRegParam(qname, exchName).signWithSM2(TEST_ACCOUNT_JSON2, TEST_PASSWD);
      String result = hyperchain.unregisterQueue(qname, exchName, TEST_FROM2, signature, 1);
      return result;
   }
   /**
    输出结果为：{"count":0, "success":true, "error":"null"}
    ***/

查询指定节点应注册的所有的queue名称
::::::::::::::::::::::::::::::::::::::::

**方法名称**：getAllQueueNames

**输入参数**：

  - 节点ID(int)

**返回值**：

  - queue名称列表

**调用示例**::

    public String testGetAllQName(int id) {
      String result = hyperchain.getAllQueueNames(id);
      return result;  
   }
   /**
    输出结果为：["node1queue"]
    ***/

通知平台rabbitmq-broker正常工作
::::::::::::::::::::::::::::::::::

**方法名称**：informNormal

**方法说明**：该方法主要用于在broker宕机后，可以通过调用该方法，通知节点，节点会尝试建立连接以及恢复曾经注册的queue

**输入参数**：

  - rabbitmq-broker的完整url
  - 节点ID(int)

**返回值**：

  - 平台建立连接，队列恢复的结果

**调用示例**::

    public void testInformNormal(String brokerUrl, int id) throws Exception {
        // 空字符串表示使用平台默认url
        String result = hyperchain.informNormal(brokerUrl, id);
        System.out.println(result);
   }
   /**
    输出结果为：{"success":true,"error":null}
    ***/

查询指定节点exchanger的名称
:::::::::::::::::::::::::::::

**方法名称**：getExchName

**输入参数**：

  - 节点ID(int)

**返回值**：

  - exchanger名称

**调用示例**::

     public String testGetExchangerName(int id) throws Exception {
        String result = hyperchain.getExchName(id);
        return result;
   }
   /**
   输出结果为："global_fa34664e_1530265355500005853" 
   ***/

删除指定的exchanger
::::::::::::::::::::::::

**方法名称**：deleteExchanger

**输入参数**：

  - exchanger名称(String)
  - 节点ID(int)

**返回值**：

  - 是否删除成功

**调用示例**::

    public String testDeleteExchanger() throws Exception {
        ...
        String exchName = "global_34d29974_1532437541218895978";
        String result = hyperchain.deleteExch(exchName, SPEC_ID);
        return result
    }
   /***
    输出结果为："{"success":true,"error":null}"
   ***/

消息消费者Demo
::::::::::::::::::::

消费者程度调用receiveMsg()接口，可以消费相应的消息，外部程序可以通过getMessages()接口，获得所有消费者消费掉的消息记录

::

    public class HPCMQClient {
        private static String qname = "node1queue1"; // should read from properties
        private final ArrayList<String> messages = new ArrayList<String>();
        public HPCMQClient() {}
        public void receiveMsg() throws Exception{
            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost("localhost");
            factory.setUsername("guest");
            factory.setPassword("guest");
            factory.setPort(5672);   // should set your own hostname and port
            final Connection connection = factory.newConnection();
            final Channel channel = connection.createChannel();
            channel.queueDeclare(qname, true, false, false, null);
            final Consumer consumer = new DefaultConsumer(channel) {
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    String message = new String(body, "UTF-8");
                    System.out.println("[CLIENT] Received '" + message + "'");
                    messages.add(message);
                    System.out.println("[CLIENT] Done");
                }
            };
            channel.basicConsume(qname, true, consumer);
        }
        public ArrayList<String> getMessages() {
            return messages;
        }
    }

目前javasdk提供了对事件验签的功能，接口为MQUtil.verifyEvent，入参为从broker中直接接收到的事件。


订阅规则
>>>>>>>>>>>>>>>>>>>>>>
订阅规则依赖于一个RegisterMeta结构实现，该结构体表现为一个json格式的字符串作为参数传入RegisterQueue()方法，RegisterMeta中不同的字段应用于不同的事件订阅规则，具体如下::

    // 排表方式为：变量名称 | 变量类型 |json格式key | 字段含义
    type RegisterMeta struct {
        //队列订阅相关的参数
        RoutingKeys    List<string> // `json:"routingKeys"` 表示queue绑定的routingKey集合
        QueueName      string       // `json:"queueName"`   表示队列名称，已经使用过的队列名称，没有手动删除前，不可以重复使用
        //请求发起人自身信息
        From           string   // `json:"from"`      表示请求发起者的账户地址，用于权限控制
        Signature      string   // `json:"signature"` 表示请求者以RegisterMeta结构体为内容，使用自己的私钥做sm2签名，格式为16进制字符串
        // MQBlock事件订阅需要的参数
        IsVerbose      boolean   // `json:"isVerbose"` 表示推送区块时是否需要推送交易列表，true表示推送交易列表
        // MQlog事件订阅需要的参数
        FromBlock      string           // `json:"fromBlock"` 表示需要推送log事件的起始区块号
        ToBlock        string           // `json:"toBlock"`   表示需要推送log事件的终止区块号
        Addresses      List<String>     // `json:"addresses"` 表示log事件需要匹配的合约地址集合
        Topics         List<String[]>   // `json:"topics"`    表示log事件需要匹配的topic集合
        // 该字段为flato（“flato”为趣链区块链平台新版英文简称）1.0.5新增，可选择队列推送模式
        // 若为true，则可更大程度成保证消息的正确性和顺序性（例如砍块造成的被砍区块对应的消息推送以及消息可能发生乱序的情况），
        // 但是需要以延迟推送为代价，消息需要在平台到达checkpoint之后才会被推送到broker
        Delay bool `json:"delay"`
        //目前Exception相关的操作不需要用户提供参数，订阅了Exception事件后，所有的异常都会主动推送
    }


异常处理
----------------------

broker未启动
>>>>>>>>>>>>>>>>>>>>>>
broker未启动时，在不注册队列，不需要消息推送时，平台正常输出日志，在用户调用API发起请求注册队列时，平台会打印错误日志。

broker宕机
>>>>>>>>>>>>>>>>>>>>>>>
rabbitmq-broker端单方面宕机，宕机之后，平台可以继续的提交区块，但会打印相关的推送消息失败的日志；

在rabbitmq-broker重启恢复后，需要用户再次注册队列，如果注册了与异常发生前同名的队列，则可以自动恢复broker宕机过程中的消息，平台会打印相关的日志提示消息已经恢复。

节点宕机
>>>>>>>>>>>>>>>>>>>>>>>>
节点宕机重启后，需要用户调用InformNormal接口，MQ服务会自动恢复队列，权限映射关系，并将缓存的数据推送到broker。


注意事项
-----------------------

1. Log事件中data的编码方式为 **hex** ，按照相应格式解码，即可得到原始的字节数组；
2. 在启动时使用kafka作为实现时，启动kafka服务之前需要保证kafka带着如下两个配置项启动：

在配置文件server.properties中增加如下两个配置项

.. list-table:: 
 :widths: 30 40 30
 :header-rows: 1

 * - 配置项
   - 含义
   - 值
 * - auto.create.topics.enable
   - 是否自动创建topic
   - false
 * - delete.topic.enable
   - 是否直接删除topic
   - true