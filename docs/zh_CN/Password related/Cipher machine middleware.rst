.. _Cipher-machine-middleware:

密码机中间件用户开发手册
^^^^^^^^^^^^^^^^^^^^^^^^^^

1. 引言
=============

1.1 编写目的
-----------------

密码机中间件的接口实现与测试涉及部分硬件环境，本文档为密码机中间件的开发与测试提供一定的指导。

2. 开发guide
================

本文档通过结合接口定义与实现demo，为密码机中间件的开发提供一定的指导。

注意：

1. plugin应该仅依赖 `__github.com/meshplus/crypto__ <https://github.com/meshplus/crypto>`_ 和go官方包进行开发

2. go版本和配套使用的平台版本保持一致，目前(flato1.0.6及之前)是go1.15.6

3. golang编译插件需要使用--trimpath编译选项

4. 开发的插件名称(动态链接库的名称)应该有前缀"plugin_"

2.1 接口定义与说明
--------------------

接口包可以从 `__https://github.com/meshplus/crypto__ <https://github.com/meshplus/crypto>`_ 获取

2.1.1 插件对外提供的方法
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

     func GetLevelArray() []crypto.Level{}


2.1.2  hash、随机数生成器、加解密等工厂接口
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

    //Level priority of plugins
    type Level interface {
       GetLevel() ([]int, uint8)
    }

    //PluginRandomFunc random function
    type PluginRandomFunc interface {
       Level
       Rander() (io.Reader, error)
    }

    //PluginHashFunc hash function
    type PluginHashFunc interface {
       Level
       GetHash(mode int) (Hasher, error)
    }

    //PluginCryptFunc symmetric encryption and decryption function
    type PluginCryptFunc interface {
       Level
       GetSecretKey(mode int, pwd, key []byte) (SecretKey, error)
    }

    //PluginEncKeyFunc asymmetric encryption function
    type PluginEncKeyFunc interface {
       Level
       //enter a raw publicKey and mod, return a VerifyKey
       //see GetVerifyKey's comment for the meaning of a raw publicKey
       GetEncKey(key []byte, mode int) (EncKey, error)
    }

    //PluginDecKeyFunc asymmetric decryption function
    type PluginDecKeyFunc interface {
       Level
       //enter index or raw privat key to generate a DecKey, key will not be persistent
       // a raw private key is a big integer
       GetDecKey(key []byte, mode int) (DecKey, error)
       //enter raw private key, return index, key should be persistent
       ImportDecKey(key []byte, mode int) (index []byte, err error)
    }

    //PluginCreateDecKeyFunc create decryption function
    type PluginCreateDecKeyFunc interface {
       Level
       //generate a DecKey, and return index or key in raw form
       //if persistent is true, key should be persistent and return index
       // or persistent is false, key should be persistent and output should in raw form
       CreateDecKey(persistent bool, mode int) (index []byte, k DecKey, err error)
    }

2.1.3签名验签接口
>>>>>>>>>>>>>>>>>>>>

 ::

    //PluginSignFunc sign function
    type PluginSignFunc interface {
       Level
       //enter index or raw privat key to generate a SignKey, key will not be persistent
       // a raw private key is a big integer
       GetSignKey(key []byte, mode int) (SignKey, error)
       //enter raw private key, return index, key should be persistent
       ImportSignKey(key []byte, mode int) (index []byte, err error)
    }

    //PluginVerifyFunc verify function
    type PluginVerifyFunc interface {
       Level
       //enter a raw publicKey and mod, return a VerifyKey
       //a raw publicKey means:
       // 1) for sm2, key is 65bytes and in 0x04||X||Y form, see GMT0009-2012 7.1
       //      http://www.gmbz.org.cn/main/viewfile/2018011001400692565.html may help
       // 2) for ecdsa, key is in 0x04||X||Y. The length depends on the curve, for example,
       //    65 bytes for secp256k1 and 133 for secp521r1, see 2.3.3 in [SEC1] uncompressed form.
       //    https://www.rfc-editor.org/rfc/rfc5480.txt may help
       // 3) for rsa, key is in PKCS#1 form, see RFC2313 RSAPublicKey
       //    RSAPublicKey ::= SEQUENCE {
       //            modulus INTEGER, -- n
       //            publicExponent INTEGER -- e }
       //    https://www.rfc-editor.org/rfc/rfc2313.txt may help
       GetVerifyKey(key []byte, mode int) (VerifyKey, error)
    }

    //PluginCreateSignFunc create SignKey
    type PluginCreateSignFunc interface {
       Level
       //generate a SignKey, and return index or key in raw form
       //if persistent is true, key should be persistent and return index
       // or persistent is false, key should be persistent and output should in raw form
       CreateSignKey(persistent bool, mode int) (index []byte, k SignKey, err error)
    }


PluginSignFunc接口实现密钥的签名功能，该接口的两个关键方法为GetSignKey和ImportSignKey。

1.GetSignKey：获取签名密钥

    - key参数的内容由插件解释，flato会从私钥索引文件中读取相关内容传递给插件。关于私钥索引文件的约定格式见下文

    - mode表示对应的算法，对于不支持的算法可以返回crypto包中定义的ErrNotSupport错误

    - 私钥索引文件的约定格式。所谓私钥索引文件是用于替代私钥文件的占位文件。该文件只有一行文本内容，由三部分组成，三部分间用空格分割，样例如下::

     plugin sm2 3081a40201010430bdb9839c08ee793d1157886a7

**第一部分** 是固定开头plugin； **第二部分** 是算法名称，为如下字符串之一：sm2、secp256k1、secp256r1、secp256k1recover，flato会解析得到算法类型后用mode参数传递给GetSignKey方法； **第三部分** 是hex编码，flato会将hex解码后的字节数组传递给GetSignKey方法作为key参数。第三部分的具体内容和含义是plugin负责解释的，对flato透明，因此第三部分可以是密钥的名称，索引，密钥本身，加密后的密钥等等。

2.ImportSignKey：导入签名密钥

    - 如果key是crypto.None，则key内容是pkcs8格式私钥，DER编码

    - 如果key是crypto中定义的具体算法，则key内容是对应算法的私钥，但是解析方式由插件确定，对flato透明（因此可以是加密格式）

    - 返回值index是该密钥导入后的索引，对应GetSignKey接口的第一个参数

    - 该方法在falto运行的主流程中不会调用，但是未来可能在ipc中增加相应调用功能，帮助用户完成密钥导入。

2.2 接口实现demo以及具体说明
----------------------------

2.2.1 对外提供函数的实现
>>>>>>>>>>>>>>>>>>>>>>>>

 ::

    func GetLevelArray() []crypto.Level{
       return []crypto.Level{new(hashManager), new(randManager), new(cryptManager), new(verifyManager),
          new(signManager), new(signCreator), new(encManager), new(decManager), new(decCreator)}
    }

本例中提供了所有类别的实现，实际插件可以仅仅实现其中的部分。例如如果仅仅需要签名验签，就只需要实现 以下几个就可以了。

 ::

    return []crypto.Level{ new(verifyManager),
          new(signManager), new(signCreator) }

2.2.2 hash工厂实现
>>>>>>>>>>>>>>>>>>>>>>>

 ::

    var priority uint8 = 2

    func getGlobalLevel() uint8 {
       return priority
    }

    type hashManager struct {

    }

    func (h *hashManager) GetLevel() ([]int, uint8) {
       return []int{crypto.SM3}, getGlobalLevel()
    }

    func (h *hashManager) GetHash(mode int) (crypto.Hasher, error){
       return NewHash(mode)
    }

1、其中GetLevel 函数返回支持的算法列表以及算法使用的优先级别，默认1最小，255最大。

2、GetHash 返回并创建一个支持的模式（mode）的 Hash实例。

2.2.3 随机数生成器工厂实现
>>>>>>>>>>>>>>>>>>>>>>>>>>>

 ::

    type randManager struct {

    }

    func (h *randManager) GetLevel() ([]int, uint8) {
       return []int{crypto.None}, getGlobalLevel()
    }

    func (h *randManager) Rander() (io.Reader, error) {
       rd := NewHRand()
       if rd == nil{
          return nil, errors.New("no session open")
       }
       return rd, nil
    }

1、GetLevel 以及以下所有的GetLevel 说明参考2.2.2

2、Rander()创建一个随机数生成器

2.2.4  签名工厂实现
>>>>>>>>>>>>>>>>>>>>>

1、签名Key的工厂实现

 ::

    type signManager struct {

    }

    func (h *signManager) GetLevel() ([]int, uint8) {
       return []int{crypto.Sm2p256v1}, getGlobalLevel()
    }

    func (h *signManager) GetSignKey(key []byte, mode int) (crypto.SignKey, error) {
       if key == nil{
          //return nil, errors.New("index nil")
          return GenerateHSm2PrivateKey(nil), nil
       }
       if mode != crypto.Sm2p256v1{
          return nil, modeNotSupport
       }
       priv := new(PrivateKey)
       priv.Curve = sm2Curve()
       priv.D = new(big.Int).SetBytes(key)
       priv.X, priv.Y = priv.ScalarBaseMult(key)
       return GenerateHSm2PrivateKey(priv), nil
    }

    var modeNotSupport = errors.New("mode is not sm2")
    var keyFmtNotSupport = errors.New("key format is not pkcs8 of sm2")


    func (h *signManager) ImportSignKey(key []byte, mode int) ( []byte, error){
       var priv *hSm2PrivateKey
       if key == nil{
          if mode != crypto.Sm2p256v1{
             return nil, modeNotSupport
          }
          priv = GenerateHSm2PrivateKey(nil)
       } else {
          k := new(PrivateKey)
          k.Curve = sm2Curve()
          k.D = new(big.Int).SetBytes(key)
          k.X, k.Y = k.ScalarBaseMult(key)
          priv = GenerateHSm2PrivateKey(k)
       }
       if priv == nil{
          return nil, keyFmtNotSupport
       }
       return MarshalPKCS8PrivateKey(ToSm2PrivateKey(&priv.k))
    }

注意事项：

1）GetSignKey 的key入参不是pkcs8的私钥，定义如下：

ecdsa、sm2 为大整数的大端序，例如10000 代表1万，而不是1

rsa key为pkcs1私钥格式

2）ImportSignKey 的key入参参照GetSignKey

ImportSignKey返回 []byte 为

3）如果key可以导出，则 key为pkcs8模式

4）如果key不可导出，则 key为实际的信息索引，例如gm0018所规定的 uint 值


2、签名Key的生成器工厂实现

 ::

    type signCreator struct {

    }

    func (h *signCreator) GetLevel() ([]int, uint8) {
       return []int{crypto.Sm2p256v1}, getGlobalLevel()
    }

    func (h *signCreator) CreateSignKey(write bool, mode int) (index []byte, k crypto.SignKey, err error) {
       if mode != crypto.Sm2p256v1{
          err = modeNotSupport
          return
       }

       s := GenerateHSm2PrivateKey(nil)
       k = s
       if s != nil{
          index, err = MarshalPKCS8PrivateKey(ToSm2PrivateKey(&s.k))
       }
       return
    }

2.2.5 验签工厂实现
>>>>>>>>>>>>>>>>>>>>>>>>

 ::

    type verifyManager struct {

    }

    func (h *verifyManager) GetLevel() ([]int, uint8) {
       return []int{crypto.Sm2p256v1}, getGlobalLevel()
    }

    func (h *verifyManager) GetVerifyKey(key []byte, mode int) (crypto.VerifyKey, error) {
       if mode != crypto.Sm2p256v1{
          return nil, modeNotSupport
       }
       x, y := elliptic.Unmarshal(sm2Curve(), key)
       if x == nil || y == nil{
          return nil, keyFmtNotSupport
       }
       pub := new(hSm2PublicKey)
       FromSm2PublickKey(&PublicKey{sm2Curve(), x, y}, &pub.k)
       return pub, nil
    }

1、GetVerifyKey的入参key 定义如下：

ecdsa、sm2为公钥点x，y 的椭圆曲线序列化 elliptic.Unmarshal(）

rsa 为pkcs1 公钥格式


3.测试说明
==============

3.1 插件自测
--------------

应针对插件所支持的算法提供测试用例，例如demo实现中对于sm3的测试，可以参考如下::

    var msgHash = []string{
       "4d2d682913a35ea55a75361a179b8ebd30633326a362a7c08213293de095a669ccb7bfb70e96777f90ef95647be7523e",
       "a49f3ef47efd3b1006faf58114888ecce3d242a8300392e3d866c4515440b98e",
       "38d679db0a70e3cdf78f0fa3c993550ef1b9f9d63f389e678577e27150e24251d26ba6152acf20023068fbb7c205e486",
       "995b376dcbbd2382ae8661e9c4ab7b1f7668332153c38f38385683aa60ed8539",
       "67adfca17d4612d17631dc71e4b928b8e7524cec141fe7c8a9df8dd1ca334fe86fcf9e99ff6dd07957cf927019dfecdc",
       "a5be8103079838d17ccf21dc5bd46d6de828e3c29c0325652a120c99ed4f1974",
       "48475fe720fc174d926a137707c789cab5250b82a848bf5bbf63a267196b7b493ef63d118e5752449d50273f1665ba3b",
       "f32afbaf007df2d354ed65ed486e7becf3b935b0eb2c3ec9350acbc56c5a1b5a",
       "959b84b08c29c1174d794b9f936c4f221b22a98c92a04cac57246043dfda5a0bdaae4e164d7eaf16a6c30b6ddb8c01d4",
       "9abf09f207fc8294f59b0657520be0cdc58d61ce2d51bf83d59ce4009b93163b",
    }


    func TestHash(t *testing.T) {
       hm := new(hashManager)
       for i := 0; i < len(msgHash); i++{
          h, err := hm.GetHash(crypto.SM3)
          if err != nil{
             t.Fatal(err)
          }
          h.Write(gets(hex.DecodeString(msgHash[i])))
          i++
          if !bytes.Equal(h.Sum(nil), gets(hex.DecodeString(msgHash[i]))){
             t.Fatal(i)
          }
       }
    }

其它签名验签的方法测试用例也要完整覆盖到，这样发现的错误可以以最小的成本解决。


3.2 hyperchain 接入测试
---------------------------

hyperchain 接入插件，需要修改hyperchain 配置文件。为使外部插件生效，如果为优先级模式，需插件的GetLevel为所有插件最大；指定插件模式，需指定当前插件。具体可参照密码机中间件的设计文档。


3.3加载成功关键日志
---------------------

example:

> NOTI [2021-02-22T15:01:38.772] [identitymanager] plugin/engine_external.go:84 start load external crypto engine: **plugin_ceb.so**

> NOTI [2021-02-22T15:01:38.834] [identitymanager] plugin/engine_external.go:100 crypto engine **[plugin_ceb.so]** have  **1** function: **[sign]**

> NOTI [2021-02-22T15:01:38.834] [identitymanager] plugin/external_algo_select.go:105 crypto engine: external function [SignGet] for Sm2p256v1 from plugin_ceb.so is loading...

> NOTI [2021-02-22T15:01:38.834] [identitymanager] plugin/engine.go:406 loading a external crypto engine (plugin_ceb.so) finish

> NOTI [2021-02-22T15:01:38.834] [identitymanager] plugin/engine.go:409 external plugin loading info:

> **[SignGet]       : Sm2p256v1 -> plugin_ceb.so**

4.注意事项
==============

1、具体实现的注意事项请参照2.2章节

2、由于密码机中间件的接口实现与测试涉及部分硬件环境，这给程序的测试带来了一定的难度，因此插件内部实现的自测就显得十分重要。

3、插件名要以plugin为前缀，例如: go build - **trimpath** -buildmode=plugin -o plugin_pcie.so plug.go