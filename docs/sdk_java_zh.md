
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](https://github.com/hyperledger/fabric-sdk-java) | Xuanyong Wu |  |


Welcome to Java SDK for Hyperledger project. The SDK helps facilitate Java applications to manage the lifecycle of Hyperledger channels  and user chaincode. The SDK also provides a means to execute user chaincode, query blocks and transactions on the channel, and monitor events on the channel.

欢迎使用Hyperledger项目的Java SDK。 该SDK帮助促进Java应用程序对Hyperledger通道和用户链码的生命周期的管理。该SDK同时提供执行用户链码、在通道上查询区块和交易以及监听通道上的事件的一些方法。

The SDK acts on behave of a particular User which is defined by the embedding application through the implementation of the SDK's `User` interface.

该SDK能够实现一些特定用户的行为，这是由嵌入的应用程序通过实现SDK的`User`接口来完成的。

Note, the SDK does ***not*** provide a means of persistence for the application defined channels and user artifacts on the client. This is left for the embedding application to best manage.

请注意，该SDK***不提供***客户端上应用定义的通道和用户结果的持久性方法。这个是预留给嵌入程序能够进行更有效的管理。

The SDK also provides a client for Hyperledger's certificate authority.  The SDK is however not dependent on this
particular implementation of a certificate authority. Other Certificate authority's maybe used by implementing the
SDK's `Enrollment` interface.

该SDK同时提供一个连接Hyperledger证书机构的客户端。但该SDK并不依赖于这种连接证书机构的方法，用户也可以通过实现SDK的`Enrollment`接口来使用其他的证书机构。


This provides a summary of steps required to get you started with building and using the Java SDK. Please note that this is not the API documentation or a tutorial for the SDK, this will only help you familiarize to get started with the SDK if you are new in this domain.

以下步骤让你可以开始编译和使用Java SDK。
请注意，因为这个文档不是API文档或者SDK指南，所以仅仅帮助新手熟悉如何开始使用SDK。

## 已知的限制(Known limitations and restrictions)

* TCerts are not supported(不支持TCerts): JIRA FAB-1401
* HSM not supported(不支持HSM): JIRA FAB-3137
* Single Crypto strength 256(单层加密长度为256位): JIRA FAB-2564
* Network configuration updates not supported(不支持网络配置更新): JIRA FAB-3103


`*************************************************`

## *v1.0.0*

There is a git tagged v1.0.0 [e976abdc658f212d0c3a80ace4499a5cff4279c6] release of the SDK where there is no need to build the Hyperledger Fabric and Hyperledger Fabric CA described below. The provided docker-compose.yaml for the integration tests should pull v1.0.0  tagged images from Docker hub.

已经有一个标记为V1.0.0[e976abdc658f212d0c3a80ace4499a5cff4279c6]的SDK发行版，所以没有必要自己打包Hyperledge Fabric和Hyperledger Fabric CA。提供集成测试的docker-compose.yaml会自动从Docker Hub下载V1.0.0的镜像。

The v1.0.0 version of the Hyperledger Fabric Java SDK is published to Maven so you can directly use in your application's pom.xml.

V1.0.0的Hyperledger Fabric Java SDK已经发布到了Maven仓库，你可以直接在你的应用程序的pom.xml文件中直接使用。

[Maven Repository Hyperledger Fabric Java SDK](https://mvnrepository.com/artifact/org.hyperledger.fabric-sdk-java/fabric-sdk-java)

*Make sure you're using docker images at the level of the Fabric that matches the level of the SDK you're using in your application.*

_请确认你使用的docker镜像的版本和你应用程序中使用的SDK版本一致。_

`*************************************************`


## 验证Fabric和Fabric-ca包(Valid builds of Fabric and Fabric-ca)

Hyperledger Fabric v1.0.1 is currently under active development and the very latest Hyperledger Fabric builds may not work with this sdk.
You should use the following commit levels of the Hyperledger projects:

Hyperledger Fabric V1.0.1 正在开发中，最新的Fabric打包可能不适用该SDK，你应该使用下面提交版本的Hyledger项目。


Project        | Commit level                               | Date                       |
---------------|:------------------------------------------:|---------------------------:|
 fabric         | f56a82e36e040e1c1a986edfceac014ba1516571   | Jul 11 12:48:33 2017 -0700 |
 fabric-ca      | 74f8f4d4c29e45a79a8849efb057dbd8de3ae8d0   | Jul 11 16:43:39 2017 +0200 |




 You can clone these projects by going to the [Hyperledger repository](https://gerrit.hyperledger.org/r/#/admin/projects/).

你可以到[Hyperledger仓库](https://gerrit.hyperledger.org/r/#/admin/projects/)克隆这些项目。

 As SDK development continues, this file will be updated with compatible Hyperledger Fabric and Fabric-ca commit levels.

随着SDK的开发进行中，这些文件将会随着Fabric和Fabric-ca的提交版本变化而更新。

 Once you have cloned `fabric` and `fabric-ca`, use the `git reset --hard commitlevel` to set your repositories to the correct commit.

你克隆`fabric`和`fabric-ca`时，使用`git reset --hard commitlevel`来设置你仓库的使用正确的提交版本。

## 使用Fabric Vagrant环境(Working with the Fabric Vagrant environment)

Vagrant is NOT required if your OS has Docker support and all the requirements needed to build directly in your
environment.  For non Vagrant envrionment, the steps would be the same as below minus those parts involving Vagrant.
 Do the following if you want to run the Fabric components ( peer, orderer, fabric-ca ) in Vagrant:

如果你的环境支持Docker，Vagrant不是必须的，你只需要你的环境中直接打包。在非Vagrant环境，牵涉到Vagrant的步骤会少许有些不同。
按照下面的步骤在Vagrant中运行Fabric组件( peer, orderer, fabric-ca )：

```
  git clone  https://github.com/hyperledger/fabric.git
  git clone  https://github.com/hyperledger/fabric-ca.git
  cd  fabric-ca
  git reset --hard fabric-ca_commitlevel from above
  cd ../fabric
  git reset --hard fabric_commitlevel from above
  cd devenv
  change the Vagrant file as suggested below:
  vagrant up
  vagrant ssh
  make docker
  cd ../fabric-ca
  make docker
  cd ../fabric/sdkintegration
  docker-compose down;  rm -rf /var/hyperledger/*; docker-compose up --force-recreate
```



 * Open the file `Vagrantfile` and verify that the following `config.vm.network` statements are set. If not, then add them:
 * 打开`Vagrantfile`文件来验证下面的`config.vm.network`参数是否设置了，如果没有，请添加以下行。


```
  config.vm.network :forwarded_port, guest: 7050, host: 7050 # fabric orderer service
  config.vm.network :forwarded_port, guest: 7051, host: 7051 # fabric peer vp0 service
  config.vm.network :forwarded_port, guest: 7053, host: 7053 # fabric peer event service
  config.vm.network :forwarded_port, guest: 7054, host: 7054 # fabric-ca service
  config.vm.network :forwarded_port, guest: 5984, host: 15984 # CouchDB service
  ### Below are probably missing.....
  config.vm.network :forwarded_port, guest: 7056, host: 7056
  config.vm.network :forwarded_port, guest: 7058, host: 7058
  config.vm.network :forwarded_port, guest: 8051, host: 8051
  config.vm.network :forwarded_port, guest: 8053, host: 8053
  config.vm.network :forwarded_port, guest: 8054, host: 8054
  config.vm.network :forwarded_port, guest: 8056, host: 8056
  config.vm.network :forwarded_port, guest: 8058, host: 8058

```

Add to your Vagrant file a folder for referencing the sdkintegration folder between the lines below:

在Vagrant文件中，增加SDK集成目录的引用：

  config.vm.synced_folder "..", "/opt/gopath/src/github.com/hyperledger/fabric"</br>

  `config.vm.synced_folder "/home/<<user>>/fabric-sdk-java/src/test/fixture/sdkintegration", "/opt/gopath/src/github.com/hyperledger/fabric/sdkintegration`</br>

  config.vm.synced_folder ENV.fetch('LOCALDEVDIR', ".."), "#{LOCALDEV}"</br>


## SDK 依赖(SDK dependencies)

SDK depends on few third party libraries that must be included in your classpath when using the JAR file. To get a list of dependencies, refer to pom.xml file or run
当你使用JAR文件时，SDK依赖的第三方库应该包含在classpath中。为了可以得到依赖清单，可以参照pom.xml文件或者运行

<code>mvn dependency:tree</code> or <code>mvn dependency:list</code>.
<code>mvn dependency:tree</code> 或 <code>mvn dependency:list</code>.

Alternatively, <code> mvn dependency:analyze-report </code> will produce a report in HTML format in target directory listing all the dependencies in a more readable format.
除此之外，<code> mvn dependency:analyze-report </code> 会在目标目录中生成一个HTML格式的报告更清晰地列出所有的依赖包。

## 使用SDK(Using the SDK)

The SDK's test cases uses chaincode in the SDK's source tree: `/src/test/fixture`
SDK的测试用例中使用链码源码树： `/src/test/fixture`

The SDK's JAR is in `target/fabric-sdk-java-1.0.0-SNAPSHOT.jar` and you will need the additional dependencies listed above.
When the SDK is published to `Maven` you will be able to simply include it in your application's `pom.xml`.

SDK的JAR包在`target/fabric-sdk-java-1.0.0-SNAPSHOT.jar`，你可能会需要增加上面提到的附加的依赖包。
当SDK发布到Maven后，你只需要简单的在你的应用的`pom.xml`包含它。

Add below code in your `pom.xml` to download fabric-sdk-java-1.0
在你的`pom.xml`文件增加下面的代码来下载fabric-sdk-java-1.0
```xml

     <dependencies>
     <dependency>
            <groupId>org.hyperledger.fabric-sdk-java</groupId>
            <artifactId>fabric-sdk-java</artifactId>
            <version>1.0.0</version>
         </dependency>
     </dependencies>
```

### 编译(Compiling)

To build this project, the following dependencies must be met

 * JDK 1.8 or above
 * Apache Maven

为了能够编译该工程，你需要满足以下要求

 * JDK 1.8 或更高
 * Apache Maven

Once your JAVA_HOME points to your installation of JDK 1.8 (or above) and JAVA_HOME/bin and Apache maven are in your PATH, issue the following command to build the jar file:
当你的JAVA_HOME指向了JDK 1.8安装目录，在PATH中设置了JAVA_HOME/bin和 Apache Maven，你可以执行下面的命令来打包jar文件：

<code>
  mvn install
</code>
or
<code>
  mvn install -DskipTests
</code> 如果你不想执行单元测试。

### 执行单元测试(Running the unit tests)

To run the unit tests, please use <code>mvn test</code> or <code>mvn install</code> which will run the unit tests and build the jar file.You must be running a local peer and orderer to be able to run the unit tests.

通过<code>mvn test</code> 来执行单元测试或 <code>mvn install</code> 执行单元测试并生成jar文件。你必须在本地运行了peer和orderer节点才能执行单元测试。

### 执行集成测试(Running the integration tests)

You must be running local instances of Fabric-ca, Fabric peers, and Fabric orderers to be able to run the integration tests. See above for running these services in Vagrant.
Use this `maven` command to run the integration tests:

你必须在本地运行了Fabric-ca, Fabric peers, and Fabric orderers节点才能够执行集成测试。参照上文中如何在Vagrant中运行这些服务。
使用下面的`maven`指令来执行集成测试：

 * _mvn failsafe:integration-test -DskipITs=false_

### 端到端测试场景(End to end test scenario)

The _src/test/java/org/hyperledger/fabric/sdkintegration/End2endIT.java_ integration test is an example of installing, instantiating, invoking and querying a chaincode.
It constructs the Hyperledger channel, deploys the `GO` chaincode, invokes the chaincode to do a transfer amount operation and queries the resulting blockchain world state.

This test is a reworked version of the Fabric [e2e_cli example](https://github.com/hyperledger/fabric/tree/master/examples/e2e_cli) to demonstrate the features of the SDK.
To better understand blockchain and Fabric concepts, we recommend you install and run the _e2e_cli_ example.

_src/test/java/org/hyperledger/fabric/sdkintegration/End2endIT.java_中的集成测试，是一个简单的安装、实例化、调用和查询链码的例子。它构建了Hyperledger通道，发布`GO`版本链码，调用链码，执行转账动作，并在区块链上查询结果。

### 端到端测试环境(End to end test environment)

The test defines one Fabric orderer and two organizations (peerOrg1, peerOrg2), each of which has 2 peers, one fabric-ca service.

该测试包含一个orderer节点，两个组织(peerOrg1, peerOrg2)，每个组织各自包含两个节点，以及一个fabric-ca服务。

#### 证书及其他加密工件(Certificates and other cryptography artifacts)

Fabric requires that each organization has private keys and certificates for use in signing and verifying messages going to and from clients, peers and orderers.
Each organization groups these artifacts in an **MSP** (Membership Service Provider) with a corresponding unique _MSPID_ .

Furthermore, each organization is assumed to generate these artifacts independently. The *fabric-ca* project is an example of such a certificate generation service.
Fabric also provides the `cryptogen` tool to automatically generate all cryptographic artifacts needed for the end to end test.
In the directory src/test/fixture/sdkintegration/e2e-2Orgs/channel

  The command used to generate end2end `crypto-config` artifacts:</br>

  ```build/bin/cryptogen generate --config crypto-config.yaml --output=crypto-config```

For ease of assigning ports and mapping of artifacts to physical files, all peers, orderers, and fabric-ca are run as Docker containers controlled via a docker-compose configuration file.

The files used by the end to end are:
 * _src/test/fixture/sdkintegration/e2e-2Orgs/channel_  (everything needed to bootstrap the orderer and create the channels)
 * _src/test/fixture/sdkintegration/e2e-2Orgs/crypto-config_ (as-is. Used by `configtxgen` and `docker-compose` to map the MSP directories)
 * _src/test/fixture/sdkintegration/docker-compose.yaml_


The end to end test case artifacts are stored under the directory _src/test/fixture/sdkintegration/e2e-2Org/channel_ .

Fabric需要每个组织都有自己的私钥和证书，用来签名和验证客户端、节点和orderer之间的消息。
每个组织的这些工件在**MSP** (Membership Service Provider)中都有统一的唯一 _MSPID_。

而且，我们假设各个组织都是独立的生成这些工件。*fabric-ca*工程是一个证书生成服务的一个例子。
Fabric同时提供`cryptogen`工具来自动生成端对端测试所需的加密用的工件。
在src/test/fixture/sdkintegration/e2e-2Orgs/channel目录

   使用 `crypto-config` 命令来生成 end2end 工件：</br>
   ```build/bin/cryptogen generate --config crypto-config.yaml --output=crypto-config```

为了方便分配端口和映射这些工件到物理文件，所有的peer， order 和fabric-ca都运行在Docker容器中，通过docker-compose配置文件来控制。

使用到了下面这些文件：
 * _src/test/fixture/sdkintegration/e2e-2Orgs/channel_  (everything needed to bootstrap the orderer and create the channels)
 * _src/test/fixture/sdkintegration/e2e-2Orgs/crypto-config_ (as-is. Used by `configtxgen` and `docker-compose` to map the MSP directories)
 * _src/test/fixture/sdkintegration/docker-compose.yaml_

端对端测试用例工件存储在_src/test/fixture/sdkintegration/e2e-2Org/channel_目录中。

### 通过TLS连接到Orderer和Peer(TLS connection to Orderer and Peers)

IBM Java needs the following properties defined to use TLS 1.2 to get an HTTPS connections to Fabric CA.
```
-Dcom.ibm.jsse2.overrideDefaultTLS=true   -Dhttps.protocols=TLSv1.2
```

We need certificate and key for each of the Orderer and Peers for TLS connection. You can generate your certificate and key files with openssl command as follows:

 * Set up your own Certificate Authority (CA) for issuing certificates
 * For each of orderers and peers:
   * generate a private key: <code>openssl genrsa 512 > key.pem</code>.
   * generate a certificate request (csr): <code>openssl req -new -days 365 -key key.pem -out csr.pem</code>, which will request your input for some information, where CN has to be the container's alias name (e.g. peer0, peer1, etc), all others can be left blank.
   * sign the csr with the CA private key to generate a certificate: <code>openssl ca -days 365 -in csr.pem -keyfile {CA's privatekey} -notext -out cert.pem</code>
   * put the resulting cert.pem and key.pem together with the CA's certificate (as the name cacert.pem) in the directory where the docker container can access.

The option -notext in the last openssl command in the above is important. Without the option, the resulting cert.pemmay does not work for some Java implementation (e.g. IBM JDK).
The certificates and keys for the end-to-end test case are stored in the directory _src/test/fixture/sdkintegration/e2e-2Org/tls/_.

Currently, the pom.xml is set to use netty-tcnative-boringssl for TLS connection to Orderer and Peers, however, you can change the pom.xml (uncomment a few lines) to use an alternative TLS connection via ALPN.

IBM Java需要以下定义一下参数才能使用TLS 1.2来和Fabric CA建立HTTPS连接。
```
-Dcom.ibm.jsse2.overrideDefaultTLS=true   -Dhttps.protocols=TLSv1.2
```

每个Orderer和Peer建立TLS连接都需要证书和私钥。你可以按以下步骤通过openssl命令来生成证书和私钥文件：
 * 设置你自己的证书认证机构（CA）来发行证书
 * 对每一个Orderer和Peer
 	* 生成私钥 <code>openssl genrsa 512 > key.pem</code>。
 	* 生成证书请求（csr）<code>openssl req -new -days 365 -key key.pem -out csr.pem</code>，这里可能需要你填写一些信息，CN应该是容器的别名（比如peer0， peer1等），其余的都可以留空
 	* 通过CS的私钥来给证书请求（CSR）签名并生成证书：<code>openssl ca -days 365 -in csr.pem -keyfile {CA's privatekey} -notext -out cert.pem</code>
 	* 将生成的cert.pem 和 key.pem 同CA的证书(名字cacert.pem)一起放到docker容器可以访问到的目录中。

上面最后一条openssl命令中的 -notext 选项非常重要。没有这个参数的话，生成的cert.pemmay没办法在某些java环境中用（比如java JDK）。
这些证书和钥匙文件放在_src/test/fixture/sdkintegration/e2e-2Org/tls/_目录中。

目前，pom.xml设置使用netty-tcnative-boringssl在Orderer和Peer之间建立TLS连接，你可以更该pom.xml(注释其中一些行)来通过ALPN来使用其他的TLS连接。


### 链码背书策略(Chaincode endorsement policies)

Policies are described in the [Fabric Endorsement Policies document](https://gerrit.hyperledger.org/r/gitweb?p=fabric.git;a=blob;f=docs/endorsement-policies.md;h=1eecf359c12c3f7c1ddc63759a0b5f3141b07f13;hb=HEAD).
You create a policy using a Fabric tool ( an example is shown in [JIRA issue FAB-2376](https://jira.hyperledger.org/browse/FAB-2376?focusedCommentId=21121&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-21121))
and give it to the SDK either as a file or a byte array. The SDK, in turn, will use the policy when it creates chaincode instantiation requests.


To input a policy to the SDK, use the **ChaincodeEndorsementPolicy** class.

For testing purposes, there are 2 policy files in the _src/test/resources_ directory
  * _policyBitsAdmin_ ( which has policy **AND(DEFAULT.admin)** meaning _1 signature from the DEFAULT MSP admin' is required_ )
  * _policyBitsMember_ ( which has policy **AND(DEFAULT.member)** meaning _1 signature from a member of the DEFAULT MSP is required_ )

and one file in the _src/test/fixture/sdkintegration/e2e-2Orgs/channel_ directory specifically for use in the end to end test scenario
  * _members_from_org1_or_2.policy_ ( which has policy **OR(peerOrg1.member, peerOrg2.member)** meaning  _1 signature from a member of either organizations peerOrg1, PeerOrg2 is required_)

 Alternatively, you can also use ChaincodeEndorsementPolicy class by giving it a YAML file that has the policy defined in it.
 See examples of this in the End2endIT testcases that use _src/test/fixture/sdkintegration/chaincodeendorsementpolicy.yaml_
 The file chaincodeendorsementpolicy.yaml has comments that help understand how to create these policies. The first section
 lists all the signature identities you can use in the policy. Currently, only ROLE types are supported.
 The policy section is comprised of `n-of` and `signed-by` elements.  Then n-of (`1-of` `2-of`) require that many (`n`) in that
 section to be true. The `signed-by` references an identity in the identities section.

背书策略在[Fabric Endorsement Policies document](https://gerrit.hyperledger.org/r/gitweb?p=fabric.git;a=blob;f=docs/endorsement-policies.md;h=1eecf359c12c3f7c1ddc63759a0b5f3141b07f13;hb=HEAD)上做了详细的阐释。
你可以通过Fabric工具来创建一个策略(在 [JIRA issue FAB-2376](https://jira.hyperledger.org/browse/FAB-2376?focusedCommentId=21121&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-21121) 有个例子)以文件或byte数组方式传给SDK。同时SDK会使用该策略来创建链码初始化请求。

使用**ChaincodeEndorsementPolicy**类将策略传给SDK。
为了测试目的，在_src/test/resources_下有两个策略文件。
  * _policyBitsAdmin_ ( 包含 **AND(DEFAULT.admin)** 策略， _需要一个DEFAULT MSP 管理员的签名_ )
  * _policyBitsMember_ ( 包含 **AND(DEFAULT.member)** 策略， _需要一个DEFAULT MSP 成员的签名_ )

在_src/test/fixture/sdkintegration/e2e-2Orgs/channel_下油一个文件，指定了端对端的测试场景
  * _members_from_org1_or_2.policy_ ( 包含 **OR(peerOrg1.member, peerOrg2.member)** 策略  _需要任一组织(peerOrg1, PeerOrg2)下成员的签名_)

另外，你也可以用ChaincodeEndorsementPolicy类，通过给他指定一个包含策略定义的YAML文件来实现。
参照_src/test/fixture/sdkintegration/chaincodeendorsementpolicy.yaml_目录下End2endIT测试用例。 
文件chaincodeendorsementpolicy.yaml中，有一些帮助理解如何创建策略的注释。第一部分列出了所有可以在策略中使用的签名。当前只支持ROLE类型。
policy区块包含`n-of` 和 `signed-by` 元素。n-of (`1-of` `2-of`) 元素需要在这个区块中将“many” (`n`) 设置为 true. `signed-by`元素引用了在identities区块的身份信息。

### 通道创建工件(Channel creation artifacts)

Channel configuration files and orderer bootstrap files ( see directory _src/test/fixture/sdkintegration/e2e-2Orgs/channel_ ) are needed when creating a new channel.
This is created with the Hyperledger Fabric `configtxgen` tool.

For End2endIT.java the commands are

 * build/bin/configtxgen -outputCreateChannelTx foo.tx -profile TwoOrgsChannel -channelID foo
 * build/bin/configtxgen -outputCreateChannelTx bar.tx -profile TwoOrgsChannel -channelID bar
 * build/bin/configtxgen -outputBlock orderer.block -profile TwoOrgsOrdererGenesis

with the configtxgen config file _src/test/fixture/sdkintegration/e2e-2Orgs/channel/configtx.yaml_


If `build/bin/configtxgen` tool is not present  run `make configtxgen`

Before running the end to end test case:
 *  you may need to modify `configtx.yaml` to change all hostname and port definitions to match
your server(s) hostname(s) and port(s).
 *  you **WILL** have to modify `configtx.yaml` to have the _MSPDir_ point to the correct path to the _crypto-config_ directories.
   * `configtx.yaml` currently assumes that you are running in a Vagrant environment where the fabric, fabric-ca and fabric-sdk-java projects exist under the _/opt/gopath/src/github.com/hyperledger_ directory.

创建通道的时候，需要有通道的配置文件以及orderer引导文件（在_src/test/fixture/sdkintegration/e2e-2Orgs/channel_目录下）。
这个配置文件是通过`configtxgen`来创建的。

对于End2endIT.java类来说，命令是：
 * build/bin/configtxgen -outputCreateChannelTx foo.tx -profile TwoOrgsChannel -channelID foo
 * build/bin/configtxgen -outputCreateChannelTx bar.tx -profile TwoOrgsChannel -channelID bar
 * build/bin/configtxgen -outputBlock orderer.block -profile TwoOrgsOrdererGenesis

配置文件是  _src/test/fixture/sdkintegration/e2e-2Orgs/channel/configtx.yaml_。

如果`build/bin/configtxgen`工具不存在，则需要执行`make configtxgen`来生成。

在你运行端对端测试用例前：
* 你需要修改`configtx.yaml`文件，将主机名和端口定义修改成你服务器真实的主机名和端口。
* 你将会需要修改`configtx.yaml`文件，将_MSPDir_指向到正确的_crypto-config_目录。
  * `configtx.yaml`文件当前假设你运行在Vagrant环境，而且fabric、fabric-ca 和fabric-sdk-java 在_/opt/gopath/src/github.com/hyperledger_目录下。

### GO语言链码(GO Lang chaincode)

Go lang chaincode dependencies must be contained in vendor folder.
 For an explanation of this see [Vender folder explanation](https://blog.gopheracademy.com/advent-2015/vendor-folder/)

Go语言链码的依赖文件必须包含在vendor目录下。
详细解释参照[Vender folder explanation](https://blog.gopheracademy.com/advent-2015/vendor-folder/)

## 简单的问题解决(Basic Troubleshooting)

**identity or token do not match**

Keep in mind that you can perform the enrollment process with the membership services server only once, as the enrollmentSecret is a one-time-use password. If you have performed a FSUser registration/enrollment with the membership services and subsequently deleted the crypto tokens stored on the client side, the next time you try to enroll, errors similar to the ones below will be seen.

``Error: identity or token do not match``

``Error: FSUser is already registered``

To address this, remove any stored crypto material from the CA server by following the instructions <a href="https://github.com/hyperledger/fabric/blob/master/docs/Setup/Chaincode-setup.md#removing-temporary-files-when-security-is-enabled">here</a> which typically involves deleting the /var/hyperledger/production directory and restarting the membership services. You will also need to remove any of the crypto tokens stored on the client side by deleting the KeyValStore . That KeyValStore is configurable and is set to ${FSUser.home}/test.properties within the unit tests.

When running the unit tests, you will always need to clean the membership services database and delete the KeyValStore file, otherwise, the unit tests will fail.

记住你只能运行成员服务的登记进程（enrollment process）一次，因为enrollmentSecret是一次性密码。如果你已经执行过一次FSUser registration/enrollment，然后删除了加密存储在客户端的token后，下一次enroll的时候，就会出现下面类似的问题：
``Error: identity or token do not match``

``Error: FSUser is already registered``

为了解决这个问题，请参照<a href="https://github.com/hyperledger/fabric/blob/master/docs/Setup/Chaincode-setup.md#removing-temporary-files-when-security-is-enabled">here</a> 删除所有存储在CA服务器上的的加密文件，文件目录在/var/hyperledger/production，然后重启成员服务。同时你还需要通过删除KeyValStore来清楚客户端所有的加密token。KeyValStore是可配置的，参照${FSUser.home}/test.properties文件。

当运行测试用例时，你需要清理成员服务器数据库并删除KeyValStore文件，否则单元测试会失败。

**java.security.InvalidKeyException: Illegal key size**

If you get this error, this means your JDK does not capable of handling unlimited strength crypto algorithms. To fix this issue, You will need to download the JCE libraries for your version of JDK. Please follow the instructions <a href="http://stackoverflow.com/questions/6481627/java-security-illegal-key-size-or-default-parameters">here</a> to download and install the JCE for your version of the JDK.

如果你碰到这个问题，就是你的JDK不支持无限长度加密逻辑。要修复这个问题，你需要下载正对你JDK版本的JCE库，请参照<a href="http://stackoverflow.com/questions/6481627/java-security-illegal-key-size-or-default-parameters">这里</a>来下载和安装JCE库。

## 和开发者交流(Communicating with developers and fellow users)

 Sign into <a href="https://chat.hyperledger.org/">Hyperledger project's Rocket chat</a>
 For this you will also need a <a href="https://identity.linuxfoundation.org/">Linux Foundation ID</a>

 Join the <b>fabric-sdk-java</b> channel.

 登录<a href="https://chat.hyperledger.org/">Hyperledger project's Rocket chat</a>。你需要一个<a href="https://identity.linuxfoundation.org/">Linux Foundation ID</a>。 然后加入<b>fabric-sdk-java</b>频道。

## 报告问题(Reporting Issues)

If your issue is with building Fabric development environment please discuss this on rocket.chat's #fabric-dev-env channel.
如果你有编译Fabric开发环境的问题，请在rocket.chat的 #fabric-dev-env channel讨论。

To report an issue please use: <a href="http://jira.hyperledger.org/">Hyperledger's JIRA</a>.
To login you will need a Linux Foundation ID (LFID) which you get at <a href="https://identity.linuxfoundation.org/">The Linux Foundation</a>
if you don't already have one.

报告问题请使用JIRA：<a href="http://jira.hyperledger.org/">Hyperledger's JIRA</a>。
你需要使用Linux基金会ID(LFID)来登录，如果没有，可以在<a href="https://identity.linuxfoundation.org/">The Linux Foundation</a>注册一个。

JIRA Fields should be:
<dl>
  <dt>Type</dt>
  <dd>Bug <i>or</i> New Feature</dd>

  <dt>Component</dt>
  <dd>fabric-sdk-java</dd>
  <dt>Fix Versions</dt>
    <dd>v1.0.1</dd>
</dl>

Pleases provide as much information that you can with the issue you're experiencing: stack traces logs.

Please provide the output of **java -XshowSettings:properties -version**

Logging for the SDK can be enabled with setting environment variables:

ORG_HYPERLEDGER_FABRIC_SDK_LOGLEVEL=TRACE

ORG_HYPERLEDGER_FABRIC_CA_SDK_LOGLEVEL=TRACE

Fabric debug is by default enabled in the SDK docker-compose.yaml file with

On Orderer:

ORDERER_GENERAL_LOGLEVEL=debug

On peers:
CORE_LOGGING_LEVEL=DEBUG

Fabric CA
by starting command have the -d parameter.

Upload full logs to the JIRA not just where the issue occurred if possible

请提供尽量多的信息，比如堆栈跟踪记录。
请提供 **java -XshowSettings:properties -version** 命令的输出结果
可能的话，请上传当问题发生时完整的日志。

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
