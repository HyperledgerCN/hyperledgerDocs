| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/channel_update.html) | Weibing Wang |  |


## 重新配置第一个网络
>注意
>>本章的这些步骤在docker镜像`1.1.0-preview`版本(tag)和相关工具中已经验证过。确保你已经下载了适合的镜像版本和二进制包，或者你从比Fabric“1.1.0-preview”标签更新的分支上构建的二进制包。  

本章是[构建第一个fabric网络](https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/)的后续，会演示增加一个新组织`Org3`到自动生成的应用信道`mychannel`。它假定你已经对[BYFN](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E5%90%AF%E5%8A%A8%E9%A6%96%E4%B8%AA%E7%BD%91%E7%BB%9Cfirst-network)示范很懂了，包括会使用工具`cryptogen`和`configtxgen`。  

这篇文章仅聚焦于集成一个新组织，然而用同样的方法可以更新其他信道配置（如更新修改规则、改变批大小等）。示范的操作是组织管理员职责，而不是链码或应用开发者职责。

>注意
>>确保已经安装了必要的Fabric镜像和实用程序，并且自动化脚本`byfn.sh`在继续操作前在你的计算机上运行没有报错。即将到来的步骤依赖于生成的网络和工件。如果尚未配置机器，请参阅[前提条件](http://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html)和[Hyperledger Fabric示范](http://hyperledger-fabric.readthedocs.io/en/latest/samples.html)文档。提供的命令还假定Fabric实用程序存在于`fabric-samples`目录下的`bin`根目录中。如果已将这些二进制文件路径导出到了PATH变量中，则可以相应地修改这些命令，而不必传递绝对路径。  

### 配置环境变量
下面的操作将位于`fabric-samples`的子目录`first-network`中。更换到这个目录。你可以打开自己喜欢的终端窗口，如git-bash。  

首先，使用`byfn.sh`脚本来进行清理工作。这个命令会杀死活动的docker容器和删除之前生成的加密材料。需要说明的是，为了执行重新配置任务并不一定要停止Fabric网络，然而为了这个教程的目的，我们需要一个已知的初始状态。因此让我们执行下列命令清理之前的环境：
```bash
./byfn.sh -m down
```
现在重新生成默认BYFN工件：
```bash
./byfn.sh -m generate
```
通过执行CLI容器中脚本启动网络：
```bash
./byfn.sh -m up
```
在另一个终端窗口中，切换到`org3-artifacts`子目录。
```bash
cd org3-artifacts
```
当前目录下有兴趣的文件有两个`org3-crypto.yaml`和`configtx.yaml`。首先，为org3生成加密材料：
```bash
../../bin/cryptogen generate --config=./org3-crypto.yaml
```
上述命令会读取新的加密yaml文件`org3-crypto.yaml`，利用`cryptogen`工具为Org3中间CA生成密钥和证书，并且有两个peer绑定到这个新组织。与BYFN实现一起，这个加密材料输出到一个新生成的`crypto-config`目录中。  

现在使用`configtxgen`工具输出JSON格式的Org3相关配置材料。作为开始的命令，告诉工具从当前目录下读取`configtx.yaml`。
```bash
export FABRIC_CFG_PATH=$PWD && ../../bin/configtxgen -printOrg Org3MSP > ../channel-artifacts/org3.json
```
上面的命令会创建一个JSON文件`org3.json`，并把它输出到`first-network`目录的子目录`channel-artifacts`下面。这个文件包含了为Org3定义的修改策略，以及三个base64格式的重要证书：管理员用户证书、CA根证书和TLS根证书。在下面的步骤中，我们将附加这个JSON对象到信道配置中。  

我们最后一项准备工作是将Orderer组织MSP材料搬移到Org3的`crypto-config`目录中。特别是，我们关注Orderer的TLS根证书，这将允许Org3实体和网络的orderer节点之间的安全通信。
```bash
cd ../ && cp -r crypto-config/ordererOrganizations org3-artifacts/crypto-config/
```
现在，我们准备好重新配置了。

### 启动configtxlator服务器
更新过程使用配置转换工具`configtxlator`。这个工具提供了一个纯无状态REST API，不依赖SDK，使Hyperledger Fabric网络的配置工作简单化。这个工具可以很容易地转换不同表现/格式的等价数据。例如，在工具操作的一种模式中，该工具可以将二进制protobuf格式转换到人类可读的JSON文本格式，反之亦然。此外，该工具可以根据两组不同的配置交易之间的差异来计算配置更新。  

首先，用docker exec命令进入CLI容器。回想一下，这个容器已经安装了BYFN`crypto-config`库，使我们能够访问两个原来的peer组织和Orderer组织的MSP材料。引导身份是Org1管理员用户，这意味着我们想要代表Org2行事的任何步骤都需要导出MSP特定的环境变量。
```bash
docker exec -it cli bash
```
在默认设置下，CLI容器会在10000秒后退出。如果容器退出了，确保在继续前重新启动它。首先，检查你的容器状态：
```bash
docker ps -a
```
如果必要，重新启动CLI:
```bash
docker start cli
```
现在在容器中安装`jq`工具。这个工具允许我们与`configtxlator`工具返回的JSON对象进行脚本交互：
```bash
# Press `y` when prompted by the command
apt update && apt install jq
```
启动`configtxlator`REST服务器(最后的`&`符号使键盘输入不锁住)：
```bash
# Press enter twice
configtxlator start &
```
设置URL:
```bash
CONFIGTXLATOR_URL=http://127.0.0.1:7059
```
导出`ORDERER_CA``CHANNEL_NAME`变量：
```bash
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  && export CHANNEL_NAME=mychannel
```
检查一下确保环境变量设置正确：
```bash
echo $ORDERER_CA && echo $CHANNEL_NAME
```
>注意
>>如果你重启了CLI容器，你需要重启REST服务器和重新导出三个环境变量`CONFIGTXLATOR_URL`、`ORDERER_CA`和`CHANNEL_NAME`。jq的安装会持久化，不用重新安装它。*

### 形成更新对象和重新配置信道
现在我们在CLI容器中有了一个运行中的REST服务器，并且我们导出了两个关键环境变量`ORDERER_CA`和`CHANNEL_NAME`。让我们提取信道`mychannel`最新配置区块。
```bash
peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
```
上面的命令对生成的二进制protobuf格式信道配置块取了一个名称`config_block.pb`。请注意，你可以修改返回的protobuf和JSON对象的命名约定，但是你应该遵循一种方法，以便于进行简单直观的识别。  

当你发出`peer channel fetch`命令的时候，终端上会显示一些输出。日志中的最后一行很有趣：
```bash
2017-11-07 17:17:57.383 UTC [channelCmd] readBlock -> DEBU 011 Received block: 2
```
这是告诉我们`mychannel`的最新配置区块是区块2，**不是**创世区块。默认情况下，`peer channel fetch config`命令返回目标信道的最新配置区块，在本例中是2号区块。当BYFN场景运行时，内嵌脚本执行了两个对信道的附加配置更新。也就是，通过两个信道更新交易定义了两个组织`Org1`和`Org2`的锚peer。象这样，我们有了如下配置序列：区块0，创世区块；区块1，Org1锚peer更新；区块2，Org2锚peer更新。  

现在我们将使用`configtxlator`服务器将这个信道配置区块解码为人类可以读写的JSON格式。
```bash
curl -X POST --data-binary @config_block.pb "$CONFIGTXLATOR_URL/protolator/decode/common.Block" | jq . > config_block.json
```
我们将编码输出命名为`confg_block.json`。（再次，你可以使用自己的命名习惯来操作此步骤。）如果你在CLI容器发出`ls`命令，你可以看到两个对象：二进制protobuff格式的信道配置文件`config_block.pb`和JSON格式对象`config_block.json`。  

现在我们需要确定`config_block.json`对象的范围，并去掉所有的封装包装。我们不关心标题、元数据、创建者签名等，但关心交易中的配置定义。我们通过`jq`工具实现这一点：
```bash
jq .data.data[0].payload.data.config config_block.json > config.json
```
这给了我们一个修整过的JSON对象`config.json`，这是我们修改配置的基础。我们将再次使用`jq`工具将Org3配置定义`org3.json`附加到信道的应用组字段，并命名输出为`updated_config.json`。
```bash
jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"Org3MSP":.[1]}}}}}' config.json ./channel-artifacts/org3.json >& updated_config.json
```
现在，在CLI容器中我们有了两个JSON文件`config.json`和`updated_config.json`。初始文件仅包含Org1和Org2的材料，而“updated config”文件中包含了全部3个组织(Orgs)。此时，只需重新编码这两个JSON文件并计算增量即可。  

首先，编码`config.json`为`config.pb`：
```bash
curl -X POST --data-binary @config.json "$CONFIGTXLATOR_URL/protolator/encode/common.Config" > config.pb
```
其次，编码`updated_config.json`为`updated_config.pb`：
```bash
curl -X POST --data-binary @updated_config.json "$CONFIGTXLATOR_URL/protolator/encode/common.Config" > updated_config.pb
```
现在，使用`configtxlator`服务器来计算两个配置proto之间的增量。这个命令会输出一个新的protobuf二进制文件`Org3_update.pb`：
```bash
curl -X POST -F channel=$CHANNEL_NAME -F "original=@config.pb" -F "updated=@updated_config.pb" "${CONFIGTXLATOR_URL}/configtxlator/compute/update-from-configs" > org3_update.pb
```
这个新proto`org3_update.pb`包含了Org3定义和指向Org1和Org2材料的高级指针。我们能够放弃大量的Org1和Org2的MSP材料和修改策略信息，因为这些数据已经存在于信道的创世区块中。因此，我们只需要两个配置的增量信息。  

在递交信道更新前，我们需要执行几个最后的步骤。首先，让我们解码这个对象到可编辑的JSON格式`org3_update.json`:
```bash
curl -X POST --data-binary @org3_update.pb "$CONFIGTXLATOR_URL/protolator/decode/common.ConfigUpdate" | jq . > org3_update.json
```
现在，我们有了一个解码的更新文件`org3_update.json`，这个文件我们需要包装进一个信封消息中。这一步骤给回我们之前剥掉的标题字段。我们命名这个文件为`org3_update_in_envelope.json`：
```bash
echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat org3_update.json)'}}}' | jq . > org3_update_in_envelope.json
```
使用我们正确构建的JSON文件`org3_update_in_envelope.json`，我们将最后一次利用`configtxlator`这个工具，并将这个对象转换为Fabric需要的完全成熟的proto格式。我们将命名我们的最终更新对象为`org3_update_in_envelope.pb`：
```bash
curl -X POST --data-binary @org3_update_in_envelope.json "$CONFIGTXLATOR_URL/protolator/encode/common.Envelope" > org3_update_in_envelope.pb
```
几乎完成！我们现在的CLI容器中有了一个protobuf二进制文件`org3_update_in_envelope.pb`，然而在可以成功递交这个更新前，我们需要必要的Admin用户签名。我们信道的更新策略(mod_policy)被设置成默认的“MAJORITY”(多数)，这意味着我们需要来自两个初始组织Org1和Org2的管理员签署这个更新。如果我们没有获得这两个签名，则排序服务会因无法满足策略而拒绝这个交易。首先，我们用Org1的Admin签署这个更新proto。记住CLI容器是用Org1 MSP材料引导的，所以我们只需要简单地发送`peer channel signconfigtx`命令：
```bash
peer channel signconfigtx -f org3_update_in_envelope.pb
```
最后一步是切换CLI容器的身份为Org2的Admin用户。我们通过导出对应Org2 MSP的4个环境变量做到这一点。  

>注意
>>下面的演示不能反映真实世界的操作。单个容器永远不应该装载这个网络的加密材料。相反，更新对象需要安全地通过“带外”(out-of-band)传递给Org2管理员进行检查和批准。  

导出Org2的环境变量：
```bash
# you can issue all of these commands at once
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
```
最后我们发出`peer channel update`命令。Org2管理员签名会附加到这个呼叫，所以不需要手工再次签署这个proto：  

>注意
>>即将到来的对排序服务的呼叫会经历一系列系统签名和策略检查。因此，你会发现浏览和查看排序节点的日志很有用。从另一个终端shell，发送`docker logs -f orderer.example.com`命令来显示它们。 

发送更新呼叫：
```bash
peer channel update -f org3_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
```
如果更新成功，您应该看到类似于以下内容的消息摘要指示：
```bash
2017-11-07 21:50:17.435 UTC [msp/identity] Sign -> DEBU 00f Sign: digest: 3207B24E40DE2FAB87A2E42BC004FEAA1E6FDCA42977CB78C64F05A88E556ABA
```
成功的信道更新呼叫返回了一个新的区块，区块5，到信道中的所有peer。区块0-2是初始信道配置，区块3-4是实例化和对链码`mycc`的调用。同样的，区块5作为最新的信道配置将Org3定义到了信道上。  

查看容器`peer0.org1.example.com`的日志：
```bash
docker logs -f peer0.org1.example.com
```
你看到的详细输出反应了确认检查和对peer状态数据库的更新(关于信道的当前配置)。你还可以看到我们配置交易的提交：
```bash
2017-11-15 15:41:05.000 UTC [kvledger] CommitWithPvtData -> DEBU 774 Channel [mychannel]: Committing block [5] to storage
```
如果您想查看新配置区块的内容，请按照演示过程获取并解码新配置区块。让我们继续...  

### 将Org3加入信道
在这时，信道配置已经更新到包含了我们的新组织`Org3`，这意味着此成员的peer可以成功加入这个信道。  

首先，让我们启动包含Org3 peer和Org3特定CLI的容器。从`first-network`目录启动Org3 docker compose：
```bash
docker-compose -f docker-compose-org3.yaml up -d
```
这个新compose文件已经被配置为可以桥接我们的初始网络，因此两个peer和CLI容器可以解析已经存在的peer(指Org1和Org2的)和排序节点。三个新容器运行后，exec进入Org3特定CLI容器：
```bash
docker exec -it Org3cli bash
```
就像我们在初始CLI容器中做的那样，导出两个关键环境变量`ORDERER_CA`和`CHANNEL_NAME`：
```bash
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem && export CHANNEL_NAME=mychannel
```
检查一下确保下面的变量已经被良好地设置：
```bash
echo $ORDERER_CA && echo $CHANNEL_NAME
```
现在让我们发送呼叫到排序服务，请求`mychannel`的创世区块。由于我们成功的信道更新，订购服务可以验证附加到此呼叫的签名。如果Org3尚未成功添加到信道配置中，则排序服务会拒绝此请求。  

>注意
>>再次，你会发现这很有用：浏览排序节点的日志查看签名和验证逻辑和策略检查。  

使用`peer channel fetch`命令获取这个区块：
```bash
peer channel fetch 0 mychannel.block -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
```
注意，我们发送了一个`0`来指定我们要获取的信道账本上的第一个区块（即创世区块）。如果我们简单地发送`peer channel fetch config`命令，则我们会接收到区块5，即定义Org3的更新配置。然而，我们不能在一个下游区块开始账本，反而需要加入区块0。  

发送`peer channel join`命令，并传入创世区块`mychannel.block`：
```bash
peer channel join -b mychannel.block
```

如果你想为Org3加入第二个peer，就导出TLS和ADDRESS变量并重新发出`peer channel join`命令：
```bash
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer1.org3.example.com/tls/ca.crt && export CORE_PEER_ADDRESS=peer1.org3.example.com:7051
peer channel join -b mychannel.block
```

### 升级和调用
拼图的最后一块是增加链码版本和更新背书策略以便包含Org3。留在Org3 CLI容器中并安装链码。由于我们知道升级即将到来，因此我们可以跳过安装链码版本1的徒劳行为。我们只关心Org3将成为背书策略的一部分的新版本，因此我们将直接跳转到版本2：
```bash
peer chaincode install -n mycc -v 2.0 -p github.com/chaincode/chaincode_example02/go/
```
如果你想在Org3的第二个peer上安装链码，修改相应环境变量和重新发送命令。请注意，第二个安装不是强制的，因为您只需要在背书peer或以其他方式与账本接口的peer（即仅查询）上安装链接代码。这种peer上将仍然运行确认逻辑和作为提交者，但没有运行链码的容器。  

现在跳回到原始CLI容器，并在Org1和Org2 peer上安装新版本链码。我们是用Org2管理员身份递交的信道更新呼叫，所以容器仍以`peer0.org2`的身份运行：
```bash
peer chaincode install -n mycc -v 2.0 -p github.com/chaincode/chaincode_example02/go/
```
切换到`peer0.org1`身份：
```
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
```
并重新安装：
```bash
peer chaincode install -n mycc -v 2.0 -p github.com/chaincode/chaincode_example02/go/
```
现在，我们准备好升级链码。这里没有修改底层源码，我们只是简单地把Org3加入到信道`mychannel`的链码`mycc`的背书策略。  

>注意
>>任何满足链码实例化策略的身份都可以发出升级呼叫。在默认情况下，这些身份是信道管理员。  

发送呼叫：
```bash
peer chaincode upgrade -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 2.0 -c '{"Args":["init","a","90","b","210"]}' -P "OR ('Org1MSP.member','Org2MSP.member','Org3MSP.member')"
```
你可以看到在上面的命令中，我们通过`v`标志指定了新版本。你还可以看到背书策略被更新到`-P "OR ('Org1MSP.member','Org2MSP.member','Org3MSP.member')"`，准确地反映了策略增加了Org3。（上述背书策略的含义是三个组织的任何一个签名都可以。）令人感兴趣的最后一项是我们用`c`标志指定的构造函数请求。与实例化调用一样，链式代码升级需要使用`init`方法。如果你的链码需要传参数进`init`方法，那么你需要提供适当的键值对来重新初始化状态。这不是推荐的做法，因为升级提交者可以任意改写世界状态。相反，请考虑编辑源代码以删除参数依赖项，或者从实例化时不需要参数的链码开始。  

升级呼叫将增加一个新的区块（区块6）到信道账本，并允许Org3 peer在背书阶段执行交易。跳回到Org3 CLI容器，并发出一个对`a`值的查询。这将需要一些时间，因为链码镜像需要为目标peer构建，并且容器需要启动：
```bash
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```
我们会看到一个响应:`Query Result: 90`。  

现在发送一个调用，从`a`移动`10`个数量到`b`：
```bash
peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
```
最后查询一下`a`的值：
```bash
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```
你会看到一个响应：`Query Result: 80`，这准确反映了这个链码的世界状态的改变。  

### 总结
重新配置过程的确涉及很多，但是各个步骤都有一个合理的方法。最终目标是形成以protobuf二进制格式表示的增量交易对象，然后收集必要数量的管理员签名，使得重新配置交易处理完成信道的修改策略。`configtxlator`和`jq`工具，与日益增长`peer channel`命令一起，为我们提供了完成这项任务所需的功能。
