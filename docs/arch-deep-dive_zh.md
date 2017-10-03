
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/arch-deep-dive.html) | Ruijun Ou | Min Luo |


The v1 architecture delivers the following advantages:

此V1版本架构实现了以下改进：

- **Chaincode trust flexibility.链码信任的灵活性** The architecture separates trust assumptions for chaincodes (blockchain applications) from trust assumptions for ordering. In other words, the ordering service may be provided by one set of nodes (orderers) and tolerate some of them to fail or misbehave, and the endorsers may be different for each chaincode.

    该架构分离了链码（区块链应用）的信任假设和排序的信任假设。换句话说，排序服务可以由一组节点（排序者）提供，可以容忍一些失败节点或恶意节点，以及每个链码的背书者可以不同。 

- **Scalability.可扩展性** As the endorser nodes responsible for particular chaincode are orthogonal to the orderers, the system may scale better than if these functions were done by the same nodes. In particular, this results when different chaincodes specify disjoint endorsers, which introduces a partitioning of chaincodes between endorsers and allows parallel chaincode execution (endorsement). Besides, chaincode execution, which can potentially be costly, is removed from the critical path of the ordering service.

    作为特定链码的背书节点和排序者是垂直交叉关系，这可以使系统性能比这些功能都在同一节点上完成要好。特别是，当不同的链码指定不同的背书者时更是如此，为此在背书者之间引入了链码分区技术，允许链码并行执行（和背书）。此外，链码的执行，可能比较耗费计算资源，所以把它从排序服务的关键路径移除。

- **Confidentiality.机密性** The architecture facilitates deployment of chaincodes that have confidentiality requirements with respect to the content and state updates of its transactions.

    该架构利于有保密性要求的链码部署，能满足交易内容和状态更新的保密性要求。

- **Consensus modularity.共识模块化** The architecture is modular and allows pluggable consensus (i.e., ordering service) implementations.

    该架构是模块化的，允许可插拔的模块化共识（即排序服务）实现。

**Part I: Hyperledger Fabric v1相关的架构要素(Elements of the architecture relevant to Hyperledger Fabric v1)**

1. System architecture / 系统架构
2. Basic workflow of transaction endorsement / 交易背书的基本工作流程
3. Endorsement policies / 背书策略

**Part II: 架构的v1后期要素(Post-v1 elements of the architecture)**

4. Ledger checkpointing (pruning) / 账本检查点（修剪）
          
## 1. 系统架构(System architecture)

The blockchain is a distributed system consisting of many nodes that communicate with each other. The blockchain runs programs called chaincode, holds state and ledger data, and executes transactions. The chaincode is the central element as transactions are operations invoked on the chaincode. Transactions have to be “endorsed” and only endorsed transactions may be committed and have an effect on the state. There may exist one or more special chaincodes for management functions and parameters, collectively called system chaincodes.

区块链是一个分布式系统，由许多相互通信的节点组成。区块链运行的程序称为链码，保存状态和账本数据、执行交易。链码是核心要素，交易操作在链码上调用。交易必须被“背书”，只有经过背书的交易才可以提交，并对状态产生影响。有可能存在一个或多个特定的链码用于管理功能和参数，统称为系统链码。

### 1.1. 交易(Transactions)

Transactions may be of two types:

交易可以有两种类型：

-  *Deploy transactions* create new chaincode and take a program as parameter. When a deploy transaction executes successfully, the chaincode has been installed “on” the blockchain.

-  *部署交易* 创建新的链码并设置一个程序作为参数。当一个部署交易执行成功，表明链码已被安装到区块链上。

-  *Invoke transactions* perform an operation in the context of previously deployed chaincode. An invoke transaction refers to a chaincode and to one of its provided functions. When successful, the chaincode executes the specified function - which may involve modifying the corresponding state, and returning an output.

-  *调用交易* 是在之前已部署链码的情况下执行一个操作。调用交易引用链码提供的一个函数。当成功时，链码执行特定的函数-它可能涉及修改相应的状态，并返回一个输出。

As described later, deploy transactions are special cases of invoke transactions, where a deploy transaction that creates new chaincode, corresponds to an invoke transaction on a system chaincode.

如后所述，部署交易是调用交易的特例，部署交易创建新的链码，对应于系统链码的一个调用交易。

**Remark:** *This document currently assumes that a transaction either creates new chaincode or invokes an operation provided by *one* already deployed chaincode. This document does not yet describe: a) optimizations for query (read-only) transactions (included in v1), b) support for cross-chaincode transactions (post-v1 feature).*

**注意：** *本文档目前假定交易不是创建链码就是调用*某个*已创建的链码。本文档尚未描述：a)交易优化查询（只读）（V1中包含）；b)跨链码交易支持（V1后期特征）。*

### 1.2. 区块链数据结构(Blockchain datastructures)

#### 1.2.1. State / 状态

The latest state of the blockchain (or, simply, state) is modeled as a versioned key/value store (KVS), where keys are names and values are arbitrary blobs. These entries are manipulated by the chaincodes (applications) running on the blockchain through `put` and `get` KVS-operations. The state is stored persistently and updates to the state are logged. Notice that versioned KVS is adopted as state model, an implementation may use actual KVSs, but also RDBMSs or any other solution.

区块链的最新状态（简称为，状态）被建模为一个版本键/值存储（KVS），键的名称和值可以是任意的。整体上由运行在区块链上的链码（应用）操控，通过`存`和`取`KVS操作实现。状态持续存储并且状态的更新也被记录。注意版本KVS被采用为状态模型，实现可以使用实际的KVS，像RDBMS或其它方案。

More formally, state `s` is modeled as an element of a mapping `K -> (V X N)`, where:

更正式地，状态s建模为一个元素映射K -> (V X N)，其中：

* `K` is a set of keys
- `K`是一组键
* `V` is a set of values
- `V`是一组值
* `N` is an infinite ordered set of version numbers. Injective function `next: N -> N` takes an element of `N` and returns the next version number.
- N是一个无限有序的版本号集。内射函数`next: N -> N`获取`N`的一个元素并返回下一个版本号。

Both `V` and `N` contain a special element `\bot`, which is in case of `N` the lowest element. Initially all keys are mapped to (\bot,\bot). For s(k)=(v,ver) we denote v by s(k).value, and `ver` by `s(k).version`.

`V`和`N`都包含一个特定的元素`\bot`，这是`N`的最底层元素的特例。最开始时所有的键都映射到`（\bot,\bot）`。对于`s(k)=(v,ver)`，我们用`s(k).value`代表`v`，
用`s(k).version`代表`ver`。

KVS operations are modeled as follows:

* `put(k,v)`, for `k\in K` and `v\in V`, takes the blockchain state `s` and changes it to `s'` such that `s'(k)=(v,next(s(k).version))` with `s'(k')=s(k')` for all `k'!=k`.
* `get(k)` returns `s(k)`.

KVS操作模型如下：

* `put(k,v)`,对于`K中的k`和`V中的v`，处理区块链状态`s`，将它变为`s'`，这样`s'(k)=(v,next(s(k).version))`，以及`s'(k')=s(k')` 以保证所有的`k'!=k`。
* `get(k)`返回`s(k)`。

State is maintained by peers, but not by orderers and clients.

状态由peer节点保持，而不是排序节点和客户端。

**State partitioning.** Keys in the KVS can be recognized from their name to belong to a particular chaincode, in the sense that only transaction of a certain chaincode may modify the keys belonging to this chaincode. In principle, any chaincode can read the keys belonging to other chaincodes. *Support for cross-chaincode transactions, that modify the state belonging to two or more chaincodes is a post-v1 feature.*

**状态划分。**KVS中的键能够通过它们的名字识别属于哪个特定的链码。从这点上说，只有某个链码的交易可以修改属于这个链码的键。原则上，任何链码能够读取属于其它链码的键。*支持跨链交易，修改属于两个或更多链码的状态是V1后期的特征。*

#### 1.2.2 账本（Ledger）
Ledger provides a verifiable history of all successful state changes (we talk about valid transactions) and unsuccessful attempts to change state (we talk about invalid transactions), occurring during the operation of the system.
 
账本提供了在系统运行过程中发生的可验证历史，它包含所有成功的状态更改（我们称为有效交易）和不成功的状态更改尝试（我们称为无效交易）。

Ledger is constructed by the ordering service (see Sec 1.3.3) as a totally ordered hashchain of blocks of (valid or invalid) transactions. The hashchain imposes the total order of blocks in a ledger and each block contains an array of totally ordered transactions. This imposes total order across all transactions.

账本是由排序服务（见1.3.3节内容）构建的一个全部有序的交易哈希链块（有效的或无效的）。哈希链强制将全部排序块置入账本，每个块包含一批全部排序交易。这个强制全部排序覆盖所有交易。

Ledger is kept at all peers and, optionally, at a subset of orderers. In the context of an orderer we refer to the Ledger as to `OrdererLedger`, whereas in the context of a peer we refer to the ledger as to `PeerLedger`. `PeerLedger` differs from the `OrdererLedger` in that peers locally maintain a bitmask that tells apart valid transactions from invalid ones (see Section XX for more details).

账本保存在所有peer节点，可选地，保存在排序者的一个子集。在谈论排序时我们说的账本是排序账本，而谈论peer节点时我们说的账本是peer账本。peer账本与排序账本的区别是，peer节点本地维护一个位掩码来表明隔离有效交易和无效交易（看XX节获取更详细的描述）。

Peers may prune `PeerLedger` as described in Section XX (post-v1 feature). Orderers maintain `OrdererLedger` for fault-tolerance and availability (of the `PeerLedger`) and may decide to prune it at anytime, provided that properties of the ordering service (see Sec. 1.3.3) are maintained.

Peer节点可以修剪peer账本，具体在XX节描述（V1后期特征）。排序节点维护排序账本用于实现容错和（peer节点账本）可用，可以在任何时刻决定修剪它，只要排序服务（见1.3.3节）的属性在维持中。

The ledger allows peers to replay the history of all transactions and to reconstruct the state. Therefore, state as described in Sec 1.2.1 is an optional datastructure.

账本允许peer节点重演所有交易的历史和重建状态。因此，在1.2.1节中描述的状态是一个可选的数据结构。

### 1.3. 节点(Nodes)

Nodes are the communication entities of the blockchain. A “node” is only a logical function in the sense that multiple nodes of different types can run on the same physical server. What counts is how nodes are grouped in “trust domains” and associated to logical entities that control them.

Node节点是区块链的通信实体。一个“node节点”仅仅是一个逻辑函数，在这个意义上，多个不同类型的node节点可以运行在同一台物理服务器上。关键在于node节点如何在“信任域”中分组和关联控制它们的逻辑实体。

There are three types of nodes:

1、**Client** or **submitting-client**: a client that submits an actual transaction-invocation to the endorsers, and broadcasts transaction-proposals to the ordering service.

2、**Peer**: a node that commits transactions and maintains the state and a copy of the ledger (see Sec, 1.2). Besides, peers can have a special endorser role.

3、**Ordering-service-node** or **orderer**: a node running the communication service that implements a delivery guarantee, such as atomic or total order broadcast.

The types of nodes are explained next in more detail.

有三种类型的node节点：

1、**客户端**或者**提交客户端**：客户端提交实际交易调用到背书者，广播交易请求到排序服务节点。

2、**Peer节点**：提交交易、维持状态和账本的拷贝（见1.2节）。此外，peer节点可以有一个特殊的背书角色。

3、**排序服务节点**或**排序者**：运行通信服务实现交付保证，像原子或全序广播。

Node节点的类型接下来进行更详细的解释。

#### 1.3.1. 客户端(Client)

The client represents the entity that acts on behalf of an end-user. It must connect to a peer for communicating with the blockchain. The client may connect to any peer of its choice. Clients create and thereby invoke transactions.

As detailed in Section 2, clients communicate with both peers and the ordering service.

客户端代表最终用户实体。它必须连接到一个peer节点以便与区块链交互。客户端可以选择连接任何peer节点。客户端创建并调用交易。

如第2节详细说明，客户端同时与peer节点和排序服务通信。

#### 1.3.2. Peer

A peer receives ordered state updates in the form of blocks from the ordering service and maintain the state and the ledger.

Peer节点以块的形式从排序服务接收有序状态更新，维护状态和账本。

Peers can additionally take up a special role of an **endorsing peer**, or an **endorser**. The special function of an *endorsing peer* occurs with respect to a particular chaincode and consists in *endorsing* a transaction before it is committed. Every chaincode may specify an *endorsement policy* that may refer to a set of endorsing peers. The policy defines the necessary and sufficient conditions for a valid transaction endorsement (typically a set of endorsers’ signatures), as described later in Sections 2 and 3. In the special case of deploy transactions that install new chaincode the (deployment) endorsement policy is specified as an endorsement policy of the system chaincode.

Peer节点能附加一个特殊的**背书节点角色**，或**背书者**。*背书节点*的特殊功能是关于特殊链码，存在于提交之前*背书*一个交易。每个链码可以指定一个*背书策略*，可以引用一组背书节点。策略定义一个有效交易背书的必要和充分条件（典型的是一组背书者签名），在后面的第2节和第3节描述。在部署交易的特殊情况下，安装链码（部署）背书策略是由系统链码的背书策略指定。

#### 1.3.3. Orderer

The *orderers* form the *ordering service*, i.e., a communication fabric that provides delivery guarantees. The ordering service can be implemented in different ways: ranging from a centralized service (used e.g., in development and testing) to distributed protocols that target different network and node fault models.

*排序者*产生*排序服务*，即，一个提供交付保证的通信架构。排序服务能以不同的方式实现：从集中服务排序（例如，开发和测试）的分布式协议，指向不同的网络和节点故障模型。

Ordering service provides a shared *communication channel* to clients and peers, offering a broadcast service for messages containing transactions. Clients connect to the channel and may broadcast messages on the channel which are then delivered to all peers. The channel supports *atomic* delivery of all messages, that is, message communication with total-order delivery and (implementation specific) reliability. In other words, the channel outputs the same messages to all connected peers and outputs them to all peers in the same logical order. This atomic communication guarantee is also called *total-order broadcast*, *atomic broadcast*, or *consensus* in the context of distributed systems. The communicated messages are the candidate transactions for inclusion in the blockchain state.

排序服务为客户端和peer节点提供共享的*通信信道*，为包含交易的消息提供广播服务。客户端连接到信道，可以在信道上广播消息，信道随后传递消息给所有peer节点。信道支持所有消息的*原子*传递，意思是，全部排序交付的消息通信和（具体实施）可靠性。换句话说，信道输出同样的消息给所有连接的peer节点并且输出的消息具有同样的逻辑顺序。这个原子通信保证也称为*全部排序广播*，*原子广播*，或是分布式系统中的*共识*。通信消息是包含在区块链状态中的申请交易。

**Partitioning (ordering service channels).** Ordering service may support multiple *channels* similar to the *topics* of a publish/subscribe (pub/sub) messaging system. Clients can connects to a given channel and can then send messages and obtain the messages that arrive. Channels can be thought of as partitions - clients connecting to one channel are unaware of the existence of other channels, but clients may connect to multiple channels. Even though some ordering service implementations included with Hyperledger Fabric v1 will support multiple channels, for simplicity of presentation, in the rest of this document, we assume ordering service consists of a single channel/topic.

**分隔（排序服务信道）。**排序服务可以支持多个*信道*，类似发布/订阅*主题*消息系统。客户端能够连接到一个给定的信道，然后能够发送消息和获得到达的消息。信道能够被认为是分区-客户端连接到一个信道而没有察觉到其它信道的存在，但客户端可以连接到多个信道。尽管一些排序服务实现包括Hyperledger Fabric v1将支持多信道，为了阐述简单，在本文档的剩余部分，我们假定排序服务包含一个单独的信道/主题。

**Ordering service API.** Peers connect to the channel provided by the ordering service, via the interface provided by the ordering service. The ordering service API consists of two basic operations (more generally *asynchronous events*):

**排序服务API。**peer节点通过排序服务提供的接口连接到排序服务提供的信道。排序服务API包含两个基本操作（更多是*异步事件*）：

**TODO** add the part of the API for fetching particular blocks under client/peer specified sequence numbers.

- broadcast(blob): a client calls this to broadcast an arbitrary message blob for dissemination over the channel. This is also called request(blob) in the BFT context, when sending a request to a service.

**待办**：增加在客户端/peer节点指定序列号下取得特定块的API。

- broadcast(blob): 客户端调用此函数来广播任意消息blob在全信道散播。这在BFT环境下也称为request(blob)，当发送一个请求到服务器时。

- deliver(seqno, prevhash, blob): the ordering service calls this on the peer to deliver the message blob with the specified non-negative integer sequence number (seqno) and hash of the most recently delivered blob (prevhash). In other words, it is an output event from the ordering service. deliver() is also sometimes called notify() in pub-sub systems or commit() in BFT systems.

- deliver(seqno, prevhash, blob):排序服务在peer节点传送带有非负整型序列号（seqno）和blob的最近哈希(prevhash)的消息blob时调用这个。换言之，它是从排序服务产生的输出事件。deliver()有时在发布/订阅系统也称为notify() ，或在BFT系统中称为commit()。

**Ledger and block formation.** The ledger (see also Sec. 1.2.2) contains all data output by the ordering service. In a nutshell, it is a sequence of deliver(seqno, prevhash, blob) events, which form a hash chain according to the computation of prevhash described before.

**账本和块构成。** 账本（见1.2.2）包含了排序服务输出的所有数据。概括地说，它是一系列deliver(seqno, prevhash, blob)事件，根据之前描述的prevhash计算形成的一个哈希链。

Most of the time, for efficiency reasons, instead of outputting individual transactions (blobs), the ordering service will group (batch) the blobs and output blocks within a single deliver event. In this case, the ordering service must impose and convey a deterministic ordering of the blobs within each block. The number of blobs in a block may be chosen dynamically by an ordering service implementation.

大多数情况下，出于效率的原因，代替输出单个交易（blobs），排序服务会批量输出blobs，而且输出块在一个单个交付事件中。在这种情况下，排序服务必须在每个块内实施和传递blobs的确定顺序。块内blobs的数量可以由排序服务实现动态选择。

In the following, for ease of presentation, we define ordering service properties (rest of this subsection) and explain the workflow of transaction endorsement (Section 2) assuming one blob per deliver event. These are easily extended to blocks, assuming that a deliver event for a block corresponds to a sequence of individual deliver events for each blob within a block, according to the above mentioned deterministic ordering of blobs within a blocs.

接下来，为了便于说明，我们定义排序服务的属性（本节的剩余部分）和解释交易背书的工作流程（第2节），假定每个deliver事件一个blob。这些容易扩展到块，假定一个块的deliver事件对应一系列单个deliver事件，对于块内的每个blob，根据上述描述确定块内blobs顺序。

**Ordering service properties**

The guarantees of the ordering service (or atomic-broadcast channel) stipulate what happens to a broadcasted message and what relations exist among delivered messages. These guarantees are as follows:

**排序服务特性**

排序服务的保证（或原子广播信道）规定了广播消息的发生和交付消息之间存在什么关系。这些保证如下：

1、**Safety (consistency guarantees)**: As long as peers are connected for sufficiently long periods of time to the channel (they can disconnect or crash, but will restart and reconnect), they will see an *identical* series of delivered (seqno, prevhash, blob) messages. This means the outputs (deliver() events) occur in the *same order* on all peers and according to sequence number and carry *identical content* (blob and prevhash) for the same sequence number. Note this is only a logical order, and a deliver(seqno, prevhash, blob) on one peer is not required to occur in any real-time relation to deliver(seqno, prevhash, blob) that outputs the same message at another peer. Put differently, given a particular seqno, no two correct peers deliver different prevhash or blob values. Moreover, no value blob is delivered unless some client (peer) actually called broadcast(blob) and, preferably, every broadcasted blob is only delivered once.

1、**安全性（一致性保证）**：只要peer节点连接到信道足够长的时间（它们能够断开或奔溃，但会重启和重新连接），它们会看到交付（seqno,prevhash,blob）消息的*同等*序列。这意味着向所有peer节点输出（deliver()events）*相同排序*，以及根据序列号和为相同序列号携带*同等内容*（blob和prevhash）。注意这仅是一个逻辑顺序，在一个peer节点上的deliver(seqno,prevhash,blob)是不需要与另外一个peer节点输出的deliver(seqno,prevhash,blob)发生任何实时关系。换句话说，给定一个特定的seqno，没有两个正确的peer节点交付不同的prehash或blob值。而且，没有值blob交付除非一些客户端（peer节点）实际是广播（blob），以及更好的，每个广播blob只交付一次。

Furthermore, the deliver() event contains the cryptographic hash of the data in the previous deliver() event (prevhash). When the ordering service implements atomic broadcast guarantees, prevhash is the cryptographic hash of the parameters from the deliver() event with sequence number seqno-1. This establishes a hash chain across deliver() events, which is used to help verify the integrity of the ordering service output, as discussed in Sections 4 and 5 later. In the special case of the first deliver() event, prevhash has a default value.

此外，deliver()事件包含之前deliver()事件（prevhash）的数据加密哈希。当排序服务实现原子广播保证，prevhash是从序列号为seqno-1的deliver()事件得到的参数的加密哈希，这在第4节和第5节讨论。对于第一个deliver()事件特例，prevhash有一个缺省值。

2、**Liveness (delivery guarantee)**: Liveness guarantees of the ordering service are specified by a ordering service implementation. The exact guarantees may depend on the network and node fault model.

In principle, if the submitting client does not fail, the ordering service should guarantee that every correct peer that connects to the ordering service eventually delivers every submitted transaction.

2、**活跃度（交付保证）**：排序服务的活跃度保证由排序服务实现确定。准确的保证可以依赖于网络和节点故障模型。

原则上，如果提交客户端没有失败，排序服务应该保证每个连接到排序服务的正确peer节点终究交付每个提交交易。

To summarize, the ordering service ensures the following properties:

概括地说，排序服务确保以下特性：

- *Agreement.* For any two events at correct peers deliver(seqno, prevhash0, blob0) and deliver(seqno, prevhash1, blob1) with the same seqno, prevhash0==prevhash1 and blob0==blob1;

- *一致. *对于任何两个具有相同seqno的正确peer节点的事件deliver(seqno, prevhash0, blob0)和deliver(seqno, prevhash1, blob1) , 则prevhash0==prevhash1，以及 blob0==blob1;

- *Hashchain integrity.* For any two events at correct peers deliver(seqno-1, prevhash0, blob0) and deliver(seqno, prevhash, blob), prevhash = HASH(seqno-1||prevhash0||blob0).

- *哈希链完整性。*对于任何在正确peer节点的两个事件deliver(seqno-1, prevhash0, blob0)和deliver(seqno, prevhash, blob), prevhash = HASH(seqno-1||prevhash0||blob0).

- *No skipping.* If an ordering service outputs deliver(seqno, prevhash, blob) at a correct peer p, such that seqno>0, then p already delivered an event deliver(seqno-1, prevhash0, blob0).

- *没有跳过.* 如果排序服务在正确peer节点p输出deliver(seqno, prevhash, blob) , 这样的话seqno>0, 然后p已经交付事件deliver(seqno-1, prevhash0, blob0).

- *No creation.* Any event deliver(seqno, prevhash, blob) at a correct peer must be preceded by a broadcast(blob) event at some (possibly distinct) peer;

- *没有创造.* 任何在正确peer节点上的事件deliver(seqno, prevhash, blob)必须之前一定有一个broadcast(blob)事件在一些(可能是不同的)peer节点上;

- *No duplication (optional, yet desirable).* For any two events broadcast(blob) and broadcast(blob'), when two events deliver(seqno0, prevhash0, blob) and deliver(seqno1, prevhash1, blob') occur at correct peers and blob == blob', then seqno0==seqno1 and prevhash0==prevhash1.

- *没有重复 (可选,但可取).* 对于任何两个事件broadcast(blob)和broadcast(blob'), 当两个事件deliver(seqno0, prevhash0, blob) 和 deliver(seqno1, prevhash1, blob') 发生在正确的节点 和 blob == blob', 那么 seqno0==seqno1 和 prevhash0==prevhash1.

- *Liveness.* If a correct client invokes an event broadcast(blob) then every correct peer “eventually” issues an event deliver(*, *, blob), where * denotes an arbitrary value.

- *活跃性。*如果正确的客户端调用事件broadcast(blob)那么每个正确的peer节点“最终”发出事件deliver(*, *, blob)，其中*表示任意值。

## 2. 交易背书的基本工作流程(Basic workflow of transaction endorsement)

In the following we outline the high-level request flow for a transaction.
**Remark:** *Notice that the following protocol does not assume that all transactions are deterministic, i.e., it allows for non-deterministic transactions.*

接下来我们概述一个交易的高级请求流程。

备注：注意下面的协议*不假定所有的交易是确定的，即，它允许不确定交易*。

### 2.1. 客户端创建交易和发送给它选择的背书peer节点（The client creates a transaction and sends it to endorsing peers of its choice）

To invoke a transaction, the client sends a PROPOSE message to a set of endorsing peers of its choice (possibly not at the same time - see Sections 2.1.2. and 2.3.). The set of endorsing peers for a given chaincodeID is made available to client via peer, which in turn knows the set of endorsing peers from endorsement policy (see Section 3). For example, the transaction could be sent to all endorsers of a given chaincodeID. That said, some endorsers could be offline, others may object and choose not to endorse the transaction. The submitting client tries to satisfy the policy expression with the endorsers available.

调用交易，客户端发送一个PROPOSE消息到它选择的一组背书peer节点（可能不是同一时间，见第2.1.2节和第2.3节）。给定chaincodeID的

背书peer节点的设置由客户端通过peer节点实现，从背书策略（见第3节）知道背书peer节点的设置。例如，交易能被发送给所有给定

chaincodeID背书者.那就是说，一些背书者能够离线，其它人可能反对和选择不为交易背书。提交客户端尝试满足背书者可用的背书策略表达。

In the following, we first detail PROPOSE message format and then discuss possible patterns of interaction between submitting client and endorsers.

接下来，我们首先描述PROPOSE消息格式，然后讨论在提交客户端和背书者之间可能的互动模式。

#### 2.1.1. `PROPOSE`消息格式(`PROPOSE` message format)

The format of a PROPOSE message is <PROPOSE,tx,[anchor]>, where tx is a mandatory and anchor optional argument explained in the following.

一个PROPOSE消息的格式是<PROPOSE,tx,[anchor]>，其中tx是强制的，anchor可选参数在下面列出。

- tx=<clientID,chaincodeID,txPayload,timestamp,clientSig>, where
  - clientID is an ID of the submitting client,
  - chaincodeID refers to the chaincode to which the transaction pertains,
  - txPayload is the payload containing the submitted transaction itself,
  - timestamp is a monotonically increasing (for every new transaction) integer maintained by the client,
  - clientSig is signature of a client on other fields of tx.

  - clientID 是提交客户端的身份，
  - chaincodeID 引用交易相关的链码，
  - txPayload 是提交交易自身的载体,
  - timestamp 是由客户端维护的一个单独递增(为每一笔交易)整型值,
  - clientSig 是tx的其它域客户端签名.

The details of txPayload will differ between invoke transactions and deploy transactions (i.e., invoke transactions referring to a deploy-specific system chaincode). 

txPayload的细节会在调用交易和部署交易之间有所不同（即，调用交易引用部署指定的系统链码）。

For an **invoke transaction**, txPayload would consist of two fields

- txPayload = <operation, metadata>, where
  - operation denotes the chaincode operation (function) and arguments,
  - metadata denotes attributes related to the invocation.

对于**调用交易**，txPayload会包含两个域

- txPayload = <operation, metadata>, 其中

  - operation 表示链码操作（函数）和参数,

  - metadata 表示调用相关的属性.

For a **deploy transaction**, txPayload would consist of three fields
- txPayload = <source, metadata, policies>, where
  - source denotes the source code of the chaincode,
  - metadata denotes attributes related to the chaincode and application,
  - policies contains policies related to the chaincode that are accessible to all peers, such as the endorsement policy. Note that endorsement policies are not supplied with txPayload in a deploy transaction, but txPayload of a deploy contains endorsement policy ID and its parameters (see Section 3).


对于**部署交易**，txPayload会包含三个域

txPayload = <source, metadata, policies>, 其中

source 表示链码的源码

metadata 表示链码和应用的相关属性

policies 包含所有peer节点可访问的链码的相关策略，像背书策略。注意背书策略在部署交易中不支持txPayload，但部署的txPayload包含背书策略ID和它的参数（见第3节）。

- anchor contains *read version dependencies*, or more specifically, key-version pairs (i.e., anchor is a subset of KxN), that binds or “anchors” the PROPOSE request to specified versions of keys in a KVS (see Section 1.2.). If the client specifies the anchor argument, an endorser endorses a transaction only upon read version numbers of corresponding keys in its local KVS match anchor (see Section 2.2. for more details).

- anchor包含*读版本依赖*，或更具体地说，键-版本对（即，anchor是KxN的一个子集），它捆绑或“锚”PROPOSE请求到指定KVS中key的版本（第1.2节）。如果客户端指定anchor参数，背书者背书交易的情况是，只基于读它本地KVS匹配anchor中的相应KEY的版本号（更详细内容见第2.2节）。

Cryptographic hash of tx is used by all nodes as a unique transaction identifier tid (i.e., tid=HASH(tx)). The client stores tid in memory and waits for responses from endorsing peers.

tx加密哈希被所有node节点用作唯一的交易标识tid（即，tid=HASH(tx)）。客户端保存tid在内存中，等待背书peer节点的响应。

#### 2.1.2.消息模式(Message patterns)

The client decides on the sequence of interaction with endorsers. For example, a client would typically send <PROPOSE, tx> (i.e., without the anchor argument) to a single endorser, which would then produce the version dependencies (anchor) which the client can later on use as an argument of its PROPOSE message to other endorsers. As another example, the client could directly send <PROPOSE, tx> (without anchor) to all endorsers of its choice. Different patterns of communication are possible and client is free to decide on those (see also Section 2.3.).

客户端决定与背书者互动的顺序。例如，客户端通常会发送<PROPOSE, tx>（即，没有anchor参数）到一个单独的背书者，背书者随后产生版本依赖（anchor）,客户端可以在晚些时候使用这个版本依赖（anchor）作为它的PROPOSE消息参数，发送给其它背书者。另外的例子，客户端能直接发送<PROPOSE, tx>（没有anchor）到它选择的所有背书者。不同的通信模式都有可能，客户端在这方面是自由的（也见第2.3节）。

### 2.2. 背书peer节点模拟交易和产生背书签名(The endorsing peer simulates a transaction and produces an endorsement signature)

On reception of a <PROPOSE,tx,[anchor]> message from a client, the endorsing peer epID first verifies the client’s signature clientSig and then simulates a transaction. If the client specifies anchor then endorsing peer simulates the transactions only upon read version numbers (i.e., readset as defined below) of corresponding keys in its local KVS match those version numbers specified by anchor.

在从客户端接收<PROPOSE,tx,[anchor]>消息时，背书peer节点epID首先校验客户端签名clientSig，然后模拟一个交易。如果客户端指定了anchor，那么背书peer节点模拟交易只基于在它本地KVS匹配的由anchor指定的版本号对应的key读版本号（即，下面定义的readset）。

Simulating a transaction involves endorsing peer tentatively executing a transaction (txPayload), by invoking the chaincode to which the transaction refers (chaincodeID) and the copy of the state that the endorsing peer locally holds.

模拟一个交易涉及背书节点尝试执行一个交易(txPayload), 通过调用链码到交易引用（chaincodeID）和背书peer节点本地持有的状态拷贝。

As a result of the execution, the endorsing peer computes read version dependencies (readset) and state updates (writeset), also called MVCC+postimage info in DB language.

作为执行的结果，背书peer节点计算读版本依赖（readset）和状态更新（writeset），也在DB语言中称为MVCC+postimage info。

Recall that the state consists of key/value (k/v) pairs. All k/v entries are versioned, that is, every entry contains ordered version information, which is incremented every time when the value stored under a key is updated. The peer that interprets the transaction records all k/v pairs accessed by the chaincode, either for reading or for writing, but the peer does not yet update its state. More specifically:

回顾状态包含键/值对。所有键/值对实体都是版本化的，那就是说，每个实体包含排序版本信息，它是在每次键的值更新时增加的。解释交易的peer节点记录了所有的被链码访问的键/值对，不管读或是写，peer节点不会更新它的状态。更具体地说：

- Given state s before an endorsing peer executes a transaction, for every key k read by the transaction, pair (k,s(k).version) is added to readset.

- 在背书节点执行一个交易前给定状态s，被交易读取的每个键k，键/值对(k,s(k).version)被添加到readset。

- Additionally, for every key k modified by the transaction to the new value v', pair (k,v') is added to writeset. Alternatively, v' could be the delta of the new value to previous value (s(k).value).

- 此外，对于每一个被交易编辑的键k到值v'，键/值对(k,v')被添加到writeset。或者，v'能成为新值与前值(s(k).value)的增量。

If a client specifies anchor in the PROPOSE message then client specified anchor must equal readset produced by endorsing peer when simulating the transaction.

如果客户端在PROPOSE消息中指定了anchor，那么客户端指定的anchor在模拟交易时必须等于背书peer节点产生的readset.

Then, the peer forwards internally tran-proposal (and possibly tx) to the part of its (peer’s) logic that endorses a transaction, referred to as endorsing logic. By default, endorsing logic at a peer accepts the tran-proposal and simply signs the tran-proposal. However, endorsing logic may interpret arbitrary functionality, to, e.g., interact with legacy systems with tran-proposal and tx as inputs to reach the decision whether to endorse a transaction or not.
然后，peer节点内部提交交易提案（可能是tx）到它的逻辑部分来背书交易，称为背书逻辑。缺省时，一个peer节点的背书逻辑接受交易提案并简单签署。无论如何，背书逻辑可以执行任意功能，到，例如，与原有系统交互交易提案和tx作为输入来得知是否背书交易。

If endorsing logic decides to endorse a transaction, it sends <TRANSACTION-ENDORSED, tid, tran-proposal,epSig> message to the submitting client(tx.clientID), where:

如果背书逻辑决定背书一个交易，它发送<TRANSACTION-ENDORSED, tid, tran-proposal,epSig> 消息到提交客户端，其中:

- tran-proposal := (epID,tid,chaincodeID,txContentBlob,readset,writeset),
where txContentBlob is chaincode/transaction specific information. The intention is to have txContentBlob used as some representation of tx (e.g., txContentBlob=tx.txPayload).

- epSig is the endorsing peer’s signature on tran-proposal

- 交易提案 ：=tran-proposal := (epID,tid,chaincodeID,txContentBlob,readset,writeset),
其中 txContentBlob 是链码/交易专用信息。目的是让txContentBlob 用作tx的一些陈述 (例如, txContentBlob=tx.txPayload).

- epSig 是背书peer节点的交易提案签名。

Else, in case the endorsing logic refuses to endorse the transaction, an endorser *may* send a message (TRANSACTION-INVALID, tid, REJECTED) to the submitting client.

否则，假使背书逻辑拒绝背书交易，背书者*可以*发送消息(TRANSACTION-INVALID, tid, REJECTED)到提交客户端。

Notice that an endorser does not change its state in this step, the updates produced by transaction simulation in the context of endorsement do not affect the state!

注意背书者在这一步不能改变它的状态，在背书没有影响状态的情况下交易模拟产生状态更新。

### 2.3. 提交客户端收集交易背书并通过排序服务广播它(The submitting client collects an endorsement for a transaction and broadcasts it through ordering service)

The submitting client waits until it receives “enough” messages and signatures on (TRANSACTION-ENDORSED, tid, *, *) statements to conclude that the transaction proposal is endorsed. As discussed in Section 2.1.2., this may involve one or more round-trips of interaction with endorsers.

提交客户端一直等待直到它在(TRANSACTION-ENDORSED, tid, *, *)上收集到“足够”的消息和签名来推断出交易提案已背书。像在2.1.2节讨论的那样，这可能涉及一个或多个与背书者的往返。

The exact number of “enough” depend on the chaincode endorsement policy (see also Section 3). If the endorsement policy is satisfied, the transaction has been endorsed; note that it is not yet committed. The collection of signed TRANSACTION-ENDORSED messages from endorsing peers which establish that a transaction is endorsed is called an endorsement and denoted by endorsement.

“足够”的准确数字取决于链码背书策略（也见第3节）。如果背书策略是安全的，交易已经背书；注意它还没提交。签署TRANSACTION-ENDORSED消息的收集从背书peer节点来，背书peer节点建立了交易是背书的称为背书并以背书为名称。

If the submitting client does not manage to collect an endorsement for a transaction proposal, it abandons this transaction with an option to retry later.

如果提交客户端没有设法为交易提案收集背书，则放弃这个交易，稍后再试。

For transaction with a valid endorsement, we now start using the ordering service. The submitting client invokes ordering service using the broadcast(blob), where blob=endorsement. If the client does not have capability of invoking ordering service directly, it may proxy its broadcast through some peer of its choice. Such a peer must be trusted by the client not to remove any message from the endorsement or otherwise the transaction may be deemed invalid. Notice that, however, a proxy peer may not fabricate a valid endorsement.

对于一个具有有效背书的交易，我们现在开始使用排序服务。提交客户端使用broadcast(blob)调用排序服务，其中blob=endorsement.如果客户端没有能力直接调用排序服务，它可以通过它选择的peer节点代理广播。这样的peer节点必须被客户端信任不会从背书移除任何消息或其它可能被无效的交易。注意一点，无论如何，代理peer节点不可能制造有效背书。

### 2.4. 排序服务向peer节点提交交易(The ordering service delivers a transactions to the peers)

When an event deliver(seqno, prevhash, blob) occurs and a peer has applied all state updates for blobs with sequence number lower than seqno, a peer does the following:

当一个事件(seqno, prevhash, blob)发生并且一个peer节点已为所有序列号低于seqno的blosbs更新状态，peer节点执行如下流程：

- It checks that the blob.endorsement is valid according to the policy of the chaincode (blob.tran-proposal.chaincodeID) to which it refers.

- 它检查blob.endorsement是有效的，根据的是它引用的链码(blob.tran-proposal.chaincodeID)。

- In a typical case, it also verifies that the dependencies (blob.endorsement.tran-proposal.readset) have not been violated meanwhile. In more complex use cases, tran-proposal fields in endorsement may differ and in this case endorsement policy (Section 3) specifies how the state evolves.

- 在典型情况下，它也验证了依赖(blob.endorsement.tran-proposal.readset)在期间没有被违反。在更复杂的用例中，背书中的交易提案域可能不同，在这种情况下，背书策略（第3节）指定状态如何形成。

Verification of dependencies can be implemented in different ways, according to a consistency property or “isolation guarantee” that is chosen for the state updates. **Serializability** is a default isolation guarantee, unless chaincode endorsement policy specifies a different one. Serializability can be provided by requiring the version associated with every key in the readset to be equal to that key’s version in the state, and rejecting transactions that do not satisfy this requirement.

依赖的验证能以不同的方式实现，根据一致性属性或为状态更新选择的“孤立保证”。**Serializability**是一个缺省的孤立保证，除非链码背书策略指定一个不同的。Serializability能够通过在readset中的每个key关联的版本被提供，相当于key在状态中的版本，并拒绝不满足这个要求的交易。

- If all these checks pass, the transaction is deemed *valid* or *committed*. In this case, the peer marks the transaction with 1 in the bitmask of the PeerLedger, applies blob.endorsement.tran-proposal.writeset to blockchain state (if tran-proposals are the same, otherwise endorsement policy logic defines the function that takes blob.endorsement).

- 如果所有这些检查通过，交易被视为*有效*或*承诺*。在这种情况下，peer节点在PeerLedger用1标记交易，适用于blob.endorsement.tran-proposal.writeset区块链状态（如果交易提案是相同的，其它背书策略逻辑定义了函数处理blob.endorsement）。

- If the endorsement policy verification of blob.endorsement fails, the transaction is invalid and the peer marks the transaction with 0 in the bitmask of the PeerLedger. It is important to note that invalid transactions do not change the state.

- 如果blob.endorsement背书策略验证失败，交易无效，并且peer节点在PeerLedger的位掩码用0标记交易。重要的是要注意无效交易不会改变状态。

Note that this is sufficient to have all (correct) peers have the same state after processing a deliver event (block) with a given sequence number. Namely, by the guarantees of the ordering service, all correct peers will receive an identical sequence of deliver(seqno, prevhash, blob) events. As the evaluation of the endorsement policy and evaluation of version dependencies in readset are deterministic, all correct peers will also come to the same conclusion whether a transaction contained in a blob is valid. Hence, all peers commit and apply the same sequence of transactions and update their state in the same way.

注意，这里有足够的让所有（正确）peer节点在处理一个给定序列号的deliver事件（块）之后具有同样的状态。即，通过排序服务的保证，所有正确的peer节点会收到相同的deliver(seqno, prevhash, blob)事件序列。当背书策略的评估和readset中版本依赖的评估是确定的，所有正确的peer节点也会得出相同的结论，关于包含在blob中的交易是否有效。因此，所有peer节点提交和应用同样交易序列并用同样的方式更新它们的状态。

![](img/flow-4.png)

   :alt: Illustration of the transaction flow (common-case path).
   
Figure 1. Illustration of one possible transaction flow (common-case path).

图1 一种可能的交易流程说明（一般情况路径）


## 3. 背书策略(Eorsement policies)

### 3.1. 背书策略规范(Endorsement policy specification)

An **endorsement policy**, is a condition on what endorses a transaction. Blockchain peers have a pre-specified set of endorsement policies, which are referenced by a deploy transaction that installs specific chaincode. Endorsement policies can be parametrized, and these parameters can be specified by a deploy transaction.

**背书策略**，是背书一个交易的条件。区块链peer节点有一组预先确定的背书策略，它被安装特定链码的部署交易引用。背书策略能参数化，这些参数能被部署交易指定。

To guarantee blockchain and security properties, the set of endorsement policies **should be a set of proven policies** with limited set of functions in order to ensure bounded execution time (termination), determinism, performance and security guarantees.

为了保证区块链和安全特性，背书策略组**应该是一组验证过的策略**，具有有限功能，为了保证有限的执行时间（终止），决定、性能和安全保证。

Dynamic addition of endorsement policies (e.g., by deploy transaction on chaincode deploy time) is very sensitive in terms of bounded policy evaluation time (termination), determinism, performance and security guarantees. Therefore, dynamic addition of endorsement policies is not allowed, but can be supported in future.

背书策略的动态添加（即，在链码部署时间由部署交易添加）是对背书评估时间限制（终止）、决定、性能和安全保证非常敏感的。因此，动态添加背书策略是不允许的，但将来能支持。

### 3.2. 针对背书策略的交易评估(Transaction evaluation against endorsement policy)

A transaction is declared valid only if it has been endorsed according to the policy. An invoke transaction for a chaincode will first have to obtain an endorsement that satisfies the chaincode’s policy or it will not be committed. This takes place through the interaction between the submitting client and endorsing peers as explained in Section 2.


交易只有经过根据背书策略的背书才会宣布有效。对于链码的调用交易首先需要的到一个满足链码策略的背书，或不提交。这通过在提交客户端和背书peer节点之间的互动发生，在第2节解释。

Formally the endorsement policy is a predicate on the endorsement, and potentially further state that evaluates to TRUE or FALSE. For deploy transactions the endorsement is obtained according to a system-wide policy (for example, from the system chaincode).

正式的背书策略是以背书为基础，以及潜在的进一步评估为真假状态。对于部署交易，获得背书的依据是系统系统范围策略（例如，来自系统链码）。

An endorsement policy predicate refers to certain variables. Potentially it may refer to:
1、keys or identities relating to the chaincode (found in the metadata of the chaincode), for example, a set of endorsers;
2、further metadata of the chaincode;
3、elements of the endorsement and endorsement.tran-proposal;
4、and potentially more.

背书策略断言引用一定的变量。潜在可能引用的是：

1、与链码有关的钥匙或身份（在链码元数据中能发现），例如，一组背书者；

2、链码进一步的元数据；

3、endorsement and endorsement.tran-proposal的元素；

4、其它更多。

The above list is ordered by increasing expressiveness and complexity, that is, it will be relatively simple to support policies that only refer to keys and identities of nodes.

上面的列表根据表现和复杂性排序，意思是说，它将会是相对简单的支持策略，只引用node节点的钥匙和身份。

The evaluation of an endorsement policy predicate must be deterministic. An endorsement shall be evaluated locally by every peer such that a peer does not need to interact with other peers, yet all correct peers evaluate the endorsement policy in the same way.

背书策略断言的评估必须被确定。背书应当被每个peer节点本地评估，这样这个peer节点就不需要和其它peer节点在这件事情上交互，但所有正确的peer节点都以相同的方式评估背书策略。

###3.3. 背书策略例子(Example endorsement policies)

The predicate may contain logical expressions and evaluates to TRUE or FALSE. Typically the condition will use digital signatures on the transaction invocation issued by endorsing peers for the chaincode.

断言可以包含逻辑表达式和评估真假。通常情况会对背书节点为链码发出的交易请求使用数字签名。

Suppose the chaincode specifies the endorser set E = {Alice, Bob, Charlie, Dave, Eve, Frank, George}. Some example policies:

假定链码指定背书者集E = {Alice, Bob, Charlie, Dave, Eve, Frank, George}.一些例子策略如下：

- A valid signature from on the same tran-proposal from all members of E.
- A valid signature from any single member of E.
- Valid signatures on the same tran-proposal from endorsing peers according to the condition (Alice OR Bob) AND (any two of: Charlie, Dave, Eve, Frank, George).

- 一个有效签名来自全体E的成员的同样的交易提案。
- 一个有效签名来自E的任一单个成员。
- 从背书peer节点来的同一交易提案的有效签名条件是：(Alice OR Bob) AND (any two of: Charlie, Dave, Eve, Frank, George).

- Valid signatures on the same tran-proposal by any 5 out of the 7 endorsers. (More generally, for chaincode with n > 3f endorsers, valid signatures by any 2f+1 out of the n endorsers, or by any group of more than (n+f)/2 endorsers.)

- 同一提案的有效签名为7名背书者的任意5名。（更常用的，链码n>3f背书者，n名背书者有任意2f+1有效签名，或任意大于(n+f)/2背书者小组有效签名）

- Suppose there is an assignment of “stake” or “weights” to the endorsers, like {Alice=49, Bob=15, Charlie=15, Dave=10, Eve=7, Frank=3, George=1}, where the total stake is 100: The policy requires valid signatures from a set that has a majority of the stake (i.e., a group with combined stake strictly more than 50), such as {Alice, X} with any X different from George, or {everyone together except Alice}. And so on.

- 假定背书者有一个“股份”或“权重”的任务，像{Alice=49, Bob=15, Charlie=15, Dave=10, Eve=7, Frank=3, George=1}, 其中全部股份是100：策略需要一组占大多数股份的有效签名（即，一组合并股份完全超过50），像{Alice, X}，X只要不是George的任何人，或{除去Alice以外的所有人}，等等。

- The assignment of stake in the previous example condition could be static (fixed in the metadata of the chaincode) or dynamic (e.g., dependent on the state of the chaincode and be modified during the execution).

- 假定前面例子中的股权条件是静态的（固定在链码的元数据中）或动态的（例如，取决于链码的状态和在执行中修改）。

- Valid signatures from (Alice OR Bob) on tran-proposal1 and valid signatures from (any two of: Charlie, Dave, Eve, Frank, George) on tran-proposal2, where tran-proposal1 and tran-proposal2 differ only in their endorsing peers and state updates.

- 交易提案1的有效签名来自(Alice OR Bob) 和交易提案2有效签名来自（Charlie, Dave, Eve, Frank, George中的任何两个），其中交易提案1和交易提案2的不同只在它们的背书peer节点和状态更新。

How useful these policies are will depend on the application, on the desired resilience of the solution against failures or misbehavior of endorsers, and on various other properties.

如何使用这些策略取决于应用、失败或恶意背书者的恢复能力和各种其它特性。

## 4 (post-v1). 证实账本和节点账本检查（修剪）(Validated ledger and PeerLedger checkpointing (pruning))

### 4.1. 验证账本（Validated ledger (VLedger)）

To maintain the abstraction of a ledger that contains only valid and committed transactions (that appears in Bitcoin, for example), peers may, in addition to state and Ledger, maintain the *Validated Ledger (or VLedger)*. This is a hash chain derived from the ledger by filtering out invalid transactions.

维护一个账本的抽象，只包含有效和提交交易（例如比特币的方案），peer节点可以，除状态和账本外，维护*证实账本（或VLedger）*。这是一个哈希链，来自过滤掉无效交易的账本。

The construction of the VLedger blocks (called here vBlocks) proceeds as follows. As the PeerLedger blocks may contain invalid transactions (i.e., transactions with invalid endorsement or with invalid version dependencies), such transactions are filtered out by peers before a transaction from a block becomes added to a vBlock. Every peer does this by itself (e.g., by using the bitmask associated with PeerLedger). A vBlock is defined as a block without the invalid transactions, that have been filtered out. Such vBlocks are inherently dynamic in size and may be empty. An illustration of vBlock construction is given in the figure below. 

证实账本块的生成按如下顺序。当节点账本块可能包含无效交易（即，交易的背书无效或版本依赖无效），这样的交易被peer节点在交易从块变为证实块之前过滤掉。每个peer节点自身实现这点（例如，使用节点账本关联的位掩码）。证实块被定义为没有无效交易的块，是进过过滤的块。这样证实块在大小上是动态的也可能是空的。证实块生成的说明在下图中给出。

![](img/blocks-3.png)

Figure 2. Illustration of validated ledger block (vBlock) formation from ledger (PeerLedger) blocks.

图2 从节点账本块形成证实账本块

vBlocks are chained together to a hash chain by every peer. More specifically, every block of a validated ledger contains:

证实块被每个peer节点链接在一起形成一个哈希链。更具体地，证实账本的每个块包含：

- The hash of the previous vBlock.
- vBlock number.
- An ordered list of all valid transactions committed by the peers since the last vBlock was computed (i.e., list of valid transactions in a corresponding block).
- The hash of the corresponding block (in PeerLedger) from which the current vBlock is derived.

- 前证实块的哈希。
- 证实块编号。
- 从上一个证实块被计算出以来所有peer节点提交交易的排序列表（即，在相应块中的有效交易列表）。
- 相应块的哈希（在节点账本中），来自得出的当前证实块。

All this information is concatenated and hashed by a peer, producing the hash of the vBlock in the validated ledger.

所有这些信息都被peer节点级联和哈希，产生证实账本中证实块的哈希。

### 4.2. 节点账本检查(`PeerLedger` Checkpointing)

The ledger contains invalid transactions, which may not necessarily be recorded forever. However, peers cannot simply discard PeerLedger blocks and thereby prune PeerLedger once they establish the corresponding vBlocks. Namely, in this case, if a new peer joins the network, other peers could not transfer the discarded blocks (pertaining to PeerLedger) to the joining peer, nor convince the joining peer of the validity of their vBlocks.


账本包含的无效交易，没有必要永久记录。然而，一旦建立相应的证实块，peer节点不能简单地丢弃节点账本块从而修剪节点账本。即，在这种情况下，如果新的peer节点加入了网络，其它peer节点不能转移丢弃块（与节点账本有关的）到新加入的节点，也不能使新加入的peer节点承认它们的证实块。

To facilitate pruning of the PeerLedger, this document describes a checkpointing mechanism. This mechanism establishes the validity of the vBlocks across the peer network and allows checkpointed vBlocks to replace the discarded PeerLedger blocks. This, in turn, reduces storage space, as there is no need to store invalid transactions. It also reduces the work to reconstruct the state for new peers that join the network (as they do not need to establish validity of individual transactions when reconstructing the state by replaying PeerLedger, but may simply replay the state updates contained in the validated ledger).

为了便于节点账本修剪，这个文档描述一个检查点机制。这个机制建立了证实块的有效性，贯穿节点网络，允许检查点证实块替换丢弃的节点账本块。这，反过来，减少了存储空间，因为没有必要存储无效交易。它也减少了新加入的peer节点重构状态的工作量（当通过重演节点账本重构状态时，因为他们不需要建立有效的单个交易，但可以简单重演包含在节点账本中的状态更新。）

#### 4.2.1. 检查点协议(Checkpointing protocol)

Checkpointing is performed periodically by the peers every CHK blocks, where CHK is a configurable parameter. To initiate a checkpoint, the peers broadcast (e.g., gossip) to other peers message <CHECKPOINT,blocknohash,blockno,stateHash,peerSig>, where blockno is the current blocknumber and blocknohash is its respective hash, stateHash is the hash of the latest state (produced by e.g., a Merkle hash) upon validation of block blockno and peerSig is peer’s signature on (CHECKPOINT,blocknohash,blockno,stateHash), referring to the validated ledger.

检查点是由peer节点每个CHK块周期性地形成，这里CHK是一个可配置参数。开辟一个检查点，peer节点广播（例如，传播）给其它peer节点 <CHECKPOINT,blocknohash,blockno,stateHash,peerSig>, 其中，blockno是当前块编号，blocknohash是各自的哈希，stateHash是最新状态的哈希（产生于，例如Merkle hash），基于确认的块编号，peerSig是peer节点的对(CHECKPOINT,blocknohash,blockno,stateHash)的签名，引用了证实账本。

A peer collects CHECKPOINT messages until it obtains enough correctly signed messages with matching blockno, blocknohash and stateHash to establish a valid checkpoint (see Section 4.2.2.).

peer节点收集CHECKPOINT消息直到它得到匹配blockno, blocknohash 和 stateHash 的足够正确的签名消息来建立一个有效的检查点。（见4.2.2节）

Upon establishing a valid checkpoint for block number blockno with blocknohash, a peer:
- if blockno>latestValidCheckpoint.blockno, then a peer assigns latestValidCheckpoint=(blocknohash,blockno),
- stores the set of respective peer signatures that constitute a valid checkpoint into the set latestValidCheckpointProof,
- stores the state corresponding to stateHash to latestValidCheckpointedState,
- (optionally) prunes its PeerLedger up to block number blockno (inclusive).

在为块编号blockno 和 blocknohash建立了有效的检查点的基础上，peer节点：
- 如果 blockno>latestValidCheckpoint.blockno, 那么peer节点分配 latestValidCheckpoint=(blocknohash,blockno),
- 存储各peer节点的签名集，它构成了有效的检查点到集合latestValidCheckpointProof,
- 存储状态相应的stateHash 到 latestValidCheckpointedState,
- （可选的）修剪它的节点账本到块编blockno (包含).

#### 4.2.2. 有效检查点(Valid checkpoints)

Clearly, the checkpointing protocol raises the following questions: When can a peer prune its ``PeerLedger``? How many ``CHECKPOINT`` messages are “sufficiently many”?. This is defined by a checkpoint validity policy, with (at least) two possible approaches, which may also be combined:

显然，检查点协议增加了下面的问题：peer节点什么时候能修剪它的节点账本？多少检查点消息是足够多的？这由检查点有效策略定义，要有（至少）两种可能的方法且也能合并：

- Local (peer-specific) checkpoint validity policy (LCVP). A local policy at a given peer p may specify a set of peers which peer p trusts and whose CHECKPOINT messages are sufficient to establish a valid checkpoint. For example, LCVP at peer Alice may define that Alice needs to receive CHECKPOINT message from Bob, or from both Charlie and Dave.

- Local (peer-specific) checkpoint validity policy (LCVP).给定peer节点p上的本地策略可以确定一组peer节点，这一组peer节点是p信任的且它的CHECKPOINT消息是足够建立一个有效的检查点。例如，在peer节点Alice上的LCVP可以定义本地（peer确定）检查点有效性策略（LCVP）。

- Global checkpoint validity policy (GCVP). A checkpoint validity policy may be specified globally. This is similar to a local peer policy, except that it is stipulated at the system (blockchain) granularity, rather than peer granularity. For instance, GCVP may specify that:

- Global checkpoint validity policy (GCVP).检查点有效策略可以确定为全局的。这类似于本地节点策略，除非在系统链间隔上规定，好于节点间隔。例如，GCVP可以指定：

  - each peer may trust a checkpoint if confirmed by 11 different peers.
  - in a specific deployment in which every orderer is collocated with a peer in the same machine (i.e., trust domain) and where up to f orderers may be (Byzantine) faulty, each peer may trust a checkpoint if confirmed by f+1 different peers collocated with orderers.

  - 每个peer节点可以信任一个由11各不同peer节点确认的检查点。
  - 在具体部署中每个排序者与peer节点配置在同一台机器上（即，信任域），多达f个排序者可以是（拜占庭）错误，每个peer节点可以信任一个检查点，如果经过f+1个排序者配置的不同的节点确认。
