
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/gossip.html) | Xiao Zhang |  |


Hyperledger Fabric optimizes blockchain network performance, security and scalability by dividing workload across transaction execution (endorsing and committing) peers and transaction ordering nodes. This decoupling of network operations requires a secure, reliable and scalable data dissemination protocol to ensure data integrity and consistency. To meet these requirements, the fabric implements a **gossip data dissemination protocol**.

HyperLedger Fabric通过把工作节点分解为执行交易（背书和提交）节点和交易排序节点来优化区块链网络性能，安全性和可扩展性。这种解耦网络操作的方式需要一个安全、可靠、可扩展的数据分发协议来保证数据的完整性和一致性。为了满足这些要求，Fabric应用了**Gossip数据分发协议**。

## Gossip协议(Gossip protocol)

Peers leverage gossip to broadcast ledger and channel data in a scalable fashion. Gossip messaging is continuous, and each peer on a channel is constantly receiving current and consistent ledger data, from multiple peers. Each gossiped message is signed, thereby allowing Byzantine participants sending faked messages to be easily identified and the distribution of the message(s) to unwanted targets to be prevented. Peers affected by delays, network partitions or other causations resulting in missed blocks, will eventually be synced up to the current ledger state by contacting peers in possession of these missing blocks.

节点利用Gossip来以一种可扩展的方式广播账本和通道数据。Gossip出来消息是连续的，并且通道上的每个节点都在不断地接收当前来自多个节点的账本中已达成一致性的数据。每个通过Gossip传输的消息都会被签名，因此由拜占庭节点发送的伪造的消息将会很容易地被识别出来，而且可以防止将消息分发到不希望发送的目标处。节点因为受到延迟、网络分区或者其他原因的影响导致缺少部分区块的情况，最终将通过联系已拥有这些缺失的区块的节点的方式，与当前账本状态进行同步。

The gossip-based data dissemination protocol performs three primary functions on a Fabric network:
1.	Manages peer discovery and channel membership, by continually identifying available member peers, and eventually detecting peers that have gone offline.
2.	Disseminates ledger data across all peers on a channel. Any peer with data that is out of sync with the rest of the channel identifies the missing blocks and syncs itself by copying the correct data.
3.	Bring newly connected peers up to speed by allowing peer-to-peer state transfer update of ledger data.

基于Gossip的数据传播协议在Fabric网络上执行三个主要功能：
1.	通过不断识别可用的成员节点并最终监测节点离线状态的方式，对节点的发现和通道中的成员进行管理。
2.	通过通道中的所有节点来分发账本数据。任何数据未同步的节点都可以通过通道中其他节点来标识缺失的区块，并通过复制正确的数据来进行同步。
3.	通过允许点对点状态传输更新账本数据，使新加入连接的节点快速得到同步。

Gossip-based broadcasting operates by peers receiving messages from other peers on the channel, and then forwarding these messages to a number of randomly-selected peers on the channel, where this number is a configurable constant. Peers can also exercise a pull mechanism, rather than waiting for delivery of a message. This cycle repeats, with the result of channel membership, ledger and state information continually being kept current and in sync. For dissemination of new blocks, the **leader** peer on the channel pulls the data from the ordering service and initiates gossip dissemination to peers.

基于Gossip的广播由节点接收来自该通道中的其他节点的消息，然后将这些消息转发到通道上的多个随机选择的节点。这个节点数是个可配置的常数。节点也可以主动拉取消息，而不是等待消息发送。循环重复这个操作，使通道中成员的账本和状态信息不断保持和当前最新状态同步。为了传播新区块，通道中的**领导者**节点从排序服务中拉取数据，并向其他节点发送Gossip消息。

## Gossip消息传输(Gossip messaging)

Online peers indicate their availability by continually broadcasting “alive” messages, with each containing the **public key infrastructure (PKI)** ID and the signature of the sender over the message. Peers maintain channel membership by collecting these alive messages; if no peer receives an alive message from a specific peer, this “dead” peer is eventually purged from channel membership. Because “alive” messages are cryptographically signed, malicious peers can never impersonate other peers, as they lack a signing key authorized by a root certificate authority (CA).

在线的节点通过持续地广播“活跃”消息来表明他们的可用性，每条消息都包含**公钥基础设施（PKI）**的ID和消息发送者对消息的签名。节点通过收集这些活跃消息来维护通道成员身份。如果没有节点能从某个特定的节点收到活跃消息，那么这个“死亡”的节点最终将从通道成员身份列表中被删除。由于“活跃”信息是通过密码学算法进行签名的，因此恶意节点无法伪装成其他节点，因为他们缺少根证书颁发机构（CA）授权的签名密钥。

In addition to the automatic forwarding of received messages, a state reconciliation process synchronizes **world state** across peers on each channel. Each peer continually pulls blocks from other peers on the channel, in order to repair its own state if discrepancies are identified. Because fixed connectivity is not required to maintain gossip-based data dissemination, the process reliably provides data consistency and integrity to the shared ledger, including tolerance for node crashes.

除了将接收到的消息的自动转发之外，状态协程还会在每个通道上同步节点间的**世界状态**。每个节点不停地从通道中的其他节点中提取区块，以便在出现差异时修正自己的状态。由于不需要固定连接来维护基于Gossip的数据传播，因此该流程可以可靠地为共享账本保证数据的一致性和完整性，包括对节点崩溃的容错。

Because channels are segregated, peers on one channel cannot message or share information on any other channel. Though any peer can belong to multiple channels, partitioned messaging prevents blocks from being disseminated to peers that are not in the channel by applying message routing policies based on peers’ channel subscriptions.

由于通道之间相互隔离，一个通道上的节点不能在其他任何通道上发送或共享信息。尽管任何节点都可能属于多个通道，但是通过将基于节点通道订阅的机制作为消息分发策略，节点无法将被分隔开的消息传播给不在通道中的节点。

**Notes:**
1. Security of point-to-point messages are handled by the peer TLS layer, and do not require signatures. Peers are authenticated by their certificates, which are assigned by a CA. Although TLS certs are also used, it is the peer certificates that are authenticated in the gossip layer. Ledger blocks are signed by the ordering service, and then delivered to the leader peers on a channel. 2. Authentication is governed by the membership service provider for the peer. When the peer connects to the channel for the first time, the TLS session binds with fabric membership identity. This essentially authenticates each peer to the connecting peer, with respect to membership in the network and channel.


**注意：**
1.	点对点消息的安全性由节点的TLS层处理，不需要签名。节点通过其由CA分配的证书进行身份验证。节点在Gossip层的身份认证会通过TLS证书体现。账本中的区块由排序服务进行签名，然后传递给通道中的领导者节点。
2.	认证过程由节点的成员管理服务的提供者进行管理。当节点第一次连接到通道中的时候，TLS会话将与Fabric成员身份绑定。这样本质上使每个节点与相连的节点进行认证，从而与网络和通道中的成员身份关联起来。