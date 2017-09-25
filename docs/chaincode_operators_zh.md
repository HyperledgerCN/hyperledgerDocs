
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html) | Bei Wang | Dinghao Liu |


# Chaincode 操作手册

## 什么是Chaincode？

Chaincode是一段由Go语言编写（支持其他编程语言，如Java），并能实现预定义接口的程序。Chaincode运行在一个受保护的Docker容器当中，与背书节点的运行互相隔离。Chaincode可通过应用提交的交易对账本状态初始化并进行管理。

一段chaincode通常处理由网络中的成员一致认可的业务逻辑，故我们很可能用“智能合约”来代指chaincode。一段chiancode创建的（账本）状态是与其他chaincode互相隔离的，故而不能被其他chaincode直接访问。不过，如果是在相同的网络中，一段chiancode在获取相应许可后则可以调用其他chiancode来访问它的账本。

在接下来的章节中，我们会以一个区块链网络操作者：诺亚，的视角了解学习chaincode。考虑到诺亚的关注点，我们将聚焦chaincode生命周期相关的操作。具体而言，就是在一个区块链网络中，将打包、安装、实例化和升级chaincode的过程作为一种可操作的chaincode生命周期函数进行调用。

## Chaincode生命周期

Hyperledger Fabric API让我们可以与区块链网络中的各种节点（peer节点、orderer节点、MSP节点）进行交互，同时我们还能在背书节点上安装、实例化及升级chaincode。Hyperledger Fabric特定语言的SDK工具将Hyperledger Fabric API的细节抽象了出来，便利了应用的开发过程;当然它也能用于管理chaincode生命周期。除此之外，Hyperledger Fabric API可以直接由CLI（命令行）访问，这正是本文接下来要做的。

我们提供了四个管理chaincode生命周期的命令：`package`, `install`, `instantiate`,`upgrade`。在未来的版本中，我们正考虑添加`stop`和`start`交易的指令，以便能方便地停止与重启chaincode，而不用非要真正卸载它才行。在成功安装与实例化chaincode后，chaincode就处于运行状态，接着就可以用`invoke`交易指令来处理交易了。一段chaincode可以在安装后的任何时间被更新。

## 打包（Packaging）

chaincode包具体包含以下三个部分：

* chaincode本身，其由`ChaincodeDeploymentSpec`或CDS定义。CDS根据代码及一些其他属性（名称，版本等）来定义chaincode。
* 一个可选的实例化策略，该策略可被 ***背书策略*** 描述。
* 一组表示chaincode所有权的签名。

channel上chaincode实例化交易的创建者可被chaincode的实例化策略验证。

### 创建包

打包chaincode有两种方式。第一种是当你想要让chaincode有多个所有者的时候，此时就需要让chaincode包被多个所有者签名。这种情况下需要我们创建一个被签名的chaincode包（`SignedCDS`），这个包依次被每个所有者签名。

另一种就比较简单了，这是当你要建立只有一个节点的签名的时候（该节点执行`install`交易）。

我们先来看看更复杂的情况。当然，如果您对多用户的情况不感兴趣，您可以直接跳到后面的安装chaincode部分。

要创建一个签名过的chaincode包，请用下面的指令：

```
peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out
```

`-s`选项创建了一个可被多个所有者签名的包，而非简单地创建一个CDS。如果使用`-s`，那么当其他所有者要签名的时候，`-S`也必须同时使用。否则，该过程将创建一个仅包含实例化策略的签名chaincode包（SignedCDS）。

`-S`选项可以使在`core.yaml`文件中被`localMspid`相关属性值定义好的MSP对包进行签名。

`-S`选项是可选的。不过，如果我们创建了一个没有签名的包，那么它就不能被任何其他所有者用`signpackage`指令进行签名。

`-i`选项也是可选的，它允许我们为chaincode指定实例化策略。实例化策略与背书策略格式相同，它指明谁可以实例化chaincode。在上面的例子中，只有OrgA的管理员才有资格实例化chaincode。如果没有提供任何策略，那么系统会采用默认策略，该策略只允许peer节点MSP的管理员去实例化chaincode。

### 包的签名

一个在创建时就被签名的chaincode包可以交给其他所有者进行检查与签名。具体的工作流程支持带外对chaincode包签名。

***ChaincodeDeploymentSpec*** 可以选择被全部所有者签名并创建一个 ***SignedChaincodeDeploymentSpec***（SignedCDS），SignedCDS包含三个部分：

* CDS包含chaincode的源码、名称与版本
* 一个chaincode实例化策略，其表示为背书策略
* chaincode所有者的列表，由Endorsement定义

***注意:***

*注意，当chaincode在某些channel上实例化时，背书策略在带外定义，并提供合适的MSP。如果没有明确实例化策略，那么默认的策略是channel的任意管理员（执行实例化）*

每个（chaincode的）所有者通过将ChaincodeDeploymentSpec与其本人的身份信息（证书）结合并对组合结果签名来认证ChaincodeDeploymentSpec。

一个chaincode所有者可以对一个之前创建好的带签名的包进行签名，具体使用如下指令：

```bash
peer chaincode signpackage ccpack.out signedccpack.out
```

指令中的`ccpack.out`和`signedccpack.out`分别是输入与输出包。`signedccpack.out`则包含一个用本地MSP对包进行的附加签名。

### 安装chaincode

`install`交易的过程会将chaincode的源码以一种被称为`ChaincodeDeploymentSpec`（CDS）的规定格式打包，并把它安装在一个将要运行该chaincode的peer节点上。

***注意:***

*请务必在一条channel上每一个要运行你chaincode的背书节点上安装你的chaincode*

如果只是简单地给`install` API一个`ChaincodeDeploymentSpec`，它将使用默认实例化策略并添加一个空的所有者列表。

***注意:***

*Chaincode应该仅仅被安装于chaincode所有者的背书节点上，以使该chaincode逻辑对整个网络的其他成员保密。其他没有chaincode的成员将无权成为chaincode影响下的交易的认证节点（endorser）。也就是说，他们不能执行chaincode。不过，他们仍可以验证交易并提交到账本上。*

下面安装chaincode。此时会发送一条 ***SignedProposal*** 到`生命周期系统chaincode` (LSCC)，该chaincode在***系统chaincode***部分会仔细描述。举个例子，使用CLI安装***简单的账本管理chaincode***章节的sacc chaincode样例时，命令如下：

```bash
peer chaincode install -n asset_mgmt -v 1.0 -p sacc
```

在CLI内部会为sacc创建SignedChaincodeDeploymentSpec，并将其发送到本地peer节点。这些节点会调用LSCC上的`Install`方法。上述的`-p`选项指明chaincode的路径，其必须在用户的`GOPATH`目录下（比如`$GOPATH/src/sacc`）。完整的命令选项详见***CLI***部分。

注意：为了在peer节点上安装（chaincode），SignedProposal的签名必须来自peer节点本地MSP的管理员中的一位。

### 实例化chaincode

实例化交易会调用`生命周期系统chaincode` (LSCC)来在一个channel上创建并初始化一段chaincode。下面是一个chaincode-channel绑定的具体过程：一段chaincode可能会与任意数量的channel绑定并在每个channel上独立运行。换句话说，chaincode在多少个channel上安装并实例化并没有什么影响，对于每个提交交易的channel，其状态都是独立而互不影响的。

一个`实例化`交易的创建者必须符合在SignedCDS中chaincode的实例化策略，且必须充当channel的写入器（这会成为channel创建配置的一部分）。这对于channel的安全至关重要，因为这样可以防止恶意实体在未绑定的channel上部署chaincode，也能防止间谍成员在未绑定的channel上执行chaincode。

举个例子，我们提到过默认的实例化策略是任何channel MSP的管理员（可以执行），所以chaincode创建者要实例化交易，其本人必须是channel管理员的一员。当交易提议到达背书成员时，它会验证创建者的签名是否符合实例化策略。在交易被提交到账本之前的交易验证阶段，以上操作还会再来一遍。

实例化交易的过程还会为channel上的chaincode建立背书策略。背书策略描述了交易的相关认证要求，以使得交易能被channel中的成员认可。

例如，使用CLI去实例化上一章的sacc chaincode并初始化`john`的状态为`0`，指令具体如下：

```bash
peer chaincode instantiate -n sacc -v 1.0 -c '{"Args":["john","0"]}' -P "OR ('Org1.member','Org2.member')"
```

***注意:***

*注意，上述背书策略（CLI使用波兰表示法）向Org1或Org2的成员询问所有sacc处理的交易。也就是说，为确保交易有效，Org1或Org2必须为调用**sacc**的结果签名。*

在成功实例化后，channel上的chaincode就进入激活状态，并时刻准备执行任何***ENDORSER_TRANSACTION***类型的交易提议。交易会在到达背书节点的同时被处理。

### 升级chaincode

一段chaincode可以通过更改它的版本（SignedCDS的一部分）来随时进行更新。至于SignedCDS的其他部分，比如所有者及实例化策略，都是可选的。不过，chaincode的名称必须一致，否则它会被当做完全不同的另一段chaincode。

在升级之前，chaincode的新版本必须安装在需要它的背书节点上。升级是一个类似于实例化交易的交易，它会将新版本的chaincode与channel绑定。其他与旧版本绑定的channel则仍旧运行旧版本的chaincode。换句话说，`升级`交易只会一次影响一个提交它的channel。

***注意:***

*注意：由于多个版本的chaincode可能同时运行，所以升级过程不会自动移除旧版本，用户必须亲自处理。*

`升级`交易与`实例化`交易有一处微妙的区别：`升级`交易采用当前的chaincode实例化策略进行检查，而非比对新的策略（如果指定了的话）。这是为了确保只有当前实例化策略指定的已有成员才能升级chaincode。

***注意:***

*注意：在升级过程中，chaincode的`Init`函数会被调用以执行数据相关的操作，或者重新初始化数据；所以要多加小心避免在升级chaincode时重设状态信息。*

### 停止与启动

注意，`停止`与`启动`生命周期交易的功能还没实现。不过，你可以通过移除chaincode容器以及从每个背书节点删除SignedCDS包来停止chaincode。具体而言，就是删除所有主机或虚拟机上peer节点运行于其中的chaincode的容器，随后从每个背书节点删除SignedCDS。

***注意:***

*TODO -为了从peer节点删除CDS，你应该需要先进入peer节点的容器内。我们的确需要提供一个可以执行此功能的脚本*

```bash
docker rm -f <container id>
rm /var/hyperledger/production/chaincodes/<ccname>:<ccversion>
```

停止功能在以受控的方式进行升级的流程中将非常有用，特别是在进行升级前，一段channel上所有节点的chaincode都可被停止。

### CLI

***注意:***

*我们正在评估为 Hyperledger Fabric `peer`的二进制文件拆分特定平台二进制文件的需求。不过目前，您可以在一个正在运行的docker容器中方便地调用指令。*

下面我们将一览现在可用的CLI指令，请在一个运行`fabric-peer`的Docker容器中执行以下指令：

```bash
docker run -it hyperledger/fabric-peer bash
# peer chaincode --help
```

我们将看到如下输出：

```bash
Usage:
  peer chaincode [command]

Available Commands:
  install     Package the specified chaincode into a deployment spec and save it on the peer’s path.
  instantiate Deploy the specified chaincode to the network.
  invoke      Invoke the specified chaincode.
  package     Package the specified chaincode into a deployment spec.
  query       Query using the specified chaincode.
  signpackage Sign the specified chaincode package
  upgrade     Upgrade chaincode.

Flags:
      --cafile string     Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
  -C, --chainID string    The chain on which this command should be executed (default "testchainid")
  -c, --ctor string       Constructor message for the chaincode in JSON format (default "{}")
  -E, --escc string       The name of the endorsement system chaincode to be used for this chaincode
  -l, --lang string       Language the chaincode is written in (default "golang")
  -n, --name string       Name of the chaincode
  -o, --orderer string    Ordering service endpoint
  -p, --path string       Path to chaincode
  -P, --policy string     The endorsement policy associated to this chaincode
  -t, --tid string        Name of a custom ID generation algorithm (hashing and decoding) e.g. sha256base64
      --tls               Use TLS when communicating with the orderer endpoint
  -u, --username string   Username for chaincode operations when security is enabled
  -v, --version string    Version of the chaincode specified in install/instantiate/upgrade commands
  -V, --vscc string       The name of the verification system chaincode to be used for this chaincode

Global Flags:
      --logging-level string       Default logging level and overrides, see core.yaml for full syntax
      --test.coverprofile string   Done (default "coverage.cov")

Use "peer chaincode [command] --help" for more information about a command.
```

为方便在脚本应用程序里使用，`peer`指令失败时总会返回一个非0值。

chaincode的指令示例如下：

```bash
peer chaincode install -n mycc -v 0 -p path/to/my/chaincode/v0
peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a", "b", "c"]}' -C mychannel
peer chaincode install -n mycc -v 1 -p path/to/my/chaincode/v1
peer chaincode upgrade -n mycc -v 1 -c '{"Args":["d", "e", "f"]}' -C mychannel
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","e"]}'
peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

### 系统chaincode

系统chaincode与普通chaincode的编程模型相同，只不过它运行于peer节点内而非一个隔离的容器中。因此，系统chaincode在节点内构建且不遵循上文描述的chaincode生命周期。特别地，***安装***，***实例化***，***升级***这三项操作不适用于系统chaincode。

系统chaincode的目的是削减peer节点和chaincode之间的gRPC通讯成本，并兼顾管理的灵活性。例如：一个系统chaincode只能通过peer节点的二进制文件升级。同时，系统chaincode只能以一组编译好的特定的参数进行注册，且不具有背书策略相关功能。

系统chaincode在Hyperledger Fabric中用于实现一些系统行为，故它们可以被系统开发者适当替换或更改。

以下是系统chaincode的列表：

* LSCC：生命周期系统chaincode处理上述生命周期相关的功能
* CSCC：配置系统chaincode处理peer侧channel的配置
* QSCC：查询系统chaincode提供账本查询API，比如获取区块及交易等
* ESCC：背书系统chaincode通过对交易响应进行签名来处理背书过程
* VSCC：验证系统chaincode处理交易的验证，包括检查背书策略以及多版本并发控制

替换或更改这些系统chaincode一定要万分小心，尤其是LSCC, ESCC 和 VSCC，因为它们处于主交易执行路径中。值得注意的是，VSCC在一个区块被提交到账本之前进行验证，故所有channel中的peer节点得出相同的验证结果以避免账本分叉（不确定因素）就很重要了。所以当VSCC被更改或替换时就要特别小心了。











