
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode.html) | Dinghao Liu | Bei Wang |


# Chaincode 指南

## 什么是Chaincode？

Chaincode是一段由Go语言编写（支持其他编程语言，如Java），并能实现预定义接口的程序。Chaincode运行在一个受保护的Docker容器当中，与背书节点的运行互相隔离。Chaincode可通过应用提交的交易对账本状态初始化并进行管理。

一段chaincode通常处理由网络中的成员一致认可的业务逻辑，故我们很可能用“智能合约”来代指chaincode。一段chiancode创建的（账本）状态是与其他chaincode互相隔离的，故而不能被其他chaincode直接访问。不过，如果是在相同的网络中，一段chiancode在获取相应许可后则可以调用其他chiancode来访问它的账本。

## 两类角色

关于chaincode，我们可以从两个不同的视角来审视它们：第一个是区块链应用/解决方案的开发者的角度，详见《Chaincode 开发手册》部分；另一个是区块链网络的操作者，他们也许并不想涉足chaincode应用的开发，不过却需要肩负管理区块链网络的责任，并运用Hyperledger Fabric API来安装、实例化与升级chaincode。该部分详见《Chaincode 操作手册》。


