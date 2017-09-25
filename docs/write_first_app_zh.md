
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/write_first_app.html) | Qiang Sheng |  |

#编写第一个应用(Writing Your First Application)
The goal of this document is to show the tasks and provide a baseline for writing your first application against a Hyperledger Fabric network.

本文档将会指引您基于Hyperledger Fabric网络编写第一个应用程序。

At the most basic level, applications on a blockchain network are what enable users to **query** a ledger (asking for specific records it contains), or to **update** it (adding records to it).

最基本的，区块链网络应用程序需要提供给用户**查询**账本（包含特定记录）以及**更新**账本（添加记录）的功能。

Our application, composed in Javascript, leverages the Node.js SDK to interact with the network (where our ledger exists). This tutorial will guide you through the three steps involved in writing your first application.

我们的应用程序基于Javascript，通过Node.js SDK与（账本所在的）网络进行交互。本教程将通过三步来指导您编写您的第一个应用程序。

>**1. Starting a test Hyperledger Fabric blockchain network.** We need some basic components in our network in order to query and update the ledger. These components – a peer node, ordering node and Certificate Authority – serve as the backbone of our network; we’ll also have a CLI container used for a few administrative commands. A single script will download and launch this test network.

>**1. 启动一个Hyperledger Fabric区块链测试网络。** 在我们的网络中，我们需要一些最基本的组件来查询和更新账本。这些组件 —— peer节点、ordering节点以及证书管理 —— 是我们网络的基础。而CLI容器则用来发送一些管理命令。有个简单的脚本将下载并启动这个测试网络。

>**2. Learning the parameters of the sample smart contract our app will use.** Our smart contracts contain various functions that allow us to interact with the ledger in different ways. For example, we can read data holistically or on a more granular level.

>**2. 学习应用程序中所用到的智能合约例子的参数。** 智能合约包含的各种功能让我们可以用多种方式和账本进行交互。如，我们可以读取整体的数据或者某一部分详尽的数据。

>**3. Developing the application to be able to query and update records.** We provide two sample applications – one for querying the ledger and another for updating it. Our apps will use the SDK APIs to interact with the network and ultimately call these functions.

>**3. 开发能够查询以及更新记录的应用程序。** 我们提供两个程序例子 —— 一个用于查询账本，另一个用户更新账本。我们的程序将使用SDK APIs来和网络进行交互，并最终调用这些功能。

After completing this tutorial, you should have a basic understanding of how an application, using the Hyperledger Fabric SDK for Node.js, is programmed in conjunction with a smart contract to interact with the ledger on a Hyperledger Fabric network.

完成本教程之后，您应该会基本了解一个使用Hyperledger Fabric Node.js SDK并带有智能合约的应用程序，是如何与Hyperledger Fabric网络中的账本进行交互的。

First, let’s launch our test network...

首先，让我们启动测试网络...

###下载测试网络（Getting a Test Network）
Visit the [Prerequisites](http://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html) page and ensure you have the necessary dependencies installed on your machine.

请先访问[Prerequisites](http://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html)网页并确保您的计算机上已经安装了必需的依赖项。

Now determine a working directory where you want to clone the fabric-samples repo. Issue the clone command and change into the `fabcar` subdirectory

选择clone fabric-samples的工作目录，运行clone命令，并进入`fabcar`子目录

	git clone https://github.com/hyperledger/fabric-samples.git
	cd fabric-samples/fabcar
This subdirectory – `fabcar`– contains the scripts and application code to run the sample app. Issue an `ls` from this directory. You should see the following:

这个子目录 - `fabcar` - 包含运行示例程序的脚本以及程序代码。在该目录运行`ls`命令，您应该会看到以下内容：

	chaincode    invoke.js       network         package.json    query.js        startFabric.sh
Now use the `startFabric.sh` script to launch the network.

现在调用`startFabric.sh`来启动网络。

Note

The following command downloads and extracts the Hyperledger Fabric Docker images, so it will take a few minutes to complete.

请注意：下面命令将会下载并解压Hyperledger Fabric Docker images，因此需要几分钟时间来完成。
```bash
./startFabric.sh
```
For the sake of brevity, we won’t delve into the details of what’s happening with this command. Here’s a quick synopsis:

为了简洁起见，我们不会深入了解这个命令的具体细节。下面是一个关于这个命令的快速简要说明：

* launches a peer node, ordering node, Certificate Authority and CLI container
* creates a channel and joins the peer to the channel
* installs smart contract (i.e. chaincode) onto the peer’s file system and instantiates said chaincode on the channel; instantiate starts a chaincode container
* calls the `initLedger` function to populate the channel ledger with 10 unique cars

* 启动peer节点、Ordering节点、证书颁发机构以及CLI容器
* 创建一个通道，并将peer加入该通道
* 将智能合约（即链码）安装到peer节点的文件系统上，并在通道上实例化该链码；实例化会启动链码容器
* 调用`initLedger`功能来向通道账本写入10个不同的汽车

Note

These operations will typically be done by an organizational or peer admin. The script uses the CLI to execute these commands, however there is support in the SDK as well. Refer to the [Hyperledger Fabric Node SDK repo](https://github.com/hyperledger/fabric-sdk-node) for example scripts.

请注意：这些操作通常由组织或者peer的管理员来完成。这个脚本使用CLI容器来执行这些命令，但SDK中也有支持。具体请参阅[Hyperledger Fabric Node SDK repo](https://github.com/hyperledger/fabric-sdk-node)中的示例脚本。

Issue a `docker ps` command to reveal the processes started by the `startFabric.sh` script. You can learn more about the details and mechanics of these operations in the [Building Your First Network](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html) section. Here we’ll just focus on the application. The following picture provides a simplistic representation of how the application interacts with the Hyperledger Fabric network.

发送`docker ps`命令可以显示`startFabric.sh`脚本启动的进程。您可以在[Building Your First Network](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)部分了解有关这些操作的详细信息和机制。在这里，我们专注于应用程序。下图简单描述了一个应用程序与Hyperledger Fabric网络交互的过程。

![](img/firstapp0.png)

Alright, now that you’ve got a sample network and some code, let’s take a look at how the different pieces fit together.

好了，现在我们有了简单的网络以及一些代码，现在看看他们是怎么一起工作的。

###应用程序如何与网络进行交互(How Applications Interact with the Network)
Applications use **APIs** to invoke smart contracts (referred to as “chaincode”). These smart contracts are hosted in the network and identified by name and version. For example, our chaincode container is titled - `dev-peer0.org1.example.com-fabcar-1.0` - where the name is `fabcar`, the version is `1.0` and the peer it is running against is `dev-peer0.org1.example.com`.

应用程序使用**APIs**来调用智能合约(即“链码”)。这些智能合同托管在网络中，并通过名称和版本进行标识。例如，标示为`dev-peer0.org1.example.com-fabcar-1.0`的容器，其名称是`fabcar`，版本号是`1.0`，运行peer是`dev-peer0.org1.example.com`。

APIs are accessible with a software development kit (SDK). For purposes of this exercise, we’ll be using the [Hyperledger Fabric Node SDK](https://fabric-sdk-node.github.io/) though there is also a Java SDK and CLI that can be used to develop applications.

API可通过软件开发工具包（SDK）访问。在本练习中，我们将使用[Hyperledger Fabric Node SDK](https://fabric-sdk-node.github.io/)，除此以外，Fabric还提供了Java SDK和CLI用于开发应用程序。

###查询账本(Querying the Ledger)
Queries are how you read data from the ledger. You can query for the value of a single key, multiple keys, or – if the ledger is written in a rich data storage format like JSON – perform complex searches against it (looking for all assets that contain certain keywords, for example).

查询是指如何从账本中读取数据。您可以查询单个或者多个键的值，如果账本是以类似于JSON这样的数据存储格式写入的，则可以执行更复杂的搜索（如查找包含某些关键字的所有资产）。

![](img/firstapp1.png)
As we said earlier, our sample network has an active chaincode container and a ledger that has been primed with 10 different cars. We also have some sample Javascript code - `query.js` - in the `fabcar` directory that can be used to query the ledger for details on the cars.

正如我们前面所说，我们的示例网络有一个活跃的链码容器和一个已经包含10种不同汽车的账本。我们还有一些示例Javascript代码，我们可以使用`fabcar`目录中的`query.js`来查询账本中关于车的详细信息。

Before we take a look at how that app works, we need to install the SDK node modules in order for our program to function. From your `fabcar` directory, issue the following:

在我们查看该应用程序的工作原理之前，我们需要安装SDK模块才能让程序运行。在`fabcar`目录中，运行下面的命令：

	npm install
Note

You will issue all subsequent commands from the `fabcar` directory.

请注意：后续命令也都是在`fabcar`目录中运行。
Now we can run our javascript programs. First, let’s run our `query.js` program to return a listing of all the cars on the ledger. A function that will query all the cars, `queryAllCars`, is pre-loaded in the app, so we can simply run the program as is:

现在我们可以运行JavaScript程序。首先，运行`query.js` 程序，返回账本上所有汽车列表。应用程序中预先加载了一个`queryAllCars`函数，用于查询所有车辆，因此我们可以简单地运行程序：

	node query.js
It should return something like this:

返回应如下：

	Query result count =  1
	Response is  [{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},
	{"Key":"CAR1",   "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},
	{"Key":"CAR2", "Record":{"colour":"green","make":"Hyundai","model":"Tucson","owner":"Jin Soo"}},
	{"Key":"CAR3", "Record":{"colour":"yellow","make":"Volkswagen","model":"Passat","owner":"Max"}},
	{"Key":"CAR4", "Record":{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}},
	{"Key":"CAR5", "Record":{"colour":"purple","make":"Peugeot","model":"205","owner":"Michel"}},
	{"Key":"CAR6", "Record":{"colour":"white","make":"Chery","model":"S22L","owner":"Aarav"}},
	{"Key":"CAR7", "Record":{"colour":"violet","make":"Fiat","model":"Punto","owner":"Pari"}},
	{"Key":"CAR8", "Record":{"colour":"indigo","make":"Tata","model":"Nano","owner":"Valeria"}},
	{"Key":"CAR9", "Record":{"colour":"brown","make":"Holden","model":"Barina","owner":"Shotaro"}}]
These are the 10 cars. A black Tesla Model S owned by Adriana, a red Ford Mustang owned by Brad, a violet Fiat Punto owned by someone named Pari, and so on. The ledger is key/value based and in our implementation the key is `CAR0` through `CAR9`. This will become particularly important in a moment.

这里有10辆车，一辆属于Adriana的黑色Tesla Model S、一辆属于Brad的红色Ford Mustang、一辆属于Pari的紫罗兰色Fiat Punto等等。账本是基于Key/Value	的，在这里，关键字是从`CAR0`到`CAR9`。这一点特别重要。

Now let’s see what it looks like under the hood (if you’ll forgive the pun). Use an editor (e.g. atom or visual studio) and open the `query.js` program.

现在让我们来看看代码内容。使用编辑器（例如atom或visual studio）打开`query.js`程序。

The inital section of the application defines certain variables such as chaincode ID, channel name and network endpoints:

应用程序的初始部分定义了变量，如链码，通道名称和网络端点：

	var options = {
		wallet_path : path.join(__dirname, './network/creds'),
		user_id: 'PeerAdmin',
		channel_id: 'mychannel',
		chaincode_id: 'fabcar',
		network_url: 'grpc://localhost:7051',
This is the chunk where we construct our query:

这是构建查询的代码块：

	// queryCar - requires 1 argument, ex: args: ['CAR4'],
	// queryAllCars - requires no arguments , ex: args: [''],
	const request = {
		chaincodeId: options.chaincode_id,
		txId: transaction_id,
		fcn: 'queryAllCars',
		args: ['']
We define the `chaincode_id` variable as `fabcar` – allowing us to target this specific chaincode – and then call the `queryAllCars` function defined within that chaincode.

我们将`chaincode_id`变量赋值为`fabcar`- 这让我们定位到这个特定的链码 - 然后调用该链码中定义的`queryAllCars`函数。

When we issued the `node query.js` command earlier, this specific function was called to query the ledger. However, this isn’t the only function that we can pass.

在前面，我们发出`node query.js`命令时，会调用了特定函数来查询账本。但是，这不是我们能够使用的唯一功能。

To take a look at the others, navigate to the `chaincode` subdirectory and open `fabcar.go` in your editor. You’ll see that we have the following functions available to call - `initLedger`, `queryCar`, `queryAllCars`, `createCar` and `changeCarOwner`. Let’s take a closer look at the `queryAllCars` function to see how it interacts with the ledger.

要查看其他，请转至到`chaincode`子目录并在编辑器中打开`fabcar.go`。你会看到，我们可以调用下面的函数- `initLedger`、`queryCar`、`queryAllCars`、`createCar`和`changeCarOwner`。让我们仔细看看`queryAllCars`函数是如何与账本进行交互的。


	func (s *SmartContract) queryAllCars(APIstub shim.ChaincodeStubInterface) sc.Response {

		startKey := "CAR0"
		endKey := "CAR999"

		resultsIterator, err := APIstub.GetStateByRange(startKey, endKey)
The function uses the shim interface function `GetStateByRange` to return ledger data between the args of `startKey` and `endKey`. Those keys are defined as `CAR0` and `CAR999` respectively. Therefore, we could theoretically create 1,000 cars (assuming the keys are tagged properly) and a `queryAllCars` would reveal every one.

该函数调用shim接口函数`GetStateByRange`来返回参数在`startKey`和`endKey`间的账本数据。这两个键值分别定义为`CAR0`和`CAR999`。因此，我们理论上可以创建1,000辆汽车（假设Keys都被正确使用），`queryAllCars`函数将会显示出每一辆汽车的信息。

Below is a representation of how an app would call different functions in chaincode.

下图演示了一个应用程序如何在链码中调用不同功能。

![](img/firstapp2.png)
We can see our `queryAllCars` function up there, as well as one called `createCar` that will allow us to update the ledger and ultimately append a new block to the chain. But first, let’s do another query.

我们可以看到我们用过的`queryAllCars`函数，还有一个叫做`createCar`，这个函数可以让我们更新账本，并最终在链上增加一个新区块。但首先，让我们做另外一个查询。

Go back to the `query.js` program and edit the constructor request to query a specific car. We’ll do this by changing the function from `queryAllCars` to `queryCar` and passing a specific “Key” to the args parameter. Let’s use `CAR4` here. So our edited `query.js` program should now contain the following:

现在我们返回`query.js`程序并编辑请求构造函数以查询特定的车辆。为达此目的，我们将函数`queryAllCars`更改为`queryCar`并将特定的“Key” 传递给args参数。在这里，我们使用`CAR4`。 所以我们编辑后的`query.js`程序现在应该包含以下内容：

	const request = {
		chaincodeId: options.chaincode_id,
		txId: transaction_id,
		fcn: 'queryCar',
		args: ['CAR4']
Save the program and navigate back to your `fabcar` directory. Now run the program again:

保存程序并返回`fabcar`目录。现在再次运行程序：

	node query.js
You should see the following:

您应该看到以下内容：

	{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}
So we’ve gone from querying all cars to querying just one, Adriana’s black Tesla Model S. Using the `queryCar` function, we can query against any key (e.g. `CAR0`) and get whatever make, model, color, and owner correspond to that car.

这样，我们就从查询所有车变成了只查询一辆车：Adriana的黑色Tesla Model S。使用`queryCar`函数，我们可以查询任意关键字（例如`CAR0`），并获得与该车相对应的制造厂商、型号、颜色和所有者。
Great. Now you should be comfortable with the basic query functions in the chaincode, and the handful of parameters in the query program. Time to update the ledger...

很好，现在您应该比较熟悉该链码的基本查询功能以及带参数的查询功能了。现在是时候更新账本了...

###更新账本(Updating the Ledger)
Now that we’ve done a few ledger queries and added a bit of code, we’re ready to update the ledger. There are a lot of potential updates we could make, but let’s just create a new car for starters.

我们已经完成了几个分类帐查询并添加了一些代码，现在更新账本。我们可以做很多更新动作，首先让我们为新手创建一辆新车。

Ledger updates start with an application generating a transaction proposal. Just like query, a request is constructed to identify the channel ID, function, and specific smart contract to target for the transaction. The program then calls the `channel.SendTransactionProposal` API to send the transaction proposal to the peer(s) for endorsement.

账本更新是从生成交易提案的应用程序开始的。就像查询一样，我们将会构造一个请求，用来识别要进行交易的通道ID、函数以及智能合约。该程序然后调用`channel.SendTransactionProposal`API将交易建议发送给peer(s)进行认证。

The network (i.e. endorsing peer) returns a proposal response, which the application uses to build and sign a transaction request. This request is sent to the ordering service by calling the `channel.sendTransaction` API. The ordering service will bundle the transaction into a block and then “deliver” the block to all peers on a channel for validation. (In our case we have only the single endorsing peer.)

网络（即endorsing peer）返回一个提案答复，应用程序以此来创建和签署交易请求。该请求通过调用`channel.sendTransaction` API发送到排序服务器。排序服务器将把交易打包进区块，然后将区块“发送”到通道上的所有peers进行认证。（在我们的例子中，我们只有一个endorsing peer。）

Finally the application uses the `eh.setPeerAddr` API to connect to the peer’s event listener port, and calls `eh.registerTxEvent` to register events associated with a specific transaction ID. This API allows the application to know the fate of a transaction (i.e. successfully committed or unsuccessful). Think of it as a notification mechanism.

最后，应用程序使用`eh.setPeerAddr` API连接到peer的事务监听端口，并调用`eh.registerTxEvent`注册与特定交易ID相关联的事务。该API使得应用程序获得事务的结果（即成功提交或不成功）。把它当作一个通知机制。

Note

We don’t go into depth here on a transaction’s lifecycle. Consult the [Transaction Flow](http://hyperledger-fabric.readthedocs.io/en/latest/txflow.html) documentation for lower level details on how a transaction is ultimately committed to the ledger.

请注意：这里我们不深入讨论交易细节。有关交易如何最终提交给账本的详细信息，请参阅[交易流程](http://hyperledger-fabric.readthedocs.io/en/latest/txflow.html)文档。

The goal with our initial invoke is to simply create a new asset (car in this case). We have a separate javascript program - `invoke.js` - that we will use for these transactions. Just like query, use an editor to open the program and navigate to the codeblock where we construct our invocation:

我们初始调用的目标是简单地创建一个新的资产（这里为汽车）。我们有一个独立的用于这些交易的JavaScript程序 - `invoke.js`。就像查询一样，使用编辑器打开程序并转到构建调用的代码块：

	// createCar - requires 5 args, ex: args: ['CAR11', 'Honda', 'Accord', 'Black', 'Tom'],
	// changeCarOwner - requires 2 args , ex: args: ['CAR10', 'Barry'],
	// send proposal to endorser
	var request = {
		targets: targets,
		chaincodeId: options.chaincode_id,
		fcn: '',
		args: [''],
		chainId: options.channel_id,
		txId: tx_id
You’ll see that we can call one of two functions - `createCar` or `changeCarOwner`. Let’s create a red Chevy Volt and give it to an owner named Nick. We’re up to `CAR9` on our ledger, so we’ll use `CAR10` as the identifying key here. The updated codeblock should look like this:

我们可以调用函数`createCar`或者`changeCarOwner`。首先我们创建一个红色的Chevy Volt，并把它归属于Nick。在账本中我们的Key值已经用到了`CAR9` ，所以这里我们将使用`CAR10`。更新代码块如下：

	var request = {
		targets: targets,
    	chaincodeId: options.chaincode_id,
    	fcn: 'createCar',
    	args: ['CAR10', 'Chevy', 'Volt', 'Red', 'Nick'],
    	chainId: options.channel_id,
    	txId: tx_id
Save it and run the program:

保存并运行程序：

	node invoke.js
There will be some output in the terminal about Proposal Response and Transaction ID. However, all we’re concerned with is this message:

终端将会输出一些提案响应和交易ID。但是，我们关心的是这个：

	The transaction has been committed on peer localhost:7053
The peer emits this event notification, and our application receives it thanks to our `eh.registerTxEvent` API. So now if we go back to our `query.js` program and call the `queryCar` function against an arg of `CAR10`, we should see the following:

peer发出此事务通知，我们的应用程序通过 `eh.registerTxEvent`API接收到该通知。现在，如果我们回到`query.js`程序并调用带有`CAR10`参数的`queryCar`函数，将会看到：

	Response is  {"colour":"Red","make":"Chevy","model":"Volt","owner":"Nick"}
Finally, let’s call our last function - `changeCarOwner`. Nick is feeling generous and he wants to give his Chevy Volt to a man named Barry. So, we simply edit `invoke.js` to reflect the following:

最后，我们来调用最后一个函数`changeCarOwner`。Nick很慷慨，他想把他的Chevy Volt送给Barry。所以，我们简单编辑`invoke.js` 如下：

	var request = {
		targets: targets,
		chaincodeId: options.chaincode_id,
		fcn: 'changeCarOwner',
		args: ['CAR10', 'Barry'],
		chainId: options.channel_id,
		txId: tx_id
Execute the program again - `node invoke.js` - and then run the query app one final time. We are still querying against `CAR10`, so we should see:

再次运行`node invoke.js`，之后再运行查询程序，我们仍然是使用`CAR10`作为参数查询，将会看到如下结果：

	Response is  {"colour":"Red","make":"Chevy","model":"Volt","owner":"Barry"}
###其他资源(Additional Resources)
The [Hyperledger Fabric Node SDK repo](https://github.com/hyperledger/fabric-sdk-node) is an excellent resource for deeper documentation and sample code. You can also consult the Hyperledger Fabric community and component experts on [Hyperledger Rocket Chat](https://chat.hyperledger.org/home).

在[Hyperledger Fabric Node SDK repo](https://github.com/hyperledger/fabric-sdk-node)可以找到更多的说明文档以及代码例子。您也可以在[Hyperledger Rocket Chat](https://chat.hyperledger.org/home)上咨询Hyperledger Fabric社区专家。