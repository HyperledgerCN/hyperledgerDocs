
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html) | Dinghao Liu | Yunxiao Wang, Bei Wang |

# 成员服务提供者 （MSP）

The document serves to provide details on the setup and best practices for MSPs.

本文提供MSP的设置细节，致力于打造MSP的最佳实践。

Membership service provider (MSP) is a component that aims to offer an abstraction of a membership operation architecture.

成员服务提供者（MSP）是一个提供抽象化成员操作框架的组件。

In particular, MSP abstracts away all cryptographic mechanisms and protocols behind issuing and validating certificates, and user authentication. An MSP may define their own notion of identity, and the rules by which those identities are governed (identity validation) and authenticated (signature generation and verification).

特别地，MSP将颁发与校验证书，以及用户认证背后的所有密码学机制与协议都抽象了出来。一个MSP可以自己定义身份，以及身份的管理（身份验证）与认证（生成与验证签名）规则。

A Hyperledger Fabric blockchain network can be governed by one or more MSPs. This provides modularity of membership operations, and interoperability across different membership standards and architectures.

一个Hyperledger Fabric区块链网络可以被一个或多个MSP管理。这提供了模块化的成员操作，以及兼容不同成员标准与架构的互操作性。

In the rest of this document we elaborate on the setup of the MSP implementation supported by Hyperledger Fabric, and discuss best practices concerning its use.

接下来我们将详细说明在Hyperledger Fabric支持下的MSP的实现步骤，并讨论其使用方面的最佳实践方式。

##MSP配置

To setup an instance of the MSP, its configuration needs to be specified locally at each peer and orderer (to enable peer, and orderer signing), and on the channels to enable peer, orderer, client identity validation, and respective signature verification (authentication) by and for all channel members.

要想初始化一个MSP实例，每一个peer节点和orderer节点都需要在本地指定其配置，并在channel上启用peer节点、orderer节点及client的身份的验证与各自的签名验证。注意channel上的全体成员均参与此过程。

Firstly, for each MSP a name needs to be specified in order to reference that MSP in the network (e.g. `msp1`, `org2`, and `org3.divA`). This is the name under which membership rules of an MSP representing a consortium, organization or organization division is to be referenced in a channel. This is also referred to as the MSP Identifier or MSP ID. MSP Identifiers are required to be unique per MSP instance. For example, shall two MSP instances with the same identifier be detected at the system channel genesis, orderer setup will fail.

首先， 为了方便地在网络中引用MSP，每个MSP都需要一个特定的名字（例如`msp1`、`org2`、`org3.divA`）。在一个channel中，当MSP的成员管理规则表示一个团体，组织或组织分工时，该名称会被引用。这又被成为MSP标识符或MSP ID。对于每个MSP实例，MSP标识符都必须独一无二。举个例子：系统channel创建时如果检测到两个MSP有相同的标识符，那么orderer节点的启动将以失败告终。

In the case of default implementation of MSP, a set of parameters need to be specified to allow for identity (certificate) validation and signature verification. These parameters are deduced by **RFC5280**, and include:

在MSP的默认情况下，身份（证书）验证与签名验证需要指定一组参数。这些参数推导自**RFC5280**，具体包括：

* A list of self-signed (X.509) certificates to constitute the root of trust

    * 一个自签名的证书列表（满足X.509标准）以构成信任源

* A list of X.509 certificates to represent intermediate CAs this provider considers for certificate validation; these certificates ought to be certified by exactly one of the certificates in the root of trust; intermediate CAs are optional parameters

    * 一个用于表示该MSP验证过的中间CA的X.509的证书列表，用于证书的校验。这些证书应该被信任源的一个证书所认证；中间的CA则是可选参数

* A list of X.509 certificates with a verifiable certificate path to exactly one of the certificates of the root of trust to represent the administrators of this MSP; owners of these certificates are authorized to request changes to this MSP configuration (e.g. root CAs, intermediate CAs)

    * 一个具有可验证路径的X.509证书列表（该路径通往信任源的一个证书），以表示该MSP的管理员。这些证书的所有者对MSP配置的更改要求都是经过授权的（例如根CA，中间CA）

* A list of Organizational Units that valid members of this MSP should include in their X.509 certificate; this is an optional configuration parameter, used when, e.g., multiple organisations leverage the same root of trust, and intermediate CAs, and have reserved an OU field for their members

    * 一个组织单元列表，该MSP的合法成员应该将其包含进他们的X.509证书。这是一个可选的配置参数，举个例子：当多个组织使用相同信任源、中间CA以及组织为他们的成员保留了一个OU区的时候，会配置此参数

* A list of certificate revocation lists (CRLs) each corresponding to exactly one of the listed (intermediate or root) MSP Certificate Authorities; this is an optional parameter

    * 一个证书吊销列表（CRLs）的清单，清单的每一项对应于一个已登记的（中间的或根）MSP证书颁发机构（CA），这是一个可选的参数

* A list of self-signed (X.509) certificates to constitute the *TLS root of trust* for TLS certificate.

    * 一个自签名的证书列表（满足X.509标准）以构成TLS信任源，服务于TLS证书

* A list of X.509 certificates to represent intermediate TLS CAs this provider considers; these certificates ought to be certified by exactly one of the certificates in the TLS root of trust; intermediate CAs are optional parameters.

    * 一个表示该provider关注的中间TLS CA的X.509证书列表。这些证书应该被TLS信任源的一个证书所认证；中间的CA则是可选参数

*Valid* identities for this MSP instance are required to satisfy the following conditions:

对于该MSP实例，*有效的*身份应符合以下条件：

* They are in the form of X.509 certificates with a verifiable certificate path to exactly one of the root of trust certificates

    * 它们应符合X.509证书标准，且具有一条可验证的路径（该路径通往信任源的一个证书）

* They are not included in any CRL

    * 它们没有包含在任何CRL中

* And they list one or more of the Organizational Units of the MSP configuration in the OU field of their X.509 certificate structure.

    * 它们列出了一个或多个MSP配置的组织单元（列出的位置是它们X.509证书结构的OU区内）。

For more information on the validity of identities in the current MSP implementation we refer the reader to **MSP Identity Validity Rules**.

关于当前MSP实现过程中身份验证的更多信息，我们隆重推荐各位读者阅读[**MSP Identity Validity Rules**](http://hyperledger-fabric.readthedocs.io/en/latest/msp-identity-validity-rules.html)。

In addition to verification related parameters, for the MSP to enable the node on which it is instantiated to sign or authenticate, one needs to specify:

除了验证相关参数外，为了使MSP可以对已实例化的节点进行签名或认证，需要指定： 

* The signing key used for signing by the node (currently only ECDSA keys are supported), and

    * 用于节点签名的签名密钥（目前只支持ECDSA密钥）

* The node’s X.509 certificate, that is a valid identity under the verification parameters of this MSP

    * 节点的X.509证书，对MSP验证参数机制而言是一个有效的身份

It is important to note that MSP identities never expire; they can only be revoked by adding them to the appropriate CRLs. Additionally, there is currently no support for enforcing revocation of TLS certificates.

值得注意的是，MSP身份永远不会过期；它们只能通过添加到合适的CRL上来被撤销。此外，现阶段不支持吊销TLS证书。

##如何生成MSP证书及其签名密钥？

To generate X.509 certificates to feed its MSP configuration, the application can use [**Openssl**](https://www.openssl.org/). We emphasise that in Hyperledger Fabric there is no support for certificates including RSA keys.

要想生成X.509证书以满足MSP配置，应用程序可以使用[**Openssl**](https://www.openssl.org/)。我们必须强调：在Hyperledger Fabric中，不支持包括RSA密钥在内的证书。

Alternatively one can use `cryptogen` tool, whose operation is explained in **Getting Started**.

另一个选择是使用`cryptogen`工具，其操作方法详见**快速入门**章节。

**Hyperledger Fabric CA** can also be used to generate the keys and certificates needed to configure an MSP.

**Hyperledger Fabric CA**也可用于生成配置MSP所需的密钥及证书。

## peer&orderer侧 MSP 的设置

To set up a local MSP (for either a peer or an orderer), the administrator should create a folder (e.g. `$MY_PATH/mspconfig`) that contains six subfolders and a file:

要想（为peer节点或orderer节点）建立本地MSP，管理员应创建一个文件夹（如`$MY_PATH/mspconfig`）并在其下包含6个子文件夹与一个文件：

* a folder `admincerts` to include PEM files each corresponding to an administrator certificate

    * 文件夹`admincerts`包含如下PEM文件：每个PEM文件对应于一个管理员证书

* a folder `cacerts` to include PEM files each corresponding to a root CA’s certificate

    * 文件夹`cacerts`包含如下PEM文件：每个PEM文件对应于一个根CA的证书

* (optional) a folder `intermediatecerts` to include PEM files each corresponding to an intermediate CA’s certificate

    * （可选）文件夹`intermediatecerts`包含如下PEM文件：每个PEM文件对应于一个中间CA的证书

* (optional) a file `config.yaml` to include information on the considered OUs; the latter are defined as pairs of `<Certificate,OrganizationalUnitIdentifier>` entries of a yaml array called `OrganizationalUnitIdentifiers`, where `Certificate` represents the relative path to the certificate of the certificate authority (root or intermediate) that should be considered for certifying members of this organizational unit (e.g. ./cacerts/cacert.pem), and `OrganizationalUnitIdentifier` represents the actual string as expected to appear in X.509 certificate OU-field (e.g. “COP”) 

    * （可选）文件`config.yaml`包含相关OU的信息；后者作为`<Certificate,OrganizationalUnitIdentifier>`（一个被称为`OrganizationalUnitIdentifiers`的yaml数组的项）的一部分被定义；其中`Certificate`表示通往（根或中间）CA的证书的相对路径，这些CA用于为组织成员发证（如./cacerts/cacert.pem）；`OrganizationalUnitIdentifier`表示预期会出现在X.509证书中的实际字符串（如“COP”）


* (optional) a folder `crls` to include the considered CRLs

    * （可选）文件夹`crls`包含相关CRL

* a folder `keystore` to include a PEM file with the node’s signing key; we emphasise that currently RSA keys are not supported

    * 文件夹`keystore`包含一个PEM文件及节点的签名密钥；我们必须强调：现阶段还不支持RSA密钥

* a folder `signcerts` to include a PEM file with the node’s X.509 certificate

    * 文件夹`signcerts`包含一个PEM文件及节点的X.509证书

* (optional) a folder `tlscacerts` to include PEM files each corresponding to a TLS root CA’s certificate

    * （可选）文件夹`tlscacerts`包含如下PEM文件：每个PEM文件对应于一个根TLS根CA的证书

* (optional) a folder `tlsintermediatecerts` to include PEM files each corresponding to an intermediate TLS CA’s certificate

    * （可选）文件夹`tlsintermediatecerts`包含如下PEM文件：每个PEM文件对应于一个中间TLS CA的证书

In the configuration file of the node (core.yaml file for the peer, and orderer.yaml for the orderer), one needs to specify the path to the mspconfig folder, and the MSP Identifier of the node’s MSP. The path to the mspconfig folder is expected to be relative to FABRIC_CFG_PATH and is provided as the value of parameter `mspConfigPath` for the peer, and `LocalMSPDir` for the orderer. The identifier of the node’s MSP is provided as a value of parameter `localMspId` for the peer and `LocalMSPID` for the orderer. These variables can be overriden via the environment using the CORE prefix for peer (e.g. CORE_PEER_LOCALMSPID) and the ORDERER prefix for the orderer (e.g. ORDERER_GENERAL_LOCALMSPID). Notice that for the orderer setup, one needs to generate, and provide to the orderer the genesis block of the system channel. The MSP configuration needs of this block are detailed in the next section.

在节点的配置文件中（对peer节点而言配置文件是core.yaml文件，对orderer节点而言则是orderer.yaml文件），我们需要指定到mspconfig文件夹的路径，以及节点的MSP的MSP标识符。到mspconfig文件夹的路径预期是一个对FABRIC_CFG_PATH的相对路径，且会作为参数`mspConfigPath`和`LocalMSPDir`的值分别提供给peer节点和orderer节点。节点的MSP的MSP标识符则会作为参数`localMspId`和`LocalMSPID`的值分别提供给peer节点和orderer节点。运行环境可以通过为peer使用CORE前缀（例如CORE_PEER_LOCALMSPID）及为orderer使用ORDERER前缀（例如 ORDERER_GENERAL_LOCALMSPID）对以上变量进行覆写。注意：对于orderer的设置，我们需要生成并为orderer提供系统channel的创世区块。MSP配置对该区块的需求详见后面的章节。

*Reconfiguration* of a “local” MSP is only possible manually, and requires that the peer or orderer process is restarted. In subsequent releases we aim to offer online/dynamic reconfiguration (i.e. without requiring to stop the node by using a node managed system chaincode).

对“本地”的MSP进行*重新配置*只能手动进行，且该过程需要重启peer节点和orderer节点。在随后的版本中我们计划提供在线/动态的重新配置的功能（通过使用一个由节点管理的系统chaincode，使得我们不必停止node）。

##Channel MSP 的设置

At the genesis of the system, verification parameters of all the MSPs that appear in the network need to be specified, and included in the system channel’s genesis block. Recall that MSP verification parameters consist of the MSP identifier, the root of trust certificates, intermediate CA and admin certificates, as well as OU specifications and CRLs. The system genesis block is provided to the orderers at their setup phase, and allows them to authenticate channel creation requests. Orderers would reject the system genesis block, if the latter includes two MSPs with the same identifier, and consequently the bootstrapping of the network would fail.

在系统起始阶段，我们需要指定在网络中出现的所有MSP的验证参数，且这些参数需要在系统channel的创世区块中指定。前文我们提到，MSP的验证参数包括MSP标识符、信任源证书、中间CA和管理员的证书，以及OU说明和CLR。系统的创世区块会在orderer节点设置阶段被提供给它们，且允许它们批准创建channel的请求。如果创世区块包含两个有相同标识符的MSP，那么orderer节点将拒绝系统创世区块，导致网络引导程序执行失败。

For application channels, the verification components of only the MSPs that govern a channel need to reside in the channel’s genesis block. We emphasise that it is **the responsibility of the application** to ensure that correct MSP configuration information is included in the genesis blocks (or the most recent configuration block) of a channel prior to instructing one or more of their peers to join the channel.

对于应用程序channel，创世区块中需要包含管理channel的那部分MSP的验证组件。我们在此强调，**应用程序要肩负以下责任**：在令一个或多个peer节点加入到channel中之前，确保channel的创世区块（或最新的配置区块）包含正确的MSP配置信息。

When bootstrapping a channel with the help of the configtxgen tool, one can configure the channel MSPs by including the verification parameters of MSP in the mspconfig folder, and setting that path in the relevant section in `configtx.yaml`.

在configtxgen工具的帮助下引导架设channel时，我们这样来配置channel MSP：将MSP的验证参数加入mspconfig文件夹，并将该路径加入到`configtx.yaml`文件的相关部分。

Reconfiguration of an MSP on the channel, including announcements of the certificate revocation lists associated to the CAs of that MSP is achieved through the creation of a `config_update` object by the owner of one of the administrator certificates of the MSP. The client application managed by the admin would then announce this update to the channels in which this MSP appears.

要想对channel中MSP的重新配置，包括发布与MSP的CA相关的证书吊销列表，需要通过MSP管理员证书的所有者创建`config_update`对象来实现。由管理员管理的客户端应用将向该MSP所在的各个channel发布更新。

##最好的实践

In this section we elaborate on best practices for MSP configuration in commonly met scenarios.

在本节，我们将详述一般情况下MSP配置的最佳实践。

**1) Mapping between organizations/corporations and MSPs**

**为组织与MSP建立映射**

We recommend that there is a one-to-one mapping between organizations and MSPs. If a different mapping type of mapping is chosen, the following needs to be to considered:

我们建议组织和MSP之间建立一一映射。如果选择其他类型的映射，那么需要注意以下几点：

* **One organization employing various MSPs.** This corresponds to the case of an organization including a variety of divisions each represented by its MSP, either for management independence reasons, or for privacy reasons. In this case a peer can only be owned by a single MSP, and will not recognize peers with identities from other MSPs as peers of the same organization. The implication of this is that peers may share through gossip organization-scoped data with a set of peers that are members of the same subdivision, and NOT with the full set of providers constituting the actual organization.

    * **一个组织对应多个MSP。**这对应于下面这种情况：（无论出于独立管理的原因还是私人原因）一个组织有各种各样的部门，每个部门以其MSP为代表。在这种情况下，一个peer节点只能被单个MSP拥有，且不会识别相同组织内标识在其他MSP的节点。这就是说，peer节点可以与相同子分支下的一系列其他peer节点共享组织数据，而不是所有构成组织的节点。

* **Multiple organizations using a single MSP.** This corresponds to a case of a consortium of organisations that are governed by similar membership architecture. One needs to know here that peers would propagate organization-scoped messages to the peers that have an identity under the same MSP regardless of whether they belong to the same actual organization. This is a limitation of the granularity of MSP definition, and/or of the peer’s configuration.

    * **多个组织对应一个MSP。**这对应于下面这种情况：一个由相似成员结构所管理的组织联盟。这时，peer节点可以与相同MSP下的其他节点互发组织范围的数据，节点是否属于同一组织并不重要。这对于MSP的定义及peer节点的配置是个限制。

**2) One organization has different divisions (say organizational units), to which it wants to grant access to different channels.**

**一个组织有多个分支（称为组织单元），各个分支连接到组织想要获取访问权限的不同channel**

有两个方法进行处理：

* **Define one MSP to accommodate membership for all organization’s members.** Configuration of that MSP would consist of a list of root CAs, intermediate CAs and admin certificates; and membership identities would include the organizational unit (OU) a member belongs to. Policies can then be defined to capture members of a specific OU, and these policies may constitute the read/write policies of a channel or endorsement policies of a chaincode. A limitation of this approach is that gossip peers would consider peers with membership identities under their local MSP as members of the same organization, and would consequently gossip with them organisation-scoped data (e.g. their status).

    * **定义一个MSP来容纳所有组织的全部成员。**MSP的配置包含一个根CA、中间CA和管理员证书的列表；成员身份会包含一个组织单元（OU）的所属关系。接下来可以定义用于获取特定OU成员的策略，这些策略可以建立channel的读写策略或者chaincode的背书策略。这种方法的局限是gossip peer节点会本地MSP下的其他peer节点当做相同组织内的成员，并与之分享组织范围内的数据。

* **Defining one MSP to represent each division.** This would involve for each division, a set of certificates for root CAs, intermediate CAs, and admin specifying Certs, such that there is no overlapping certification path across MSPs. This would mean that, for example, a different intermediate CA per subdivision is employed. Here the disadvantage is the management of more than one MSPs instead of one, but this circumvents the issue present in the previous approach. One could also define one MSP for each division by leveraging an OU extension of the MSP configuration.

    * **定义一个MSP来表示每个分支。**这需要为每个分支引入一组根CA证书、中间CA证书和管理员证书，这样每条通往MSP的路径都不会重叠。这意味着，每个子分支的不同中间CA都会被利用起来。这样做的缺点是要管理多个MSP，不过这避免了前面方法出现的问题。我们也可以利用MSP配置的OU扩展来为每个分支定义一个MSP。

**3) Separating clients from peers of the same organization.**

**将客户从相同组织的peer节点中分离**

In many cases it is required that the “type” of an identity is retrievable from the identity itself (e.g. it may be needed that endorsements are guaranteed to have derived by peers, and not clients or nodes acting solely as orderers).

多数情况下，一个身份的“类型”被要求能够从身份本身获取（可能当背书要保证：背书节点由peers充当，而非客户端或者仅充当orders的节点时，需要该特性支持）。

There is limited support for such requirements.

下面是对这些要求的有限支持。

One way to allow for this separation is to to create a separate intermediate CA for each node type - one for clients and one for peers/orderers; and configure two different MSPs - one for clients and one for peers/orderers. Channels this organization should be accessing would need to include both MSPs, while endorsement policies will leverage only the MSP that refers to the peers. This would ultimately result in the organization being mapped to two MSP instances, and would have certain consequences on the way peers and clients interact.

一种支持这种分离的方法是为每个节点类型创建一个分离的中间CA：一个为客户，一个为peer节点或orderer节点；并配置两个不同的MSP：一个为客户，一个为peer节点或orderer节点。该组织要访问的channel需要同时包含两个MSP，不过背书策略将只用到服务peer节点的MSP。这最终导致组织与两个MSP实例建立映射，并对peer节点与客户间的交流产生特定影响。

Gossip would not be drastically impacted as all peers of the same organization would still belong to one MSP. Peers can restrict the execution of certain system chaincodes to local MSP based policies. For example, peers would only execute “joinChannel” request if the request is signed by the admin of their local MSP who can only be a client (end-user should be sitting at the origin of that request). We can go around this inconsistency if we accept that the only clients to be members of a peer/orderer MSP would be the administrators of that MSP.

由于所以同一组织的peer节点仍属于相同的MSP，所以通讯不会受到严重影响。peer节点可以把特定系统chaincode的执行控制在本地MSP的策略范围内。例如：只有请求被本地MSP的管理员签署（其只能是一个客户），peer节点才会执行“joinChannel”的请求（终端用户应该处于该请求的起点）。如果我们接受这样一个前提：只有客户成为MSP peer节点或orderer节点的一员，才能成员MSP的管理员，那么我们就可以绕过这个矛盾。

Another point to be considered with this approach is that peers authorize event registration requests based on membership of request originator within their local MSP. Clearly, since the originator of the request is a client, the request originator is always doomed to belong to a different MSP than the requested peer and the peer would reject the request.

该方法还要注意，peer节点授权事件登记的请求，是基于本地MSP内请求的发起成员。简而言之，由于请求的发起者是一个客户，故请求发起者必定隶属于和被请求的peer节点不同的MSP，这会导致peer节点拒绝该请求。

**4) Admin and CA certificates.**

**管理员和CA的证书**

It is important to set MSP admin certificates to be different than any of the certificates considered by the MSP for `root of trust`, or intermediate CAs. This is a common (security) practice to separate the duties of management of membership components from the issuing of new certificates, and/or validation of existing ones.

将MSP管理员证书设置得与任何MSP，或中间CA处理的其他证书都不同是很重要的。这是一种常见的安全做法，即将成员管理的责任从发行新证书与验证已有证书中拆分出来。

**5) Blacklisting an intermediate CA.**

**将中间CA加入黑名单**

As mentioned in previous sections, reconfiguration of an MSP is achieved by reconfiguration mechanisms (manual reconfiguration for the local MSP instances, and via properly constructed `config_update` messages for MSP instances of a channel). Clearly, there are two ways to ensure an intermediate CA considered in an MSP is no longer considered for that MSP’s identity validation:

就像上文所述，重新配置MSP是通过一种重配置机制完成的（手动重新配置本地MSP实例，并通过channel合理构建发送给MSP实例的`config_update`消息）。显然，我们有两种方法保证一个中间CA被MSP身份验证机制彻底忽视：

* Reconfigure the MSP to no longer include the certificate of that intermediate CA in the list of trusted intermediate CA certs. For the locally configured MSP, this would mean that the certificate of this CA is removed from the `intermediatecerts` folder.

    * 重新配置MSP并使它的*信任中间CA证书列表*不再包含该中间CA的证书。对于本地重新配置的MSP，这意味着该CA的证书从`intermediatecerts`文件夹中被删除了。

* Reconfigure the MSP to include a CRL produced by the root of trust which denounces the mentioned intermediate CA’s certificate.
    
    * 重新配置MSP并使它包含由信任源产生的CRL，该CRL会通知MSP废止中间CA证书的使用。

In the current MSP implementation we only support method (1) as it is simpler and does not require blacklisting the no longer considered intermediate CA.

在目前的MSP实现中，我们只支持上述的第一个方法，因为它更加简单，且并不需要把早就不用考虑的中间CA列入黑名单。

***5) CAs and TLS CAs*

***5) CA 和 TLS CA*

MSP identities’ root CAs and MSP TLS certificates’ root CAs (and relative intermediate CAs) need to be declared in different folders. This is to avoid confusion between different classes of certificates. It is not forbidden to reuse the same CAs for both MSP identities and TLS certificates but best practices suggest to avoid this in production.

MSP 身份的根CA及MSP TLS证书的根CA（以及相关的中间CA）需要在不同的文件夹中声明。这是为了避免混淆不同等级的证书。且MSP身份与TLS证书都允许重用相同的CA，不过我们建议最好在实际中避免这样做。




