
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/kafka.html) | Shaoxi Qiu |  |


须知（Caveat emptor）
-------------

This document assumes that the reader generally knows how to set up a Kafka
cluster and a ZooKeeper ensemble. The purpose of this guide is to identify the
steps you need to take so as to have a set of Hyperledger Fabric ordering
service nodes (OSNs) use your Kafka cluster and provide an ordering service to
your blockchain network.

该文档假设读者已经基本了解如何去搭建Kafka集群和ZooKeeper集群。本文档的目的是确定您使用Kafka集群搭建一套Hyperledger Fabric排序服务节点集(OSNs)以及为你的区块链网络提供排序服务所需要采取的步骤。


概览（Big picture）
-----------

Each channel maps to a separate single-partition topic in Kafka. 

每一个通道(channel)在Kafka中被映射到一个单独的单分区(partition)类别(topic)。(译者注：通常每个Topic包含一个或多个Partition，此处每个Topic只包含一个Partition)

When an OSN receives transactions via the ``Broadcast`` RPC, it checks to make sure
that the broadcasting client has permissions to write on the channel, then
relays (i.e. produces) those transactions to the appropriate partition in Kafka.

当排序节点通过RPC``广播``(``Broadcast``)接收到交易时，它会检查广播交易的客户端是否有权限去修改通道(channel)数据，然后反馈（即产生）这些交易到Kafka的适当分区(partition)中。

This partition is also consumed by the OSN which groups the received
transactions into blocks locally, persists them in its local ledger, and serves
them to receiving clients via the ``Deliver`` RPC. 

该分区也被排序节点所消费(consume)，排序节点将接收到的交易分组写入到本地区块，将其保留在本地账本中，并通过``Deliver`` RPC提供给需要接收的客户端。

For low-level details, refer
to `the document that describes how we came to this design
<https://docs.google.com/document/d/1vNMaM7XhOlu9tB_10dKnlrhy5d7b1u8lSY8a-kVjCO4/edit>`_
-- Figure 8 is a schematic representation of the process described above.

更多详细的信息，请参考`the document that describes how we came to this design
<https://docs.google.com/document/d/1vNMaM7XhOlu9tB_10dKnlrhy5d7b1u8lSY8a-kVjCO4/edit>`_
-- 图8是上述过程的示意图。


步骤（Steps）
-----

Let ``K`` and ``Z`` be the number of nodes in the Kafka cluster and the
ZooKeeper ensemble respectively:

设定变量 ``K`` 和 ``Z`` 分别是Kafka集群和ZooKeeper集群的节点数量：

i. At a minimum, ``K`` should be set to 4. (As we will explain in Step 4 below,
this is the minimum number of nodes necessary in order to exhibit crash fault
tolerance, i.e. with 4 brokers, you can have 1 broker go down, all channels will
continue to be writeable and readable, and new channels can be created.)

i. ``K``的最小值需要是4。(我们将在步骤4中解释，这是实现 故障容错(crash fault tolerance) 所需要的最小数值，也就是说，
4个节点可以容许1个节点宕机，所有的通道能够继续读写且可以创建通道。)(译者：Kafka节点被称为broker)

ii. ``Z`` will either be 3, 5, or 7. It has to be an odd number to avoid
split-brain scenarios, and larger than 1 in order to avoid single point of
failures. Anything beyond 7 ZooKeeper servers is considered an overkill.

ii. ``Z``可以是3、5或者7。它必须是一个奇数来避免分裂(split-brain)情景，大于1以避免单点故障。
超过7个ZooKeeper服务器则被认为是多余的。

Proceed as follows:

请按照以下步骤进行:

1. Orderers: **Encode the Kafka-related information in the network's genesis
block.** If you are using ``configtxgen``, edit ``configtx.yaml`` -- or pick a
preset profile for the system channel's genesis block --  so that:

 Orderers: **Kafka 相关信息被写在网络的初始区块中.** 如果你使用 ``configtxgen`` 工具, 编辑 ``configtx.yaml`` 文件-- 或者挑一个现成的系统通道的初始区块配置文件 --  其中:

        a. ``Orderer.OrdererType`` is set to ``kafka``.
        
        a. ``Orderer.OrdererType`` 字段被设置为 ``kafka``.

        b. ``Orderer.Kafka.Brokers`` contains the address of *at least two* of the
        Kafka brokers in your cluster in ``IP:port`` notation. The list does not
        need to be exhaustive. (These are your seed brokers.)
        
        b. ``Orderer.Kafka.Brokers`` 字段包含 *至少两个* Kafka集群中的节点``IP:port`` 样式的地址。这个列表没有必要详尽无遗(这些是你的 seed brokers.)

2. Orderers: **Set the maximum block size.** Each block will have at most
`Orderer.AbsoluteMaxBytes` bytes (not including headers), a value that you can
set in ``configtx.yaml``. Let the value you pick here be ``A`` and make note of
it -- it will affect how you configure your Kafka brokers in Step 4.

  Orderers: **设置区块最大容量.** 每一个区块最多只能有 `Orderer.AbsoluteMaxBytes` bytes的容量(不含区块头信息), 这是一个你可以修改的值，存放在 ``configtx.yaml`` 配置文件中. 假设此处你设置的数值为``A``,将此数字记下来 -- 这会影响你在步骤4中对于Kafka brokers 的配置.

3. Orderers: **Create the genesis block.** Use ``configtxgen``. The settings you
picked in Steps 1 and 2 above are system-wide settings, i.e. they apply across
the network for all the OSNs. Make note of the genesis block's location.

 Orderers:  使用 ``configtxgen`` 工具 **创建初始区块.** 在步骤1和2中的设置是全局的设置, 也就是说这些设置的生效范围是网络中所有的排序节点. 记录下初始区块的位置. 

4. Kafka cluster: **Configure your Kafka brokers appropriately.** Ensure that
every Kafka broker has these keys configured:

 Kafka 集群: **适当配置你的Kafka集群.** 确保每一个Kafka节点都配置了以下的值:

    a. ``unclean.leader.election.enable = false`` -- Data consistency is key in
    a blockchain environment. We cannot have a channel leader chosen outside of
    the in-sync replica set, or we run the risk of overwriting the offsets that
    the previous leader produced, and --as a result-- rewrite the blockchain
    that the orderers produce.
    
    a. ``unclean.leader.election.enable = false`` -- 数据一致性是区块链环境的关键. 我们不能选择不在同步副本集中的channel leader, 也不能冒风险去覆盖前一leader所产生的偏移量, 那样的结果就是重写orderers所产生的区块链数据.

    b.  ``min.insync.replicas = M`` -- Where you pick a value ``M`` such that
    1 < M < N (see ``default.replication.factor`` below). Data is considered
    committed when it is written to at least ``M`` replicas (which are then
    considered in-sync and belong to the in-sync replica set, or ISR). In any
    other case, the write operation returns an error. Then:
    
    b.  ``min.insync.replicas = M`` --  ``M`` 的值需要满足
    1 < M < N (N的值参考后面的 ``default.replication.factor``). 数据被认为是完成提交当它被写入到至少 ``M`` 个副本中(也就是说它被认为是同步的,然后被写入到同步副本集中,也成为ISR). 其他情况, 写入操作返回错误信息. 然后: 

        i. If up to N-M replicas -- out of the N that the channel data is
        written to -- become unavailable, operations proceed normally.
        i. 如果有 N-M 个副本不可访问, 操作将正常进行.
        ii. If more replicas become unavailable, Kafka cannot maintain an ISR
        set of M, so it stops accepting writes. Reads work without issues.
        The channel becomes writeable again when M replicas get in-sync.
        ii. 如果更多副本不可访问, Kafka 不能位置数量 M 的同步副本集(ISR), 所以它会停止接受写入操作. 读操作可以正常运行.
        当M个副本重新同步后,通道就可以再次变为可写入状态.
        

    c. ``default.replication.factor = N`` -- Where you pick a value ``N`` such
    that N < K. A replication factor of ``N`` means that each channel will have
    its data replicated to ``N`` brokers. These are the candidates for the ISR
    set of a channel. As we noted in the ``min.insync.replicas section`` above,
    not all of these brokers have to be available all the time. ``N`` should be
    set *strictly smaller* to ``K`` because channel creations cannot go forward
    if less than ``N`` brokers are up. So if you set N = K, a single broker
    going down means that no new channels can be created on the blockchain
    network -- the crash fault tolerance of the ordering service is
    non-existent.
    
    c. ``default.replication.factor = N`` -- 选择一个 ``N`` 的数值满足 N < K (Kafak集群数量). 参数 ``N`` 表示每个channel 的数据会复制到 ``N`` 个 broker 中. 这些是 channel 同步副本集的候选. 正如前面 ``min.insync.replicas`` 部分所说的, 不是所有broker都需要是随时可用的. ``N`` 值需要设置为绝对小于 ``K`` , 因为channel的创建需要不少于 ``N`` 个broker是启动的. 所以如果设置 N = K , 一个 broker 宕机就意味着区块链网络不能再创建channel. 那么故障容错的排序服务也就不存在了.


    d. ``message.max.bytes`` and ``replica.fetch.max.bytes`` should be set to a
    value larger than ``A``, the value you picked in
    ``Orderer.AbsoluteMaxBytes`` in Step 2 above. Add some buffer to account for
    headers -- 1 MiB is more than enough. The following condition applies:
    
    d. ``message.max.bytes`` 和 ``replica.fetch.max.bytes`` 的值需要大于 ``A``, 就是在步骤2中选取的 ``Orderer.AbsoluteMaxBytes`` 的值. 再为区块头增加一些余量 -- 1 MiB 就足够了. 需要满足以下条件:

    ::

        Orderer.AbsoluteMaxBytes < replica.fetch.max.bytes <= message.max.bytes

    (For completeness, we note that ``message.max.bytes`` should be strictly
    smaller to ``socket.request.max.bytes`` which is set by default to 100 MiB.
    If you wish to have blocks larger than 100 MiB you will need to edit the
    hard-coded value in ``brokerConfig.Producer.MaxMessageBytes`` in
    ``fabric/orderer/kafka/config.go`` and rebuild the binary from source.
    This is not advisable.)
    
    (补充, 我们注意到 ``message.max.bytes`` 需要严格小于 ``socket.request.max.bytes`` , 这个值默认是100Mib. 如果你希望区块大于100MiB, 你需要去修改硬代码中的变量 ``brokerConfig.Producer.MaxMessageBytes`` , 代码位置是 ``fabric/orderer/kafka/config.go`` , 再重新编译代码, 不建议这么做.)

    e. ``log.retention.ms = -1``. Until the ordering service adds
    support for pruning of the Kafka logs, you should disable time-based
    retention and prevent segments from expiring. (Size-based retention -- see
    ``log.retention.bytes`` -- is disabled by default in Kafka at the time of
    this writing, so there's no need to set it explicitly.)
    
    e. ``log.retention.ms = -1``. 直到排序服务增加了对于 Kafka 日志分割(pruning)的支持之前, 应该禁用基于时间分割的方式以避免单个日志文件到期分段. (基于文件大小的分割方式 -- 看参数 ``log.retention.bytes`` -- 在本文书写时, 在 Kafka 中是默认被禁用的, 所以这个值没有必要指定地很明确. )

    Based on what we've described above, the minimum allowed values for ``M``
    and ``N`` are 2 and 3 respectively. This configuration allows for the
    creation of new channels to go forward, and for all channels to continue to
    be writeable.
    
    基于上文所描述的, ``M`` 和 ``N`` 的最小值分别为 2 和 3 . 这个配置可以创建 channel 并让所有 channel 都是随时可以写入的.

5. Orderers: **Point each OSN to the genesis block.** Edit
``General.GenesisFile`` in ``orderer.yaml`` so that it points to the genesis
block created in Step 3 above. (While at it, ensure all other keys in that YAML
file are set appropriately.)

 Orderers: **将所有排序节点指向初始区块.** 编辑 ``orderer.yaml`` 文件中的参数 ``General.GenesisFile`` 使其指向步骤3中所创建的初始区块. (同时, 确保YAML文件中所有其他参数都是正确的.)

6. Orderers: **Adjust polling intervals and timeouts.** (Optional step.)
 Orderers: **调整轮询间隔和超时时间.** (可选步骤.)

    a. The ``Kafka.Retry`` section in the ``orderer.yaml`` file allows you to
    adjust the frequency of the metadata/producer/consumer requests, as well as
    the socket timeouts. (These are all settings you would expect to see in a
    Kafka producer or consumer.)
    
    a.  ``orderer.yaml`` 文件中的 ``Kafka.Retry`` 区域让你能够调整  metadata/producer/consumer 请求的频率以及socket的超时时间. (这些应该就是所有在 kafka 的生产者和消费者 中你需要的设置)

    b. Additionally, when a new channel is created, or when an existing channel
    is reloaded (in case of a just-restarted orderer), the orderer interacts
    with the Kafka cluster in the following ways:
    
    b. 另外, 当一个 channel 被创建, 或当一个现有的 channel 被重新读取(刚启动 orderer 的情况), orderer 通过以下方式和 Kafka 集群进行交互. 

        a. It creates a Kafka producer (writer) for the Kafka partition that
        corresponds to the channel.
        
        a. 为 channel 对应的 Kafka 分区 创建一个 Kafka 生产者.

        b. It uses that producer to post a no-op ``CONNECT`` message to that
        partition.
        
        b. 通过生产者向这个分区发一个空的连接信息.

        c. It creates a Kafka consumer (reader) for that partition.
        
        c. 为这个分区创建一个 Kafka 消费者. 

        If any of these steps fail, you can adjust the frequency with which they
        are repeated. Specifically they will be re-attempted every
        ``Kafka.Retry.ShortInterval`` for a total of ``Kafka.Retry.ShortTotal``,
        and then every ``Kafka.Retry.LongInterval`` for a total of
        ``Kafka.Retry.LongTotal`` until they succeed. Note that the orderer will
        be unable to write to or read from a channel until all of the steps
        above have been completed successfully.
        
        如果任意步骤出错, 你可以调整其重复的频率. 
        这些步骤会在每一个 Kafka.Retry.ShortInterval 指定的时间间隔后进行重试 Kafka.Retry.ShortTotal 次, 
        再以 Kafka.Retry.LongInterval 规定的时间间隔重试 Kafka.Retry.LongTotal 次直到成功. 
        需要注意的是 orderer 不能读写该 channel 的数据直到所有上述步骤都成功执行.

7. **Set up the OSNs and Kafka cluster so that they communicate over SSL.**
(Optional step, but highly recommended.) Refer to `the Confluent guide
<http://docs.confluent.io/2.0.0/kafka/ssl.html>`_ for the Kafka cluster side of
the equation, and set the keys under ``Kafka.TLS`` in ``orderer.yaml`` on every
OSN accordingly.

 **将排序节点和 Kafka 集群间设置为通过 SSL 通讯.** 
(可选步骤,强烈推荐) 参考 `the Confluent guide
<http://docs.confluent.io/2.0.0/kafka/ssl.html>`_ 文档中关于 Kafka 集群的设置, 来设置每个排序节点 ``orderer.yaml`` 文件中  ``Kafka.TLS`` 部分的内容.

8. **Bring up the nodes in the following order: ZooKeeper ensemble, Kafka
cluster, ordering service nodes.**

 **启动节点请按照以下顺序: ZooKeeper 集群, Kafka 集群, 排序节点**


其他注意事项（Additional considerations）
-------------------------

1. **Preferred message size.** In Step 2 above (see `Steps`_ section) you can
also set the preferred size of blocks by setting the
``Orderer.Batchsize.PreferredMaxBytes`` key. Kafka offers higher throughput when
dealing with relatively small messages; aim for a value no bigger than 1 MiB.

 **首选的消息大小.** 在上面的步骤2中, 你也能通过参数 ``Orderer.Batchsize.PreferredMaxBytes`` 设置首选的区块大小.
Kafka 处理相对较小的信息有更高的吞吐量; 针对小于 1 MiB 大小的值.

2. **Using environment variables to override settings.** You can override a
Kafka broker or a ZooKeeper server's settings by using environment variables.
Replace the dots of the configuration key with underscores --
e.g. ``KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false`` will allow you to override
the default value of ``unclean.leader.election.enable``. The same applies to the
OSNs for their *local* configuration, i.e. what can be set in ``orderer.yaml``.
For example ``ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s`` allows you to override the
default value for ``Orderer.Kafka.Retry.ShortInterval``.

 **使用环境变量重写设置.** 你能够通过设置环境变量来重写 Kafka 节点和 Zookeeper 服务器的设置. 替换配置参数中的 点 为 下划线 -- 例如 ``KAFKA_UNCLEAN_LEADER_ELECTION_ENABLE=false`` 环境变量重写配置参数 ``unclean.leader.election.enable``. 环境变量重写同样适用于排序节点的*本地*配置, 即 ``orderer.yaml`` 中所能设置的. 例如 ``ORDERER_KAFKA_RETRY_SHORTINTERVAL=1s`` 环境变量可以重写本地配置文件中的 ``Orderer.Kafka.Retry.ShortInterval``.

Supported Kafka versions and upgrading
--------------------------------------
支持的 Kafka 版本和升级
--------------------------------------

Supported Kafka versions for v1 are ``0.9`` and ``0.10``. (Hyperledger Fabric
uses the `sarama client library <https://github.com/Shopify/sarama>`_
and vendors a version of it that supports Kafka 0.9 and 0.10.)

Fabric V1 支持的 Kafka 版本是 ``0.9`` 和 ``0.10``. (Hyperledger Fabric 使用代码库: `sarama client library <https://github.com/Shopify/sarama>`_ 支持的 Kafka 版本是 ``0.9`` 和 ``0.10``)

Out of the box the Kafka version defaults to ``0.9.0.1``. If you wish to use a
different supported version, specify a supported version using the
``Kafka.Version`` key in ``orderer.yaml``.

默认的 Kafka 版本是 ``0.9.0.1``. 如果你使用其他支持的版本, 修改 ``orderer.yaml`` 文件中的参数 ``Kafka.Version``.

The current supported Kafka versions are:

目前支持的 Kafka 版本是:

* ``Version: 0.9.0.1``
* ``Version: 0.10.0.0``
* ``Version: 0.10.0.1``
* ``Version: 0.10.1.0``


调试（Debugging）
---------

Set ``General.LogLevel`` to ``DEBUG`` and ``Kafka.Verbose`` in ``orderer.yaml``
to ``true``.

设置 ``orderer.yaml`` 文件中 ``General.LogLevel`` 为 ``DEBUG`` 和 ``Kafka.Verbose`` 为 ``true``.


例子（Example）
-------

Sample Docker Compose configuration files inline with the recommended settings
above can be found under the ``fabric/bddtests`` directory. Look for
``dc-orderer-kafka-base.yml`` and ``dc-orderer-kafka.yml``.

包含了推荐的设置的Docker Compose 配置文件示例能够在 ``fabric/bddtests`` 目录中找到. 包括 ``dc-orderer-kafka-base.yml`` 文件和 ``dc-orderer-kafka.yml`` 文件.

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/