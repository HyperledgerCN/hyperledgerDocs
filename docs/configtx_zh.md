
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/configtx.html) | Linsheng Yu |  |


Shared configuration for a Hyperledger Fabric blockchain network is stored in a collection configuration transactions, one per channel. Each configuration transaction is usually referred to by the shorter name ***configtx***.

Hyperledger Fabric 区块链网络的共享配置存储在每个channel的配置交易集合中。配置交易简称***configtx***。

Channel configuration has the following important properties:

Channel配置有以下重要特性：

1. **版本 Versioned**：All elements of the configuration have an associated version which is advanced with every modification. Further, every committed configuration receives a sequence number.
	
	配置中的所有项都有一个相关联的版本，每次修改都会有个更高的版本。此外每个提交的配置都会有个序列号。
2. **许可 Permissioned**：Anyone with a copy of the previous configtx (and no additional info) may verify the validity of a new config based on these policies.
	
	配置中的所有项都有一个相关联的策略，该策略控制该元素是否可修改。任何有前一个configtx（无需额外信息）的“人”，都可基于这些策略验证新配置的有效性。
3. **分层 Hierarchical**：A root configuration group contains sub-groups, and each group of the hierarchy has associated values and policies. These policies can take advantage of the hierarchy to derive policies at one level from policies of lower levels.

	根配置组包含子组，每个分层组具有相关的值和策略。这些策略可以利用分层结构从较低级别的策略中导出。

## Anatomy of a configuration / 配置剖析

Configuration is stored as a transaction of type `HeaderType_CONFIG` in a block with no other transactions. These blocks are referred to as ***Configuration Blocks***, the first of which is referred to as the ***Genesis Block***.

配置作为一种`HeaderType_CONFIG`类型的交易单独存储在一个block中（也就是说这个block不再包含其他交易），这个block被称为***配置区块***，其中的第一个称为 ***创世区块***。

The proto structures for configuration are stored in `fabric/protos/common/configtx.proto`. The Envelope of type `HeaderType_CONFIG` encodes a `ConfigEnvelope` message as the `Payload` `data` field. The proto for `ConfigEnvelope` is defined as follows:

配置的数据结构在文件`fabric/protos/common/configtx.proto`中，编码后的`ConfigEnvelope`信息作为`HeaderType_CONFIG`类型的`Envelope`中`Payload`的`Data`字段值。*（译注：Envelope.payload.Header.channel_header.type= HeaderType_CONFIG，Envelope.payload.data = []byte(ConfigEnvelope)）*`ConfigEnvelope`定义如下：

	message ConfigEnvelope {
	    Config config = 1;        // A marshaled Config structure
	    Envelope last_update = 2; // The last CONFIG_UPDATE message which generated this current configuration
	                              // Note that CONFIG_UPDATE has a Payload.Data of a Marshaled ConfigUpdate
	}

The `last_update` field is defined below in the Updates to configuration section, but is only necessary when validating the configuration, not reading it. Instead, the currently committed configuration is stored in the `config` field, containing a `Config` message.

`last_update`字段在下面的“更新配置”一节定义，该字段只用于验证配置，而不会读他。当前提交的配置存储在`config`字段，是一个`Config`信息。
	
	// Config represents the config for a particular channel
	message Config {
	    uint64 sequence = 1;
	    ConfigGroup channel_group = 2;
	}

The `sequence` number is incremented by one for each committed configuration. The `channel_group` field is the root group which contains the configuration. The `ConfigGroup` structure is recursively defined, and builds a tree of groups, each of which contains values and policies. It is defined as follows:

其中`sequence`字段是每次提交配置递增的数字；`channel_group`字段是包含该配置的根组。`ConfigGroup`结构是递归定义的，构建了一个组树，其中每个组都包含值和策略。其结构如下：

	// ConfigGroup is the hierarchical data structure for holding config
	message ConfigGroup {
	    uint64 version = 1;
	    map<string,ConfigGroup> groups = 2;
	    map<string,ConfigValue> values = 3;
	    map<string,ConfigPolicy> policies = 4;
	    string mod_policy = 5;
	}

Because `ConfigGroup` is a recursive structure, consider an example hierarchical arrangement of ``ConfigGroup``s (expressed for clarity in golang notation).

`ConfigGroup`是递归结构，此处给出一个用golang表示的分层排列的示例：

	// 假设定义了以下group
	var root, child1, child2, grandChild1, grandChild2, grandChild3 *ConfigGroup
	
	// Set the following values
	root.Groups["child1"] = child1
	root.Groups["child2"] = child2
	child1.Groups["grandChild1"] = grandChild1
	child2.Groups["grandChild2"] = grandChild2
	child2.Groups["grandChild3"] = grandChild3
	
	// The resulting config structure of groups looks like:
	// root:
	//     child1:
	//         grandChild1
	//     child2:
	//         grandChild2
	//         grandChild3

Each group defines a level in the config hierarchy, and each group has an associated set of values (indexed by string key) and policies (also indexed by string key).

每个组都定义了一个分层结构中的级别，且每个组都有一个相关的值集（以string为key）和策略（以string为key）。

Values are defined by:

值定义：

	// ConfigValue represents an individual piece of config data
	message ConfigValue {
	    uint64 version = 1;
	    bytes value = 2;
	    string mod_policy = 3;
	}

Policies are defined by:
策略定义：

	message ConfigPolicy {
	    uint64 version = 1;
	    Policy policy = 2;
	    string mod_policy = 3;
	}

Note that Values, Policies, and Groups all have a `version` and a `mod_policy`.

注意，ConfigGroup、ConfigValue、ConfigPolicy都有`version`和`mod_policy`字段。

The `ersion` of an element is incremented each time that element is modified. The `mod_policy` is used to govern the required signatures to modify that element. 

每次修改元素时，其`version `递增，`mod_policy`用于管理修改该元素所需的签名。

For Groups, modification is adding or removing elements to the Values, Policies, or Groups maps (or changing the `mod_policy`). For Values and Policies, modification is changing the Value and Policy fields respectively (or changing the mod_policy). 

对于Groups，修改就是添加或删除Values、Policies、或Groups中的元素（或者是改变`mod_policy`）*（译注，其实就是`ConfigGroup`中除`version`字段外其他字段的变化）*；对于Values和Policies，修改就是改变`value`或`policy`字段（或者改变`mod_policy`）。

Each element’s `mod_policy` is evaluated in the context of the current level of the config.
 
每个元素的`mod_policy`都只在当前配置级别(level)中有效。

Consider the following example mod policies defined at `Channel.Groups["Application"]` (Here, we use the golang map reference syntax, so `Channel.Groups["Application"].Policies["policy1"]` refers to the base `Channel` group’s `Application` group’s `Policies` map’s `policy1` policy.)

下面是一个定义在`Channel.Groups["Application"]`中的策略的示例（这里用的是golang语法，因此`Channel.Groups["Application"].Policies["policy1"]`表示根组`Channel`的子组`Application`的`Policies`里的`policy1`对应的策略）

* `policy1`对应`Channel.Groups["Application"].Policies["policy1"]`
* `Org1/policy2`对应`Channel.Groups["Application"].Groups["Org1"].Policies["policy2"]`
* `/Channel/policy3`对应`Channel.Policies["policy3"]`

Note that if a `mod_policy` references a policy which does not exist, the item cannot be modified.

注意，如果`mod_policy`引用了一个不存在的策略，那么该元素不可修改。

## Configuration updates / 更新配置

Configuration updates are submitted as an `Envelope` message of type `HeaderType_CONFIG_UPDATE`. The `Payload` `data` of the transaction is a marshaled `ConfigUpdateEnvelope`. The `ConfigUpdateEnvelope` is defined as follows:

更新配置是提交一个`HeaderType_CONFIG_UPDATE`类型的`Envelope`消息，交易的`Payload.data `字段是序列化的`ConfigUpdateEnvelope`，其定义如下：

	message ConfigUpdateEnvelope {
	    bytes config_update = 1;                 // A marshaled ConfigUpdate structure
	    repeated ConfigSignature signatures = 2; // Signatures over the config_update
	}

The `signatures` field contains the set of signatures which authorizes the config update. Its message definition is:

其中`signatures`字段包含了授权更新配置的签名集，定义如下：

	message ConfigSignature {
	    bytes signature_header = 1; // A marshaled SignatureHeader
	    bytes signature = 2;        // Signature over the concatenation signatureHeader bytes and config bytes
	}

The `signature_header` is as defined for standard transactions, while the signature is over the concatenation of the `signature_header` bytes and the `config_update` bytes from the `ConfigUpdateEnvelope` message.

`signature_header`如标准交易所定义，而签名则是`signature_header`字节和`ConfigUpdateEnvelope`中的`config_update`字节的拼接。

The `ConfigUpdateEnvelope` `config_update` bytes are a marshaled `ConfigUpdate` message which is defined as follows:

`ConfigUpdateEnvelope`中的`config_update`字段是序列化的`ConfigUpdate`，其定义为：

	message ConfigUpdate {
	    string channel_id = 1;     // Which channel this config update is for
	    ConfigGroup read_set = 2;  // ReadSet explicitly lists the portion of the config which was read, this should be sparse with only Version set
	    ConfigGroup write_set = 3; // WriteSet lists the portion of the config which was written, this should included updated Versions
	}

The `channel_id` is the channel ID the update is bound for, this is necessary to scope the signatures which support this reconfiguration.

其中`channel_id`是配置更新所对应的channel ID，该字段是必要，因为它界定了支持本次配置更新的所需的签名范围。

The `read_set` specifies a subset of the existing configuration, specified sparsely where only the `version` field is set and no other fields must be populated. The particular `ConfigValue` `value` or `ConfigPolicy` `policy` fields should never be set in the `read_set`. The `ConfigGroup` may have a subset of its map fields populated, so as to reference an element deeper in the config tree. For instance, to include the `Application` group in the `read_set`, its parent (the `Channel` group) must also be included in the read set, but, the `Channel` group does not need to populate all of the keys, such as the `Orderer` `group` key, or any of the `values` or `policies` keys.

`read_set`是现有配置的一个子集，其中仅含`version`字段，`ConfigValue.value`和`ConfigPolicy.policy`等其他字段不包含在`read_set`中。`ConfigGroup`会map字段组成的子集，以便引用配置树的深层元素。例如，为使`Application`group包含到`read_set`，它的上层（`Channel`group）也必须包含到`read_set`中，但不必将`Channel`group中所有的key都包括进去，比如`Orderer``group`或者任何`values`或`policies`。

The `write_set` specifies the pieces of configuration which are modified. Because of the hierarchical nature of the configuration, a write to an element deep in the hierarchy must contain the higher level elements in its `write_set` as well. However, for any element in the `write_set` which is also specified in the `read_set` at the same version, the element should be specified sparsely, just as in the `read_set`.

`write_set`指定了要被修改的那部分配置。由于配置的分层特性，修改深层元素就必须在`write_set`中包含其上层元素。`write_set`中的任意元素都会在`read_set`中指定相同版本的该元素。

For example, given the configuration:

例如，给出如下配置：

	Channel: (version 0)
	    Orderer (version 0)
	    Appplication (version 3)
	       Org1 (version 2)
	       
To submit a configuration update which modifies `Org1`, the `read_set` would be:
	       
修改`Org1`提交的`read_set`应为：
	
	Channel: (version 0)
	    Application: (version 3)

and the write_set would be

对应的`write_set`应是：

	Channel: (version 0)
	    Application: (version 3)
	        Org1 (version 3)

When the `CONFIG_UPDATE` is received, the orderer computes the resulting `CONFIG` by doing the following:
	  
接收到`CONFIG_UPDATE`后，orderer会通过以下步骤计算`CONFIG`结果：

1. Verifies the `channel_id` and `read_set`. All elements in the `read_set` must exist at the given versions.
	
	校验`channel_id`和`read_set`，`read_set`中所有元素必须存在对应的版本。
2. Computes the update set by collecting all elements in the `write_set` which do not appear at the same version in the `read_set`.
	
	收集`read_set`与`write_set`中版本不一致的元素，计算更新集。
3. Verifies that each element in the update set increments the version number of the element update by exactly 1.

	校验更新集中的元素的版本号是否递增1
4. Verifies that the signature set attached to the `ConfigUpdateEnvelope` satisfies the `mod_policy` for each element in the update set.
	
	校验更新集中每个元素，`ConfigUpdateEnvelope`的签名满足`mod_policy`。
5. Computes a new complete version of the config by applying the update set to the current config.
	
	通过将更新集应用于当前配置，计算该配置的完整新版本
6. 	Writes the new config into a `ConfigEnvelope` which includes the `CONFIG_UPDATE` as the `last_update` field and the new config encoded in the `config` field, along with the incremented `sequence` value.
	
	将新配置写成`ConfigEnvelope`作为`CONFIG_UPDATE`赋给`last_update`字段，新的配置赋给`config`字段，`sequence`字段自增。
7. Writes the new `ConfigEnvelope` into a `Envelope` of type `CONFIG`, and ultimately writes this as the sole transaction in a new configuration block.

	将`ConfigEnvelope`写成`CONFIG`类型的`Envelope`，最终将此作为唯一交易写入配置区块。、

When the peer (or any other receiver for `Deliver`) receives this configuration block, it should verify that the config was appropriately validated by applying the last_update message to the current config and verifying that the orderer-computed `config` field contains the correct new configuration.
	
当peer（或者任意其他接收`Deliver`者）接收到这个配置区块后，就会通过将`last_update`信息应用到当前配置并校验orderer计算的`config`字段是否包含正确的新配置，来验证该配置是否被正确校验。

## Permitted configuration groups and values / 组和值得配置许可

Any valid configuration is a subset of the following configuration. Here we use the notation `peer.<MSG>` to define a `ConfigValue` whose `value` field is a marshaled proto message of name `<MSG>` defined in `fabric/protos/peer/configuration.proto`. The notations `common.<MSG>`, `msp.<MSG>`, and `orderer.<MSG>` correspond similarly, but with their messages defined in `fabric/protos/common/configuration.proto`, `fabric/protos/msp/mspconfig.proto`, and `fabric/protos/orderer/configuration.proto` respectively.

有效的配置都是下面配置的子集。在此，用`peer.<MSG>`表示一个`ConfigValue`，其`value`字段是称为`<MSG>`的序列化后的信息，定义在`fabric/protos/peer/configuration.proto`。`common.<MSG>`，`msp.<MSG>`和`orderer.<MSG>`分别定义在`fabric/protos/common/configuration.proto`，`fabric/protos/msp/mspconfig.proto`和`fabric/protos/orderer/configuration.proto`。

Note, that the keys `{{org_name}}` and `{{consortium_name}}` represent arbitrary names, and indicate an element which may be repeated with different names.

注意，下面的`{{org_name}}` 和 `{{consortium_name}}`是任意的名字，表示可以重复使用不同名称的元素。
	
	&ConfigGroup{
	    Groups: map<string, *ConfigGroup> {
	        "Application":&ConfigGroup{
	            Groups:map<String, *ConfigGroup> {
	                {{org_name}}:&ConfigGroup{
	                    Values:map<string, *ConfigValue>{
	                        "MSP":msp.MSPConfig,
	                        "AnchorPeers":peer.AnchorPeers,
	                    },
	                },
	            },
	        },
	        "Orderer":&ConfigGroup{
	            Groups:map<String, *ConfigGroup> {
	                {{org_name}}:&ConfigGroup{
	                    Values:map<string, *ConfigValue>{
	                        "MSP":msp.MSPConfig,
	                    },
	                },
	            },
	
	            Values:map<string, *ConfigValue> {
	                "ConsensusType":orderer.ConsensusType,
	                "BatchSize":orderer.BatchSize,
	                "BatchTimeout":orderer.BatchTimeout,
	                "KafkaBrokers":orderer.KafkaBrokers,
	            },
	        },
	        "Consortiums":&ConfigGroup{
	            Groups:map<String, *ConfigGroup> {
	                {{consortium_name}}:&ConfigGroup{
	                    Groups:map<string, *ConfigGroup> {
	                        {{org_name}}:&ConfigGroup{
	                            Values:map<string, *ConfigValue>{
	                                "MSP":msp.MSPConfig,
	                            },
	                        },
	                    },
	                    Values:map<string, *ConfigValue> {
	                        "ChannelCreationPolicy":common.Policy,
	                    }
	                },
	            },
	        },
	    },
	
	    Values: map<string, *ConfigValue> {
	        "HashingAlgorithm":common.HashingAlgorithm,
	        "BlockHashingDataStructure":common.BlockDataHashingStructure,
	        "Consortium":common.Consortium,
	        "OrdererAddresses":common.OrdererAddresses,
	    },
	}

## Orderer system channel configuration / Order channel 配置

The ordering system channel needs to define ordering parameters, and consortiums for creating channels. There must be exactly one ordering system channel for an ordering service, and it is the first channel to be created (or more accurately bootstrapped). It is recommended never to define an Application section inside of the ordering system channel genesis configuration, but may be done for testing. Note that any member with read access to the ordering system channel may see all channel creations, so this channel’s access should be restricted.

ordering系统channel定义了创建channel的ordering参数和consortiums。ordering service必须有一个ordering系统channel，这是被创建的第一个channel。建议不要在ordering系统channel初始配置中定义application部分，但是测试是可以这么做。注意，任何对ordering系统channel有读权限的成员都可以查看所有channel创建，因此channel的访问应受限制。

The ordering parameters are defined as the following subset of config:

ordering参数定义如下：

	&ConfigGroup{
	    Groups: map<string, *ConfigGroup> {
	        "Orderer":&ConfigGroup{
	            Groups:map<String, *ConfigGroup> {
	                {{org_name}}:&ConfigGroup{
	                    Values:map<string, *ConfigValue>{
	                        "MSP":msp.MSPConfig,
	                    },
	                },
	            },
	
	            Values:map<string, *ConfigValue> {
	                "ConsensusType":orderer.ConsensusType,
	                "BatchSize":orderer.BatchSize,
	                "BatchTimeout":orderer.BatchTimeout,
	                "KafkaBrokers":orderer.KafkaBrokers,
	            },
	        },
	    },

Each organization participating in ordering has a group element under the `Orderer` group. This group defines a single parameter `MSP` which contains the cryptographic identity information for that organization. The `Values` of the `Orderer` group determine how the ordering nodes function. They exist per channel, so `orderer.BatchTimeout` for instance may be specified differently on one channel than another.

ordering中的每个组织都在`Orderer`组下有一个组元素，这个组定义了一个`MSP`参数，这个参数包含该组织的加密身份信息。`Orderer`组中的`Values`决定了ordering节点的功能。他们存在于每个channel中，所以像`orderer.BatchTimeout`就可在不同channel中指定不同值。

At startup, the orderer is faced with a filesystem which contains information for many channels. The orderer identifies the system channel by identifying the channel with the consortiums group defined. The consortiums group has the following structure.

启动时，orderer面对含有很多channel信息的文件系统，orderer通过识别带有consortiums组定义的channel来标识系统channel。consortiums组结构如下。
	
	&ConfigGroup{
	    Groups: map<string, *ConfigGroup> {
	        "Consortiums":&ConfigGroup{
	            Groups:map<String, *ConfigGroup> {
	                {{consortium_name}}:&ConfigGroup{
	                    Groups:map<string, *ConfigGroup> {
	                        {{org_name}}:&ConfigGroup{
	                            Values:map<string, *ConfigValue>{
	                                "MSP":msp.MSPConfig,
	                            },
	                        },
	                    },
	                    Values:map<string, *ConfigValue> {
	                        "ChannelCreationPolicy":common.Policy,
	                    }
	                },
	            },
	        },
	    },
	},

Note that each consortium defines a set of members, just like the organizational members for the ordering orgs. Each consortium also defines a `ChannelCreationPolicy`. This is a policy which is applied to authorize channel creation requests. Typically, this value will be set to an `ImplicitMetaPolicy` requiring that the new members of the channel sign to authorize the channel creation. More details about channel creation follow later in this document.

注意，每个consortium定义一组成员，就行ordering组织的组织成员一样。每个consortium也都定义了一个`ChannelCreationPolicy`，它是一种应用于授权channel创建请求的策略，通常这个值设为`ImplicitMetaPolicy`，要求channel的新成员签名授权channel创建。有关channel创建更信息的内容，请参阅文档后面的内容。

## Application channel configuration / APP channel 配置

Application configuration is for channels which are designed for application type transactions. It is defined as follows:

应用程序配置用于为应用类型交易设计的channel。其定义如下：
	
	&ConfigGroup{
	    Groups: map<string, *ConfigGroup> {
	        "Application":&ConfigGroup{
	            Groups:map<String, *ConfigGroup> {
	                {{org_name}}:&ConfigGroup{
	                    Values:map<string, *ConfigValue>{
	                        "MSP":msp.MSPConfig,
	                        "AnchorPeers":peer.AnchorPeers,
	                    },
	                },
	            },
	        },
	    },
	}

Just like with the `Orderer` section, each organization is encoded as a group. However, instead of only encoding the `MSP` identity information, each org additionally encodes a list of `AnchorPeers`. This list allows the peers of different organizations to contact each other for peer gossip networking.

就像`Orderer`部分，每个组织被编码为一个组。然而，app channel不仅有`MSP`身份信息，每个组织都附加了一个`AnchorPeers`列表。这个列表允许不同组织的节点彼此联系。

The application channel encodes a copy of the orderer orgs and consensus options to allow for deterministic updating of these parameters, so the same `Orderer` section from the orderer system channel configuration is included. However from an application perspective this may be largely ignored.

应用程序channel通过对orderer组织和共识选项的编码，以允许对这些参数进行确定性更新，因此包含了orderer系统channel配置的相同`Orderer`部分。但从应用角度看，这会在很大程度上被忽略。

## Channel creation / 创建channel

When the orderer receives a `CONFIG_UPDATE` for a channel which does not exist, the orderer assumes that this must be a channel creation request and performs the following.

当Orderer 接收到对一个不存在的channel的`CONFIG_UPDATE`信息时，orderer就会假设这是个创建channel的请求并执行以下操作：

1. The orderer identifies the consortium which the channel creation request is to be performed for. It does this by looking at the `Consortium` value of the top level group.
	
	通过查看高层组中的`Consortium`值，orderer标识所要执行创建channel请求的consortium***（译注：这个词暂时不知翻译成什么好）***。
2. The orderer verifies that the organizations included in the `Application` group are a subset of the organizations included in the corresponding consortium and that the `ApplicationGroup` is set to `version` `1`.

	orderer验证Application组中的组织是对应的consortium中组织的一部分，并验证`ApplicationGroup`的版本是1。
3. The orderer verifies that if the consortium has members, that the new channel also has application members (creation consortiums and channels with no members is useful for testing only).

	orderer验证consortium是否有成员，新的channel也会有application成员（创建没有成员的consortiums和channel只用于测试）。
4. The orderer creates a template configuration by taking the `Orderer` group from the ordering system channel, and creating an `Application` group with the newly specified members and specifying its `mod_policy` to be the `ChannelCreationPolicy` as specified in the consortium config. Note that the policy is evaluated in the context of the new configuration, so a policy requiring `ALL` members, would require signatures from all the new channel members, not all the members of the consortium.

	orderer从ordering系统channel取得`Orderer`组，并创建一个包含新指定成员的`Application`组，并将其`mod_policy`指定为在consortium config中指定的`ChannelCreationPolicy`，从而创建一个模板配置。注意，这个策略（mod_policy）是基于新配置的上下文的，因此需要所有成员的策略就是要需要新channel中所有成员的签名，而不是consortium中的所有成员。
5. The orderer then applies the `CONFIG_UPDATE` as an update to this template configuration. Because the `CONFIG_UPDATE` applies modifications to the `Application` group (its `version` is 1), the config code validates these updates against the `ChannelCreationPolicy`. If the channel creation contains any other modifications, such as to an individual org’s anchor peers, the corresponding mod policy for the element will be invoked.

	orderer用`CONFIG_UPDATE`更新这个模板配置。因为`CONFIG_UPDATE`用于`Application`组（其版本是1）的修改，所以配置码根据`ChannelCreationPolicy`验证这些更新。如果channel创建包含任何其它修改，例如修改单个组织的锚节点，则调用该元素的相应mod策略。
6. The new `CONFIG` transaction with the new channel config is wrapped and sent for ordering on the ordering system channel. After ordering, the channel is created.

	带有新channel配置的`CONFIG`交易被包装并通过order系统channel发送到ordering，ordering之后channel就创建完成。