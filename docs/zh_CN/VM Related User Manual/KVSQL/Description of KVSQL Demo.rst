.. _Description-of-KVSQL-Demo:

一键链改Demo说明
^^^^^^^^^^^^^^^^^^^^^

> 一键链改Demo是为用户更快速的了解一键链改特性而准备的示例项目

> This project is the Demo project of Application for use kvsql with hyperchain.

> There are two branches:

> - hibernate : This branch is the Demo of use Spring Boot and hibernate

> - Mybatis：This branch is the Demo of use Spring Boot and mybatis

运行步骤
============

1. 启动对应hyperchain平台

步骤详见部署文档

2. 运行CreateDataBase中的main方法，进行"库"和"表"的创建

3. 修改application-dev.yml配置文件，修改"库"地址以及urls

4. 启动应用，打开swagger

 ::

 http://localhost:8089/swagger-ui.html

 For more usage step, please see the Usage Document.

附Demo文件包（demo包需要根据最新的sdk重新打包  否则2.7.0链使用交易签名非法）：

`sql-driver-demo.tar.gz <https://upload.filoop.com/RTD-Hyperchain%2Fsql-driver-demo.tar.gz>`_

外网使用时的依赖(手动添加到工程中)：

`kvsql-jdbc-driver-0.2.0-jar-with-dependencies.jar <https://upload.filoop.com/RTD-Hyperchain%2Fkvsql-jdbc-driver-0.2.0-jar-with-dependencies.jar>`_
