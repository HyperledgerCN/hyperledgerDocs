
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html) | Zhangjiong Xuan | Zhangjiong Xuan |

**注意：**这些说明已经被验证，适用于被标记“1.0.0-rc1”的Docker镜像和提供tar文件中的预编译的实用程序。如果你在当前的主分支下使用下列命令以及镜像或者工具，你可能会看到一些配置和panic错误。

构建你的第一个网络（BYFN）场景提供了由两个组织组成的示例Hyperledger Fabric网络，每个组织持有2个peer节点，以及一个“solo”排序服务。

## 1.1. 安装预置环境

在我们开始之前，如果你还没有这样做，你可能需要检查一下在你开发区块链应用程序或者Hyperledger Fabric的平台上是否已经安装了预置环境。

你还需要下载并安装`Hyperledger Fabric Samples`。你会注意到`fabric-samples`文件夹中包含了许多示例。我们将使用`first-network`这个例子。现在让我们打开这个子目录。
```bash
cd first-network
```

>注意
>>本文档中提供的命令必须运行在`fabric-network`的子目录`first-network`中。如果你选择从其他位置运行命令，提供的一些列脚本将无法找到对应的二进制。

## 1.2. 想要现在运行吗？

我们提供一个完全注释的脚本`byfn.sh`，利用这些Docker镜像可以快速引导一个由4个代表2个不同组织的peer节点以及一个排序服务节点的`Hyperledger fabric`网络。它还将启动一个容器来运行一个将peer节点加入channel、部署实例化链码服务以及驱动已经部署的链码执行交易的脚本。

以下是该`byfn.sh`脚本的帮助文档：
```bash
./byfn.sh -h
Usage:
  byfn.sh -m up|down|restart|generate [-c <channel name>] [-t <timeout>]
  byfn.sh -h|--help (print this message)
    -m <mode> - one of 'up', 'down', 'restart' or 'generate'
      - 'up' - bring up the network with docker-compose up
      - 'down' - bring up the network with docker-compose up
      - 'restart' - bring up the network with docker-compose up
      - 'generate' - generate required certificates and genesis block
    -c <channel name> - config name to use (defaults to "mychannel")
    -t <timeout> - CLI timeout duration in microseconds (defaults to 10000)

Typically, one would first generate the required certificates and
genesis block, then bring up the network. e.g.:

  byfn.sh -m generate -c <channelname>
  byfn.sh -m up -c <channelname>
```

如果你选择不提供channel名称，则脚本将使用默认名称`mychannel`。CLI超时参数（用-t标志指定）是一个可选值;如果你选择不设置它，那么CLI容器将会在脚本执行完之后退出。

## 1.3. 生成网络神器

准备好了吗？好吧！执行以下命令。你将会看到会发生什么伴随yes/no命令行提示的简要说明。输入y来执行描述的动作。

```bash
./byfn.sh -m generate
Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10000'
Continue (y/n)?y
proceeding ...
/Users/xxx/dev/fabric-samples/bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
org1.example.com
2017-06-12 21:01:37.334 EDT [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.
...

/Users/xxx/dev/fabric-samples/bin/configtxgen
##########################################################
#########  Generating Orderer Genesis block ##############
##########################################################
2017-06-12 21:01:37.558 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 002 intermediate certs folder not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts]. Skipping.: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts: no such file or directory]
...
2017-06-12 21:01:37.588 EDT [common/configtx/tool] doOutputBlock -> INFO 00b Generating genesis block
2017-06-12 21:01:37.590 EDT [common/configtx/tool] doOutputBlock -> INFO 00c Writing genesis block

#################################################################
### Generating channel configuration transaction 'channel.tx' ###
#################################################################
2017-06-12 21:01:37.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-06-12 21:01:37.644 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2017-06-12 21:01:37.645 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########
#################################################################
2017-06-12 21:01:37.674 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-06-12 21:01:37.678 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2017-06-12 21:01:37.679 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########
#################################################################
2017-06-12 21:01:37.700 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```

第一步生成我们各种网络实体的所有证书和密钥，`genesis block`用于引导排序服务，以及配置`Channel`所需要的一组交易配置集合。

## 1.4. 启动网络

接下来，你可以使用以下命令来启动整个网络。再试提示你是否继续。回答y：

```bash
./byfn.sh -m up
Starting with channel 'mychannel' and CLI timeout of '10000'
Continue (y/n)?y
proceeding ...
Creating network "net_byfn" with the default driver
Creating peer0.org1.example.com
Creating peer1.org1.example.com
Creating peer0.org2.example.com
Creating orderer.example.com
Creating peer1.org2.example.com
Creating cli


 ____    _____      _      ____    _____
/ ___|  |_   _|    / \    |  _ \  |_   _|
\___ \    | |     / _ \   | |_) |   | |
 ___) |   | |    / ___ \  |  _ <    | |
|____/    |_|   /_/   \_\ |_| \_\   |_|

Channel name : mychannel
Creating channel...
```

The logs will continue from there. This will launch all of the containers, and then drive a complete end-to-end application scenario. Upon successful completion, it should report the following in your terminal window:

日志将继续。然后启动所有容器，驱动一个端到端的应用场景。成功以后，在终端窗口中会报告以下内容：
```bash
2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
Query Result: 90
2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
===================== Query on PEER3 on channel 'mychannel' is successful =====================

===================== All GOOD, BYFN execution completed =====================


 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

你可以滚动这些日志去查看各种交易。如果你没有获得这个结果，请移步疑难解答部分，看看我们是否可以帮助你发现问题。

## 1.5. 关闭网络

最后，让我们把它全部停下来，这样我们可以一步一步地探索网络设置。以下操作将关闭你的容器，移除加密材料和4个配置信息，并且从Docker仓库删除chaincode镜像。你将再一次被提示是否继续，回答y：
```bash
./byfn.sh -m down
Stopping with channel 'mychannel' and CLI timeout of '10000'
Continue (y/n)?y
proceeding ...
WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
Removing network net_byfn
468aaa6201ed
...
Untagged: dev-peer1.org2.example.com-mycc-1.0:latest
Deleted: sha256:ed3230614e64e1c83e510c0c282e982d2b06d148b1c498bbdcc429e2b2531e91
...
```

如果你想了解关于底层工具和引导材料相关的更多信息，请继续阅读。在接下来的章节中，我们将浏览构建功能齐全的`Hyperledger fabric`网络的各种要求和步骤。

## 1.6. 加密生成器

我们将使用`cryptogen`工具为我们生成各种网络实体的加密材料（x509证书）。这些证书是身份的代表，它们允许在我们的网络实体进行交流和交易时进行签名/验证身份验证。

### 1.6.1. 它是如何工作的？

`Cryptogen`消费一个包含网络拓扑的`crypto-config.yaml`，并允许我们为组织和属于这些组织的组件生成一组证书和密钥。每个组织都配置了唯一的根证书(`ca-cert`),它将特定组件（peers和orders）绑定到该组织。通过为每一个组织分配唯一的CA证书，我们正在模仿一个经典的网络，这个网络中的成员将使用自己的证书颁发机构。Hyperledger Fabric中的交易和通信是通过存储在`keystore`中的实体的私钥签名，然后通过公钥手段进行验证（`signcerts`）。

你将注意到在这个文件里有一个`count`变量。我们将使用它来指定每个组织中`peer`的数量;在我们的例子中，每个组织有两个peer。我们现在不会深入研究[x.509证书和公钥基础设施](https://en.wikipedia.org/wiki/Public_key_infrastructure)的细节。如果你有兴趣，你可以在自己的时间细读这些主题。

在运行该工具之前，让我们快速浏览一下这段代码`crypto-config.yaml`。特别注意在`OrdererOrgs`头下的`Name`，`Domain`和`Specs`参数：
```bash
OrdererOrgs:
#---------------------------------------------------------
# Orderer
# --------------------------------------------------------
- Name: Orderer
  Domain: example.com
  # ------------------------------------------------------
  # "Specs" - See PeerOrgs below for complete description
# -----------------------------------------------------
  Specs:
    - Hostname: orderer
# -------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
# ------------------------------------------------------
PeerOrgs:
# -----------------------------------------------------
# Org1
# ----------------------------------------------------
- Name: Org1
  Domain: org1.example.com
```

网络实体的命名约定如下："{{.Hostname}}.{{.Domain}}"。所以使用我们的排序节点作为参考点，它与`Order`的MSP ID相关联。该文件包含了有关定义和语法的大量文档。你还可以参考[Membership Service Providers(MSP)](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html)，以便更深入地了解MSP。

我们运行`cryptogen`工具，生成的证书和密钥将被保存到名为`crypto-config`的文件夹中。

## 1.7. 配置交易生成器

`configtxgen tool`用于创建4个配置工作：
* order的`genesis block`,
* channel的`channel configuration transaction`,
* 以及两个`anchor peer transactions`一个对应一个Peer组织。

有关此工具的完整说明，请参阅[Channel Configuration(configtxgen)](http://hyperledger-fabric.readthedocs.io/en/latest/configtxgen.html)。

`order block`是一个ordering service的[创世区块](http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#genesis-block)，`channel transaction`文件在`Channel`创建的时侯广播给order。`anchor peer transactions`，正如名称所示，指定了每个组织在此channel上的[`Anchor peer`](http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#anchor-peer)。

### 1.7.1. 它是如何工作的？

Configtxgen使用一个包含示例网络的`configtx.yaml`文件。有3个成员-一个排序服务组织`OrdererOrg`以及两个节点组织（`Org1`&`Org2`）,每个组织管理和持有2个peer节点。该文件还指定了一个`SampleConsortium`的联盟，由上述2个节点组织构成。
请特别注意此文件顶部的"Profiles"部分。你会注意到我们有两个独特的标题。一个是orderer的创世区块-`TwoOrgsOrdererGenesis`-另一个是针对管道的`TwoOrgsChannel`。

这些标题很重要，因为在我们创建我们的工作的时侯她们将作为传递的参数。

>注意
>>请注意我们的`SampleConsortium`在系统界别的配置文件中定义，然后由渠道级别配置文件引用。管道存在于联盟的范围内，所有的联盟必须定义在整个网络范围内。

此文件还包含两个值得注意的附加规格。首先，我们为每个组织指定了锚点节点（`peer0.org1.example.com`和`peer0.org2.example.com`）。其次，我们为每个成员指定MSP文件夹，用来存储每个组织在`orderer genesis block`中指定的根证书。这是一个关键的概念。现在任意和ordering service通信的网络实体都可以对其数字签名进行验证。

## 1.8. 运行工具

你可以用`configtxgen`和`cryptogen`命令来手动生成证书/密钥和各种配置文件。或者，你可以尝试使用`byfn.sh`脚本来完成你的目标。

### 1.8.1. 手动生成配置文件

必要的话，你可以参考byfn.sh脚本中的`generateCerts`函数去生成相关定义在`crypto-config.yaml`文件中用于你的网络配置的相关证书。然而，为了方便起见，我们也将在此提供参考。

首先，我们来运行`cryptogen`这个工具。我们的二进制文件在`bin`目录中，所以我们需要提供工具所在的相对路径。

```
../bin/cryptogen generate --config=./crypto-config.yaml
```

你可能会看到以下警告。这是无害的，请忽略它： 

```
[bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.
```

接下来，我们需要告诉`configtxgen`工具需要提取的`configtx.yaml`所在的位置。我们会告诉它在我们当前所在工作目录：

首先，我们需要设置一个环境变量来告诉`configtxgen`哪里去寻找configtx.yaml。然后，我们将调用`configtxgen`工具去创建`orderer genesis block`：
```bash
export FABRIC_CFG_PATH=$PWD
../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```

你可以忽略有关中间证书，证书撤销列表（crls）和MSP配置的日志警告。我们没有在示例网络中使用其中的任何一个。

接下来，我们需要创建`channel transaction`配置。请确保替换`$CHANNEL_NAME`或者将`CHANNEL_NAME`设置为整个说明中可以使用的环境变量：

```
export CHANNEL_NAME=mychannel

# this file contains the definitions for our sample channel
../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```

接下来，我们将在正在构建的通道上定义`Org1`的`anchor peer`。请再次确认$CHANNEL_NAME已被替换或者为以下命令设置了环境变量：

```
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
```

现在，我们将在同一个通道定义`Org2`的`anchor peer`：

```
../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```

## 1.9. 启动网络

我们将利用`docker-compose`脚本来启动我们的区块链网络。`docker-compose`文件利用我们之前下载的镜像，并用以前生成的`genesis.block`来引导`orderer`。

```
working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
# command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
volumes
```

如果没有注释，该脚本将在网络启动时执行所有命令，正如我们在[幕后发生的情况](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#behind-scenes)中所描述的那样。然而，我们想手动执行命令，以便公开每个调用的语法和功能。

适当地为`TIMEOUT`传递较高的值（以秒为单位）;默认情况下CLI容器将在60秒之后退出。

启动你的网络：
```bash
CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml up -d
```

如果要实时查看你的区块链网络的日志，请不要提供`-d`标志。如果你需要日志流，你需要打开第二个终端来执行CLI命令。

### 1.9.1. 环境变量

为了使针对`peer0.org1.example.com`的CLI命令起作用，我们需要使用下面给出四个环境变量来介绍我们的命令。为`peer0.org1.example.com`涉及的这些变量将被拷贝到CLI容器中，因此我们不需要复制它们。然而，如果你发送调用到其他的peer节点或者orderer，则需要相应地提供这些值。检查`docker-compose-base.yaml`中的具体路径：

```
# Environment variables for PEER0

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

### 1.9.2. 创建&加入信道

我们将使用`docker exec`命令进入CLI容器：
```bash
docker exec -it cli bash
```

如果成功，你将看到下列信息：
```
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```

回想以下，我们使用configtxgen工具生成信道配置-`channel.tx`。我们将这个配置作为请求的一部分传递给order。

>注意
>>注意`-- cafile`会作为命令的一部分。这是orderer的root cert的本地路径，允许我们去验证TLS握手。

我们使用`-c`标志指定channel的名字，`-f`标志指定配置交易。在这个例子中它是`channel.tx`，当然你也可以使用不同的名称，挂载你自己的交易配置。

```bash
export CHANNEL_NAME=mychannel

# the channel.tx file is mounted in the channel-artifacts directory within your CLI container
# as a result, we pass the full path for the file
# we also pass the path for the orderer ca-cert in order to verify the TLS handshake
# be sure to replace the $CHANNEL_NAME variable appropriately

peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```

此命令返回一个创世区块-`<channel-ID.block>`-我们将使用它加入信道。它包含了`channel.tx`中的配置信息。

>注意
>>剩下的命令将会留在CLI容器内执行。你必须记住所有的命令必须在相应的环境变量下执行当目标节点是除了`peer0.org1.example.com`以外的节点。

现在让我们加入`peer0.org1.example.com`频道。
```bash
# By default, this joins ``peer0.org1.example.com`` only
# the <channel-ID>.block was returned by the previous command

 peer channel join -b <channel-ID.block>
```

你可以修改4个环境变量来让别的节点加入信道。

### 1.9.3. 安装和实例化链码

>注意
>>我们将利用一个现有的简单链码，来学习如何编写自己的链码，请参考[链码服务开发指南](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)

应用程序和区块链账本会相互影响通过`chaincode`。因此，我们需要在每个会执行以及背书我们交易的peer节点安装chaincode，然后在信道上实例化chaincode。

首先，在将示例代码安装到4个peer节点中的其中一个。这个命令将源代码放到peer节点的文件系统中。
```bash
peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
```

接下来，在信道上实例化chaincode。这将初始化信道上的链码，设置链码的背书策略，为目标peer节点启动一个chaincode容器注意`-P`参数。这是我们需要指定的当这个chaincode的交易需要被验证的时侯的背书策略。

在下面的命令中，你会注意到我们指定`-P "OR ('Org0MSP.member','Org1MSP.member')"`作为背书策略。这意味着我们需要Org1或者Org2组织中的其中一个的节点的背书即可（即只有一个背书）。如果我们改变语法为`AND`那么我们就需要2个背书者。
```bash
# be sure to replace the $CHANNEL_NAME environment variable
# if you did not install your chaincode with a name of mycc, then modify that argument as well

peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```

有关更多背书策略的详细信息请参考[背书策略](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)

### 1.9.4. 查询

让我们查询一下`a`的值，以确保链码被正确实例化，`state DB`被填充。查询的语法如下：
```bash
# be sure to set the -C and -n flags appropriately

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```

### 1.9.5. 调用

现在让我们从`a`账户转`10`到`b`账户。这个交易将创建一个新的区块并更新`state DB`。调用语法如下：
```bash
# be sure to set the -C and -n flags appropriately

peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

### 1.9.6. 查询

让我们确认下我们之前的调用被正确地执行了。我们初始化了`a`的值为`100`，在上一次调用的时侯转移了`10`给`b`。因此，查询`a`应该展示`90`。查询的语法如下：

```bash
# be sure to set the -C and -n flags appropriately

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```

我们应该看到以下内容：

```bash
Query Result: 90
```

随时重新开始并操作键值对和随后的调用。

### 1.9.7. 幕后发生了什么?

>注意
>>这些步骤描述了在`script.sh`脚本中没有注释掉的`docker-compose-cli.yaml`文件中的场景。使用`./byfn.sh -m down`并确保命令执行成功。然后使用相同的docker-compose提示去启动你的网络。

* `script.sh`脚本被拷贝到CLI容器中。这个脚本驱动了使用提供的channel name以及信道配置的channel.tx文件的`createChannel`命令。

* `createChannel`命令的产出是一个创世区块-`<your_channel_name>.block`-这个创世区块被存储在peer节点的文件系统中同时包含了在channel.tx的信道配置。

* `joinChannel`命令被4个peer节点执行，作为之前产生的genesis block的输入。这个命令介绍了peer节点加入`<your_channel_name>`以及利用`<your_channel_name>.block`去创建一条链。

* 现在我们有了由4个peer节点以及2个组织构成的信道。这是我们的`TwoOrgsChannel`配置文件。

* `peer0.org1.example.com`和`peer1.org1.example.com`属于Org1;`peer0.org2.example.com`和`peer1.org2.example.com`属于Org2

* 这些关系是通过crypto-config.yaml定义的，MSP路径在docker-compose文件中被指定。

* Org1MSP(`peer0.org1.example.com`)和Org2MSP(`peer0.org2.example.com`)的anchor peers将在后续被更新。我们通过携带channel的名字传递`Org1MSPanchors.tx`和`Org2MSPanchors.tx`配置到排序服务来实现anchor peer的更新。

* 一个链码-`chaincode_example02`被安装在`peer0.org1.example.com`和`peer0.org2.example.com`

* 这个链码在`peer0.org2.example.com`被实例化。实例化过程将链码添加到信道上，并启动peer节点对应的容器，并且初始化和链码服务有关的键值对。示例的初始化的值是`[”a“,”100“，”b“，”200“]`。实例化的结果是一个名为`dev-peer0.org2.example.com-mycc-1.0`的容器启动了。

* 实例化过程同样为背书策略传递相关参数。策略被定义为`-P "OR    ('Org1MSP.member','Org2MSP.member')"`，意思是任何交易必须被Org1或者Org2背书。

* 一个针对`a`的查询发往`peer0.org1.example.com`。链码服务已经被安装在了`peer0.org1.example.com`，因此这次查询将启动一个名为`dev-peer0.org1.example.com-mycc-1.0`的容器。查询的结果也将被返回。没有写操作出现，因此查询的结果的值将为`100`。

* 一次`invoke`被发往`peer0.org1.example.com`，从`a`转移`10`到`b`。

* 然后链码服务被安装到`peer1.org2.example.com`

* 一个`query`请求被发往`peer1.org2.example.com`用于查询`a`的值。这将启动第三个链码服务名为`dev-peer1.org2.example.com-mycc-1.0`。返回`a`的值为90,正确地反映了之前的交易，`a`的值被转移了10。

### 1.9.8. 这指明了什么？

为了能够正确地在账本上进行读写操作，链码服务必须被安装在peer节点上。此外，每个peer节点的链码服务的容器除了`init`或者传统的交易-读/写-针对该链码服务执行（例如查询`a`的值），在其他情况下不会启动。交易导致容器的启动。当然，所有信道中的节点都持有以块的形式顺序存储的不可变的账本精确的备份，以及状态数据库来保存前状态的快照。这包括了没有在其上安装链码服务的peer节点（`peer1.org2.example.com`如上所示）。最后，链码在被安装后将是可达状态，因为它已经被实例化了。

### 1.9.9. 我如何查询这些交易？

检查CLI容器的日志。
```bash
docker logs -f cli
```
你应该看到以下输出：
```bash
2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
Query Result: 90
2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
===================== Query on PEER3 on channel 'mychannel' is successful =====================

===================== All GOOD, BYFN execution completed =====================


 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

你可以滚动这些日志来查看各种交易。

### 1.9.10. 我如何查看链码日志？

检查每个独立的链码服务容器来查看每个容器内的分隔的交易。下面是每个链码服务容器的日志的组合：
```bash
$ docker logs dev-peer0.org2.example.com-mycc-1.0
04:30:45.947 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Init
Aval = 100, Bval = 200

$ docker logs dev-peer0.org1.example.com-mycc-1.0
04:31:10.569 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Invoke
Query Response:{"Name":"a","Amount":"100"}
ex02 Invoke
Aval = 90, Bval = 210

$ docker logs dev-peer1.org2.example.com-mycc-1.0
04:31:30.420 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
ex02 Invoke
Query Response:{"Name":"a","Amount":"90"}
```

## 1.10. 了解 Docker Compose 技术

BYFN示例给我们提供了两种风格的Docker Compose文件，它们都继承自`docker-compose-base.yaml`（`base`目录下）。我们的第一种类型，`docker-compose-cli.yaml`给我们提供了一个CLI容器，以及一个orderer容器，四个peer容器。我们用此文件来展开这个页面上的所有说明。

>注意
>>本节的剩余部分涵盖了为SDK设计的docker-compose文件。有关运行这些测试的详细信息，请参阅[Node SDK](https://github.com/hyperledger/fabric-sdk-node)仓库。

第二种风格是`docker-compose-e2e.yaml`，被构造为使用Node.js SDK来运行端到端测试。除了SDK的功能之外，它主要的区别在于它有运行fabric-ca服务的容器。因此，我们能够向组织的CA节点发送REST的请求用于注册和登记。

如果你在没有运行`byfn.sh`脚本的情况下，想使用`docker-compose-e2e.yaml`，我们需要进行4个轻微的修改。我们需要指出本组织CA的私钥。你可以在`crypto-config`文件夹中找到这些值。举个例子，为了定位Org1的私钥，我们将使用`crypto-config/peerOrganizations/org1.example.com/ca/`。Org2的路径为`crypto-config/peerOrganizations/org2.example.com/ca/`。

在`docker-compose-e2e.yaml`里为ca0和ca1更新FABRIC_CA_SERVER_TLS_KEYFILE变量。你同样需要编辑command中去启动ca server的路径。你为每个CA容器提供了2次同样的私钥。

## 1.11. 使用CouchDB

状态数据库可以从默认的`goleveldb`切换到`CouchDB`。链码功能同样能使用`CouchDB`。但是，`CouchDB`提供了额外的能力来根据JSON形式的链码服务数据提供更加丰富以及复杂的查询。

使用CouchDB代替默认的数据库（goleveldb），除了在启动网络的时侯传递`docker-compose-couch.yaml`之外，请遵循前面提到的生成配置文件的过程：

```bash
CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d
```

chaincode_example02现在应该使用下面的CouchDB。

>注意
>>如果你选择将fabric-couchdb容器端口映射到主机端口，请确保你意识到了安全性的影响。在开发环境中映射端口可以使CouchDB REST API可用，并允许通过CouchDB Web界面（Fauxton）对数据库进行可视化。生产环境将避免端口映射，以限制对CouchDB容器的外部访问。

你可以使用上面列出的步骤使用CouchDB来执行chaincode_example02，然而为了执行执行CouchDB的查询能力，你将需要使用被格式化为JSON的数据（例如marbles02）。你可以在`fabric/examples/chaincode/go`目录中找到`marbles02`链码服务。

我们将按照上述[创建和加入频道](#createandjoin)部分所述的相同过程创建和加入信道。一旦你将peer节点加入到了信道，请使用以下步骤与marbles02链码交互：

* 在`peer0.org1.example.com`上安装和实例化链码：

```bash
# be sure to modify the $CHANNEL_NAME variable accordingly for the instantiate command

peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02
peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.member','Org1MSP.member')"
```

* 创建一些marbles并移动它们：
```bash
# be sure to modify the $CHANNEL_NAME variable accordingly

peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'
```

* 如果你选择在docker-compose文件中映射你的CouchDB的端口，那么你现在就可以通过CouchDB Web界面（Fauxton）通过打开浏览器导航下列URL：`http://localhost:5984/_utils`

你应该可以看到一个名为`mychannel`（或者你的唯一的信道名字）的数据库以及它的文档在里面：

>Note
>>For the below commands, be sure to update the $CHANNEL_NAME variable appropriately.

>注意
>>对于下面的命令，请确定$CHANNEL_NAME变量被更新了。

你可以CLI中运行常规的查询（例如读取`marble2`）：
```bash
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'
```

marble2的输出应该显示为如下：
```bash
Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}
```

你可以检索特定marble的历史记录-例如`marble1`:
```bash
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'
```

输出应该在`marble1`的交易：
```bash
Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}},{"TxId":"755d55c281889eaeebf405586f9e25d71d36eb3d35420af833a20a2f53a3eefd", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}},{"TxId":"819451032d813dde6247f85e56a89262555e04f14788ee33e28b232eef36d98f", "Value":}]
```

你还可以对数据内容执行丰富的查询，例如通过拥有者`jerry`查询marble：
```bash
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'
```

输出应该显示2个属于`jerry`的marble：
```bash
Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]
```

## 1.12. 关于数据持久化的提示

如果需要在peer容器或者CouchDB容器进行数据持久化，一种选择是将docker容器内相应的目录挂载到容器所在的宿主机的一个目录中。例如，你可以添加下列的两行到`docker-compose-base.yaml`文件中peer的约定中：
```bash
volumes:
 - /var/hyperledger/peer0:/var/hyperledger/production
```

对于CouchDB容器，你可以在CouchDB的约定中添加两行：
```bash
volumes:
 - /var/hyperledger/couchdb0:/opt/couchdb/data
```

## 1.13. 故障排除

* 始终保持你的网络是全新的。使用以下命令来移除之前生成的artifacts,crypto,containers以及chaincode images：
```bash
./byfn.sh -m down
```

* 你将会看到错误信息，如果你不移除容器和镜像

* 如果你看到相关的Docker错误信息，请检查你的版本（应为1.12或更高版本），然后重启你的Docker进程。Docker的问题通常不会被立即识别。例如，你可能看到由于容器内加密材料导致的错误。

* 如果她们坚持删除您的镜像，并从头开始：
```bash
docker rm -f $(docker ps -aq)
docker rmi -f $(docker images -q)
```

* 如果你发现你的创建、实例化，调用或者查询命令，请确保你已经更新了信道和链码的名字。提供的示例命令中有占位符。

* 如果你看到如下错误：
```bash
Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)
```

你可能由以前运行的链码服务（例如`dev-peer1.org2.example.com-mycc-1.0`或`dev-peer0.org1.example.com-mycc-1.0`）。删除它们，然后重试。
```bash
docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')
```

* 如果你看到类似以下内容的错误信息：

```bash
Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
```

请确保你的fabric网络运行在被标记为`latest`的`1.0.0-rc1`镜像上。

如果你看到了类似以下错误的内容：
```bash
[configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
panic: Error reading configuration: Unsupported Config Type ""
```

那么你没有正确设置`FABRIC_CFG_PATH`环境变量。configtxgen工具需要这个变量才能找到configtx.yaml。返回并执行`export FABRIC_CFG_PATH=$PWD`，然后重新创建channel配置。

* 要清理网络，请使用`down`选项：

```bash
./byfn.sh -m down
```

* 如果你看到一条指示你依然有“active endpoints”，然后清理你的Docker网络。这将会清除你之前的网络并且给你一个全新的环境：

```bash
docker network prune
```

你将看到以下消息：

```bash
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N]
```

选择`y`。

* 如果你仍旧看到了错误，请在[Hyperledger Rocket Chat](https://chat.hyperledger.org/home)的`# fabric-questions`频道上分享你的日志。
