
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/configtxlator.html) | Shaoxi Qiu |  |


概览（Overview）
--------

The ``configtxlator`` tool was created to support reconfiguration independent
of SDKs. Channel configuration is stored as a transaction in configuration
blocks of a channel and may be manipulated directly, such as in the bdd behave
tests.  However, at the time of this writing, no SDK natively supports
manipulating the configuration directly, so the ``configtxlator`` tool is
designed to provide an API which consumers of any SDK may interact with to
assist with configuration updates.

开发 ``configtxlator`` 工具是为了支持独立于SDK来进行重新配置。通道配置通过一个交易的形式存储在通道的配置区块中，并且能够直接被修改，就像bdd行为测试。
然而，在本文写作的时间，还没有SDK原生支持直接修改配置，所以 ``configtxlator`` 工具被设计为提供一个API让任意一个SDK的用户都能够与之交互来更新配置。

The tool name is a portmanteau of *configtx* and *translator* and is intended to
convey that the tool simply converts between different equivalent data
representations. It does not generate configuration. It does not submit or
retrieve configuration. It does not modify configuration itself, it simply
provides some bijective operations between different views of the configtx
format.

工具的名称是 *configtx* 和 *translator* 的拼接，意在传达该工具简单地在不同的等效数据之间进行转换。
它不产生配置。也不提交或撤回配置。它不修改配置本身，只是简单地提供一些配置格式的不同的双射展现。

译者注：既是单射又是满射的函数称为双射. 函数为双射当且仅当每个可能的像有且仅有一个变量与之对应。参考：http://www.cnblogs.com/wanghetao/archive/2012/03/16/2400619.html

The standard usage is expected to be:

  1. SDK retrieves latest config
  2. ``configtxlator`` produces human readable version of config
  3. User or application edits the config
  4. ``configtxlator`` is used to compute config update representation of
     changes to the config
  5. SDK submits signs and submits config
  
 
标准用法：

  1. SDK 取出最新的配置
  2.  ``configtxlator`` 工具产生可读版本的配置文件
  3. 用户或者应用编辑配置文件
  4. 使用 ``configtxlator`` 工具计算更新的配置与原有配置的差异
  5. SDK 提交配置以及签名


The ``configtxlator`` tool exposes a truly stateless REST API for interacting
with configuration elements.  These REST components support converting the
native configuration format to/from a human readable JSON representation, as
well as computing configuration updates based on the difference between two
configurations.

 ``configtxlator`` 工具暴露一个完全无状态的 REST API 接口用来和配置匀速进行交互。 
 这些 REST 组件支持本地的配置和可读的JSON格式配置文件进行相互转换， 同时根据配置文件的差异计算配置的更新。

Because the ``configtxlator`` service deliberately does not contain any crypto
material, or otherwise secret information, it does not include any authorization
or access control. The anticipated typical deployment would be to operate as
a sandboxed container, locally with the application, so that there is a
dedicated ``configtxlator`` process for each consumer of it.

因为 ``configtxlator`` 工具特意没有包含任何密码工具和密钥信息， 所有它没有任何权限控制。
预计的典型部署方式是运行在沙盒容器中， 所以在本地的应用中， 有一个专用的 ``configtxlator`` 进程给每一个使用者。



运行 configtxlator 工具（Running the configtxlator）
-------------------------

The ``configtxlator`` tool can be downloaded with the other Hyperledger Fabric
platform-specific binaries. Please see :ref:`download-platform-specific-binaries`
for details.

 ``configtxlator`` 工具可以和其他 Hyperledger Fabric 平台专用工具一样被下载使用。详情请查看 ref:`download-platform-specific-binaries` 。

The tool may be configured to listen on a different port and you may also
specify the hostname using the ``--port`` and ``--hostname`` flags. To explore
the complete set of commands and flags, run ``configtxlator --help``.

该工具可以配置去监听不同的端口和地址，只用  ``--port`` 和 ``--hostname`` 参数。
查看所有参数的详细信息，执行 ``configtxlator --help``.

The binary will start an http server listening on the designated port and is now
ready to process request.

工具启动一个服务器监听指定的端口且等待处理请求。

To start the ``configtxlator`` server:

执行命令启动 ``configtxlator`` 服务：

.. code:: bash

  configtxlator start
  2017-06-21 18:16:58.248 HKT [configtxlator] startServer -> INFO 001 Serving HTTP requests on 0.0.0.0:7059

原型翻译（Proto translation）
-----------------

For extensibility, and because certain fields must be signed over, many proto
fields are stored as bytes.  This makes the natural proto to JSON translation
using the ``jsonpb`` package ineffective for producing a human readable version
of the protobufs.  Instead, the ``configtxlator`` exposes a REST component to do
a more sophisticated translation.

为了可扩展性，以及特定的字段需要被签名，许多原型字段被存储为字节。使用 ``jsonpb`` 工具包来转换原型和可读的 JSON 格式因此变得无效。
替代的方式是， ``configtxlator`` 暴露一个REST 组件去做更复杂的翻译。

To convert a proto to its human readable JSON equivalent, simply post the binary
proto to the rest target
``http://$SERVER:$PORT/protolator/decode/<message.Name>``,
where ``<message.Name>`` is the fully qualified proto name of the message.

要转换原型到可读的 JSON 格式，只要发送二进制原型到 rest 目标
``http://$SERVER:$PORT/protolator/decode/<message.Name>``,
 ``<message.Name>`` 是合法原型名的全称。

For instance, to decode a configuration block saved as
``configuration_block.pb``, run the command:

例如，为了解析一个存储为 ``configuration_block.pb`` 的配置区块，执行命令：

.. code:: bash

  curl -X POST --data-binary @configuration_block.pb http://127.0.0.1:7059/protolator/decode/common.Block

To convert the human readable JSON version of the proto message, simply post the
JSON version to ``http://$SERVER:$PORT/protolator/encode/<message.Name``, where
``<message.Name>`` is again the fully qualified proto name of the message.

转换可读的JSON版本为原型数据，只要发送JSON版本到 ``http://$SERVER:$PORT/protolator/encode/<message.Name>`` ，
这里的 ``<message.Name>`` 是合法原型的全称。

For instance, to re-encode the block saved as ``configuration_block.json``, run
the command:

例如，重新编码存储为 ``configuration_block.json`` 的配置区块，执行命令：

.. code:: bash

  curl -X POST --data-binary @configuration_block.json http://127.0.0.1:7059/protolator/encode/common.Block

Any of the configuration related protos, including ``common.Block``,
``common.Envelope``, ``common.ConfigEnvelope``, ``common.ConfigUpdateEnvelope``,
``common.Configuration``, and ``common.ConfigUpdate`` are valid targets for
these URLs.  In the future, other proto decoding types may be added, such as
for endorser transactions.


任何原型相关的配置，包括  ``common.Block``,
``common.Envelope``, ``common.ConfigEnvelope``, ``common.ConfigUpdateEnvelope``,
``common.Configuration``, 和 ``common.ConfigUpdate`` 都是这些地址的合法的目标。
未来，其他解析类型可能会被增加，比如背书交易。


配置更新计算（Config update computation）
-------------------------

Given two different configurations, it is possible to compute the config update
which transitions between them.  Simply POST the two ``common.Config`` proto
encoded configurations as ``multipart/formdata``, with the original as field
``original`` and the updated as field ``updated``, to
``http://$SERVER:$PORT/configtxlator/compute/update-from-configs``.

两个不同的配置，可以计算出两个配置更新所需要的交易。
向  `http://$SERVER:$PORT/configtxlator/compute/update-from-configs``  
发送两个已编码的 ``common.Config`` 原型配置作为 ``multipart/formdata`` ，其中原始配置填入 ``original`` 域，更新配置填入  ``updated`` 域。

For example, given the original config as the file ``original_config.pb`` and
the updated config as the file ``updated_config.pb`` for the channel
``desiredchannel``:

例如，对于通道 ``desiredchannel`` 的原始配置文件 ``original_config.pb`` 和更新配置文件  ``updated_config.pb`` ：

.. code:: bash

  curl -X POST -F channel=desiredchannel -F original=@original_config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs


引导实例（Bootstraping example）
--------------------

First start the ``configtxlator``:

首先，启动启动 ``configtxlator`` 工具:

.. code:: bash

  $ configtxlator start
  2017-05-31 12:57:22.499 EDT [configtxlator] main -> INFO 001 Serving HTTP requests on port: 7059

First, produce a genesis block for the ordering system channel:

然后，为通道产生初始区块

.. code:: bash

  $ configtxgen -outputBlock genesis_block.pb
  2017-05-31 14:15:16.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-05-31 14:15:16.646 EDT [common/configtx/tool] doOutputBlock -> INFO 002 Generating genesis block
  2017-05-31 14:15:16.646 EDT [common/configtx/tool] doOutputBlock -> INFO 003 Writing genesis block

Decode the genesis block into a human editable form:

解析初始区块为可编辑的形式

.. code:: bash

  curl -X POST --data-binary @genesis_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > genesis_block.json

Edit the ``genesis_block.json`` file in your favorite JSON editor, or manipulate
it programatically.  Here we use the JSON CLI tool ``jq``.  For simplicity, we
are editing the batch size for the channel, because it is a single numeric
field. However, any edits, including policy and MSP edits may be made here.

使用你喜欢的JSON编辑器编辑 ``genesis_block.json`` 文件，或使用程序编辑。 这里需要使用JSON  工具  ``jq`` .
为了方便，这里编辑通道的区块大小，因为这是一个数字字段。
然而，任何修改，包括策略和MSP都是可以做的。

First, let's establish an environment variable to hold the string that defines
the path to a property in the json:

首先，建立一个环境变量来存储变量的路径

.. code:: bash

  export MAXBATCHSIZEPATH=".data.data[0].payload.data.config.channel_group.groups.Orderer.values.BatchSize.value.max_message_count"

Next, let's display the value of that property:

然后，显示变量的值

.. code:: bash

  jq "$MAXBATCHSIZEPATH" genesis_block.json
  10

Now, let's set the new batch size, and display the new value:

现在，设置新的区块大小，并且显示新值：

  jq "$MAXBATCHSIZEPATH = 20" genesis_block.json  > updated_genesis_block.json
  jq "$MAXBATCHSIZEPATH" updated_genesis_block.json
  20

The genesis block is now ready to be re-encoded into the native proto form to be
used for bootstrapping:

初始区块现在已经可以被重新编码为可用于引导启动的原型格式：

.. code:: bash

  curl -X POST --data-binary @updated_genesis_block.json http://127.0.0.1:7059/protolator/encode/common.Block > updated_genesis_block.pb

The ``updated_genesis_block.pb`` file may now be used as the genesis block for
bootstrapping an ordering system channel.

现在， ``updated_genesis_block.pb`` 文件可以作为初始区块来引导通道启动了。


重配置示例（Reconfiguration example）
-----------------------

In another terminal window, start the orderer using the default options,
including the provisional bootstrapper which will create a ``testchainid``
ordering system channel.

打开另一个终端窗口，使用默认选项启动orderer，
包括临时的引导程序，将会创建一个名称为 ``testchainid`` 的排序通道。

.. code:: bash

  ORDERER_GENERAL_LOGLEVEL=debug orderer

Reconfiguring a channel can be performed in a very similar way to modifying a
genesis config.

重配置一个通道与修改初始配置类似。

First, fetch the config_block proto:

首先，获取配置区块原型：

.. code:: bash

  $ peer channel fetch config config_block.pb -o 127.0.0.1:7050 -c testchainid
  2017-05-31 15:11:37.617 EDT [msp] getMspConfig -> INFO 001 intermediate certs folder not found at [/home/yellickj/go/src/github.com/hyperledger/fabric/sampleconfig/msp/intermediatecerts]. Skipping.: [stat /home/yellickj/go/src/github.com/hyperledger/fabric/sampleconfig/msp/intermediatecerts: no such file or directory]
  2017-05-31 15:11:37.617 EDT [msp] getMspConfig -> INFO 002 crls folder not found at [/home/yellickj/go/src/github.com/hyperledger/fabric/sampleconfig/msp/intermediatecerts]. Skipping.: [stat /home/yellickj/go/src/github.com/hyperledger/fabric/sampleconfig/msp/crls: no such file or directory]
  Received block:  1
  Received block:  1
  2017-05-31 15:11:37.635 EDT [main] main -> INFO 003 Exiting.....

Next, send the config block to the ``configtxlator`` service for decoding:

然后，发送配置区块到 ``configtxlator`` 服务进行解析：

.. code:: bash

  curl -X POST --data-binary @config_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > config_block.json

Extract the config section from the block:

从区块中提取配置区域

.. code:: bash

  jq .data.data[0].payload.data.config config_block.json > config.json

Edit the config, saving it as a new ``updated_config.json``.  Here, we set the
batch size to 30.

编辑配置，将编辑后的内容存放在 ``updated_config.json`` 。这里我们设计区块大小为30.

.. code:: bash

  jq ".channel_group.groups.Orderer.values.BatchSize.value.max_message_count = 30" config.json  > updated_config.json

Re-encode both the original config, and the updated config into proto:

重新将原配置与新配置进行编码：

.. code:: bash

  curl -X POST --data-binary @config.json http://127.0.0.1:7059/protolator/encode/common.Config > config.pb

.. code:: bash

  curl -X POST --data-binary @updated_config.json http://127.0.0.1:7059/protolator/encode/common.Config > updated_config.pb

Now, with both configs properly encoded, send them to the `configtxlator`
service to compute the config update which transitions between the two.

现在，将编码后的文件发送到 `configtxlator` 服务进行计算比对两个的差异。

.. code:: bash

  curl -X POST -F original=@config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs -F channel=testchainid > config_update.pb

At this point, the computed config update is now prepared. Traditionally,
an SDK would be used to sign and wrap this message. However, in the interest of
using only the peer cli, the `configtxlator` can also be used for this task.

到此，计算出的配置更新已经准备好了。一般，SDK会对该消息进行签名打包。
然而，为了那些只使用节点命令（peer cli）的情况， `configtxlator` 工具也能进行这个工作。

First, we decode the ConfigUpdate so that we may work with it as text:

首先，按上文所说对配置更新进行编码：

.. code:: bash

  $ curl -X POST --data-binary @config_update.pb http://127.0.0.1:7059/protolator/decode/common.ConfigUpdate > config_update.json

Then, we wrap it in an envelope message:

然后，讲消息进行打包：

.. code:: bash

  echo '{"payload":{"header":{"channel_header":{"channel_id":"testchainid", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' > config_update_as_envelope.json

Next, convert it back into the proto form of a full fledged config
transaction:

接着，将它转换为完整配置的交易的原型结构

.. code:: bash

  curl -X POST --data-binary @config_update_as_envelope.json http://127.0.0.1:7059/protolator/encode/common.Envelope > config_update_as_envelope.pb

Finally, submit the config update transaction to ordering to perform a config
update.

最后，将配置更新交易提交到排序服务。

.. code:: bash

  peer channel update -f config_update_as_envelope.pb -c testchainid -o 127.0.0.1:7050


增加组织（Adding an organization）
----------------------

First start the ``configtxlator``:

首先启动 ``configtxlator`` 服务：

.. code:: bash

  $ configtxlator start
  2017-05-31 12:57:22.499 EDT [configtxlator] main -> INFO 001 Serving HTTP requests on port: 7059

Start the orderer using the ``SampleDevModeSolo`` profile option.

使用 ``SampleDevModeSolo`` 属性配置来启动排序服务。

.. code:: bash

  ORDERER_GENERAL_LOGLEVEL=debug ORDERER_GENERAL_GENESISPROFILE=SampleDevModeSolo orderer

The process to add an organization then follows exactly like the batch size
example. However, instead of setting the batch size, a new org is defined at
the application level. Adding an organization is slightly more involved because
we must first create a channel, then modify its membership set.

增加组织的国政和修改区块大小的过程类似。然而，不同于设置区块大小，一个新的组织被定义在应用层。
增加一个组织涉及更多内容因为需要先创建通道，然后修改它的成员集。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
