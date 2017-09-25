
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html) | Dinghao Liu | Bei Wang |


# Chaincode 开发手册

##什么是Chaincode？

Chaincode is a program, written in Go that implements a prescribed interface. Eventually, other programming languages such as Java, will be supported. Chaincode runs in a secured Docker container isolated from the endorsing peer process. Chaincode initializes and manages the ledger state through transactions submitted by applications.

Chaincode是一段由Go语言编写（支持其他编程语言，如Java），并能实现预定义接口的程序。Chaincode运行在一个受保护的Docker容器当中，与背书节点的运行互相隔离。Chaincode可通过应用提交的交易对账本状态初始化并进行管理。

A chaincode typically handles business logic agreed to by members of the network, so it similar to a “smart contract”. Ledger state created by a chaincode is scoped exclusively to that chaincode and can’t be accessed directly by another chaincode. Given the appropriate permission, a chaincode may invoke another chaincode to access its state within the same network.

一段chaincode通常处理由网络中的成员一致认可的业务逻辑，故我们很可能用“智能合约”来代指chaincode。一段chiancode创建的（账本）状态是与其他chaincode互相隔离的，故而不能被其他chaincode直接访问。不过，如果是在相同的网络中，一段chiancode在获取相应许可后则可以调用其他chiancode来访问它的账本。

In the following sections, we will explore chaincode through the eyes of an application developer. We’ll present a simple chaincode sample application and walk through the purpose of each method in the Chaincode Shim API.

在接下来的章节中，我们会以一个应用开发者的视角了解学习chaincode。我们将展示一个chaincode的应用范例，并步步为营搞清每一个Chaincode Shim API的来龙去脉。

##Chaincode API

Every chaincode program must implement the ***Chaincode interface*** whose methods are called in response to received transactions. In particular the `Init` method is called when a chaincode receives an `instantiate` or `upgrade` transaction so that the chaincode may perform any necessary initialization, including initialization of application state. The `Invoke` method is called in response to receiving an `invoke` transaction to process transaction proposals.

每个chaincode程序都必须实现 ***chiancode接口*** ，接口中的方法会在响应传来的交易时被调用。特别地，`Init`（初始化）方法会在chaincode接收到`instantiate`（实例化）或者`upgrade`(升级)交易时被调用，进而使得chaincode顺利执行必要的初始化操作，包括初始化应用的状态；`Invoke`（调用）方法会在响应`invoke`（调用）交易时被调用以执行交易。

The other interface in the chaincode “shim” APIs is the ***ChaincodeStubInterface*** which is used to access and modify the ledger, and to make invocations between chaincodes.

其他chaincode shim APIs中的接口被称为***chaincode存根接口***，用于访问及修改账本，并实现chaincode之间的互相调用。

In this tutorial, we will demonstrate the use of these APIs by implementing a simple chaincode application that manages simple “assets”.

在本篇指南中，我们会通过实现一个能管理简单“资产”的chaincode应用范例来演示这些接口的使用方法。

##简单的资产管理Chaincode

Our application is a basic sample chaincode to create assets (key-value pairs) on the ledger.

我们的应用是一个在账本上创建资产（键值对）的基本范例。

###选择一个代码存放位置

If you haven’t been doing programming in Go, you may want to make sure that you have ***Go Programming Language*** installed and your system properly configured.

如果您之前还未编写过Go程序，那么您也许需要先确保 ***Go程序设计语言*** 在您的系统中已经被正确配置。

Now, you will want to create a directory for your chaincode application as a child directory of `$GOPATH/src/`.

现在，您需要为chaincode应用创建一个位于`$GOPATH/src/`目录下的子目录。

To keep things simple, let’s use the following command:

简单起见，我们使用如下指令：

```bash
mkdir -p $GOPATH/src/sacc && cd $GOPATH/src/sacc
```

Now, let’s create the source file that we’ll fill in with code:

现在，输入以下指令来创建程序的源文件吧！

```bash
touch sacc.go
```

###内务处理

First, let’s start with some housekeeping. As with every chaincode, it implements the ***Chaincode interface*** in particular, `Init` and `Invoke` functions. So, let’s add the go import statements for the necessary dependencies for our chaincode. We’ll import the chaincode shim package and the ***peer protobuf package***.

首先， 我们先进行内务处理。对于每一个chaincode，它都会实现预定义的***chaincode接口***，特别是`Init`和`Invoke`函数接口。所以我们首先为我们的chaincode引入必要的依赖。我们将在此引入chaincode shim package和***peer protobuf package***。

```bash
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)
```

###初始化Chaincode

接下来，我们将实现`Init`函数。

```bash
// Init is called during chaincode instantiation to initialize any data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {

}
)
```

  ****注意:***

*Note that chaincode upgrade also calls this function. When 	    writing a chaincode that will upgrade an existing one, make 	sure to modify the Init function appropriately. In particular, provide an empty “Init” method if there’s no “migration” or nothing to be initialized as part of the upgrade.*
  
*值得留意的是chaincode升级同样会调用该函数。当我们编写的chaincode会升级现有chaincode时，需要确保适当修正Init函数。特别地，如果没有“迁移”操作或其他需要在升级中初始化的东西，那么就提供一个空的“Init”方法。*


Next, we’ll retrieve the arguments to the `Init` call using the *ChaincodeStubInterface.GetStringArgs* function and check for validity. In our case, we are expecting a key-value pair.

下面，我们将调用*ChaincodeStubInterface.GetStringArgs*函数来获取`Init`所需的参数，并进行有效性检查。在我们的例子中，我们希望传入参数是一组键值对。

```bash
// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data, so be careful to avoid a scenario where you
// inadvertently clobber your ledger’s data!
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
  // Get the args from the transaction proposal
  args := stub.GetStringArgs()
  if len(args) != 2 {
    return shim.Error("Incorrect arguments. Expecting a key and a value")
  }
}
```

Next, now that we have established that the call is valid, we’ll store the initial state in the ledger. To do this, we will call *ChaincodeStubInterface*.PutState with the key and value passed in as the arguments. Assuming all went well, return a peer.Response object that indicates the initialization was a success.

接下来，既然我们已经确定调用有效，那么我们就将初始状态放心地存入账本。为了实现该目标，我们将调用*ChaincodeStubInterface*并以键值为参数传入。如果一切正常，那么我们会收到表明初始化成功的peer.Response返回对象。

```bash
// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data, so be careful to avoid a scenario where you
// inadvertently clobber your ledger’s data!
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
  // Get the args from the transaction proposal
  args := stub.GetStringArgs()
  if len(args) != 2 {
    return shim.Error("Incorrect arguments. Expecting a key and a value")
  }

  // Set up any variables or assets here by calling stub.PutState()

  // We store the key and the value on the ledger
  err := stub.PutState(args[0], []byte(args[1]))
  if err != nil {
    return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
  }
  return shim.Success(nil)
}
```

###调用Chaincode

First, let’s add the `Invoke` function’s signature.

首先，添加`Invoke`函数签名。

```bash
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The 'set'
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {

}
```

As with the `Init` function above, we need to extract the arguments from the `ChaincodeStubInterface`. The `Invoke` function’s arguments will be the name of the chaincode application function to invoke. In our case, our application will simply have two functions: `set` and `get`, that allow the value of an asset to be set or its current state to be retrieved. We first call *ChaincodeStubInterface.GetFunctionAndParameters* to extract the function name and the parameters to that chaincode application function.

就如上述`Init`函数一样，我们需要调用`ChaincodeStubInterface`来获取参数。`Invoke`函数所需的传入参数正是应用想要调用的chaincode的名称。在我们的例子中，我们的应用只有两个简单的功能函数：`set`和`get`;前者允许对资产的数值进行设定;后者允许获取当前资产的状态。我们先调用*ChaincodeStubInterface.GetFunctionAndParameters*来获取chaincode应用所需的函数名与参数。

```bash
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

}
```

Next, we’ll validate the function name as being either `set` or `get`, and invoke those chaincode application functions, returning an appropriate response via the `shim.Success` or `shim.Error` functions that will serialize the response into a gRPC protobuf message.

下面，我们将使`set`与`get`这两个函数名正式生效，并调用这些chaincode应用函数，经由`shim.Success`或`shim.Error`函数返回一个合理的响应。这两个`shim`成员函数可以将响应序列化为gRPC protobuf消息。

```bash
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

    var result string
    var err error
    if fn == "set" {
            result, err = set(stub, args)
    } else {
            result, err = get(stub, args)
    }
    if err != nil {
            return shim.Error(err.Error())
    }

    // Return the result as success payload
    return shim.Success([]byte(result))
}
```

###实现Chaincode应用

As noted, our chaincode application implements two functions that can be invoked via the `Invoke` function. Let’s implement those functions now. Note that as we mentioned above, to access the ledger’s state, we will leverage the *ChaincodeStubInterface.PutState* and *ChaincodeStubInterface.GetState* functions of the chaincode shim API.

如上文所述，我们的chaincode应用实现了两个函数，并可以被`Invoke`函数调用。下面我们就来真正实现这些函数。注意，就像上文一样，我们调用chaincode shim API中的*ChaincodeStubInterface.PutState*和*ChaincodeStubInterface.GetState*函数来访问账本。

```bash
// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 2 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    }

    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return "", fmt.Errorf("Failed to set asset: %s", args[0])
    }
    return args[1], nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 1 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    }

    value, err := stub.GetState(args[0])
    if err != nil {
            return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    }
    if value == nil {
            return "", fmt.Errorf("Asset not found: %s", args[0])
    }
    return string(value), nil
}
```

###整合全部代码

Finally, we need to add the `main` function, which will call the *shim.Start* function. Here’s the whole chaincode program source.

终于到了写`main`函数的最后关头，它将调用*shim.Start*函数。下面是包含整个chaincode程序的代码：

```bash
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)

// SimpleAsset implements a simple chaincode to manage an asset
type SimpleAsset struct {
}

// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
    // Get the args from the transaction proposal
    args := stub.GetStringArgs()
    if len(args) != 2 {
            return shim.Error("Incorrect arguments. Expecting a key and a value")
    }

    // Set up any variables or assets here by calling stub.PutState()

    // We store the key and the value on the ledger
    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
    }
    return shim.Success(nil)
}

// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

    var result string
    var err error
    if fn == "set" {
            result, err = set(stub, args)
    } else { // assume 'get' even if fn is nil
            result, err = get(stub, args)
    }
    if err != nil {
            return shim.Error(err.Error())
    }

    // Return the result as success payload
    return shim.Success([]byte(result))
}

// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 2 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    }

    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return "", fmt.Errorf("Failed to set asset: %s", args[0])
    }
    return args[1], nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 1 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    }

    value, err := stub.GetState(args[0])
    if err != nil {
            return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    }
    if value == nil {
            return "", fmt.Errorf("Asset not found: %s", args[0])
    }
    return string(value), nil
}

// main function starts up the chaincode in the container during instantiate
func main() {
    if err := shim.Start(new(SimpleAsset)); err != nil {
            fmt.Printf("Error starting SimpleAsset chaincode: %s", err)
    }
}
```

###编译Chaincode

现在来编译你的chaincode吧！

```bash
go get -u --tags nopkcs11 github.com/hyperledger/fabric/core/chaincode/shim
go build --tags nopkcs11
```

Assuming there are no errors, now we can proceed to the next step, testing your chaincode.

如果没有报错，那么我们就可以进行下一步：测试chaincode。

###在开发模者式下测试

Normally chaincodes are started and maintained by peer. However in “dev mode”, chaincode is built and started by the user. This mode is useful during chaincode development phase for rapid code/build/run/debug cycle turnaround.

通常，chaincode由peer节点启动并维护。不过，在“开发者模式”下，chaincode可以由用户创建并启动。当用户处于以快速编码/构建/运行/调试的循环周期为主的chaincode开发阶段时，该模式十分有用。

We start “dev mode” by leveraging pre-generated orderer and channel artifacts for a sample dev network. As such, the user can immediately jump into the process of compiling chaincode and driving calls.

我们借助构建自带区块链样例网络时已经预先生成好的order和channel来启动“开发者模式”。这样，用户可以立即编译chaincode并调用函数。

##安装Hyperledger Fabric样例

If you haven’t already done so, please install the **Hyperledger Fabric Samples**.

如果您之前还没有进行过这一步，请先安装 **Hyperledger Fabric Sample**

Navigate to the `chaincode-docker-devmode` directory of the `fabric-samples` clone:

下面进入到安装好的`fabric-samples`下的`chaincode-docker-devmode`目录。

```bash
cd chaincode-docker-devmode
```

##下载Docker镜像

We need four Docker images in order for “dev mode” to run against the supplied docker compose script. If you installed the `fabric-samples` repo clone and followed the instructions to download-platform-specific-binaries, then you should have the necessary Docker images installed locally.

跟据下载样例中自带的docker构建脚本，我们需要四个Docker镜像来确保“开发者模式”成功运行。如果您已经安装了`fabric-samples`克隆仓库，并按照指示下载了platform-specific-binaries，那么你的本地理应早已安装好了所需的Docker镜像。

****注意:***

*If you choose to manually pull the images then you must retag them as `latest`.*

*如果您选择手动拉取镜像，那么您务必将其重新标记为`latest`。*

Issue a `docker images` command to reveal your local Docker Registry. You should see something similar to following:

输入`docker images`指令可以方便地查询本地的Docker镜像列表。您应该会看到类似下面的内容：

```bash
docker images
REPOSITORY                     TAG                                  IMAGE ID            CREATED             SIZE
hyperledger/fabric-tools       latest                               e09f38f8928d        4 hours ago         1.32 GB
hyperledger/fabric-tools       x86_64-1.0.0                         e09f38f8928d        4 hours ago         1.32 GB
hyperledger/fabric-orderer     latest                               0df93ba35a25        4 hours ago         179 MB
hyperledger/fabric-orderer     x86_64-1.0.0                         0df93ba35a25        4 hours ago         179 MB
hyperledger/fabric-peer        latest                               533aec3f5a01        4 hours ago         182 MB
hyperledger/fabric-peer        x86_64-1.0.0                         533aec3f5a01        4 hours ago         182 MB
hyperledger/fabric-ccenv       latest                               4b70698a71d3        4 hours ago         1.29 GB
hyperledger/fabric-ccenv       x86_64-1.0.0                         4b70698a71d3        4 hours ago         1.29 GB
```

****注意:***

*If you retrieved the images through the download-platform-specific-binaries, then you will see additional images listed. However, we are only concerned with these four.*

*如果您通过download-platform-specific-binaries来获取镜像，那么您将会看到更多的镜像资源列表。不过这里我们只关心以上四个。*

Now open three terminals and navigate to your `chaincode-docker-devmode` directory in each.

现在请在`chaincode-docker-devmode`目录下面打开三个独立的终端。

##1号终端

```bash
docker-compose -f docker-compose-simple.yaml up
```

The above starts the network with the `SingleSampleMSPSolo` orderer profile and launches the peer in “dev mode”. It also launches two additional containers - one for the chaincode environment and a CLI to interact with the chaincode. The commands for create and join channel are embedded in the CLI container, so we can jump immediately to the chaincode calls.

上述指令启动了一个带有`SingleSampleMSPSolo`orderer profile的网络，并将节点在“开发者模式”下启动。它还启动了另外两个容器：一个包含chaincode运行环境;另一个是CLI命令行，可与chaincode进行交互。创建并加入channel（管道）的指令内嵌于CLI容器中，所以我们下面马上跳转到chaincode调用部分。

##2号终端

```bash
docker exec -it chaincode bash
```

执行完上述指令后，您应该会看到如下内容：

```bash
root@d2629980e76b:/opt/gopath/src/chaincode#
```

（此时您已经进入chaincode容器）下面编译您的chaincode:

```bash
cd sacc
go build
```

现在运行chaincode：

```bash
CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc
```

The chaincode is started with peer and chaincode logs indicating successful registration with the peer. Note that at this stage the chaincode is not associated with any channel. This is done in subsequent steps using the `instantiate` command.

chaincode被peer节点启动，chaincode日志表明peer节点成功注册。注意：现阶段chaincode还没有与任何channel关联。这会在接下来使用`instantiate`指令后实现。

##3号终端

Even though you are in `--peer-chaincodedev` mode, you still have to install the chaincode so the life-cycle system chaincode can go through its checks normally. This requirement may be removed in future when in `--peer-chaincodedev` mode.

即便您处于`--peer-chaincodedev`模式，安装chaincode这一步仍必不可少，这样生命周期系统chaincode才能正常进行检查。也许这一步会在日后的`--peer-chaincodedev`模式中省去。

We’ll leverage the CLI container to drive these calls.

下面我们将进入CLI容器进行chaincode调用。

```bash
docker exec -it cli bash
```

```bash
peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a","10"]}' -C myc
```

现在我们执行一次将a的值设为20的调用：

```bash
peer chaincode invoke -n mycc -c '{"Args":["set", "a", "20"]}' -C myc
```

最后查询a的值，我们会看到20。

```bash
peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
```

##测试新的chaincode

By default, we mount only `sacc`. However, you can easily test different chaincodes by adding them to the `chaincode` subdirectory and relaunching your network. At this point they will be accessible in your `chaincode` container.

虽然我们只实现了`sacc`，不过您可以通过将不同的chaincode添加到`chaincode`子目录下重启网络来轻松地测试它们。重启它们将可在`chaincode`容器中被访问。



