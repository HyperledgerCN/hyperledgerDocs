
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/Fabric-FAQ.html) | Shaoxi Qiu |  |

# Hyperledger Fabric FAQ<br />Hyperledger Fabric 答疑


## Endorsement<br />背书

**Endorsement architecture**:

**背书操作架构**:

Q. How many peers in the network need to endorse a transaction?

问题：多少个网络中的节点需要对交易进行背书？

A. The number of peers required to endorse a transaction is driven by the endorsement
policy that is specified at chaincode deployment time.

答案：需要对交易进行背书操作的节点数量由背书策略决定，背书策略在链码部署的时候指定。

Q. Does an application client need to connect to all peers?

问题：应用客户端需要连接到所有节点吗？

A. Clients only need to connect to as many peers as are required by the
endorsement policy for the chaincode.

答案：客户端只需要连接到链码对应的背书策略所要求的足够多的节点。

## Security & Access Control<br />安全和权限控制

**Data Privacy and Access Control**:

**数据隐私和权限控制**:


Q. How do I ensure data privacy?

问题：我如何能够保证数据的隐私？

A. There are various aspects to data privacy.
First, you can segregate your network into channels, where each channel
represents a subset of participants that are authorized to see the data
for the chaincodes that are deployed to that channel.
Second, within a channel you can restrict the input data to chaincode to the
set of endorsers only, by using visibility settings. The visibility setting
will determine whether input and output chaincode data is included in the
submitted transaction,  versus just output data.
Third, you can hash or encrypt the data before calling chaincode. If you hash
the data then you will need to provide a means to share the source data.
If you encrypt the data then you will need to provide a means to share the
decryption keys.
Fourth, you can restrict data access to certain roles in your organization, by
building access control into the chaincode logic.
Fifth, ledger data at rest can be encrypted via file system encryption on
the peer, and data in-transit is encrypted via TLS.

答案：数据隐私包括几个方面。

第一、你可以将你的网络拆分为多个通道，每个通道代表参与者的子集，这些参与者有权访问该通道所部署的链码所包含的数据。

第二、通过通道你可以限制链码的输入数据只对背书节点可见。可见性设置将决定链码的输入输出数据是否包含在提交交易中，可以只包含输出数据。

第三、你能在调用链码前对数据进行哈希操作或者加密。如果对数据进行哈希操作，你需要提供获取原数据的方式。如果对数据进行加密，你需要提供获取密钥的方式。

第四、你可以通过链码中的访问控制逻辑限制数据访问权限，只有机构中特定的角色可以访问。

第五、节点中的账本数据可以通过文件系统进行加密，节点间数据传输通过TLS进行加密。

Q. Do the orderers see the transaction data?

问题：排序节点能否看到交易数据？

A. No, the orderers only order transactions, they do not open the transactions.
If you do not want the data to go through the orderers at all, and you are only
concerned about the input data, then you can use visibility settings. The
visibility setting will determine whether input and output chaincode data is
included in the submitted transaction,  versus just output data. Therefore,
the input data can be private to the endorsers only.
If you do not want the orderers to see chaincode output, then you can hash or
encrypt the data before calling chaincode. If you hash the data then you will
need to provide a meansto share the source data. If you encrypt the data then
you will need to provide a means to share the decryption keys.

答案：不能，排序节点只是排序交易，不能解析交易。如果你只关心输入数据，不希望经过排序节点，你可以使用可见性设置。可见性设置将决定链码的输入输出数据是否包含在提交交易中，可以只包含输出数据，这样只有背书节点能看到输入数据。如果你不希望排序节点看到交易输出数据，你能在调用链码前对数据进行哈希操作或者加密。如果对数据进行哈希操作，你需要提供获取原数据的方式。如果对数据进行加密，你需要提供获取密钥的方式。

## Application-side Programming Model<br />应用端编程模型

**Transaction execution result**:

**交易执行结果**:

Q. How do application clients know the outcome of a transaction?

问题：应用客户端如何知道交易的输出？

A. The transaction simulation results are returned to the client by the
endorser in the proposal response.  If there are multiple endorsers, the
client can check that the responses are all the same, and submit the results
and endorsements for ordering and commitment. Ultimately the committing peers
will validate or invalidate the transaction, and the client becomes
aware of the outcome via an event, that the SDK makes available to the
application client.

答案：背书节点会返回提案模拟交易的结果。如果有多个背书节点，客户端能检查返回结果是否相同后再提交交易结果以及排序和生效所需要的背书。最终，生效节点会验证交易是否合法，客户端通过SDK提供的事件通知方式接收交易结果。

**Ledger queries**:

**账本查询**:

Q. How do I query the ledger data?

问题：如何查询账本数据？

A. Within chaincode you can query based on keys. Keys can be queried by range,
and composite keys can be modeled to enable equivalence queries against multiple
parameters. For example a composite key of (owner,asset_id) can be used to
query all assets owned by a certain entity. These key-based queries can be used
for read-only queries against the ledger, as well as in transactions that
update the ledger.

If you model asset data as JSON in chaincode and use CouchDB as the state
database, you can also perform complex rich queries against the chaincode
data values, using the CouchDB JSON query language within chaincode. The
application client can perform read-only queries, but these responses are
not typically submitted as part of transactions to the ordering service.

答案：通过链码可以通过键值查询数据。可以对键值进行范围查询，复合主键可以允许查询出部分键值相同的值，例如由 (owner,asset_id) 组成的复合主键可以用来查询特定实体所拥有的所有asset。

Q. How do I query the historical data to understand data provenance?

问题：如何查询数据历史来理解数据出处？

A. The chaincode API ``GetHistoryForKey()`` will return history of
values for a key.

答案： 链码中API  ``GetHistoryForKey()`` 会返回键值对应的数据历史。

Q. How to guarantee the query result is correct, especially when the peer being
queried may be recovering and catching up on block processing?

问题：如何保证查询结果是正确的，特别是当查询的节点可能正处在恢复和获取最新区块的过程中？

A. The client can query multiple peers, compare their block heights, compare
their query results, and favor the peers at the higher block heights.

答案：客户端可以查询多个节点，比较他们的区块高度查询结果，选用区块高度最高的结果。

## Chaincode (Smart Contracts and Digital Assets)<br />链码（智能合约和数字资产）

Q. Does Hyperledger Fabric support smart contract logic?

问题：Hyperledger Fabric 是否支持智能合约逻辑？

A. Yes. We call this feature :ref:`chaincode`. It is our interpretation of the
smart contract method/algorithm, with additional features.

A chaincode is programmatic code deployed on the network, where it is
executed and validated by chain validators together during the consensus
process. Developers can use chaincodes to develop business contracts,
asset definitions, and collectively-managed decentralized applications.

答案：是的，我们称这个功能为 :ref:`chaincode`. 这是我们对带有附加功能的智能合同方法/算法的理解。

链码是部署在网络中的程序代码，通过共识流程被链的验证者共同执行并验证。开发者能使用链码来开发商业合同、资产定义和集体管理的去中心化应用。

Q. How do I create a business contract?

问题：如何创建商业合同？

A. There are generally two ways to develop business contracts: the first way is
to code individual contracts into standalone instances of chaincode; the
second way, and probably the more efficient way, is to use chaincode to
create decentralized applications that manage the life cycle of one or
multiple types of business contracts, and let end users instantiate
instances of contracts within these applications.

答案：通常有两种方法来开发商业合同：第一个方法是开发单独的合同在独立的链码实例;第二个也是最有效的方法是使用链码创建去中心化的应用来管理一种或多种类型的商业合同的生命周期，并让终端用户实例化带有这些应用的合同实例。

Q. How do I create assets?

如何创建资产？

A. Users can use chaincode (for business rules) and membership service (for digital tokens) to
design assets, as well as the logic that manages them.

There are two popular approaches to defining assets in most blockchain
solutions: the stateless UTXO model, where account balances are encoded
into past transaction records; and the account model, where account
balances are kept in state storage space on the ledger.

Each approach carries its own benefits and drawbacks. This blockchain
technology does not advocate either one over the other. Instead, one of our
first requirements was to ensure that both approaches can be easily
implemented.

答案：用户可以使用链码（在商业规则方面）和成员管理服务（在数字密钥方面）来设计资产和资产管理逻辑。

区块链解决方案中有两种流行的方法来设计资产：无状态的UTXO模型，账户余额是过去交易记录中没有使用的交易输出的合计;另一种是账户模型，账户余额被存储在账本中的状态值中。

每一个方法都有好处和弊端。本区块链技术对两种方法没有倾向。我们最初的要求就是确保两个方法都能够被容易实现。

Q. Which languages are supported for writing chaincode?

问题：链码支持哪些语言？

A. Chaincode can be written in any programming language and executed in
containers.  The first fully supported chaincode language is Golang.

Support for additional languages and the development of a templating language
have been discussed, and more details will be released in the near future.

It is also possible to build Hyperledger Fabric applications using
[Hyperledger Composer](<https://hyperledger.github.io/composer/>).

答案：链码可以使用任何编程语言编写并在容器内执行。第一个全功能支持的语言是Golang。

对其他语言的支持和模板语言的开发正在被讨论，更多细节会在近期发布。

还可以通过[Hyperledger Composer](<https://hyperledger.github.io/composer/>)来构建Hyperledger Fabric应用。

Q. Does the Hyperledger Fabric have native currency?

问题：Hyperledger Fabric是否有原生的货币？

A. No. However, if you really need a native currency for your chain network,
you can develop your own native currency with chaincode. One common attribute
of native currency is that some amount will get transacted (the chaincode
defining that currency will get called) every time a transaction is processed
on its chain.

答案：没有。然而，如果你的区块链网络确实需要原生的货币，你可以使用链码来开发货币。原生货币的一个基本的属性是一些数量会被转移（定义货币的链码会被调用）当每次链上交易被执行。

## Differences in Most Recent Releases<br />最近版本的区别

Q. As part of the v1.0.0 release, what are the highlight differences between v0.6 and v1.0?

问题：V0.6版本和V1.0版本的最大区别是什么？

A. The differences between any subsequent releases are provided together with the
[Release Notes](<http://hyperledger-fabric.readthedocs.io/en/latest/releases.html>).
Since Fabric is a pluggable modular framework, you can refer to the [design-docs](<https://wiki.hyperleger.org/projects/fabric/design-docs>) for further information of these difference.

答案：任何后续版本的差异都在[版本说明](<http://hyperledger-fabric.readthedocs.io/en/latest/releases.html>)中展现。因为Fabric是一个模块可替换的框架，你可以参考[设计文档](<https://wiki.hyperleger.org/projects/fabric/design-docs>)获取更多差异信息。

Q. Where to get help for the technical questions not answered above?

问题：哪里可以获得上文中没被解答的技术问题的帮助？

A. Please use [StackOverflow](<https://stackoverflow.com/questions/tagged/hyperledger>).

答案：请使用 [StackOverflow](<https://stackoverflow.com/questions/tagged/hyperledger>).


.. Licensed under Creative Commons Attribution 4.0 International License

.. 本文通过Creative Commons Attribution 4.0 International License协议进行授权

   https://creativecommons.org/licenses/by/4.0/