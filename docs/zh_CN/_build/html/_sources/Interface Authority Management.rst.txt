接口权限管理
^^^^^^^^^^^^^

功能概述
------------------
权限，即是资源和操作的一套组合。接口权限控制，即是某种用户/角色对接口这一资源对操作控制，权限控制中有三个重要元素： **人（用户/角色）、资源（要控制的对象）、操作（如何控制如增删改查）**。

当前的平台接口权限设计中，只有对于交易类的接口权限管理，其通过SDKCert进行证书验证，而RPC方法中很多是没有签名验签的，需要在RPC层做权限拦截控制，对用户访问接口进行权限验证，保证业务场景中用户访问链上信息的安全性。

节点管理员主要参与节点级别的权限管理，主要负责 **机构内部节点的接口访问权限管理**，节点管理员一般由节点运维人员担任，负责 **功能的开关管理以及初始规则设定**，节点管理员可以预先设置好对应的账户角色和接口权限管理规则，当接口权限管理的开关开启时，所有的查询请求（不包含发送交易的接口）都需要携带请求发送者的信息，即需要知道是哪个账户发送了此次请求，进行账户级别的接口访问控制，保障区块链系统接口访问安全。

**注意**：如果某一节点针对某一namespace设置了该节点的接口访问规则，则只对此节点中对应对namespace生效，对于此节点对其他namespace和其他节点是不生效的。


使用说明
------------------
接口权限管理的使用步骤如下：

1. **设置开关** （接口权限管理是节点级的，默认关闭）；
2. **角色管理** （为账户设置节点级的角色，角色的设定没有限制，不需要先创建角色再设置，直接为账户设置角色即可）；
3. **规则管理** （基于角色来设置管理规则，当开关开启时，规则生效）。


设置开关
>>>>>>>>>>>>>>>>>>>
接口权限管理的开关通过ipc命令进行管理。

ipc启动命令::

    ./flato（“flato”为趣链区块链平台新版英文简称） -s --ipc=hpc_1.ipc

通过此命令进入ipc操作页面。


开启开关
:::::::::::::::::::::

`auth namespace_name start` 命令即可开启当前节点中对应namespace的接口权限管理功能。

建议：节点运维人员在开启对应namespace的接口权限管理功能之前，最好预先设置好对应的账户角色和接口权限管理规则，开启后，规则生效。另外，当用户设置的规则发生死锁或其他不可控的情况（例如，没有用户可以访问接口），可由节点运维人员将开关关闭，重新设置好角色、规则后再开启。

另外，当接口权限管理的开关开启时，所有的请求（无论是查询类的还是发交易类的）都需要携带请求发送者的信息，即需要知道是那个账户发送了此次请求，对应的在sdk中也有提供相应的接口，在调用查询之前，设置使用的账户。

示例：通过ipc命令开启global的接口权限管理功能，其命令如下::

    auth global start

使用litesdk设置发送请求的账户，其接口如下::

    public class DefaultHttpProvider {
        // 设置发送请求时使用的账户
        public void setAccount(Account account);
        public static class Builder {
            /**
            * use inspector.
            * @param defaultAccount the account to send request.
            * @return {@link Builder}
            */
            public Builder enableInspector(Account defaultAccount);//设置默认账户
            // 由于sdk在创建HttpProvider时会查询当前节点的txVersion，此时无法使用提供的接口设置发送请求的账户
            // 因此，使用enableInspector设置默认的发送请求的账户，用于查询txVersion，需要注意的是，要保证此账户有权限访问此接口
        }
    }

示例：使用litesdk设置默认的发送请求的账户::

    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder()
            .setUrl(DEFAULT_URL)
            .enableInspector(account)
            .build();

关闭开关
:::::::::::::::::::::::::

`auth namespace_name stop` 命令即可关闭当前节点中对应namespace的接口权限管理功能。

示例：通过ipc命令关闭global的接口权限管理功能，其命令如下::

    auth global stop


角色管理
>>>>>>>>>>>>>>>>>>>>
此处描述的角色管理是指的账户与节点级角色的管理。

添加角色
:::::::::::::::::::::

通过auth_addRole接口，可为账户添加节点级角色。

示例：使用litesdk为账户 `bfa5bd992e3eb123c8b86ebe892099d4e9efb783` 添加角色 `accountReader` 和 `blockReader` ，其代码如下::

    String accountJson = "{\"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",\"algo\":\"0x03\",\"version\":\"4.0\",\"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",\"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
    private static String DEFAULT_URL = "localhost:8081";
    private Account account = Account.fromAccountJson(accountJson,"");
    // 1. build provider manager
    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(DEFAULT_URL).enableInspector(account).build();
    private ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);
    // 2. build service
    private AuthService authService = ServiceManager.getAuthService(providerManager);
    public void testAddRole() {
        List<String> roles = new ArrayList<>();
        roles.add("accountReader");
        roles.add("blockReader");
        Request<Response> request = authService.addRoles(“bfa5bd992e3eb123c8b86ebe892099d4e9efb783
        ”, roles);
        try {
            Response response = request.send();
            System.out.println(response);
        } catch (RequestException e) {
            System.out.println("add roles error:"+e.getMsg());
        }
    }

删除角色
:::::::::::::::::::::

通过auth_deleteRole接口，可以删除账户的节点级角色。

示例：使用litesdk为账户 `bfa5bd992e3eb123c8b86ebe892099d4e9efb783` 删除角色 `accountReader`  ，其代码如下::

    String accountJson = "{\"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",\"algo\":\"0x03\",\"version\":\"4.0\",\"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",\"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
    private static String DEFAULT_URL = "localhost:8081";
    private Account account = Account.fromAccountJson(accountJson,"");
    // 1. build provider manager
    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(DEFAULT_URL).enableInspector(account).build();
    private ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);
    // 2. build service
    private AuthService authService = ServiceManager.getAuthService(providerManager);
    public void testDeleteRole() {
        List<String> roles = new ArrayList<>();
        roles.add("accountReader");
        Request<Response> request = authService.deleteRoles(“bfa5bd992e3eb123c8b86ebe892099d4e9efb783
        ”, roles);
        try {
            Response response = request.send();
            System.out.println(response);
        } catch (RequestException e) {
            System.out.println("add roles error:"+e.getMsg());
        }
    }


查询账户角色
::::::::::::::::::::::::::

通过auth_getRole接口，可以查询账户的角色信息。

示例：使用litesdk查询账户 `bfa5bd992e3eb123c8b86ebe892099d4e9efb783` 的角色，其代码如下::

    String accountJson = "{
    \"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",
    \"algo\":\"0x03\",
    \"version\":\"4.0\",
    \"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",
    \"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
    private static String DEFAULT_URL = "localhost:8081";
    private Account account = Account.fromAccountJson(accountJson,"");
    // 1. build provider manager
    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(DEFAULT_URL).enableInspector(account).build();
    private ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);
    // 2. build service
    private AuthService authService = ServiceManager.getAuthService(providerManager);
    public void testGetRole() {
        Request<RolesResponse> request = authService.getRolesByAddress(“bfa5bd992e3eb123c8b86ebe892099d4e9efb783”);
        try {
            RolesResponse response = request.send();
            System.out.println(response.getRoles());
        } catch (RequestException e) {
            System.out.println("get role error:"+e.getMsg());
        }
    }


查询角色账户
::::::::::::::::::::::

通过auth_getAddress接口，可以查询拥有某角色的账户信息。

示例：使用litesdk查询拥有 `accountReader` 角色的账户，其代码如下::

    String accountJson = "{
    \"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",
    \"algo\":\"0x03\",
    \"version\":\"4.0\",
    \"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",
    \"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
    private static String DEFAULT_URL = "localhost:8081";
    private Account account = Account.fromAccountJson(accountJson,"");

    // 1. build provider manager
    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(DEFAULT_URL).enableInspector(account).build();
    private ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);

    // 2. build service
    private AuthService authService = ServiceManager.getAuthService(providerManager);
    public void testGetAddress() {
        Request<AddressesResponse> request = authService.getAddressByRole("accountReader");
        try {
            AddressesResponse response = request.send();
            System.out.println(response.getAddresses());
        }catch (RequestException e) {
            System.out.println("get address error:"+e.getMsg());
        }
    }

查询所有角色
:::::::::::::::::::::::

通过auth_getAllRoles接口，可以查询所有的角色信息。

示例：使用litesdk查询所有的角色信息，其代码如下::

    String accountJson = "{
    \"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",
    \"algo\":\"0x03\",
    \"version\":\"4.0\",
    \"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",
    \"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
    private static String DEFAULT_URL = "localhost:8081";
    private Account account = Account.fromAccountJson(accountJson,"");

    // 1. build provider manager
    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(DEFAULT_URL).enableInspector(account).build();
    private ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);
    
    // 2. build service
    private AuthService authService = ServiceManager.getAuthService(providerManager);
    public void testGetAllRole() {
        Request<RolesResponse> request = authService.getAllRoles();
        try {
            RolesResponse response = request.send();
            System.out.println(response.getRoles());
        }catch (RequestException e) {
            System.out.println("get all roles error:"+e.getMsg());
        }    
    }

规则管理
>>>>>>>>>>>>>>>>>>>>
此处的规则管理描述的是对节点级接口权限管理规则进行管理。

设置规则
:::::::::::::::::::

通过auth_setRules接口，设置节点的接口权限管理规则。

示例：通过litesdk设置规则，只有拥有 `accountReader` 角色的账户才能访问 `account` 类的接口，其代码如下::

    String accountJson = "{
    \"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",
    \"algo\":\"0x03\",
    \"version\":\"4.0\",
    \"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",
    \"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
    private static String DEFAULT_URL = "localhost:8081";
    private Account account = Account.fromAccountJson(accountJson,"");
    
    // 1. build provider manager
    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(DEFAULT_URL).enableInspector(account).build();
    private ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);
    
    // 2. build service
    private AuthService authService = ServiceManager.getAuthService(providerManager);
    public void testSetRules() {
        List<InspectorRuleParam> rules = new ArrayList<>();
        List<String> authorizedRoles = new ArrayList<>();
        authorizedRoles.add("accountReader");
        List<String> methods = new ArrayList<>();
        methods.add("account_*");
        rules.add(new InspectorRuleParam.Builder().allowAnyone(false).authorizedRoles(authorizedRoles).forbiddenRoles(forbiddenRoles).methods(methods).build());
        Request<Response> request = authService.setRules(rules);
        try {
            Response response = request.send();
            System.out.println(response);
        } catch (RequestException e) {
            System.out.println("set rules error"+ e.getMsg());
        }    
    }

读取规则
:::::::::::::::::::::

通过auth_getRules接口，查询节点的接口权限管理规则。

示例：通过litesdk设置规则，查询节点接口权限管理规则，其代码如下::

    String accountJson = "{
    \"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",
    \"algo\":\"0x03\",
    \"version\":\"4.0\",
    \"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",
    \"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
    private static String DEFAULT_URL = "localhost:8081";
    private Account account = Account.fromAccountJson(accountJson,"");
    
    // 1. build provider manager
    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(DEFAULT_URL).enableInspector(account).build();
    private ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);
    
    // 2. build service
    private AuthService authService = ServiceManager.getAuthService(providerManager);
    public void testGetRules() {
        Request<InspectorRulesResponse> request = authService.getRules();
        try {
            InspectorRulesResponse response = request.send();
            System.out.println(response.getRules());
        } catch (RequestException e) {
            System.out.println("delete roles error:"+e.getMsg());
        }    
    }


操作实例
-------------------

场景介绍
>>>>>>>>>>>>>>>>>>>>
限制特定的账户才能访问特定的接口，具体步骤如下：

1. 为账户 `bfa5bd992e3eb123c8b86ebe892099d4e9efb783`设置角色 `accountReader`;
2. 设置规则，拥有 `accountReader`  或者 `accountManager` 角色的用户可以访问 `account` 相关的查询接口;
3. 开启接口权限管理（开启后也可以管理角色和规则）;
4. 使用账户 `bfa5bd992e3eb123c8b86ebe892099d4e9efb783`访问 `account_getBalance` 接口成功，其他没有相应角色的账户访问提示没有权限。


具体操作
>>>>>>>>>>>>>>>>>>>>>

::

    String accountJson = "{
    \"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",
    \"algo\":\"0x03\",
    \"version\":\"4.0\",
    \"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",
    \"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
    private static String DEFAULT_URL = "localhost:8081";
    private Account account = Account.fromAccountJson(accountJson,"");
    
    // 1. build provider manager
    private DefaultHttpProvider defaultHttpProvider = new DefaultHttpProvider.Builder().setUrl(DEFAULT_URL).enableInspector(account).build();
    private ProviderManager providerManager = ProviderManager.createManager(defaultHttpProvider);
    
    // 2. build service
    private AuthService authService = ServiceManager.getAuthService(providerManager);
    private AccountService accountService = ServiceManager.getAccountService(providerManager);
    public void test() throws RequestException {
        // add role
        List<String> roles = new ArrayList<>();
        roles.add("accountReader");
        Request<Response> request = authService.addRoles("bfa5bd992e3eb123c8b86ebe892099d4e9efb783", roles);
        Response response = request.send();
        System.out.println(response);
        
        // set rules
        List<InspectorRuleParam> rules = new ArrayList<>();
        List<String> authorizedRoles = new ArrayList<>();
        authorizedRoles.add("accountManager");
        authorizedRoles.add("accountReader");
        List<String> methods = new ArrayList<>();
        methods.add("account_*");
        rules.add(new InspectorRuleParam.Builder().allowAnyone(false).authorizedRoles(authorizedRoles).methods(methods).build());
        request = authService.setRules(rules);
        response = request.send();
        System.out.println(response);
        
        // use bfa5bd992e3eb123c8b86ebe892099d4e9efb783 getBalance
        String accJson = "{\"address\":\"0xfc546753921c1d1bc2d444c5186a73ab5802a0b4\",\"algo\":\"0x03\",\"version\":\"4.0\",\"publicKey\":\"0x04a06184e6617da8183b194497688eb1395fbef9be58d3c41fadbc45c0d1273c704f0dab0a87e794233cfe331bf618b5258f1455978bd9d94190f70db559043d4e\",\"privateKey\":\"fed0f46b931f24740fe351d45ac6bb5a88d74ef851ecf51d15615fabbcf16184\"}";
        Account ac = Account.fromAccountJson(accountJson,"");
        defaultHttpProvider.setAccount(ac);
        Request<BalanceResponse> balance = accountService.getBalance(account.getAddress());
        BalanceResponse send = balance.send();
        System.out.println(send.getBalance());
    }

    

