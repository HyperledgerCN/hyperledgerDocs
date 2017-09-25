
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/configtxgen.html) | Fei Cao |  |


This document describe the usage for the configtxgen utility for manipulating Hyperledger Fabric channel configuration.

本文档描述了`configtxgen`工具的用法，该工具用来操作超级账本Fabric的通道配置。

For now, the tool is primarily focused on generating the genesis block for bootstrapping the orderer, but it is intended to be enhanced in the future for generating new channel configurations as well as reconfiguring existing channels.

目前，该工具主要侧重于生成引导共识节点的创世纪块，但是将来预计增加生成新通道的配置以及重新配置已有的通道。

## Configuration Profiles - 配置文件

The configuration parameters supplied to the configtxgen tool are primarily provided by the configtx.yaml file. This file is located at fabric/sampleconfig/configtx.yaml in the fabric.git repository.

configtxgen工具的配置参数主要由`configtx.yaml`文件提供。在fabric库中，配置文件在`fabric/sampleconfig/configtx.yaml`。

This configuration file is split primarily into three pieces.

此配置文件主要分为三部分。

1.The Profiles section. By default, this section includes some sample configurations which can be used for development or testing scenarios, and refer to crypto material present in the fabric.git tree. These profiles can make a good starting point for construction a real deployment profile. The configtxgen tool allows you to specify the profile it is operating under by passing the -profile flag. Profiles may explicitly declare all configuration, but usually inherit configuration from the defaults in (3) below.

1.`Profiles`部分。默认情况下，这部分包含一些用于开发或测试场景的示例配置，这些配置涉及fabric目录中加密部分。这些配置能为构建一个真正开发配置做一个良好开始。`configtxgen`工具允许你通过`-profile`标签来指定配置文件。`Profiles`部分可以显式声明所有配置，但是通常都是从一下（3）默认配置中继承。

2.The Organizations section. By default, this section includes a single reference to the sampleconfig MSP definition. For production deployments, the sample organization should be removed, and the MSP definitions of the network members should be referenced and defined instead. Each element in the Organizations section should be tagged with an anchor label such as &orgName which will allow the definition to be referenced in the Profiles sections.

2.`Organizations`部分。默认情况下，这部分包含实力配置MSP定义的单一引用。对于生产部署，应该删除示例配置，并应引用和定义网络成员的MSP定义。`Organizations`部分每一个元素都必须带有锚标签，如`&orgName`，这些标签在`Profiles`部分引用。

3.The default sections. There are default sections for Orderer and Application configuration, these include attributes like BatchTimeout and are generally used as the base inherited values for the profiles.

3.默认部分。此部分包括共识和应用部分的默认配置，包括一些属性配置，如`BatchTimeout`和一般用作继承的基础值。

This configuration file may be edited, or, individual properties may be overridden by setting environment variables, such as CONFIGTX_ORDERER_ORDERERTYPE=kafka. Note that the Profiles element and profile name do not need to be specified.

这个配置文件可以被编辑，或者通过设置环境变量来重写属性值，如`CONFIGTX_ORDERER_ORDERERTYPE=kafka`。注意，不需要指定配置文件元素和配置文件名称。

## Bootstrapping the orderer - 引导共识

After creating a configuration profile as desired, simply invoke

创建配置文件后，简单调用

~~~
configtxgen -profile <profile_name>
~~~

This will produce a genesis.block file in the current directory. You may optionally specify another filename by passing in the -path parameter, or, you may skip the writing of the file by passing the dryRun parameter if you simply wish to test parsing of the file.

这会在当前文件目录下生成`genesis.block`。你也可以通过`-path`参数指定其他文件名。如果你只希望测试这个文件，可以通过`dryRun`参数来跳过创建文件。

Then, to utilize this genesis block, before starting the orderer, simply specify ORDERER_GENERAL_GENESISMETHOD=file and ORDERER_GENERAL_GENESISFILE=$PWD/genesis.block or modify the orderer.yaml file to encode these values.

然后为了使用生成的创世快，在启动orderer之前，简单的通过指定`ORDERER_GENERAL_GENESISMETHOD=file` 和`ORDERER_GENERAL_GENESISFILE=$PWD/genesis.block` 或者修改`orderer.yaml`文件编辑这些属性值。

## Creating a channel - 创建通道

The tool can also output a channel creation tx by executing

此工具同样可以创建通道交易通过执行

~~~
configtxgen -profile <profile_name> -channelID <channel_name> -outputCreateChannelTx <tx_filename>
~~~

This will output a marshaled Envelope message which may be sent to broadcast to create a channel.

这将输出一个`Envelope`消息，用来发送广播来创建通道。

## Reviewing a configuration - 检查配置

In addition to creating configuration, the configtxgen tool is also capable of inspecting configuration.

除了创建配置，`configtxgen`工具同样提供检查配置的功能。

It supports inspecting both configuration blocks, and configuration transactions. You may use the inspect flags -inspectBlock and -inspectChannelCreateTx respectively with the path to a file to inspect to output a human readable (JSON) representation of the configuration.

它支持检查配置块和配置交易。你可以用检查标签`-inspectBlock` 和 `-inspectChannelCreateTx` 分别指定文件路径来输出可读的（JSON）配置。

You may even wish to combine the inspection with generation. For example:

你甚至可能希望将创建与检查相结合。例如：

~~~
$ build/bin/configtxgen -channelID foo -outputBlock foo.block -inspectBlock foo.block
2017/03/01 21:24:24 Loading configuration
2017/03/01 21:24:24 Checking for configtx.yaml at:
2017/03/01 21:24:24 Checking for configtx.yaml at:
2017/03/01 21:24:24 Checking for configtx.yaml at: /home/yellickj/go/src/github.com/hyperledger/fabric/common/configtx/tool
2017/03/01 21:24:24 map[orderer:map[BatchSize:map[MaxMessageCount:10 AbsoluteMaxBytes:99 MB PreferredMaxBytes:512 KB] Kafka:map[Brokers:[127.0.0.1:9092]] Organizations:<nil> OrdererType:solo Addresses:[127.0.0.1:7050] BatchTimeout:10s] application:map[Organizations:<nil>] profiles:map[SampleInsecureSolo:map[Orderer:map[BatchTimeout:10s BatchSize:map[MaxMessageCount:10 AbsoluteMaxBytes:99 MB PreferredMaxBytes:512 KB] Kafka:map[Brokers:[127.0.0.1:9092]] Organizations:<nil> OrdererType:solo Addresses:[127.0.0.1:7050]] Application:map[Organizations:<nil>]] SampleInsecureKafka:map[Orderer:map[Addresses:[127.0.0.1:7050] BatchTimeout:10s BatchSize:map[AbsoluteMaxBytes:99 MB PreferredMaxBytes:512 KB MaxMessageCount:10] Kafka:map[Brokers:[127.0.0.1:9092]] Organizations:<nil> OrdererType:kafka] Application:map[Organizations:<nil>]] SampleSingleMSPSolo:map[Orderer:map[OrdererType:solo Addresses:[127.0.0.1:7050] BatchTimeout:10s BatchSize:map[MaxMessageCount:10 AbsoluteMaxBytes:99 MB PreferredMaxBytes:512 KB] Kafka:map[Brokers:[127.0.0.1:9092]] Organizations:[map[Name:SampleOrg ID:DEFAULT MSPDir:msp BCCSP:map[Default:SW SW:map[Hash:SHA3 Security:256 FileKeyStore:map[KeyStore:<nil>]]] AnchorPeers:[map[Host:127.0.0.1 Port:7051]]]]] Application:map[Organizations:[map[Name:SampleOrg ID:DEFAULT MSPDir:msp BCCSP:map[Default:SW SW:map[Hash:SHA3 Security:256 FileKeyStore:map[KeyStore:<nil>]]] AnchorPeers:[map[Port:7051 Host:127.0.0.1]]]]]]] organizations:[map[Name:SampleOrg ID:DEFAULT MSPDir:msp BCCSP:map[Default:SW SW:map[Hash:SHA3 Security:256 FileKeyStore:map[KeyStore:<nil>]]] AnchorPeers:[map[Host:127.0.0.1 Port:7051]]]]]
2017/03/01 21:24:24 Generating genesis block
2017/03/01 21:24:24 Writing genesis block
2017/03/01 21:24:24 Inspecting block
2017/03/01 21:24:24 Parsing genesis block
Config for channel: foo
{
    "": {
        "Values": {},
        "Groups": {
            "/Channel": {
                "Values": {
                    "HashingAlgorithm": {
                        "Version": "0",
                        "ModPolicy": "",
                        "Value": {
                            "name": "SHA256"
                        }
                    },
                    "BlockDataHashingStructure": {
                        "Version": "0",
                        "ModPolicy": "",
                        "Value": {
                            "width": 4294967295
                        }
                    },
                    "OrdererAddresses": {
                        "Version": "0",
                        "ModPolicy": "",
                        "Value": {
                            "addresses": [
                                "127.0.0.1:7050"
                            ]
                        }
                    }
                },
                "Groups": {
                    "/Channel/Orderer": {
                        "Values": {
                            "ChainCreationPolicyNames": {
                                "Version": "0",
                                "ModPolicy": "",
                                "Value": {
                                    "names": [
                                        "AcceptAllPolicy"
                                    ]
                                }
                            },
                            "ConsensusType": {
                                "Version": "0",
                                "ModPolicy": "",
                                "Value": {
                                    "type": "solo"
                                }
                            },
                            "BatchSize": {
                                "Version": "0",
                                "ModPolicy": "",
                                "Value": {
                                    "maxMessageCount": 10,
                                    "absoluteMaxBytes": 103809024,
                                    "preferredMaxBytes": 524288
                                }
                            },
                            "BatchTimeout": {
                                "Version": "0",
                                "ModPolicy": "",
                                "Value": {
                                    "timeout": "10s"
                                }
                            },
                            "IngressPolicyNames": {
                                "Version": "0",
                                "ModPolicy": "",
                                "Value": {
                                    "names": [
                                        "AcceptAllPolicy"
                                    ]
                                }
                            },
                            "EgressPolicyNames": {
                                "Version": "0",
                                "ModPolicy": "",
                                "Value": {
                                    "names": [
                                        "AcceptAllPolicy"
                                    ]
                                }
                            }
                        },
                        "Groups": {}
                    },
                    "/Channel/Application": {
                        "Values": {},
                        "Groups": {}
                    }
                }
            }
        }
    }
}
~~~

