
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/ledger.html) | Yuan Jin | Linsheng Yu |

## Ledger - 账本

The ledger is the sequenced, tamper-resistant record of all state transitions in the fabric. State transitions are a result of chaincode invocations (‘transactions’) submitted by participating parties. Each transaction results in a set of asset key-value pairs that are committed to the ledger as creates, updates, or deletes.

账本是Fabric中所有状态转换的记录，具有有序和防篡改的特点。状态转换是参与各方提交链代码调用（交易）产生的结果。每个交易会产生一组资产键值对，这些键值对作为“创建”、“更新”或者“删除”提交给账本。

The ledger is comprised of a blockchain (‘chain’) to store the immutable, sequenced record in blocks, as well as a state database to maintain current fabric state. There is one ledger per channel. Each peer maintains a copy of the ledger for each channel of which they are a member.

账本由一个区块链（链）构成，并将不可变的、有序的记录存放在区块中；同时包含一个状态数据库来记录当前的Fabric状态。每个通道中各有一个账本。各个节点对于它所属的每个通道，都会保存一份该通道的账本副本。

## Chain - 链

The chain is a transaction log, structured as hash-linked blocks, where each block contains a sequence of N transactions. The block header includes a hash of the block’s transactions, as well as a hash of the prior block’s header. In this way, all transactions on the ledger are sequenced and cryptographically linked together. In other words, it is not possible to tamper with the ledger data, without breaking the hash links. The hash of the latest block represents every transaction that has come before, making it possible to ensure that all peers are in a consistent and trusted state.

链是一个交易日志，它由哈希值链接的区块构造而成，每个区块包含N个有序的交易。块头中包含了本区块所记录交易的哈希值，以及上一个区块头的哈希值。通过这种方式，账本中的所有交易都被有序的、加密的形式串联在了一起。换言之，如果不破坏哈希链的话，是无法篡改账本数据的。最新区块的哈希是之前每一笔交易的体现，从而可以保证所有的节点处于一致的可信任的状态。

The chain is stored on the peer file system (either local or attached storage), efficiently supporting the append-only nature of the blockchain workload.

链被存放于节点的文件系统中（本地的或者挂载的），有效地支持着区块链工作量只追加的特性。

## State Database - 状态数据库

The ledger’s current state data represents the latest values for all keys ever included in the chain transaction log. Since current state represents all latest key values known to the channel, it is sometimes referred to as World State.

账本的当前状态信息呈现的是链交易日志中记录过的所有键的最新值。由于当前状态表示的是通道已知的所有键的最新值，由此也常被称作世界状态。

Chaincode invocations execute transactions against the current state data. To make these chaincode interactions extremely efficient, the latest values of all keys are stored in a state database. The state database is simply an indexed view into the chain’s transaction log, it can therefore be regenerated from the chain at any time. The state database will automatically get recovered (or generated if needed) upon peer startup, before transactions are accepted.

链码调用基于当前的状态数据执行交易。为了使链码调用高效运行，所有键的最新值被存储在状态数据库中。状态数据库是链的交易日志的索引视图，因此它可以随时从链中重新导出。节点启动的时候，在接受交易之前，状态数据库将被自动恢复（或者根据需要产生）。

## Transaction Flow - 交易流程

At a high level, the transaction flow consists of a transaction proposal sent by an application client to specific endorsing peers. The endorsing peers verify the client signature, and execute a chaincode function to simulate the transaction. The output is the chaincode results, a set of key/value versions that were read in the chaincode (read set), and the set of keys/values that were written in chaincode (write set). The proposal response gets sent back to the client along with an endorsement signature.

概括而言，交易流程由应用客户端发送给背书节点交易提案组成。背书节点验证客户端的签名，然后执行链码来模拟交易。产生的输出就是链码结果，一组链码读取的键值版本（读集合），和一组被写入链码的键值集合（写集合）。交易提案的响应被发送回客户端，同时包含了背书签名。

The client assembles the endorsements into a transaction payload and broadcasts it to an ordering service. The ordering service delivers ordered transactions as blocks to all peers on a channel.

客户端汇总所有的背书到一个交易有效载荷中，并将它广播到排序服务。排序服务将排好序的交易放入区块并发送到通道内的所有节点。

Before committal, peers will validate the transactions. First, they will check the endorsement policy to ensure that the correct allotment of the specified peers have signed the results, and they will authenticate the signatures against the transaction payload.

在提交之前，节点们会验证交易。首先它们会检查背书策略来保证足够的指定节点正确地对结果进行了签名，并且会认证交易有效载荷中的签名。

Secondly, peers will perform a versioning check against the transaction read set, to ensure data integrity and protect against threats such as double-spending. The fabric has concurrency control whereby transactions execute in parallel (by endorsers) to increase throughput, and upon commit (by all peers) each transaction is verified to ensure that no other transaction has modified data it has read. In other words, it ensures that the data that was read during chaincode execution has not changed since execution (endorsement) time, and therefore the execution results are still valid and can be committed to the ledger state database. If the data that was read has been changed by another transaction, then the transaction in the block is marked as invalid and is not applied to the ledger state database. The client application is alerted, and can handle the error or retry as appropriate.

其次，节点们会对交易的读集合进行版本检查，从而保证数据的一致性并防范一些攻击，比如双花。Fabric拥有并发控制，从而交易可以（被背书节点）并行运行来提高吞吐量，而且当交易（被节点）提交时，每个交易都会被验证来保证它所读取的数据没有被其他交易更改。换言之，它确保链码执行期间所读取的数据从执行（背书）开始后没有变动。如果读取的数据被其他交易改动了，那么区块中的交易将被标记成无效的，也不会被应用到账本状态数据库。客户端应用会收到提醒，从而进行纠错或适当重试。

See the [Transaction Flow](http://hyperledger-fabric.readthedocs.io/en/latest/txflow.html) and [Read-Write set semantics](http://hyperledger-fabric.readthedocs.io/en/latest/readwrite.html) topics for a deeper dive on transaction structure, concurrency control, and the state DB.

要进一步了解交易的结构，并发控制和状态数据库的相关内容，可以参考[交易流程](http://hyperledger-fabric.readthedocs.io/en/latest/txflow.html)和[读写集合语言学](http://hyperledger-fabric.readthedocs.io/en/latest/readwrite.html)。

## State Database options - 状态数据库选项

State database options include LevelDB and CouchDB (beta). LevelDB is the default key/value state database embedded in the peer process. CouchDB is an optional alternative external state database. Like the LevelDB key/value store, CouchDB can store any binary data that is modeled in chaincode (CouchDB attachment functionality is used internally for non-JSON binary data). But as a JSON document store, CouchDB additionally enables rich query against the chaincode data, when chaincode values (e.g. assets) are modeled as JSON data.

状态数据库选项包括LevelDB和CouchDB(beta)。LevelDB是节点流程中集成的缺省键值状态数据库。CouchDB是可选的外部状态数据库。类似于LevelDB的键值库，CouchDB能存储任何链码中建模的二进制数据（CouchDB附件功能被内部用于非JSON格式的二进制数据）。但作为一个JSON格式文档库，当链码的数据（比如资产）以JSON格式建模时，CouchDB额外提供了许多针对链码数据的查询方式。

Both LevelDB and CouchDB support core chaincode operations such as getting and setting a key (asset), and querying based on keys. Keys can be queried by range, and composite keys can be modeled to enable equivalence queries against multiple parameters. For example a composite key of (owner,asset_id) can be used to query all assets owned by a certain entity. These key-based queries can be used for read-only queries against the ledger, as well as in transactions that update the ledger.

LevelDB和CouchDB都支持核心的链码操作，比如获取和设置一个键（资产），以及基于键进行查询等。键的查询可以通过设置范围，而且可以通过构建组合键来达到按多个参数进行查询的同等效果。比如一个组合键（拥有者，资产编号）可以被用来查询某实体所拥有的所有资产。这些基于键的查询可以被用做针对账本的只读查询，同时也可以被应用在对账本进行更新的交易中。

If you model assets as JSON and use CouchDB, you can also perform complex rich queries against the chaincode data values, using the CouchDB JSON query language within chaincode. These types of queries are excellent for understanding what is on the ledger. Proposal responses for these types of queries are typically useful to the client application, but are not typically submitted as transactions to the ordering service. In fact the fabric does not guarantee the result set is stable between chaincode execution and commit time for rich queries, and therefore rich queries are not appropriate for use in update transactions, unless your application can guarantee the result set is stable between chaincode execution time and commit time, or can handle potential changes in subsequent transactions. For example, if you perform a rich query for all assets owned by Alice and transfer them to Bob, a new asset may be assigned to Alice by another transaction between chaincode execution time and commit time, and you would miss this ‘phantom’ item.

如果你将资产以JSON格式进行建模，并且使用的是CouchDB，那你可以通过CouchDB的JSON查询语言，对链码的数据值进行复杂多样的查询。这些查询类型可以很好的帮助理解账本中包含什么。查询类型的提案响应对客户端应用通常很有用，但并不会被作为交易提交到排序服务。实际上对于富查询（rich query），Fabric并不保证结果集在链码执行和提交过程中间是稳定的，或者能处理后续交易中潜在的变化。比如说，如果你对所有Alice拥有的资产进行富查询，并转移给Bob，那在链码执行和提交的过程中，可能会有另一个交易将一个新的资产分配给了Alice，你将会错失这个“幻影”项。

CouchDB runs as a separate database process alongside the peer, therefore there are additional considerations in terms of setup, management, and operations. You may consider starting with the default embedded LevelDB, and move to CouchDB if you require the additional complex rich queries. It is a good practice to model chaincode asset data as JSON, so that you have the option to perform complex rich queries if needed in the future.

CouchDB作为独立的数据库进程跟节点一起运行，所以安装、管理和操作的时候需要一些额外的考虑。你可以尝试开始的时候用缺省集成的LevelDB，然后当你需要额外的复杂查询时再切换到CouchDB。将链码的资产数据以JSON格式建模是一个非常好的实践，这样有利于你将来进行复杂多样的查询。

To enable CouchDB as the state database, configure the /fabric/sampleconfig/core.yaml stateDatabase section.

如果要用CouchDB作为状态数据库，需要对/fabric/sampleconfig/core.yaml stateDatabase这部分进行配置。

