
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](https://fabric-sdk-node.github.io) | Qingpeng Cai | Dinghao Liu, Bei Wang |

The Hyperledger Fabric SDK for Node.js prvides a powerful API to interact with a Hyperledger Fabric v1.0 blockchain. The SDK is designed to be used in the Node.js JavaScript runtime.

Hyperledger Fabric中的Node.js SDK提供了强大的API接口用于同Hyperledger Fabric v1.0 blockchain进行交互。该SDK被设计用于Node.js JavaScript的运行。

##概述

Hyperledger Fabric is the operating system of an enterprise-strength permissioned blockchain network. For a high-level overview of the fabric, visit http://hyperledger-fabric.readthedocs.io/en/latest/.

Hyperledger Fabric是企业级许可的区块链网络的操作系统。对于高等级fabric的概述，请访问http://hyperledger-fabric.readthedocs.io/en/latest/.

Applications can be developed to interact with the blockchain network on behalf of the users. APIs are available to:

我们可以开发应用来帮助区块链网络中的代表用户进行交互。API可以用于：

* 创建[通道](https://fabric-sdk-node.github.io/Channel.html#queryInfo)(create channels)
* 请求[节点](https://fabric-sdk-node.github.io/)加入通道(ask peer nodes to join the channel)
* 在节点中安装[链码](http://hyperledger-fabric.readthedocs.io/en/latest/fabric_model.html#chaincode)(install chaincodes in peers)
* 在通道中实现链码实例化(instantiate chaincodes in a channel)
* 通过调用链码来调用事务(invoke transactions by calling the chaincode)
* 查询事务或区块的[账本](http://hyperledger-fabric.readthedocs.io/en/latest/fabric_model.html#ledger-features)(query the ledger for transactions or blocks)

##Fabric的不同组成部分如何协调工作

The Transaction Flow document provides an excellent description of the application/SDK, peers, and orderers working together to process transactions and producing blocks.

该[交易流程](http://hyperledger-fabric.readthedocs.io/en/latest/txflow.html)文件提供一个关于应用程序/SDK，peer和orderer共同处理事务并产生区块的很好的描述。

Security on the Fabric is enforced with digital signatures. All requests made to the fabric must be signed by users with appropriate enrollment certificates. For a user's enrollment certificate to be considered valid on the Fabric, it must be signed by a trusted Certificate Authority (CA). Fabric supports any standard CAs. In addition, Fabric provides a CA server. See this overview.

Fabric的安全是通过`数字签名`来实现的。在Fabric中的所有请求都必须具有有效注册证书的用户签名。对于在Fabric中被认为有效的注册证书，必须具有受信任的证书颁发机构(CA)签名。Fabric支持CA的所有标准。此外，Fabric还提供了一个CA服务器。请看这个[概述](http://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#overview)。

##Node.js SDK功能

The Hyperledger Fabric SDK for Node.js is designed in an Object-Oriented programming style. Its modular construction enables application developers to plug in alternative implementations of key functions such as crypto suites, the state persistence store, and logging utility.

Hyperledger Fabric中的Node.js SDK是根据面向对象的程序风格进行设计的。该模块化的结构是应用程序开发人员能够可选择的插入关键函数的实现，比如加密套件，持久性状态存储，日志记录等等。

The SDK's list of features include:

该SDK的功能列表如下：

· fabric-client:

 * [创建一个新的通道](https://fabric-sdk-node.github.io/Client.html#createChannel)(create a new channel)
 * [将通道信息发送给节点用于节点加入](https://fabric-sdk-node.github.io/Channel.html#joinChannel)(send channel information to a peer to join)
 * [在节点上安装链码](https://fabric-sdk-node.github.io/Client.html#installChaincode)(install chaincode on a peer)
 * 在通道中进行链码实例化，其中包括两步：[提议](https://fabric-sdk-node.github.io/Channel.html#sendInstantiateProposal)和[交易](https://fabric-sdk-node.github.io/Channel.html#sendTransaction)(instantiate chaincode in a channel, which involves two steps: propose and transact)
 * 提交交易，其中包括两步：[提议](https://fabric-sdk-node.github.io/Channel.html#sendInstantiateProposal)和[交易](https://fabric-sdk-node.github.io/Channel.html#sendTransaction)(submitting a transaction, which also involves two steps: propose and transact)
 * [查询链码的最新状态](https://fabric-sdk-node.github.io/Channel.html#queryByChaincode)(query a chaincode for the latest application state)
 * 多种查询功能(various query capabilities):
 	* [查询通道长度](https://fabric-sdk-node.github.io/Channel.html#queryInfo)(channel height)
   	* [通过区块数字查询](https://fabric-sdk-node.github.io/Channel.html#queryBlock)(block-by-number),[通过区块hash值查询](https://fabric-sdk-node.github.io/Channel.html#queryBlockByHash)(block-by-hash)
   	* [查询一个节点所在的所有通道](https://fabric-sdk-node.github.io/Client.html#queryChannels)(all channels that a peer is part of)
   	* [查询节点中安装的所有链码](https://fabric-sdk-node.github.io/Client.html#queryInstalledChaincodes)(all installed chaincodes in a peer)
   	* [查询通道中的所有实例化链码](https://fabric-sdk-node.github.io/Channel.html#queryInstantiatedChaincodes)(all instantiated chaincodes in a channel)
  	* [通过事务ID查询](https://fabric-sdk-node.github.io/Channel.html#queryTransaction)(transaction-by-id)
   	* [查询通道的配置数据](https://fabric-sdk-node.github.io/Channel.html#getChannelConfig)(channel configuration data)
   
 * 监控事件(monitoring events):
 	* [连接一个节点的事件流](https://fabric-sdk-node.github.io/EventHub.html#connect)(connect to a peer's event stream)
   	* 监听一个[区块的事件](https://fabric-sdk-node.github.io/EventHub.html#registerBlockEvent)(listen on block events)
   	* 监听交易时间并且确定交易是否成功提交进行标记(listen on transactions events and find out if the transaction was successfully committed to the ledger or marked invalid)
   	* 监听由链码产生的[自定义事件](https://fabric-sdk-node.github.io/EventHub.html#registerChaincodeEvent)(listen on custom events produced by chaincodes)

 * 序列化用户对象及其签名功能(serializable User object with signing capabilities)

 * 多层覆盖式的分层配置设置(hierarchical configuration settings with multiple layers of overrides):文件(files), 环境变量(environment variable),程序参数(program arguments), 内存设置(in-memory settings)

 * 使用内置的记录器(winston)进行日志记录并且被多种流行的日志记录器进行覆盖，包括 log4js 和 bunyan等等。logging utility with a built-in logger (winston) and can be overriden with a number of popular loggers including log4js and bunyan

 * 可插式CryptoSuite 界面描述了与Fabric进行成功交互时需要的加密操作。其中包括两种实现：pluggable CryptoSuite interface describe the cryptographic operations required for successful interactions with the Fabric. Two implementations are provided out of box:
 	* [Software-based ECDSA](https://fabric-sdk-node.github.io/CryptoSuite_ECDSA_AES.html)
  	* [PKCS#11-compliant ECDSA](https://fabric-sdk-node.github.io/CryptoSuite_PKCS11.html)
 * 可插式状态存储界面，用于持续性状态缓存，比如用户。pluggable State Store interface for persisting state caches such as users

   	* [基于文件的存储](https://fabric-sdk-node.github.io/FileKeyValueStore.html)(File-based store)
   	* [基于CouchDB的存储](https://fabric-sdk-node.github.io/CouchDBKeyValueStore.html)，与CouchDB数据库和IBM Cloudant兼容(CouchDB-base store which works with both CouchDB database and IBM Cloudant)

 * 自定义的密钥存储用于所有基于软件的加密套件实现。customizable Crypto Key Store for any software-based cryptographic suite implementation
 * 同时支持TLS和非TLS链接到peer和orderer，请参阅peer和orderer的超类。 supports both TLS (grpcs://) or non-TLS (grpc://) connections to peers and orderers, see Remote which is the superclass for peers and orderers


· fabric-ca-client:

 * [注册](https://fabric-sdk-node.github.io/FabricCAServices.html#register)新用户(register a new user)
 * [登记](https://fabric-sdk-node.github.io/FabricCAServices.html#enroll)用户同时获得由Fabric CA签署的注册证书(enroll a user to obtain the enrollment certificate signed by the Fabric CA)
 * 通过注册ID[废除](https://fabric-sdk-node.github.io/FabricCAServices.html#revoke)已有用户或废除特定证书(evoke an existing user by enrollment ID or revoke a specific certificate)
 * [自定义的持久储存](https://fabric-sdk-node.github.io/FabricCAServices.html)(customizable persistence store)

##API参考

The SDK is made up of 3 top-level modules that can be accessed through the navigation menu Modules:

SDK由3个顶级模块组成，可以通过导航菜单访问模块：

* api:可插式API，用于应用程序开发人员来通过SDK来提供有选择性关键界面的替代。对于每个接口，都有内置的默认实现。api: pluggable APIs for application developers to supply alternative implementations of key interfaces used by the SDK. For each interface there are built-in default implementations.


* fabric-client: 该模块提供API来进行同基于Hypreledger Fabric的区块链网络的交互，也就是同peer，orderer和事件流。fabric-client: this module provides APIs to interact with the core components of a Hypreledger Fabric-based blockchain network, namely the peers, orderers and event streams.


* fabric-ca-client:该模块提供能与包含成员管理资格的服务的可选组件(fabric-ca)进行交互的API。该工作具有[Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)。fabric-ca-client: this module provides APIs to interact with the optional component, fabric-ca, that contains services for membership management. Creative Commons License
This work is licensed under a Creative Commons Attribution 4.0 International License.

