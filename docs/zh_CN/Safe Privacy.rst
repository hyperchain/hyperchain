隐私保护
^^^^^^^^^^

数据作为新型生产要素，是数字化、网络化、智能化的基础，已快速融入生产、分配、流通、消费和社会服务管理等各个环节，深刻改变着生产方式、生活方式和社会治理方式。区块链因为有去中心化、不可篡改和支持智能合约等特性，成为数据交易、流通、使用等环节的重要和富有潜力的平台和载体，但是数据上链后的隐私保护问题亟待解决。

可验证计算
-------------

可验证计算（Verifiable Computing，VC）是将计算任务外包给第三方算力提供者，（不受信任的）第三方算力提供者需要在完成计算任务的同时，提交一份关于计算结果的正确性证明。

平台支持链接基于零知识证明或者TEE的可验证计算节点进行计算证明上链。可验证计算节点旨在解决区块链在隐私数据使用中面临的隐私保护问题，主要思路可以总结为“链下计算-链上验证”。计算节点在链下进行计算时，可以使用本地的隐私数据，而本地隐私数据仅仅是参与计算，并不会离开所有方控制。通过隐私合约，链上在不获取数据本身的同时使用到了数据的价值。平台还提供了统一的API接口，降低用户使用难度，使用者无需感知复杂的TEE和零知识证明的细节，只需专注于自己业务逻辑部分的开发。

零知识证明
>>>>>>>>>>>>>>

零知识证明是指证明方能够在不向验证方提供任何有用的信息的情况下，使验证方相信某个陈述是正确的。证明者（Prover）有可能在不透露具体数据的情况下让验证者（Verifier）相信数据的真实性。

零知识证明具备以下三个基本特性：

* **完整性**：完整性意味着陈述的真实性，使验证者相信证明者拥有所需的输入这一事实，即证明者能够说服验证者接受一条正确的陈述。
* **可靠性**：不诚实的证明者无法说服验证者在他们的陈述为假时他们有所需的输入这一事实，即证明者不能说服验证者接受一个错误的陈述。
* **零知识性**：ZKP 中零知识方面的主要指示指向信息的不公开。无论该陈述是真是假，验证者都不得了解有关该信息的任何信息

基于上述特性，零知识证明的优势主要包括：1.证明方式简单，用户可以以简单的形式利用零知识证明某个结果，而不需要复杂的加密方法；2.安全性更高，基于密码学算法，正确性、安全性及零知识证明不依赖第三方或外部条件；3.保护数据安全，避免隐私泄露，用户在与其他机构进行信息交互的过程中不必担心数据外泄。

平台支持使用Groth16算法或PLONK算法对计算内容生成零知识证明。其具备以下特点：

* 深度的性能优化：提供从计算到证明/验证算法的高效实现，零知识证明验证时间小于1ms，保障高性能；
* 基于电路结构的计算：可验证计算节点支持高级语言定义电路结构。传统实现方法通过高级语言分别处理得到用于计算的通用编程语言和用于证明的电路结构，利用两部分分别完成计算和证明功能，较为繁琐，而平台采用了不同的思路，直接在转化后的电路基础上进行计算，避免了向其他通用编程语言的转化和编译。

.. image:: ../../images/zero_knowledge.png

在实际应用场景中，通过零知识证明可以使用户在不透露具体信息的情况下证明其信息的可信任性。例如，抵押贷款申请人可以证明他们的收入在可接受的范围内，而无需透露他们的确切工资；投票场景中，公共区块链记录了投票，则不需要值得信赖的第三方来验证结果，故零知识证明也常用于匿名投票系统，因为合格的选民可以在不透露身份的情况下证明他们投票的权利。


TEE可信执行环境
>>>>>>>>>>>>>>>>>

TEE全名为可信执行环境，是计算平台上由软硬件方法构建的一个安全区域，可保证在安全区域内加载的代码和数据的机密性和完整性。其目标是确保一个任务按照预期执行，保证初始状态的机密性、完整性，以及运行时状态的机密性、完整性。可信执行环境的最本质属性是隔离,通过芯片等硬件技术并与上层软件协同对数据进行保护,且同时保留与系统运行环境之间的算力共享。

TEE具有如下优势：

* 全链路可信，基于TEE中的所有数据运算交互全链路加密；
* 性能强大，能够解决复杂多样的业务场景诉求；
* 开发成本低，能够迅速根据需求产出满足业务诉求的产品。

.. image:: ../../images/TEE.png

通用计算芯片厂商发布的TEE技术方案包括X86指令集架构的Intel SGX(Intel Software Guard Extensions)技术、AMDSEV(Secure Encrypted Virtualization)技术以及高级RISC机器(Advanced RISC Machine,ARM)指令集架构的 Trustzone技术。而国内计算芯片厂商推出的TEE功能则包括兆芯 ZX-TCT(TrustedComputing Technology)技术、海光CSV(China Security Virtualization)技术，以及 ARM架构的飞腾、鯤鹏也已推出自主实现的 TrustZone功能。由此诞生了很多基于以上产品的商业化实现方案,如百度 MesaTEE、华为 iTrustee 等。Intel 的 SGX 和 ARM 的 TrustZone 处于 TEE 硬件的垄断地位。 TrustZone 在2008 年推出,而 SGX 最早在 2013 年推出,目前市场上可信执行环境的商业化落地都是基于 TrustZone 或 SGX 的解决方案。

目前平台已支持SGX和CSV，同时支持TEE插件化，将TEE以密码机中间件的形式加载到平台中，实现可插拔式TEE加密。


账本加密
------------

账本加密即针对用户的账户信息和业务数据进行按需加密操作，确保数据写入磁盘后的安全性和隐私性。

目前账本加密主要分为对Filelog和Multicache（LevelDB）类型的数据库进行加密。Filelog存储区块（Block）、日志（Journal）、回执（Receipt）数据，Multicache主要存储账本（State）、账户（Account）、链级元数据（Meta）、Did以及Credential数据。

目前平台支持的账本加密方式主要包含了AES、DES、SM4、TEE。区块链首次启动后，开启账本加密的默认算法为AES加解密，当用户需要使用其他算法时，无需对节点进行改写配置并重启节点，只需要用户通过SDK发送一笔交易，即可完成账本加密算法的变更。

