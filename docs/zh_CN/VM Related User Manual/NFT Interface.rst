.. _NFT-Interface:

数字资产相关接口
^^^^^^^^^^^^^^^

说明
========

从Hyperchain2.2.0版本开始，NFT资产可映射为平台底层账户，由区块链原生提供数字资产操作功能，因此在合约层面，也提供了能够操作底层数字资产账户的接口。

接口
=======

从hvm-sdk1.1.4版本开始，内置了类ERC721的接口定义，用于协定发布的数字资产的合约的管理行为，接口为约定内容，并非数字合约必须实现的，可按自己实际业务进行实现。

HPC721接口
-------------

hvm智能合约提供类ERC721的HPC721标准，在合约灵活方便的基础上，提供标准化接口的实现。配合底层数字资产账户，即可灵活定制数字玩法，也可高效完成数字交易::

    public interface HPC721 {
        /**
         * Emitted when `id` property is transferred from `from` to `to`.
         */
        void eventTransfer(String from, String to, long id);

        /**
         * Emitted when `owner` enables `approved` to manage the `id` property.
         */
        void eventApproval(String owner, String approved, long id);

        /**
         * Emitted when `owner` enables or disables (`approved`) `operator` to manage all of its assets.
         */
        void eventApprovalForAll(String owner, String operator, boolean approved);

        /**
         * Returns the number of properties in `owner`'s account.
         */
        long balanceOf(String owner);

        /**
         * Returns the owner of the `id` property.
         */
        String ownerOf(long id);

        /**
         * Safely transfers `id` property from `from` to `to`
         */
        void transferFrom(String from, String to, long id);

        /**
         * Safely transfers `id` property from `from` to `to`
         */
        void transferFrom(String from, String to, long id, byte[] calldata);

        /**
         * Gives permission to `to` to transfer `id` property to another account.
         */
        void approve(String to, long id);

        /**
         * Returns the account approved for `id` property.
         */
        String getApproved(long id);

        /**
         * Approve or remove `operator` as an operator for the caller.
         */
        void setApprovalForAll(String operator, boolean approved);

        /**
         * Returns if the `operator` is allowed to manage all of the assets of `owner`.
         */
        boolean isApprovedForAll(String owner, String operator);
    }

HPC721Metadata接口
----------------------

HPC721Metadata作为HPC721的补充接口，增加了NFT项目的扩展指定内容，本身改接口继承自HPC721接口::

    public interface HPC721Metadata extends HPC721 {

        /**
         * Returns the property collection name.
         */
        String name();

        /**
         * Returns the property collection symbol.
         */
        String symbol();

        /**
         * Returns the Uniform Resource Identifier (URI) for `id` property.
         */
        String uri(long id);
    }

资产操作接口
================

所有NFT的操作都是基于一个智能合约的，因此对于操作NFT的接口均基于智能合约的内置方法，具体介绍可参考《HVM合约内置方法使用手册》，这里描述与NFT相关的接口。

发布NFT资产
-------------

**emit0**

铸造nft账户，默认会为创建的nft资产账户设置的平台地址为当前合约地址，默认初始状态为0::

 public final native String emit0(byte[] identity, String owner, byte[] meta);

===== ==== ==================
方法  参数 返回值
===== ==== ==================
emit0      资产的底层账户地址
===== ==== ==================

**示例**::

 String propertyAddr = this.emit0(id, "0x123456", "meta".getBytes());

获取NFT资产
--------------

**getProperty0**

通过nft资产唯一标识获取到当前合约下发布的nft资产账户

该只能获取到当前合约地址先发布的nft，若资产不存在，则会返回null。 **需要注意的是取出来的资产为账本中资产账户的映射，对应的setXXX方法将会对账本进行修改，需要注意好访问权限问题。**

 ::

 public final native PropertyV1 getProperty0(byte[] identity);

============ ==== ============
方法         参数 返回值
============ ==== ============
getProperty0      资产账户结构
============ ==== ============

**示例**::

 PropertyV1 property = this.getProperty0();

资产结构
------------

 ::

    public class PropertyV1 {

        private byte[] entityID;
        private String metaData;
        private int status;
        private String owner;
        private String updateFrom;

        private byte[] addr;

        public byte[] getEntityID() {
            return entityID;
        }

        public String getMetaData() {
            return metaData;
        }

        public int getStatus() {
            return status;
        }

        public String getOwner() {
            return owner;
        }

        public native void setMetaData(String metaData);

        public native void setStatus(int status);

        public native void setOwner(String owner);
    }

PropertyV1可通过合约内置方法获取得到，其属性的含义如下：

- entityID：合约内资产唯一标识

- metaData：资产的自定义meta信息

- status：资产状态，数字表示，每个数字对应的含义由合约决定，默认为0

- owner：资产拥有者地址

- updateFrom：是否从某个资产更新而来，暂未开放更新地址修改功能

同时PropertyV1提供了 `getEntityID` ， `getMetaData` ， `getStatus` ， `getOwner` 接口来获取上述的属性。

PropertyV1接口同时还提供了 `setMetaData` ， `setStatus` ， `setOwner` 方法，直接从合约内置方法中获取到的PropertyV1对象的set方法的修改将直接对账本中的资产账户结构产生影响，因此需要注意权限问题，合约内对外返回的Property对象需要经过拷贝。

demo
=======

`property.zip <https://upload.filoop.com/RTD-Hyperchain%2Fproperty%20(1).zip>`_
