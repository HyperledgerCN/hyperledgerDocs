
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/channels.html) | Yi Zeng |  |

A Hyperledger Fabric channel is a private “subnet” of communication between two or more specific network members, for the purpose of conducting private and confidential transactions. A channel is defined by members (organizations), anchor peers per member, the shared ledger, chaincode application(s) and the ordering service node(s). Each transaction on the network is executed on a channel, where each party must be authenticated and authorized to transact on that channel. Each peer that joins a channel, has its own identity given by a membership services provider (MSP), which authenticates each peer to its channel peers and services.

在超级账本Fabric中，一个通道是指一个在两个或多个特定网络成员间的专门为私人的和机密的交易为目的而建立的私有"子网"。一个通道的定义中包含：成员（组织），每个成员的锚节点，共享帐本，链上代码应用程序和排序服务节点。网络上的每个交易都在一个指定的通道中执行，在通道中交易必须通过通道中的每部分的认证和授权。要加入一个通道的每个节点都必须有自己的通过成员服务提供商（MSP）获得的身份标识，用于鉴定每个节点在通道中的是什么节点和服务。

To create a new channel, the client SDK calls configuration system chaincode and references properties such as anchor peer**s, and members (organizations). This request creates a **genesis block for the channel ledger, which stores configuration information about the channel policies, members and anchor peers. When adding a new member to an existing channel, either this genesis block, or if applicable, a more recent reconfiguration block, is shared with the new member.

要创建一个通道，客户端SDK调用配置系统链上代码和参考属性，比如锚节点和成员（组织）。这个请求会为通道的账本创建一个创世区块，用于存储关于通道的策略，成员和锚节点的配置信息。当需要添加一个新成员到现有通道时，这个创世区块，或者最新的新配置区块（如果可用），将会共享给这个新成员。

Note
注意

See the Channel Configuration (configtx) section for more more details on the properties and proto structures of config transactions.

参考通道配置（configtx）章节，可以查看更多关于交易的配置属性和典型的结构的明细信息。

The election of a leading peer for each member on a channel determines which peer communicates with the ordering service on behalf of the member. If no leader is identified, an algorithm can be used to identify the leader. The consensus service orders transactions and delivers them, in a block, to each leading peer, which then distributes the block to its member peers, and across the channel, using the gossip protocol.

从通道的所有节点中选举出的领导节点决定哪个节点用于代表其他成员节点与排序服务通讯。如果还没有领导节点，那么一个算法可以用于标识出领导节点。共识服务对交易进行排序，并打包成区块，发送区块给每个领导节点，然后领导节点把区块分发给其成员节点，然后使用gossip协议穿过通道。

Although any one anchor peer can belong to multiple channels, and therefore maintain multiple ledgers, no ledger data can pass from one channel to another. This separation of ledgers, by channel, is defined and implemented by configuration chaincode, the identity membership service and the gossip data dissemination protocol. The dissemination of data, which includes information on transactions, ledger state and channel membership, is restricted to peers with verifiable membership on the channel. This isolation of peers and ledger data, by channel, allows network members that require private and confidential transactions to coexist with business competitors and other restricted members, on the same blockchain network.

虽然任意一个锚节点都可以属于多个通道，而且维护了多个账本，但是不会有任何账本数据会从一个通道传到另一个通道。这就是根据通道对账本的分离，这种分离是在配置链上代码，成员标识服务和gossip传播协议中定义和实现。数据的传播，包括交易的信息，账本状态和通道成员等都在通道内受限制的验证成员身份的节点之间。这种根据通道对节点和账本数据进行隔离，允许网络成员可以在同一个区块链网络中请求私有的和保密的交易给业务上的竞争对手和其他受限的成员。