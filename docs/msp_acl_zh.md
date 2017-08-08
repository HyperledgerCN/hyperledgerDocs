
|  原文  |  翻译  |  审核修正  |
|:-------|:-------|:-------  |
|  [原文](https://docs.google.com/document/d/1Qg7ZEccOIsrShSHSNl4kBHOFvLYRhQ3903srJ6c_AZE/edit)  |  梁永甫  |    |



# Membership Service Providers & Access Control in Hyperledger Fabric

__Authors__: Elli Androulaki, Angelo De Caro, Binh Nguyen, Alessandro Sorniotti, Murali Srinivasan, Jason Yellick

翻译：梁永甫

# 1. Terminology
A __Blockchain network__ consists of the following entities: 

区块链网络包含如下组成部分：
- Application(s) network (can include peers and clients or only clients)
- 应用程序网络（可以包含peer和client，也可以只包含client）
- Network of peers (if not part of the application network)
- peer网络（除了应用网络中的peer组成的网络）
- Ordering service (can be decentralized or centralized)
- 排序服务（可以是中心化的，也可以分布式）
- Set of protocols to facilitate the communication between the clients, peers and the ordering service, enabling the application to create one or more chains throughout the operation of the system and submit transactions to it, as well as manage the access to these chains.
- 一组通讯协议，这些协议用于：client，peer，orderer之间的通信；应用通过系统操作创建一个或者多个chain，并提交交易；管理对chains的访问。

For some of these terms we refer the reader to Hyperledger Fabric architecture.

__Ordering service__ is the component of a Blockchain network that offers atomic broadcast services. It can create one or more “atomic broadcast channels” upon authenticated request from appropriately authorized entities. The internal structure of the ordering service may differ from one implementation to the other, as it can be a service offered by one entity (centralized version) or more entities, e.g., running BFT or CFT protocols (decentralized version). In any case, ordering service should come with its client that should expose the following functionalities to ordering service users, i.e., entities that leverage atomic broadcast functionalities of the ordering service:

__排序服务__ 是区块链网络中提供原子广播服务的组件。 它能够通过一些经过认证的实体创建多个“原子广播通道” 。 根据不同的实现方式，排序服务内部结构可能不同， 它可以是中心化的，也可以是分布式的（比如运行BFT或者CFT协议的多个实体）。 不管那种方式， 排序服务和其客户端要为用户提供如下功能，

- Create and submit channel creation requests
- 创建和提交创建channel的请求
- Reconfigure channel permissions, e.g., update the channel access policies
- 重新配置channel权限，如：更新channel准入策略
- Reconfigure permissions of members concerning the ordering service functionalities (e.g., channel creation)
- 为连接到排序服务的成员重新配置权限（如创建channel）

The ordering client is also equipped with reconfiguration mechanisms that are triggered by properly authenticated transactions advertised within application or system channel. E.g., if the ordering service is decentralized, the ordering service client would need to know & understand the policies associated to updates of that ordering service’s membership mechanism, and other parameters, e.g., batch size, etc.

排序客户端也要具备重新配置的机制，这些机制可以通过在应用或者系统channel中广播交易来触发。 如，如果排序服务是分布式的， 客户端应该知道排序服务的成员策略更新机制，以及其他的一些参数，如batch size等。

__Channel__ is an atomic broadcast channel managed by the ordering service. One or more channels may be created within the ordering service after request from the application network (see below for application network definition).

__channel__是排序服务管理的一个原子广播通道。 应用可以通过发送请求的方式，在排序服务中创建多个channel。

__Chain__ is bound to a channel and is the data structure that maintains the history and state of transactions advertised within a channel. In particular, a chain consist of 

chain 和channel绑定， 是一种数据结构，用来维护channel内广播的交易的历史和状态。chain由一下组成部分：

- The total order of transactions as provided by the channel orderers
- channel 的orderer提供的全部交易序列
- The state associated to chaincodes defined within the channel (i.e., the set of transactions of the channel that are valid)
- channel中连代码的状态（channel中成功的交易）
- Membership and access control information for this chain’s operations (read-access to transactions, write-access to the state, etc).
- chain的成员和准入控制信息（读取交易，写状态等） 

__Membership service provider (MSP)__: A set of cryptographic mechanisms and protocols for issuing and validating certificates and identities in the Blockchain network. Identities issued in the scope of a membership service provider can be evaluated within that membership service provider’s rules. 

__MSP __： 一组加密机制和协议，用来在区块链网络中发放验证证书和identities 。msp中发放的identities 可以在msp中验证。


__Clients__ are defined by their membership service provider type and respective (public) configuration. The latter for the default membership service provider includes a list of root CAs and intermediate CAs, and a list of administrators.

__Clients__ 通过它的msp type来定义，默认的msp包含根ca，中间ca和管理员。

__Network of peers__ are defined either the same way as clients. A peer network, like  the client network, comes along with a description of the membership service provider and its configuration. 

__peer网络__ 用和client网络同样的方式定义。peer网络也是有msp描述和自己的配置。


__Application network__ is a broader term to cover entities that could be part of the application infrastructure. This could include a set of clients, and potentially also sets of peers. To understand this better, let’s take the example of an auctioning service that is implemented as a chaincode that runs on peers A, and B. Now, let’s assume that for the purpose of this use-case, the application service has implemented client-application software that runs on the end-user machine, and application server software that runs on the application server. Both cases leverage the client-sdk and are perceived by fabric network as clients. In this scenario the application owns the application-server side, as well as peers A, and B, that would install and instantiate/simulate the chaincodes controlled and submitted by that application.

__应用网络__ 是一个广义的定义，指那些应用架构中的实体。包括客户端和一些peer。为了更好理解， 我们举一个拍卖服务的例子，chaincode在peer A和B上运行。现在，假设应用运行在终端用户的设备上， 应用服务端运行在应用服务器上。两者都是用client-sdk，并且被fabric网络视为客户端。在这种场景下，应用包括应用服务端，peer A 和peer B。


__Interaction between the ordering service, and application entities (clients, and peers)__: This requires that the ordering service nodes have installed some piece of software to be able to minimally process messages coming from the application network. This includes membership-related functionalities, i.e., check whether a certain entity is authorized to do certain things or not, and membership-update functionalities. We will call this appshim. At the same time, as mentioned before, application network should run the ordering service client that is able to process ordering service specific messages that are advertised through a channel. We will call this osshim.

__排序服务和应用实体间的交互__： 这需要排序服务安装一些软件来处理来自应用网络的消息。包括成员相关的功能（如：检查一个实体是否有权限做某些事情）和 成员更新功能。我们称之为appshim。 同样，应用网络需要运行排序服务客户端， 处理排序服务在channel中广播的消息，我们称之为osshim。


__Application chaincodes__: Applications can deploy and invoke application chaincodes. At deploy time the owner of a chaincode (application) should specify policies that will govern the impact of invocations of that chaincode in that chaincode’s state. These policies are known as endorsement policies. These policies usually contain:

__应用链代码__： 应用程序可以部署和调用应用链代码，在部署的时候，应用要指定链代码的背书策略，管理链代码的调用方式。背书策略通常包含：

- an endorsement specific part, i.e., defining how an endorser should “endorse” such invocations (e.g., add signatures, included in the endorsing peer code in the form of ESCC), and 
- 背书方式，定义背书者如何背书一个调用（如增加自己的签名，以ESCC的形式包在含背书节点的代码中）
- a validation part, that is used by the committing peers to  assess if the endorsement policy is satisfied for a certain chaincode invocation transaction (included in the committing peer code in the form of the validation system chaincode (VSCC)). 
- 验证方式，这部分将在peer提交的时候用来验证对链代码的调用是否满足背书策略（以VSCC的形式包含在提交peer的代码中）。

Thus, at deploy time, the deployer needs to specify the id of a pair of ESCC, VSCC this chaincode should adhere to. Fabric equips the peers with default ESCC and VSCC implementations that  cover certain types of policies. Blockchain network provider may develop additional ESCC and VSCC to support specific endorsements that are not covered by the default. What is important is that VSCC given a transaction concludes to a decision in a deterministic manner. 

因此，部署的时候，部署者需要指定一对ESCC和VSCC的ID， Fabric已经实现了默认的ESCC和VSCC，包括了一些指定的策略。区块链网络提供者可以开发额外的ESCC和VSCC来支持其他的背书。重要的是VSCC能给明确的决定交易是否有效。


__Organizations__: Logical entities or corporations that constitute the stakeholders of a Blockchain network installation. Members of such organization could be authorized by that organization’s membership service providers to submit transactions to certain chains.

__组织__： 组成区块链网络的逻辑实体或者公司。组织的成员经过组织的msp认证后才能提交交易到指定的chain。


__Members__: Represent the end-users of the Blockchain network. Each organization may have one or more members and acts as the root of trust (MSP) for its members. For the default MSP used by fabric applications, each member has one long-term identity and can use its long-term identity to generate one or more ephemeral identities.

__成员__：区块链网络的终端用户。每个组织中可以有一个或多个成员，作为可信的msp来验证其他成员。 对于应用程序使用的默认MSP，每个成员都有一个长期identity ，并且可以使用其长期identity 来生成一个或多个短暂identity 。


__Mapping between an organization and an MSP__: This mapping is not enforced by the fabric configuration. It is up to the consortium of entities building their Blockchain network to decide how to leverage the modular nature of membership service providers, and the ability for many of them to co-exist in the network. 

__组织和msp的映射__：这种映射不是通过fabric配置强制执行的。这取决于区块链的参与者如何利用msp的模块化特性，以及他们在网络中共存的能力。


The simplest case would be that there is one to one mapping between an organization and a membership service provider. In this case, root certificates of the organization MSP could carry that organization’s name, that can be used as the MSP’s identifier within a chain. If an organization has more than one subdivisions, e.g., the ones that appear in the OU field of an X.509 based identity, then the identities of these divisions should be considered using the identity of the MSP/Organization as namespace for it.

最简单的情况下，一个组织和一个msp映射。这种情况下，组织msp的根证书中可以包含组织的名称，名称在可以用来做msp在chain中的identifier 。如果一个组织有多个分支。例如，分支名字作为x.509身份证书中的ou字段，那么这些分支的identity应该应该考虑用msp/organization的形式表示。 


In the last section we elaborate on best practices associated to membership service providers and their mapping to organizations.

# 2. Membership Service Providers (MSPs) in a Blockchain network
## 2.1 Definition of a Membership Service Provider
MSP定义

1. A Blockchain network may be governed by one or more MSPs. An MSP can be logically defined by the following components:

一个区块链网络可以由多个MSP管理，一个MSP包含如下组件：

2. An identity format, also known as certificate, and optionally the algorithm to generate one identity

identity格式，即证书格式，以及可选的生成生identity的算法

3. A signing algorithm that utilizes the secret associated to an identity, and a message to produce a byte array that is also bound to the identity

与identity对应的签名的算法，将identity序列化的方法。

4. A signature verification algorithm that takes as input an identity, a message, and a signature (byte array), and outputs “accept” if the signature bytes correspond to a valid signature of the input message assuming the information in the input identity; otherwise, the algorithm outputs “reject”

签名验证算法，算法输入identity，消息，签名，判断签名和identity是否匹配，匹配就输出“接受”，否则输出“拒绝”。

5. The set of rules that need to be satisfied by an identity for the identity to be considered valid for this MSP

一组判断identity是否符合MSP要求的验证规则。

A set of administration identities, that are authorized to change configuration parameters that are MSP-specific

一组管理者identity，管理者可以修改MSP的配置信息。


From an implementation perspective, many MSPs are similar in items (1) and (2), and (3) but  differ in (4) and (5). For the purpose of this document we will overload the MSP notation to refer to a unique tuple of algorithms: 

对于具体的实现，大部分MSP中，上述1,2,3条都是相似的。但是在4,5两条上有差异。本文档中，我们用下面的格式分别表示上述5条。

```bash
<MSP.id, MSP.sign, MSP.verify, MSP.validateid, MSP.admin> 
```
## 2.2 Examples

Examples of MSPs used by Peers. Here we describe how the above MSP features are instantiated in the case of two popular MSP scenarios on the peer side. Notice that peers in the network are agnostic to the identity issuing process, as their role is restricted to the endorsement of client proposals, client identity validation and client identity signature validation. 

Peer中使用的MSP的例子。 我们描述peer端两种场景下如何实现上面的特性。注意，peer并不感知identity的产生机制是（就是证书发行）。identity的功能分布在客户端提案背书， 客户端identity验证，客户端签名验证。


__Example 1: Classic MSP__. Identities (i.e., MSP.id) in this case have the form of standard X.509 certificates, that are signed by exactly one root CA. The certificate of the root CA, that can also be a commercial CA, is part of this MSP description. Signing and signature verification algorithms (i.e., MSP.sign and MSP.verify) are ECDSA-based or RSA-based depending on the key-material in the certificate.  

__场景1： 基本MSP__。 这种场景下，identity（即MSP.id）是经过根CA签名的x.509格式的证书。根CA（可以是商业的CA） 也是MSP的一部分。签名算法（MSP.sign）和签名验证算法(MSP.verify) 可以是ECDSA，也可以是RSA，具体依赖于证书中的说明。 

Validation of an identity (i.e,. MSP.validateid) in this case involves: 

这种场景下，identity验证（MSP.validateid）包括

- Verifying the correctness of the signature (chain) included in the identity (X.509 certificate) assuming the trusted root CA, 

- 验证证书中的签名链，假设相信根CA。

- Confirming that the identity is not within the list of identities that have been revoked; 

- 确认，identity不在revodedidentity列表中。

- depending on the MSP implementation this can be done either by means of “MSP identity revocation list” (IdRL), or of “MSP identity white-list” (IdWL), that are updated regularly. In the Blockchain setting the IdRL/IdWL is passed as parameter at MSP setup time, and is updated through (properly authenticated) reconfiguration messages advertized through the Blockchain.

- 上面的过程可以实现为黑名单（idRL），也可以实现为白名单（idWL），并且定期更新。在MSP启动的时候通过参数设置IdRL/IdWL， 并且可以通过在区块链网络中广播重配置消息更新这些信息。

Admin of such an MSP (i.e., MSP.admin) can be the X.509 certificate of the administrator of that MSP, i.e., the entity that can update the root CA certificate this MSP is governed by. By default, IdRLs or IdWLs are only updatable by the root CA itself, or the administrator. 

MSP的管理员（即MSP.admin）可以是MSP管理员的X.509数字证书， 管理员可以更新MSP的根CA证书。默认情况下，只有根CA自己和管理员可以更新IdRL和IdWL。

__Example 2: MSP allowing for cross-signed certificates__. As in the previous example, identities in this case also have the form of an X.509 certificates. However, in this case, the MSP client leverages standard X.509 certificate structure to accommodate within a single certificate signatures of one or more root CAs. In this case, the MSP is parameterized with a list of trusted CAs (by means of standard X.509 certificates) and the threshold number of these whose signature should appear in a valid identity. As before, validation of certificates, apart from the signature validation relates to IdRLs/IdWLs, which can be advertised by either the administrator, or the identity issuer CA.  

__场景2：MSP允许交叉签名证书__。 和上一种场景一样，identity在这种场景写也是X.509证书。 但是MSP客户端在在一个X.509证书中加入多个根CA签名。通过参数配置MSP信任的MSP列表（通过X.509证书指定）和一个有效证书中需要有的根CA签名数阀值。和以前一样，证书验证和签名验证分离， 证书验证依赖于IdRL/IdWL, 而IdRL/IdWL只能由根CA自己或者管理才能更新。 

Signing, and signature verification algorithm is an ECDSA based one or RSA based one depending on the key-material inside the certificate.  

签名算法，和签名验证算法可以是ECDSA ，也可以是RSA，在证书中指定。 

## 2.3 Generic interfaces for a fabric platform MSP
Reflecting the definition of an MSP from Section 2.1, we define generic interfaces for a membership service provider. These interfaces are shown in Figure1, and are strongly coupled with the notion of identity. Identity, that reflects the notion of publicly verifiable certificate is also defined in a generic way, through the interfaces described in Figure 2. Finally, Figure 3 depicts a SigningIdentity interface, i.e., an Identity with signing capabilities.

为了体现2.1中MSP的定义，我们定义MSP的通用接口，如下图1所示。这些接口和identity的概念是强相关的。identity体现证书的概念，identity也是以一种通用接口的形式定义，如图2所示。最后在图3中描述SigningIdentity 接口， 即具备签名功能的Identity。


```bash
// MSP is the minimal Membership Service Provider Interface to reflect Membership
// Service Provider needs to be used on the peer and ordering node side. Notice that
// on these nodes, MSP is needed for verifying purposes.
type MSP interface {

      // Setup the MSP instance according to configuration information defined through
      // an MSPConfig data structure. This struct is generic enough so as to capture
      // any MSP configuration
      Setup(config *MSPConfig) error

     // Returns the type this MSP leverages. The default MSP type is “Fabric” that implements
     // a standard X.509 certificate based signature generation and verification. Notice that such 
     // a provider type can also parse and evaluate transaction certificate signatures 
      GetType() ProviderType

      // GetIdentifier returns the identifier of this MSP; this is an identifier assigned to
      // the MSP via configuration (Setup).
      GetIdentifier() (string, error)

       // GetSigningIdentity returns a signing identity that this MSP already manages,
       // and that corresponds to the input identifier; IdentityIdentifier consists of two strings,
       // the first is the provider identifier, and the second includes the identity’s identifier
       // within the provider.
      GetSigningIdentity(identifier *IdentityIdentifier) (SigningIdentity, error)

      // GetDefaultSigningIdentity returns the default signing identity of this MSP; this is 
      // helpful in cases where there is a main signing identity that is used throughout a
      // node’s operation
      GetDefaultSigningIdentity() (SigningIdentity, error)

      // DeserializeIdentity deserializes an identity, according to this MSP’s deserialization
      // rules and instantiates an Identity object that this MSP can “understand”
      DeserializeIdentity(serializedIdentity []byte) (Identity, error)

      // Validate checks whether the supplied identity is valid under this MSP’s validation rules
      Validate(id Identity) error

     // SatisfiesPrincipal checks whether the identity matches
     // the description supplied in MSPPrincipal. The check may
     // involve a byte-by-byte comparison (if the principal is
     // a serialized identity) or may require MSP validation). MSPPrincipal 
    // functionality will be discussed in Section 2.5.2.
     SatisfiesPrincipal(id Identity, principal *common.MSPPrincipal) error
}
```
Figure1. Description of the generic platform MSP interface.


````bash
// Identity interface defining operations associated to a "certificate".  That is, the public part of the 
// identity could be thought to be a certificate, and offers solely signature verification capabilities.
// This is to be used at the peer side when verifying certificates that transactions are signed
// with, and verifying signatures that correspond to these certificates.
type Identity interface {

      // GetIdentifier returns the identifier of that identity
      GetIdentifier() *IdentityIdentifier

      // GetMSPIdentifier returns the MSP Id for this instance
      GetMSPIdentifier() string

      // Validate uses the rules that govern this identity to validate it.
      // E.g., if it is a fabric TCert implemented as identity, validate
      // will check the TCert signature against the assumed root certificate
      // authority.
      Validate() error

      // GetOrganizationalUnits returns zero or more organization units or
      // divisions this identity is related to as long as this is public
      // information. Certain MSP implementations may use attributes
      // that are publicly associated to this identity, or the identifier of
      // the root certificate authority that has provided signatures on this
      // certificate.
      // Examples:
      //  - if the identity is an x.509 certificate, this function returns one
      //    or more string which is encoded in the Subject's Distinguished Name
      //    of the type OU
      GetOrganizationalUnits() []string

      // Verify a signature over some message using this identity as reference
      Verify(msg []byte, sig []byte) error

      // Serialize converts an identity to bytes
      Serialize() ([]byte, error)

      // SatisfiesPrincipal checks whether this instance matches
      // the description supplied in MSPPrincipal. The check may
      // involve a byte-by-byte comparison (if the principal is
      // a serialized identity) or may require MSP validation
      SatisfiesPrincipal(principal *common.MSPPrincipal) error
}  
````

Figure 2  Description of a generic Identity interface devised for Fabric platform needs.

Identities equipped with the secret signing information that correspond to their public key, are called in our infrastructure __SigningIdentities__. In the case of an X.509 based MSP, Identity would be instantiated as an X.509 certificate. __SigningIdentity__ in this case, would also carry a reference to the signing key of the certificate’s public key. SigningIdentity interface is described in Figure 3.

包含和公钥对应的私钥信息的identity 称为 SigningIdentity。 Identity就是一个X.509证书。 SigningIdentity 要保存证书公钥的签名私钥（跟第一句同一个意思？）。 SigningIdentity接口定义如图3


````bash
// SigningIdentity is an extension of Identity to cover signing capabilities. E.g., signing identity
// should be requested in the case of a peer who wishes to sign proposal responses. A form
// of signing identity is also used at the client side who would sign proposals and transactions.
type SigningIdentity interface {

      // Extends Identity
      Identity

      // Sign the input message using the singing identity’s signing key
      Sign(msg []byte) ([]byte, error)
     
      // Removed SignOpts, GetAttributeProof for the same reasons as for VerifyOpts and VerifyAttributes

      // GetPublicVersion returns the public parts of this identity. In a signing identity corresponding 
      // to X.509 certificates, GetPublicVersion would output the Identity object representing the 
      // actual X.509 certificate
      GetPublicVersion() Identity

 }

````

Figure 3. Description of a generic interface for a signing identity.

## 2.4 Coupling node signing abilities with a (local) MSP

Orderers, and peers need to be equipped with signing abilities. To do so, the administrator of a node needs to specify at node setup time the configuration of the MSP that would carry the signing identity of the peer or orderer. As the MSP instance included here is created solely to instantiate the node’s signing identity, we refer to this MSP by __SignerMSP__. The latter is only possible to be updated manually by that node’s administrator, and can naturally vary from node to node. For simplicity and for V.1, to setup SignerMSP, and assuming the default MSP type for fabric, the administrator is requested to copy to dedicated location in the node’s file system four sets of files:

orderer和peer需要具备签名的能力。为了做到这一点，节点的管理员在节点启动的时候指定签名Identity的信息。这里的MSP实现了签名Identity， 我们称之为SignerMSP。创建后，只有节点的管理员可以更新，各个节点可以不同。 简单描述，在V1.0中，假设关键一个类型为Fabric的MSP， 管理员需要在节点的指定目录拷贝下面四类文件。

- cacerts: PEM files containing the root authority certificates of the MSP
- cacerts: PEM格式的MSP的根CA证书
- admincerts: PEM files containing the administrators’ certificates of this MSP
- admincerts： MSP的管理员证书
- keystore: PEM files containing the signing private key of the node
- keystore: 节点的签名私钥
- signcerts: PEM encoded certificate files corresponding to the identity of the node
- signcerts:节点Identity的证书（这个其实就是节点的证书）


In particular, the node admin is required to modify the setup .yaml file with the information depicted in the figure below. First of all, the BC crypto service provider of the node is configured, where it needs to be determined whether a software (SW) or HSM based CSP is used to store the key-material of the node.  In the example below a SW provider is configured with the location of where the key material of the peer is to reside. In addition, for the default MSP case, the node is to retrieve from its .yaml file the location of the msp-related files (cacerts, admincerts, intermediatecas, and crls) are stored (parameter mspConfigPath), and identifier of the node’s MSP (localMspId).

特殊情况下，节点的管理员要修改启动配置文件，如下所示。首先，配置BC加密服务， 要选择存储私钥信息时用的加密方式，SW或者HSM。另外，如果用默认MSP，节点会从配置的目录下获取相关的证书信息，以及节点的Msp标识（localMSpId）


````bash
# BCCSP (Blockchain crypto provider): Select which crypto implementation or
# library to use
BCCSP:
   Default: SW
   SW:
       # TODO: The default Hash and Security level needs refactoring to be
       # fully configurable. Changing these defaults requires coordination
       # SHA2 is hardcoded in several places, not only BCCSP
       Hash: SHA2
       Security: 256
       # Location of Key Store, can be subdirectory of SbftLocal.DataDir
       FileKeyStore:
           # If "", defaults to 'mspConfigPath'/keystore
           KeyStore:

# Path on the file system where peer will find MSP local configurations
mspConfigPath: msp/sampleconfig

# Identifier of the local MSP
# ----!!!!IMPORTANT!!!-!!!IMPORTANT!!!-!!!IMPORTANT!!!!----
# Deployers need to change the value of the localMspId string.
# In particular, the name of the local MSP ID of a peer needs
# to match the name of one of the MSPs in each of the channel
# that this peer is a member of. Otherwise this peer's messages
# will not be identified as valid by other nodes.
localMspId: DEFAULT



````
## 2.5 Coupling chain participation with MSPs

The genesis block of a chain must contain the specification (description) of the MSPs that govern the chain participants’ identities. If an MSP covers multiple chains, it is important that we keep the state of that MSP on each chain. This is to avoid reconfiguration inconsistency attacks that can be caused by reconfiguration transactions of the organization’s MSP arriving in each chain in a different order. 

区块链的创始块中必须包含管理参与者身份的MSP信息。如果一个MSP涉及多个链， 就需要在每个链中保存MSP的状态。这样可以避免重新配置攻击（让配置变更交易在不同的链中以不同的顺序到达节点）。


MSPs defined in the context of a chain or channel would enable the orderers and peers to authenticate chain transaction signers, endorsers, and/or creators of requests for chain/channel creation/termination,  channel broadcast & delivery and others. 

在chain或者channel环境中定义的MSP，能够使orderer和peer校验链交易签名者，背书者和重配置请求的创建者，channel广播和推送，以及其他。


In particular, MSPs specified in the orderer system channel would allow the specification of the policies governing channel readers (to authenticate & validate channel delivery requests), writers (to authenticate & validate channel broadcast requests), chainCreators (to evaluate chain creation requests), and admins (to authenticate & validate channel reconfiguration requests). MSPs specified in an application chain or channel allow the specification of policies that govern chain readers, writers, admins, and chaincodeAdmins (to authenticate and validate chaincode instantiation requests). Evidently, MSPs within a chain have a __verifier role__, that comes in contrast to the __signer role__ that local MSP(s) mean to offer. More specifically, peers and orderers are required to setup MSPs in the context of a channel (for orderer system channel) or chain to be able to authenticate transactions and configuration-related requests, and carry __no signature generation responsibility/ability__.

尤其是，orderer系统channel中指定的msp，可以定义对channel的访问策略，管理相关成员，包括：reader（向channel发获取请求），writer（向channel发送广播请求），chainCreator（发送创建链代码请求），admin（可以发送重新配置channel请求）。应用链或者channel中指定的msp，可以定义访问策略，管理相关成员：reader， writer，admins， chaincodeAdmin(可以发送链代码实例化请求)。 显然，链内的msp具有签名验证角色，与本地msp提供的签名角色相反。更详细的说，peer和orderer需要在channel（orderer 系统channel）和chain上下文中 启动msp，来认证交易或者配置相关的请求，具备生成签名的能力。


Clearly, peers and orderers need to be able to verify signatures that correspond to identities issued by multiple MSPs. To facilitate this, Hyperledger fabric introduces the concept of an MSP manager. In particular, an MSPManager interface is the fabric component that would instantiate one or more MSPs at chain setup time (which is also relevant for the orderer channel setup that takes place at orderer bootstrap), and use these to validate transaction signatures transparently to the rest of the code. MSPManager interface brings in two important advantages to the fabric. 

显然，peer和orderer 要能够验证多个msp发行的身份（Identity）的签名。为了满足这一点，Hyperledger Fabric 引入了msp管理器的概念。MSPManager 接口作为Fabric组件，在建立chain的时候（如果是orderer就是在创建channel的时候）实例化多个MSP，并利用这些来验证交易签名。MSPManager给Fabric带来两个好处。

- Pluggability of MSPs
- MSP可插拔化。
- Support for multiple MSP providers simultaneously
- 可同时提供多个MSP。
- Hiding the complexity of internal policies of a single MSP and its architecture from the rest of the MSPs in the Blockchain network.
- 对区块链网络隐藏MSP内部策略的复杂性以及其架构。

MSPManager uses the information from the configuration block of the chain (i.e., the Genesis block) to instantiate the MSPs, as shown in Figure 4.

MSPManager 使用chain的配置块信息（即创始块）实例化MSP。如下图所示。


![](img/MSP_ACL_Figure4.png)

Figure 4. Setup flow from chain configuration components, to MSPManager setup, and individual MSP setup.

从链配置组件到MSPManager和个体MSP启动流程。


The MSPManager exposes an interface to the rest of the fabric-code that is simple for components to integrate. This interface is in Figure 5.

MSPManager 暴漏简单接口给区块链的其他组件，方便其他组件 集成。接口如下：


````bash
// MSPManager is an interface defining a manager of one or more MSPs. This essentially acts
// as a mediator to MSP calls and routes MSP related calls to the appropriate MSP. This object
// is immutable, it is initialized once and never changed.
type MSPManager interface {

    // DeserializeIdentity deserializes an identity.
    // Deserialization will fail if the identity is associated to
    // an msp that is different from this one that is performing
    // the deserialization.
    DeserializeIdentity(serializedIdentity []byte) (Identity, error)


    // Setup the MSP manager instance according to configuration information
    Setup(msps []MSP) error

    // GetMSPs Provides a list of Membership Service providers
     GetMSPs() (map[string]MSP, error)
}  
````

Figure 5. In this Figure we can see the definition of MSPManager interface. Notice, that in the code, “DeserializeIdentity” function is part of a “IdentityDeserializer” interface, that MSPManager extends. However, for simplicity of description we list directly the function inside MSPManager interface. 

在这幅图，我们能看到MSPManager接口定义。注意，在代码中“DeserializeIdentity”函数是MSPManager 扩展的IdentifyDeserializer接口的一部分。为了简化描述，在MSPManager接口中直接列出了此函数。


### 2.5.1 Setup of chain MSPs

An MSPManager instance is created for every new chain that is created through the “Setup” method, that takes as input a list of MSP configuration objects, i.e., “msp.MSPConfig”. The proto message governing the structure of msp.MSPConfig is depicted in Figure 6.

在创建chain的时候，通过SetUp接口创建一个MSPManager实例。此接口的输入是msp的配置信息。即msp.MSPConfig. 封装此结构的消息结构如下：

````bash
// MSPConfig collects all the configuration information for
// an MSP. The Config field should be unmarshalled in a way
// that depends on the Type
message MSPConfig {
   // Type holds the type of the MSP; the default one would
   // be of type FABRIC implementing an X.509 based provider
   int32 Type = 1;

   // Config is MSP dependent configuration info
   bytes Config = 2;
}

````

Figure 6. Protocol message for generic configuration of an MSP.

Marshalling of field “Config” (and hence the way to be unmarshalled) is defined by the value of field “Type”. For the default case where the default MSP type (Farbric) is used and being configured, “Config” has the content shown in Figure 7.

通过Type指定Config的编码方式和解码方式。默认情况下，MSP类型是Fabirc， Config的内如如下：

````bash
// FabricMSPConfig collects all the configuration information for a Fabric MSP. 
// Here we assume a default certificate validation policy, where any certificate
// signed by any of the listed rootCA certs would be considered as valid
// under this MSP. This MSP may or may not come with a signing identity. If
// it does, it can also issue signing identities. If it does not, it can only
// be used to validate and verify certificates.
message FabricMSPConfig {
   // Name holds the identifier of the MSP; MSP identifier is chosen by the
   // application that governs this MSP. For example, and assuming the default
   // implementation of MSP, that is X.509-based and considers a single Issuer,
   // this can refer to the Subject OU field or the Issuer OU field.
   string Name = 1;

   // List of root certificates associated
   repeated bytes RootCerts = 2;

   // Identity denoting the administrator of this MSP
   repeated bytes Admins = 3;

   // Identity revocation list
   repeated bytes RevocationList = 4;

   // SigningIdentity holds information on the signing identity
   // this peer is to use, and which is to be imported by the
   // MSP defined before
   SigningIdentityInfo SigningIdentity = 5;
}

````

Figure 7. Protocol message for configuration of the default MSP.


The configuration transaction that constitutes the genesis transaction of the chain, contains a list of one or more MSPs that would govern the chain. An example of the content related to MSPManager setup content with three MSPs of type “Fabric” is depicted in Figure 8.

链的创始交易产生的配置交易，包含管理链的MSP列表。下图描述了一个MSPManager启动三个Fabric类型MSP的例子。

````bash
{
 "MSPManager":
  [
     {
       "Type":0,
       "Config":{
           "Name":"org1",
           "RootCerts":["org1-identity1bytes","org1-identity2bytes","org1-identity3bytes"],
           "Admins":["adminidOrg1bytes"]
         }
     },
     {
       "Type":0,
       "Config":{
           "Name":"org2",
           "RootCerts":["org2-identity1bytes","org2-identity2bytes","org2-identity3bytes"],
           "Admins":["adminidOrg2bytes"]
         }
     },
     {
       "Type":0,
       "Config":{
           "Name":"org3",
           "RootCerts":["org3-identity1bytes","org3-identity2bytes","org3-identity3bytes"],
           "Admins":["adminidOrg3bytes"]
         }
     }
   ]
 }

````

Figure 8. Example of configuration content included in a chain genesis block. We emphasize that in this Figure, we use json format for simplicity of presentation.

创始块内容的例子，要强调的是，此图中，我们用json格式描述内容。


Configuration information of a (simple) MSP: To setup an MSP one would need the type of MSP be supported by that node’s executable, i.e., the MSP type to be among the ones already defined and implemented in the node’s peer or orderer executable. Each MSP is required to implement the interface presented in Figure1 and can be configured with instructions provided at chain genesis time. The information included in the genesis block of a chain for each MSP, is MSP-type-dependent. For the the default fabric MSP type, MSP configuration includes the following parts:

一个简单MSP的配置信息：为了建立MSP，需要知道节点支持的MSP类型，即peer或者 orderer中已经实现的MSP。 每个MSP需要实现图1中的接口，并且要能够用在链创建的时候提供的信息进行配置。这些信息（包括链的创始块中每个MSP的信息）依赖于MSP的类型。默认的MSP类型为Fabric，其配置包括如下信息。


- A name to identify the MSP within the chain/Blockchain network; In the above example we use  “org1”, “org2”, and “org3” as MSPIDs of the MSPS that govern the chain.  This is because we follow the convention of Section 1. Terminology, where each MSP represents an organization. 
- 标识MSP在链或者区块链网络中的身份的名字； 在上述例子中我们用 org1， org2 和org3 作为MSPID来管理chain。
- The type of MSP this provider uses; the provider mentioned in the example below is of type “Fabric”, which aims to be the (default) MSP implementation in fabric. Alternative implementations may require cross-signed certificates from two or more root CAs etc, and would have a different type reference.
- MSP类型；下面例子中提的类型为Fabric，这是Fabric中默认实现的MSP。 其他的实现可能需要来自多个RootCa的交叉签名证书，那么就需要一个不同的类型。
- A set of parameter values that include
- 一组包含如下值的参数信息：
  - a set of identities/certificates to constitute the root of trust, or CA server(s), e.g.,  
````
"RootCerts":["org1-identity1bytes","org1-identity2bytes","org1-identity3bytes"],
````

Notice that all these identities in the default case have the form of plain X.509 certificates in PEM format.
  - 一组构成信任rootCA的证书，比如："RootCerts":["org1-identity1bytes","org1-identity2bytes","org1-identity3bytes"]。 这些Identity默认情况下是X.509的PEM格式的证书。
  - the admin authorized to perform updates to this MSP parameters/configuration (consisting primarily on the root of trust/ root CA certificates, and CRLs); for simplicity we show a single certificate/identity as admin: 

````
  "Admins":["adminidOrg1bytes"],
````

  - 管理员，可以更新MSP参数和配置信息；简单起见，一般提供一个证书或者Identity作为管理员。"Admins":["adminidOrg1bytes"],
  - the current list of revoked certificates in bytes, which, if omitted, an empty list is implied. Notice that exact structure of revocation identities depends on the MSP type. However, one could represent it in the configuration file as an array of bytes.
  - 撤销证书列表，如果没有，可以为空。 撤销identity的结构依赖于MSP的类型。但是可以在配置文件中进行配置。

This structure is depicted in the protocol message of Figure 7.

图7的协议消息中定义了这个配置的结构。


### 2.5.2 MSP principals

MSP principals constitute the building blocks of definition of access control policies within a chain or channel. In particular, they are used to describe one or more identities that share a common feature that a specific MSP manages.

MSP主题组成了定义chain或者channel的访问控制策略的区块。它们被用来描述一个MSP管理的具有共同特性的多个Identity 。 


In the simplest case,  an MSPPrincipal can be the group of identities that are valid under a specific MSP’s identity validation logic, or the administrator(s) of an MSP. In this case we say that the MSPPrincipal is defined using the role of the identity inside that MSP as a classification criteria.

在简单的情况下，一个MSP主题可以是在MSPIdentity验证逻辑下合法的Identity 组（member）， 或管理员组。这种情况下，我们称MSP主题用MSP中Identity的角色分类定义。


Alternatively MSPPrincipal, can be tuned to define a specific (serialized) identity of an MSP configuration (MSPConfig object from above). In this case we say that the MSPPrincipal is defined using an identity based classification.

可选的MSP主题定义，可以是msp配置定义的一个identity。这种情况下，我们称 MSP主题用身份分类来定义。


Finally, MSPPrincipal can be defined as a set of identities that are valid under a specific MSP’s configuration, and belong to a certain organization unit. In this case we say that the MSPPrincipal is defined using an OrganizationUnit based classification.

最后，MSP主题可以用一组在MSP配置中合法的identity来定义，并且隶属于特定组织。这种情况下，我们称MSP主题使用组织分类来定义。


In the future, MSPPrincipal, can be tuned to describe a set of identities that are valid under a specific MSP’s configuration (MSPConfig object from above) and have a certain attribute in common. In this case we would say that the MSPPrincipal is defined using an attribute based classification.

未来，MSP主题可以用一组在MSP配置中合法的identity来定义，并且有一些特定属性。这种情况下，我们将称 MSP主题 使用属性分类来定义。


MSPPrincipal structure is defined in Figure 9.

MSP主题结构如下：

````bash

// MSPPrincipal aims to represent an MSP-centric set of identities.  In particular,
// this structure allows for definition of
//  - a group of identities that are member of the same MSP
//  - a group of identities that are member of the same organization unit
//    in the same MSP
//  - a group of identities that are administering a specific MSP
//  - a specific identity
// Expressing these groups is done given two fields of the fields below
//  - Classification, that defines the type of classification of identities
//    in an MSP this principal would be defined on; Classification can take
//    three values:
//     (i)  ByMSPRole: that represents a classification of identities within
//          MSP based on one of the two pre-defined MSP rules, "member" and "admin"
//     (ii) ByOrganizationUnit: that represents a classification of identities
//          within MSP based on the organization unit an identity belongs to
//     (iii)ByIdentity that denotes that MSPPrincipal is mapped to a single
//          identity/certificate; this would mean that the Principal bytes
//          message
//  -Principal that contains either a serialized MSPRole message, or a serialized 
// MSPOrganizationUnit message, or a serialized SerializedIdentityMessage depending
// on the Classification value..

message MSPPrincipal {

   enum Classification {
       ByMSPRole = 0;  // Represents the one of the dedicated MSP roles, the
       // one of a member of MSP network, and the one of an
       // administrator of an MSP network
       ByOrganizationUnit = 1; // Denotes a finer grained (affiliation-based)
       // groupping of entities, per MSP affiliation
       // E.g., this can well be represented by an MSP's
       // Organization unit
       ByIdentity  = 2;    // Denotes a principal that consists of a single
       // identity
   }

   // Classification describes the way that one should process
   // Principal. An Classification value of "ByOrganizationUnit" reflects
   // that "Principal" contains the name of an organization this MSP
   // handles. A Classification value "ByIdentity" means that
   // "Principal" contains a specific identity. Default value
   // denotes that Principal contains one of the groups by
   // default supported by all MSPs ("admin" or "member").
   Classification PrincipalClassification = 1;

   // Principal completes the policy principal definition. For the default
   // principal types, Principal can be either "Admin" or "Member".
   // For the ByOrganizationUnit/ByIdentity values of Classification,
   // PolicyPrincipal acquires its value from an organization unit or
   // identity, respectively.
   bytes Principal = 2;
}


// OrganizationUnit governs the organization of the Principal
// field of a policy principal when a specific organization unity members
// are to be defined within a policy principal.
message OrganizationUnit {

   // MSPIdentifier represents the identifier of the MSP this organization unit
   // refers to
   string MSPIdentifier = 1;

   // OrganizationUnitIdentifier defines the organization unit under the
   // MSP identified with MSPIdentifier
   string OrganizationUnitIdentifier = 2;

}

// MSPRole governs the organization of the Principal
// field of an MSPPrincipal when it aims to define one of the
// two dedicated roles within an MSP: Admin and Members.
message MSPRole {

   // MSPIdentifier represents the identifier of the MSP this principal
   // refers to
   string MSPIdentifier = 1;

   enum MSPRoleType {
       Member = 0; // Represents an MSP Member
       Admin  = 1; // Represents an MSP Admin
   }

   // MSPRoleType defines which of the available, pre-defined MSP-roles
   // an identiy should posess inside the MSP with identifier MSPidentifier
   MSPRoleType Role = 2;

}

````

Figure 9. MSPPrincipal related protocol messages.

MSPPrincipal 相关协议消息

MSP interface is to be extended with the following function 

SatisfiesPrincipal(id Identity, principal *MSPPrincipal) error

That would return true if the list of identities passed as parameters satisfy the provided MSPPrincipal. 

MSP的接口中增加函数：

SatisfiesPrincipal(id Identity, principal *MSPPrincipal) error

当参数identity 满足MSPPrincipal，返回true


>> Note: This function can be extended to include signatures if the only way to ensure that an identity is valid w.r.t. A certain MSP is through signatures. An example of such case is IBM Identity Mixer. 

>> 注：此函数可以扩展为支持签名，如果唯一确保identity是合法的方式是wrt， 一个特定的通过签名的MSP。 一个这样的例子是IBM的Identity Mixer。

### 2.5.3 Reconfiguration of a channel MSP

A channel’s MSPs are reconfigured through configuration blocks that are submitted to the channel and committed to that channel. More information on this is to be added soon.

Channel的MSP可以通过向channnel提交配置块进行重新配置。更多信息后续将会加入。

As happens with the reconfiguration of parameters included in other configuration items, reconfiguration of an MSP includes the following steps: 

像其他配置变更一样，MSP的重新配置也包含如下步骤：

1. Validate the authorization of the signers of configuration items included in the configuration transaction/block to reconfigure the signed configuration items using chain’s ConfigManager (+jason.anonymous@gmail.com to add more information as needed here :)). For the specific case of MSPs we discuss later how this takes place.  

验证配置块（或者交易）配置项的签名者的权限。 更多详细信息后面会讨论。

2. Create a new chain object, and a fresh MSPManager using the MSPManager Setup function and the list of MSPConfig objects created after parsing the Configuration Block & transaction. 

创建一个新的链对象，通过MSPManager的setup函数和配置块中包含的MSPConfig对象，刷新MSPManager。

3. Do the same for the rest of components that configuration items define, and have the new chain object point to the most recent instances of the reconfigured chain’s components. 

对配置定义的组件的其他部分，做同样的操作，并且让新的链对象指向那些新配置的链的组件。

4. After ensuring that the creation of each of the new components completes successfully, substitute chain pointers for MSPManager and other components to point to the freshly generated ones. Otherwise abandon the operation. 

确认所有新的组件都创建成功后，替换链指针指向新创建的对象（为MSPManager和其他组件创建的对象）。否则回滚操作。

5. Destroy the obsolete chain and linked components.

销毁过时是链对象和它关联的组件。

More information on how a chain reconfiguration occurs is described in the last section.

关于如何重新配置chain的更多信息，将在下一章节讨论。


# 3.Channel Access Control

In the genesis block of a channel, the following policies need to be defined:

在channel的创始块中，下面的策略需要定义：

- The readers of the chain or “channelReaders”, i.e., the policy to authenticate any request associated to read access to the transactions of a chain. For example this policy could define the identity or groups of identities (MSPPrincipals) that are allowed read-access to the chain; the same policy governs access to ordering service delivery requests and event access requests for that chain. 

- chain的reader，即关于访问链交易的请求的验证策略。比如，这个策略可以定义允许访问chain的Identity或者Identiy组（MSPPrincipals）；这个策略也会管理对orderer中关于这个链的访问。
- The writers of the chain or “channelWriters”, i.e., the policy to authenticate any request associated to submitting transaction to a chain. For example, this policy may include the identities or groups of identities (MSPPrincipals) that should be allowed to submit transactions to the chain. In particular this policy governs the set of signatures that need to be acquired in a transaction for the latter to be allowed to be submitted to the chain. The same policy governs permissions of clients to submit proposals to endorsers concerning that chain. 

- chain的writer，即关于提交交易的请求的验证策略。比如，这个策略可以包含允许向chain提交交易的Identity或者Identity组（MSPPrincipals）。尤其是，这个策略管理签名组，向chain中提交交易时需要用这些签名。同样的策略管理允许哪些客户端向背书节点提交proposals 。

- The admins of the channel or “channelAdmins”,  i.e., the policy to authenticate any request associated to reconfiguration of specific channel parameters. Admins may determine the identities or groups of identities (MSPPrincipals, and the way to combine them) that have admin access to the chain configuration. Such policies specify the (combination of) MSPPrincipals that should sign chain-specific reconfigurations for the reconfiguration to be applied. 

- channel的管理员，即关于重新配置channel参数的请求的验证策略。管理员可以决定哪些identity对chain的配置有管理访问权限。这些策略定义MSPPrincipals 用来对重新配置请求进行签名。

- The chaincodeAdmins of the chain,  i.e., the policy to authenticate any request associated to chaincode deployment within a chain. In the default policy case, chainDeployers would determine the identities or groups of identities (MSPPrincipals, and the way to combine them) that have permission to create or upgrade chaincodes on that chain. 

- chain的管理员，即对链中部署的链代码相关的请求的验证策略。对于默认策略，链部署者决定哪些identity 有权限在链中创建和更新链代码。


For the orderer system channel in particular the application needs to specify another policy called “chainCreators”, that would be used to evaluate chain creation requests by the orderers. Again chainCreators policy definition would be based on MSPPrincipals of MSPs that have been defined on the system channel.

对于orderer的系统channel，应用需要指定另外一种策略“chainCreators”，orderer用它来评估chain创建请求。chainCreator策略要基于系统channel中定义的MSP的 MSPPrincipals定义。


Initially within fabric, we plan to provide default policies, that are defined with the use of the policy framework in coauthdsl package. That is,

在fabric的最初版本中， 我们计划提供默认策略， 用coauthdsl 包的策略框架定义：

- For read-permissions we allow that the chain content is readable within a selection of MSP networks that govern the chain. In the genesis block, this is done by:

- 对于读权限，我们允许管理链的msp中的部分msp可读链的内容。 在创始块中定义。


  - Including in the genesis transaction a signed configuration item of type “Policy” whose value is the actual policy definition. In our example, we would use an OR (SignaturePolicy) type of policy of two MSP principals, one defining the members of org2, and one defining the members of org3. Let this configuration item be referred using key channelReadersPolicy. Its content is schematically described  as follows:

  - 在创始交易中，“policy”配置项的值就是一个策略。在我们的例子中，我们用两个MSP 。principal的OR（签名策略）类型作为策略，一个是org2的成员，一个是org3的成员。


   {“classification":"bymsprole","principal":{"mspid":"org2","msproletype": "member"}}
               OR 
   {“classification":"bymsprole","principal":{"mspid":"org3","msproletype": "member"}}


  - Including a (signed) configuration item of type “Chain”, referenced by identifier “channelReaders” and takes as value the policy identifier channelReaderPolicy. Table 2 shows an example of how read policy of a chain/channel is defined in a genesis block.

  - Identifier 的“channelReaders" 引用 配置项”Chain“ 作为channel的reader策略。表2是创始块中链/chennal的读策略定义

- For write permissions we allow that the members of a selection of MSP networks that govern the chain are allowed to submit transactions to the chain. As in the case to channelReaders, this is reflected in the chain genesis transaction through the use of two configuration items: 

- 对于写权限，我们允许管理chain的MSP中的部分成员向链中提交交易。跟Reader一样， 它也通过在创始交易中的两个配置项 进行配置。

  - A configuration item of type “Policy” whose value is the actual policy definition. In our example, we would define this policy as an OR (SignaturePolicy) type of policy of two MSP principals, one defining the members of org2, and one defining the members of org3. Let this configuration item be referred using key channelWritersPolicy. Its content is schematically described  as follows:

  - 在创始交易中，“policy”配置项的值就是一个策略。在我们的例子中，我们用两个MSP 。principal的OR（签名策略）类型作为策略，一个是org2的成员，一个是org3的成员。这些配置项用“channelWritersPolicy”作为键来定义。内容如下：


   {“classification":"bymsprole","principal":{"mspid":"org2","msproletype": "member"}}
                                  OR
   {“classification":"bymsprole","principal":{"mspid":"org3","msproletype": "member"}}


  - Including a (signed) configuration item of type “Chain”, referenced by identifier “channelWriters” and takes as value the policy identifier channelWritersPolicy. Table 2 shows an example of how write policy of a chain/channel is defined in a genesis block.

  - Identifier 的“channelWriters" 引用 配置项”Chain“ 作为channel的Writer策略。表2是创始块中链/chennal的写策略定义

- As admin policies we allow that the chain is re-configurable as long as reconfiguration requests on chain items are signed by the admins of all MSPs (via an external tool) that govern that chain. This is expressed in the genesis block in the same as the two previously described policies and using the following as content of the policy:

- 作为管理策略，可以通过重配置请求进行配置，这种请求必须包含管理此chain的所有MSP的签名。这跟前两个策略一样在创始块中描述，内容如下：

````bash

   {“classification":"bymsprole","principal":{"mspid":"org2","msproletype": "admin"}} 
                                              AND
   {“classification":"bymsprole","principal":{"mspid":"org3","msproletype": "admin"}}

````
For the purpose of this document we will refer to the related policy configuration item and chain parameter by channelAdminsPolicy, and channelAdmins respectively as reflected in Table 3.

在本文档中，相关的配置策略和chain参数会被channelAdminsPolicy 和 channelAdmins引用，可参考表3.

As chaincodeAdmins policy of a chain we use the same methodology as the one listed before.
Definition of these policies is done through cauthdsl framework of fabric codebase that leverage MSPPrincipals, and satisfiesPrincipal(id, principal) function of MSP interface where principal denotes the  MSPPrincipal of MSP’s network,, and id is an identity.

chain的链代码管理策略，我们使用跟上面相同的机制。这些策略也通过cauthdsl框架定义，会用到MSP接口的MSPPrincipals 和 satisfiesPrincipal（id，principal）函数。principal是MSP网络的MSPPrincipal， id是一个Identity。

For chain creation policies, that concern solely the system channel, the orderer defines chain creation policies within its orderer system chain. These policies may be arbitrary, and restrict the default policy further.  For instance, the chain creation policy might require that a minimum of 4 parties are involved in a new chain, or that one particular party be involved in all new chains etc.  The characteristics of this policy are negotiated with the orderer admins when an ordering service is configured to accept chain creation requests from a group of MSPs. The default policy in this case would require the signature of all application MSPs involved in the new chain.

对于chain创建策略，与系统channel强相关，orderer在它的系统chain中定义chain的创建策略。这些策略可以随意定义，未来会受限于默认策略。例如，chain创建策略要求chain最少有4个参与方，或者每个chain中都要有一个特定的参与方。当orderer配置为接受来自一组MSP的创建chain请求时，这个策略就是orderer的管理员协商的结果。默认策略中创建chain需要其中所有的MSP的签名。

An example of the structure of orderer system channel genesis is depicted in Table 2.   

表2描述了一个orderer系统channel例子的结构。
# 4. Using the default MSP: Best Practices

In this section we elaborate on best practices for membership service providers configuration for v1 in commonly met scenarios.

本节详细描述在通用场景下使用V1的默认MSP配置。

## 1. Mapping between organizations/corporations and membership service providers

组织跟MSP的映射关系

We recommend that there is a one-to-one mapping between organizations and MSPs. If a different mapping ration is chosen the following needs to be to considered:

我们推荐组织和MSP一一对应。如果选择了其他映射，下面这些需要考虑：

- One organization employing various MSPs. This corresponds to the case of an organization including a variety of divisions each represented by its membership service provider, either for management independence reasons, or for privacy reasons. One needs to know in this case that peers can only be owned by a single MSP, and will not recognize peers with identities from other MSPs as peers of the same organization. Implications of this is that peers may share organization-scoped data with a set of peers that are members of the same subdivision, and not with the full set of organizations. 

- 一个组织定义多个MSPs。它满足的场景是，一个组织包含多个部门，由于管理独立性或者隐私的原因，每个部门由自己的MSP。我们需要清楚，一个peer只能归属于一个MSP，并且peer不会把其他MSP下的peer看作同一个组织。这意味着peer只会跟本部门的peer分享信息，而不会跟同一组织中的其他peer分享。

- Multiple organizations using a single  MSP. This corresponds to a case of an organization consortium whose membership architecture of individual organizations is compatible. One needs to know here that peers would propagate organization-scoped messages to the peers that have identity under the same MSP regardless of whether they belong to the same actual organization. This is a limitation of granularity of MSP definition, and/or of peer’s configuration. 

- 多个组织使用一个MSP。它满足的场景是，一个组织联盟，各个组织的成员相互兼容。peer会跟同一MSP下的其他peer共享信息，而不管他们实际上是否为同一组织。这是MSP或者peer定义粒度的限制。

In future versions of fabric this can change as we move towards (i) an identity channel that contains all membership related information of the network, (ii) peer notion of “trust-zone” being configurable, a peer’s administrator specifying at peer setup time whose MSP members should be treated by peers as authorized to receive “organization”-scoped messages.

Fabric未来的版本中，会向两个方向转变，（1）一个channel中包含网络中所有成员的相关信息。（2），增加peer信任域的概念，peer的管理员可以在peer启动的时候指定哪些MSP的成员可信，并共享“组织范围”的消息。


## 2. On organization has different divisions (say organizational units), to which it wants to grant access to different channels. 

组织有不同的部门，希望不同部门访问不同的channel。

Two ways to deal with this:

两种方法可以解决这个问题：

- 1 Define one MSP to accommodate membership for all organization’s members. Configuration of that MSP would consist of a list of root CAs, intermediate CAs and admin CAs, and membership identities would include the organizational unit (OU) the member belongs to. Policies can then be defined to capture members of a specific OU, and these policies can be read/write policies of a channel or even chaincode administrators. Limitation of this approach, is that gossip peers would still consider peers under their local MPS as members of the same organization, and therefore share state / ledger related information with these peers even though they belong to an OU that is forbidden access to a certain channel.

- 定义一个MSP管理组织的所有成员。MSP配置多个root CA， 中间CA 和 管理CA，成员的身份中包含所属组织（OU）。定义策略的时候可以指定成员的OU，这些策略可以是channel的读写策略，也可以是链代码的管理策略。这种方法也有限制，peer会认为同一MSP下的peer属于同一组织，并且共享状态/账本相关的信息，但是他们属于不同的OU，这些OU禁止访问特定的channel。

- 2 Defining one MSP to represent each division, i.e., specify for each division a set of certificates, for root CAs, intermediate CAs, and admin Certs, such that there is no common certification path across MSPs. Here the disadvantage is the management of more than one MSPs instead of one, but this circumvents the issue present in approach (1).

- 每个分支机构定义1个MSP，每个分支机构都有自己的root CA， 中间CA 和管理CA。这样MSP之间没有公共认证路径。这种方式的缺点是要管理多个MSP而不是1个，但是它解决了方法1中的问题。
- 3 (available in the future) Define one MSP for each division by leveraging an OU extension of the MSP configuration.
- （未来可用）利用MSP配置的OU扩展属性为每一个分支机构定义一个MSP。

## 3. Separating clients from peers of the same organization. 

在同一组织中分离客户端和peer

In many cases it is required that the “type” of an identity is retrievable from the identity itself,e.g., it may be needed that endorsements are guaranteed to have derived by peers, and not clients or nodes acting solely as orderers. 

很多情况下，要求从Identity本身获取Identity的类型，例如，背书只能由peer做，而不是客户端或者orderer。

There is limited support for such requirements in v1.0. That is to allow for this separation currently, we would be required to create a separate intermediate CA, one for clients and one for peers, and configure two different MSPs one for clients, and one for peers. Channels this organization should be accessing, would need to include both MSPs, while endorsement policies will leveraging only the MSP that refers to the peers. This would ultimately result into the organization being mapped to two membership service provider instances, and would have certain consequences into the way peers and clients interact:

在V1.0中，对这中需求只有很有限的支持。为了实现这种分离，我们需要创建独立的中间CA，一个给客户端用，一个给peer用。并且配置两个MSP，一个给客户端，一个给peer。组织需要访问的channel，需要包含这两个MSP，而背书策略只利用peer对应的MSP。最终的结果是一个组织映射2个MSP，并且对peer和客户端的通信方式有一定的影响。

- Gossip would not be drastically impacted as all peers of the same organization would still belong to one MSP

- Gossip并不受显著影响，因为所有peer 仍然在同一个MSP。

- Peers allow the restrict the execution of certain system chaincodes to MSP-principals, e.g., local MSP based policies. For example, peers would only execute “JoinChannel” request if the request is signed by the admin of their local MSP who can only be a client (end-user should be sitting at the origin of that request). We can remedy this, if we exclude administrators of peer MSP to have a dual role.

- Peer允许限定某些系统链代码的执行只针对MSP-principals。例如， 基于本地MSP的策略。举例，如果请求有本地MSP的管理员的签名，而他只是一个客户端，peer只执行“joinChannel”请求。

- At the first phase peers would authorize event registration requests based on membership of request originator within their local MSP. Clearly if the originator of the request belongs to a different MSP, e.g., a client MSP, the peer would reject the request. This is to be changed soon, as registration requests are soon to be served using the check of whether identities are part of the readers of all the channels a peer has joined.

- 在第一阶段，peers只允许同一个MSP下的发起者的事件注册请求。显然如果请求的发起者属于不同的MSP，比如，客户端的MSP，peer将拒绝这个请求。这个很快会改变，peer会进一步检查，如果发起者是peer加入的channel的reader，peer一样会接受这个请求。

In the (near) future policy language is to be extended to support definition of organizational unit based policies. E.g., it would be possible to restrict endorsement policies to owners of identities that are members of a certain organizational unit of an MSP. This, would allow for another way of separating clients a peers of a given organization, e.g., by allocating identities of each type to  different organizational unit. 

未来，将会对策略语言进行扩展，来支持基于组织单元的策略定义。如，可以限定背书策略为MSP中的一个特定的组织单元内的成员。这将为分离客户端和peer提供新的解决方案，如，将不同的身份类型分配给不同组织单元。

# 5 example

>> Note: This section needs to be moved to a different document and be updated with the most recent re-configuration framework.
>> 注：这部分需要迁移到其他文档中，跟最新的配置框架文档一起更新。

In this section we give an end-to-end example of how peers/orderers and chains MSPs are setup and what is the configuration information each of them uses to bootstrap its operation.

在这个一节，我们给一个e2e的例子，展示peer orderer MSP如何启动，以及他们的配置信息。

For the purpose of this example we assume that four organizations Org1, Org2, Org3, and Org4 have decided to deploy a Blockchain network using an ordering service that would consist of orderer nodes owned by Org1, and Org2. Notice that Org1 is only to contribute orderers in this network. Thus, for the purpose of our example:

我们假设区块链网络中有4个组织，Org1,Org2,Org3,Org4。Org1和Org2配置orderer服务。并且Org1只提供orderer服务。
- Org1: orderers
- Org2: clients, peers, orderers
- Org3: clients, peers
- Org4: clients, peers

Step 1: The applications of each organization of the peer network (Org2, Org3, Org4) decide on the form of the Blockchain network genesis configuration from the application perspective, i.e., they decide on 

步骤1：peer网络的每个组织（Org2，Org3，Org4）的应用决定区块链网络的初始配置。如：
- the MSP that represents each organization, each MSP’s configuration, and each MSP identifier throughout the chain.
- 每个组织的MSP，MSP的配置， 每个MSP在链中的身份。
- the list of admins of the chain on behalf of the application 
- 链的管理员列表

In particular, the applications of the three organizations, Org2, Org3, Org4, agree on the list of configuration components for each MSP of theirs, depicted in Figure 10 (in JSON for presentation simplicity).

尤其是，Org2，Org3，Org4的应用要对各自MSP的配置达成一致。如图10中描述的（为了简单，我们用json格式描述）

````bash

     "msplist":[
      {
         "Type":0,
         "MSPConfig":{
           "Name":"org2",
           "RootCerts":["org2-cert-bytes-1","org2-cert-bytes-2","org2-cert-bytes-3"],
           "Admins":["org2-admin-cert-bytes"]
         }
       },
       {
         "Type":0,
         "MSPConfig":{
           "Name":"org3",
           "RootCerts":["org3-cert-bytes-1","org3-cert-bytes-2","org3-cert-bytes-3"],
           "Admins":["org3-admin-cert-bytes"]
         }
       },
       {
         "Type":0,
         "MSPConfig":{
           "Name":"org4",
           "RootCerts":["org4-cert-bytes-1","org4-cert-bytes-2","org4-cert-bytes-3"],
           "Admins":["org4-admin-cert-bytes"]
         }

       }
     ]
````

Figure 10. Example of list of MSPs included in the Blockchain network description on application behalf in json. We emphasize that though json is not used in practice, we present this information here in json for simplicity of  presentation of the configuration content. 

图10：用json格式描述的区块链网络中的msp列表，我们强调实际中不是用json格式描述的，我们这里用json格式只是为了更好的呈现其内容。

Notice that for simplicity, the identifier of the MSP associated to organization Org1 (2, 3) has been chosen to be “org1” (org2, org3, respectively). This information is submitted to the orderers (agnostic to whether it is orderer administrator or kafka cluster admin or some system channel) by the application. 
注意，为了简单，组织的标识符我们选“org1”（org2，org3），应用会将这些信息提交给orderer（是否是orderer管理员或者kafka集群管理员或者系统channel都无关紧要）。


Disclaimer: We emphasize that for simplicity of presentation in the previous figure we ignore the organization of MSP configuration in configuration items inside a genesis block. It is assumed that each MSP’s configuration data, is marshalled into a separate ConfigurationItem that is signed by identities such that the defined modification policy is satisfied. (Specifically for genesis blocks we can simply ignore the validation of the signatures in ConfigurationItems against the modification policies of these items). As we will see later, checking the signature against established modification policies is imperative in re-configuration blocks that have the same structure as genesis blocks.  

否认声明：我们强调为了简单描述，我们在创始块中忽略了MSP配置中的组织信息，我们假设每个MSP的配置数据，可以排序为独立的配置项，这些配置项通过Identity签名，来满足定义变更策略（对创始块，我们忽略与变更策略不一致的配置项的签名验证）。后面会看到，在重配置块中（与创始块结构一致），则必须检查与已建立的变更策略不一致的签名。

Step 2: The ordering service administrator configures the orderers. This configuration consists of two sets of data:

步骤2：orderer的管理员配置，这个配置包括两组数据：
1. Local configuration of each orderer that includes setup of the crypto service provider, key-manager, and the node’s SignerMSP, and any consensus related local information (e.g., where certain files are to be stored, etc).This is depicted in Figure 11.  The configured MSP is only possible to be updated manually by that orderer’s administrator, and can naturally vary from orderer to orderer. For simplicity and for V.1, to setup the local MSP, and assuming our default MSP type the administrator is requested to copy to dedicated location in orderer’s file system four sets of files:
- 每个orderer的本地配置，包括加密服务，私钥管理，节点的签名MSP，共识相关的信息（存储的文件等）在图11中描述。MSP的配置只能由orderer的管理员手动更新，并且各个orderer会有所不同。 为了简化，在V1中，启动本地MSP，假设默认的MSP类型。管理员需要拷贝4组文件到orderer的文件系统。
  - cacerts: PEM files containing the root authority certificates of the MSP
  - cacerts: PEM文件，是MSP的根证书
  - admincerts: PEM files containing the administrators’ certificates of this MSP
  - admincerts：PEM文件， MSP的管理员证书。
  - keystore: PEM files containing the private signing key of the orderer
  - keystore：PEM文件，orderer的私钥文件。
  - signcerts: PEM public cert files corresponding to the singing identity of the orderer
  - signcerts：PEM文件，跟签名私钥对应的签名证书。
Disclaimer: Currently SignerMSP is not used for Signature verification, and hence. updating the cacerts or admincerts of this MSP is not of crucial importance. However this is to be revisited as gossip communication may require frequent updates of these 
values.
否认声明：当前签名MSP不用于签名验证，因此，更新MSP的cacerts和admincerts并不十分重要。然而，在gossip通信中用到的时候，可能会经常更新他们的值。

2. Configuration parameters that are to be in common by all orderers participating in the system, and that is imperative that are consistently updated across all orderers of the ordering service. These parameters are organized in a structure that would constitute the system / orderer channel genesis configuration, that is depicted in Figure 12 in json format, and include the following parameters:

- 配置系统中所有orderer的公共参数，而必须保证这些信息在各个orderer间同步更新。这些参数包含在orderer的 channel的初始配置中。图12中有描述，包含如下参数：

  - The list of MSPs that are to be used throughout the Blockchain network (followed by the identifiers chosen by their owner organization), and reside below the "msp-manager" label. The MSP descriptions listed here aim to enable the orderers to validate peer/client signatures on system requests, e.g., chain creation requests. Hence MSPs in this case have more a “verifier” role, and will henceforth be referred to as VerifierMSP. Notice, that there is one MSP listed for each of Org2, Org3, and Org4, an one MSP for Org1 as the latter contributes orderers.  
  - 区块链网络中使用的MSP列表（后面跟对应组织的标示），放在“msp-manager”标签下面。这里包含MSP列表信息是为了让orderer能验证来自客户端和peer的系统请求中的签名，如创建链请求。因此MSP在这里有多个验证角色，并且将被成为VertifierMSP。注意，Org2，Org3，Org4每个组织一个MSP，Org1一个MSP给orderder用。
  - The list of parameters associated to ordering client, and server. Ordering client includes parameters that anyone invoking broadcast and deliver request to the ordering service would need to know (e.g., peers) while the ordering server includes parameters that are in common across ordering nodes, and need to be consistently updated across the orderers.
  - orderer服务端和r客户端相关的参数，客户端参数包括向orderer broadcast和deliver请求的时候需要用到的参数，服务器端参数包括各个orderer节点的公共参数，这部分参数要再orderer节点间同步更新。
  - The policies for readers, writers, and chain creators of the ordering channel that is to be created.  that for now includes only the networks of orderers, i.e., members of Organization 1, and Organization 2.
  - channel的读、写和创建策略。现在只有orderer网络需要创建channel，即：Org1和Org2. 


Configuration of the non-local part of orderers can have the form of a genesis transaction a diagram of which is depicted in Table 2, since it is anyway the genesis block of system channel. For simplicity of presentation we make the same convention as before and ignore mapping of MSP configuration to configuration items.

orderer的非本地配置部分，可以以创始交易的形式进行。如表2描述，它会用来做系统channel的创始块。为了简化描述，我们也像以前也一样进行转化为json格式，并忽略MSP配置跟配置项的映射关系。

Figure 11. Local orderer .yaml configuration file

图11. 本地orderer.yaml配置文件

````bash

General:

    # Local MSP config file
    LocalMSP:  file location of root folder of cacerts, admincerts, signcert, and keystore.

    # Orderer Type: The orderer implementation to start
    # Available types are "solo" and "kafka"
    OrdererType: solo

    # Ledger Type: The ledger type to provide to the orderer (if needed)
    # Available types are "ram", "file". When "kafka" is chosen as the
    # OrdererType, this option is ignored.
    LedgerType: ram

    # Batch Timeout: The amount of time to wait before creating a batch
    BatchTimeout: 10s

    # possibly other local orderer config

````

Figure 12. This can be used to generate the genesis block which is needed for orderer bootstrap.

图12：这些可以用来生成orderer启动的时候需要的创始块。

````bash

{
 "description":"Orderer channel genesis (Orgs: 01, 02)",
 "chain-id":"OrdererChannel-01-02",
 "msp-manager":[
     {
       "msp-type":0,
       "msp-config":{
         "msp-identifier":"org1",
         "rootca-identities":["org1-cert-bytes-1","org1-cert-bytes-2","org1-cert-bytes-3"],
         "admins":["org1-admin-cert-bytes"]
       }
     },
     {
       "msp-type":0,
       "msp-config":{
         "msp-identifier":"org2",
         "rootca-identities":["org2-cert-bytes-1","org2-cert-bytes-2","org2-cert-bytes-3"],
         "admins":["org2-admin-cert-bytes"]
       }
     },
     {
       "msp-type":0,
       "msp-config":{
         "msp-identifier":"org3",
         "rootca-identities":["org3-cert-bytes-1","org3-cert-bytes-2","org3-cert-bytes-3"],
         "admins":["org3-admin-cert-bytes"]
       }
     },
     {
       "msp-type":0,
       "msp-config":{
         "msp-identifier":"org4",
         "rootca-identities":["org4-cert-bytes-1","org4-cert-bytes-2","org4-cert-bytes-3"],
         "admins":["org4-admin-cert-bytes"]
       }
     }
   ]
 },
 "readers":[
   {"msp-identifier":"org1","group":"member"},
   {"msp-identifier":"org2","group":"member"}
 ],
 "writers":[
   {"msp-identifier":"org1","group":"member"},
   {"msp-identifier":"org2","group":"member"}
 ],
 "admins":[
   {"msp-identifier":"org1","group":"admin"},
   {"msp-identifier":"org2","group":"admin"}
 ]
}

````

Step 3: The orderers run and can identify now members of all participant organizations.

步骤3：orderer运行，并且能识别参与组织的成员。



|  Type  |  Key  |  Value  |  Modification Policy  |
|:-------|:-------|:-------|:-------|
|  Chain  |  HashingAlgorithm  |  SHAKE256  |  OrdererAdminPolicy  |
|  Chain  |  BlockDataHashStructure  |  Merkle tree width 10  |  OrdererAdminPolicy  |
|  Chain  |  OrdererAddresses  |  [addr1, addr2, …]  |  OrdererAdminPolicy  |
|  Orderer  |  ChainCreationPolicyNames  |  ChainCreationPolicy1, ChainCreationPolicy2  |  OrdererAdminPolicy  |
|  Orderer  |  BatchSize  |  100  |  OrdererAdminPolicy  |
|  Orderer  |  BatchTimeout  |  10s  |  OrdererAdminPolicy  |
|  Orderer  |  IngressPolicyNames  |  OrdererWriterPolicy, PeerWriterPolicy  |  OrdererAdminPolicy  |
|  Orderer  |  EgressPolicyNames  |  OrdererReaderPolicy, PeerReaderPolicy  |  OrdererAdminPolicy  |
|  Orderer  |  ConsensusType  |  Kafka  |  OrdererAdminPolicy  |
|  Orderer  |  KafkaBrokersKey  |  [addr1, addr2, …]  |  OrdererAdminPolicy  |
|  MSP  |  Org1ID  |  Org1 MSPConf  |  MSPInternal  |
|  MSP  |  Org2ID  |  Org2 MSPConf  |  MSPInternal  |
|  MSP  |  Org3ID  |  Org3 MSPConf  |  MSPInternal  |
|  MSP  |  Org4ID  |  Org4 MSPConf  |  MSPInternal  |
|  Policy  |  OrdererReaderPolicy  |  Org1.User or Org2.User  |  OrdererAdminPolicy  |
|  Policy  |  OrdererWriterPolicy  |  Org1.User or Org2.User  |  OrdererAdminPolicy  |
|  Policy  |  OrdererAdminPolicy  |  Org1.admin AND Org2.admin  |  OrdererAdminPolicy  |
|  Policy  |  BlockValidationPolicy  |  Org1.Cert or Org2.Cert  |  OrdererAdminPolicy  |
|  Policy  |  SignedByOrdererPolicy  |  Org1.Cert or Org2.Cert  |  OrdererAdminPolicy  |
|  Policy  |  ChainCreationPolicy1  |  Org3.Admin and (Org2.admin or Org4.admin)  |  OrdererAdminPolicy  |
|  Policy  |  ChainCreationPolicy2  |  Org4.admin  |  OrdererAdminPolicy  |
|  Policy  |  NewConfigItemCreationPolicy  |  Org1.admin AND Org2.admin  |  OrdererAdminPolicy  |
|  Policy  |  MSPInternal  |  empty  |  RejectAlwaysPolicy  |
|  Policy  |  RejectAlwaysPolicy  |  1 out of 0  |  RejectAlwaysPolicy  |



Table 2. Example of list of configuration items in orderer channel genesis transaction. Each row in the table corresponds to a (signed) configuration item of the genesis block. 

表2：orderer channel创始交易配置示例。每一行对应创始块中一个经过签名的配置项。


Step 4: For their initialization, peers need similar information as the orderers to setup the (local) MSP the peer belongs to, and signing identity/key information the peer would use to sign messages to the rest of Blockchain participants. Finally, the peer initialization information provides the reference to the signing identity the peer is to use when asked to create endorsements. As in the case of orderers, we instantiate SignerMSP for peers by having the administrator fill up three folders with the appropriate certificates/key-material:

步骤4：在初始化时，peer需要跟orderer类似的MSP信息，对消息签名用到的证书和私钥信息。最后，peer的初始化信息提供签名身份，这个在创建背书节点的时候用。 和orderer一样，我们通过4类文件给peer初始化一个SignerMSP。
- cacerts: PEM files containing the root authority certificates of the MSP
- cacerts：PEM文件，MSP的根证书。
- admincerts: PEM files containing the administrators’ certificates of this MSP
- admincerts：PEM文件，MSP的管理员证书
- keystore: PEM files containing the signing private key of the peer
- keystore：PEM文件，签名用的私钥
- signcerts: PEM public cert files corresponding to the singing identity of the peer
- signcerts：PEM文件，跟签名私钥对应的证书。

Disclaimer: Currently SignerMSP is not used for signature verification, and hence updating the cacerts or admincerts of this MSP is not of crucial importance. However this is to be revisited as GOSSIP communication may require frequent updates of these values. 

否认声明：当前签名MSP不用于签名验证，因此，更新MSP的cacerts和admincerts并不十分重要。然而，在gossip通信中用到的时候，可能会经常更新他们的值。

Step 5. Now let’s assume that Org2, and Org3 want to create a chain for their bilateral transactions. Applications of Org2, and Org3 agree off-band on certain configuration aspects of the chain, i.e., the configuration of the MSP contributed by each organization, the readers and the writers of the new chain, as well as the admins of the resulting chain. This information the application of Org2, and Org3 combines with the ordering information, to result to a configuration transaction that includes the information included in Table 3. Notice that Org4, does not appear anywhere in this genesis transaction.

5：现在，假设Org2和Org3想为他们的双边交易创建一个链。Org2和Org3的应用在对链的一些配置信息达成共识。即：每个组织的MSP，新链的reader和writer，以及新链的管理员。Org2和Org3的应用把这些信息和orderer的信息组合在一起，形成一个配置交易，如表3所示。注，Org4没有出现在这个创始交易中。

In the example of Table 3 below, Organization 1 and 2 are the only ones contributing orderers, while Organization 2 and 3 are the ones deploying and invoking application chaincodes. 

在表3的例子中，Org1和Org2组成orderer。Org2和Org3部署和调用应用链代码。
- Readers of the chain have been set to all listed MSP’s groups.
- 所有MSP都是链的reader。
- Writers of the chain have been set to all application MSP’s groups, i.e., Organization 2 and 3 members.
- 所有应用MSP都是链的writer，即 Org2和3个成员
- Chaincode deployer access is given to the admins of the two application MSPs, i.e., Organization 2 and 3.
- 链代码的部署权限分配给了两个应用MSP的管理员，即Org2和Org3
- Finally, re-configuration of the chain requires approval from admins of the MSPs of Organization 2 and Organization 3. 
- 最后，链的重配置请求需要Org2和Orgs的MSP的管理同意才能进行。

In the table below, we assumed the following mapping between parts of configuration and (signed) configuration items. 

在下表中，我们假设配置部分和签名配置项的映射如下：
- Two configuration items per MSPConfig, of type “MSP”, whose Key is the organization name and whose value is a marshaled MSPConfig proto and whose modification policy is [organization name + “Internal”], and another configuration item of type Policy, of Key of [organization name + “Internal”] of Policy Type MSP, and nil value with modification policy of itself
- 每个MSPConfig，有两个配置项。一个类型为MSP，键是组织名，值是序列化过的MSPConfig原型，变更策略是组织名+“internal”；另一个类型是Policy，键是组织名+“internal”， 值为空，变更策略为自己。
- One configuration item named “ChannelAdmins” of type “Policy” and Policy.Type “SignaturePolicy” whose value is a SignaturePolicy containing the conditions for administrative action.  For example “2 out of the following three MSP principals must sign (each an admin).  The “ChannelAdmins”, has a modification policy “ChannelAdmins” (itself)  
- 类型为Plolicy的配置项“ChannelAdmins”和“SignaturePlolicy”， 值是一个管理动作条件，比如“3个MSP中的2个管理员签名”， “ChannelAdmins”， 有一个变更策略“ChannelAdmins”（跟自己一样）。
- One configuration item of type “Policy” and Policy.Type MSP for “chain-readers”, and modification policy “chain-admins” 
- 一个Policy类型的配置项“chain-readers”, 变更策略为”chain-admins”。
- One configuration item named “ChannelReaders” of type “Policy” and Policy.Type “SignaturePolicy” whose value is a SignaturePolicy containing the conditions for reading the chain.  Since this applies to Deliver calls, it should be “1 out of the following n MSP principals must sign (each a reader)”.  The “ChannelReaders”, has a modification policy “ChannelAdmins”
- 一个类型Policy的配置项“ChannelReaders”，Policy.Type 为SignaturePolicy，值为”SignatruePolicy，包含读链的条件。因为Deliver的时候要用，它应为“下列N个MSP中的一个签名（每个都是一个reader）”。 “ChannelReaders”的变更策略为“ChannelAdmins”
- One configuration item named “ChannelWriters” of type “Policy” and Policy.Type “SignaturePolicy” whose value is a SignaturePolicy containing the conditions for reading the chain.  Since this applies to Broadcast calls, it should be “1 out of the following n MSP principals must sign (each a writer)”.  The “ChannelWriters”, has a modification policy “ChannelAdmins”
- 一个Policy的配置项“ChannelWriters”，Policy.Type”SignaturePolicy”， 值为包含写条件的SignaturePolicy，它用在Broadcast调用，应该为”下列N个MSP中的一个成员的签名（每个都是writer）“。”ChannelWriters“的变更策略为”ChannelAdmins“。
- One configuration item named “ChaincodeLifecycleAdmins” of type “Policy” and Policy.Type “SignaturePolicy” whose value is a SignaturePolicy containing the conditions for deploying a chaincode to the chain.  This might be something like “3 out of the following 4 MSP principals must sign (each a developer)”. The “ChaincodeLifecycleAdmins”, has a modification policy “ChannelAdmins”
- 一个Policy类型的配置”ChaincodeLifeCycleAdmins“， Poliy.Type为”SignaturePolicy”，它的值为包含向链中部署链代码的条件。可以如下“4个MSP中的3个的成员签名”。“ChaincodeLifecycleAdmins“ 的变更策略为”ChannelAdmins“。

The convention for items of type Orderer is to have the Key <KeyString> have a value of a marshaled <KeyString> message from the protos/common/orderer/configuration.proto.  The same is true for items of type Chain, and Peer, but corresponding to protos/common/configuration.proto and protos/peer/configuration.proto.  For type Policy, the value is always a marshaled protos/peer/configuration.proto.Policy message, and for type MSP the value is always a marshaled MSPConf from msp/protos (this should probably be moved to protos/msp.

按照惯例，orderer的类型配置有键<keystring>，值就是序列化的<keystring>消息（在/protos/common/orderer/configuration.proto中定义）。Chain（protos/common/configuration.proto）和Peer（protos/peer/configuration.proto）的配置也是如此。Policy类型的配置，值总是序列化的protos/peer/configuration.proto.Policy消息。 MSP类型的配置，值总是序列化的MSPConf（msp/protos, 可能会移动到protos/msp）。


|  Type  |  Key  |  Value  |  Modification Policy  |
|:-------|:-------|:-------|:-------|
|  Chain  |  HashingAlgorithm  |  SHAKE256  |  OrdererAdminPolicy  |
|  Chain  |  BlockDataHashStructure  |  Merkle tree width 10  |  OrdererAdminPolicy  |
|  Chain  |  OrdererAddresses  |  [addr1, addr2, …]  |  OrdererAdminPolicy  |
|  Orderer  |  BatchSize  |  100  |  OrdererAdminPolicy  |
|  Orderer  |  BatchTimeout  |  10s  |  OrdererAdminPolicy  |
|  Orderer  |  IngressPolicyNames  |  OrdererWriterPolicy, PeerWriterPolicy  |  OrdererAdminPolicy  |
|  Orderer  |  EgressPolicyNames  |  OrdererReaderPolicy, PeerReaderPolicy  |  OrdererAdminPolicy  |
|  Orderer  |  ConsensusType  |  Kafka  |  OrdererAdminPolicy  |
|  Orderer  |  KafkaBrokersKey  |  [addr1, addr2, …]  |  OrdererAdminPolicy  |
|  MSP  |  Org1ID  |  Org1 MSPConf  |  MSPInternal  |
|  MSP  |  Org2ID  |  Org2 MSPConf  |  MSPInternal  |
|  MSP  |  Org3ID  |  Org3 MSPConf  |  MSPInternal  |
|  MSP  |  Org4ID  |  Org4 MSPConf  |  MSPInternal  |
|  Peer  |  ReaderPolicyNames  |  PeerReaderPolicy  |  PeerAdminPolicy  |
|  Peer  |  WriterPolicyNames  |  PeerWriterPolicy  |  PeerAdminPolicy  |
|  Peer  |  AdminPolicyNames  |  PeerAdminPolicy  |  PeerAdminPolicy  |
|  Peer  |  ChaincodeLifecycleAdmins  |  ChaincodeLifecyclePolicy  |  AdminsPolicy  |
|  Policy  |  PeerReaderPolicy  |  org1.members OR org2.members OR org3.members   |  PeerAdminPolicy  |
|  Policy  |  PeerWriterPolicy  |  org1.members OR org3.members OR org4.members  |  PeerAdminPolicy  |
|  Policy  |  PeerAdminPolicy  |  org2.admin AND org3.admin   |  PeerAdminPolicy  |
|  Policy  |  ChaincodeLifecyclePolicy  |  org2.admin OR org3.admin  |  PeerAdminPolicy  |
|  Policy  |  OrdererReaderPolicy  |    |  OrdererAdminPolicy  |
|  Policy  |  OrdererWriterPolicy  |    |  OrdererAdminPolicy  |
|  Policy  |  OrdererAdminPolicy  |    |OrdererAdminPolicy  |
|  Policy  |  BlockValidationPolicy  |    |  OrdererAdminPolicy  |
|  Policy  |  SignedByOrdererPolicy  |    |  OrdererAdminPolicy  |
|  Policy  |  NewConfigItemCreationPolicy  |    |  OrdererAdminPolicy  |
|  Policy  |  MSPInternal  |  empty  |  RejectAlwaysPolicy  |
|  Policy  |  RejectAlwaysPolicy  |  1 out of 0  |RejectAlwaysPolicy  |




Table 3. Example of organization of chain genesis transaction in configuration items. Each row in the table corresponds to a (signed) configuration item of the genesis block. 

表3:。链创始交易组织配置的例子。每行对应创始块中一个经签名的配置项。

Step 6. Now from the config file, Org2, and Org3 construct a configuration transaction based on the resulting config file, that would constitute the only transaction included in the new channel’s genesis block. Although we are agnostic to the way the application constructs this configuration transaction, it is the application’s responsibility to submit the constructed genesis block (carrying a chain identity that has not yet been used) to the ordering service via a broadcast request. 

步骤6。现在，Org2,Org3通过配置文件构造一个配置交易，这是新channel的创始块中的唯一交易。应用是如何构建这个交易，对我们来说是透明的，应用负责将此创始块通过broadcast请求提交给排序服务。

Step 7. Now each orderer that receives this genesis block for a channel that does not exist assumes this to be a chain creation request. Assuming that the orderer approves of the chain creation request (assuming that is requested by properly authorized requestors), it embeds the (verbatim) configuration transaction as the contents of the genesis block for the new chain. If chain deployers policy exists in orderers’ system channel, then the orderers would need to check that the signatures in the configuration block received match the chain deployers’ policy. Finally, they create a new channel and use the constructed genesis block as the first block of the new channel. 

步骤7. 每个排序服务收到一个不存在的channel的创始块，就会认为这是一个创建链请求。假设排序服务同意链创建请求（假设认证通过），它把配置交易加入区块，作为新链的创始区块。如果在排序服务的系统channel中有部署部署者的策略，排序服务需要检查配置交易的签名符合部署者的策略。最后，它们创建一个新的channel，并且用这个创始块作为新channel的首个区块。

Step 8. Application of the two organizations calls deliver on the new channel and obtains the genesis block. It checks the validity of the configuration parameters in there to ensure that it is the same parameters Org2, and Org3, had agreed on (included in the chain creation request configuration).

步骤8，两个组织的应用在新的channel上调用deliver，获取创始块。然后验证配置参数跟Org2和Org3协商好的配置参数（在配置请求中）一样。 

Step 9. Application of Org2 send the obtained genesis block to its peers within a JoinChannel request and asks these peers to join the channel. The peer uses the local MSP to authenticate the JoinChannel request.

步骤9. Org2的应用通过“JoinChannel”请求发送获取的创始块给peer，告诉peer加入channel。peer利用本地MSP认证JoinChannel请求。 

Step 10. Upon joining a channel, and processing the genesis block included in the JoinChannel request  the peers retrieve the list of anchor peers. The gossip layer of the peer notifies the gossip layer of all peers in the organizationgossip leader of the organization the peer belong to, that it has joined the channel. One of the peers in that organization that has also joined the channel, That leader peer can then connects to the ordering service on the peer's behalf of all peers of that organization that have joined the channel. The peer receives the channel related ledger blocks by connecting directly to the ordering service itself (leader peer) or from the leader peer. As a result, the  peer receives the new channel’s transactions (normally it will be only the genesis block) and will have to check that the genesis block it received matches the one the application provided to it with the JoinChannel request. After that, it parses the genesis block and instantiates the corresponding chain instance, i.e., MSPManager, Ledger, and cache.

步骤10。在加入channel，处理JoinChannel请求中的创始块的时候，peer获取锚peer列表。peer的gossip层通知peer所属的组织的gossip leader peer，它已经加入了channel。有新的peer加入，然后leader peer代表组织的所有peer连接排序服务。peer（leader peer）通过直接连接排序服务获取账本区块或者从leader peer获取。结果，peer获的新channel的交易（正常只有创始块），并且必须验证获取的创始块与应用在JoinChannel请求中提供给它的是否一致。之后，它解析创始块并初始化相应的链实例，即，MSPManager， 账本和缓存。

Step 11. After Initializing the channel’s MSP, the peers that joined the channel disseminate via the gossip layer the channel’s MSP to all peers of their organization, in order to ensure that peers that belong to different organizations but their organizations share a channel, can communicate with each other. 
步骤11. 初始化channel的MSP之后，peer通过gossip层传播channel的MSP信息给组织中的所有peer，从而保证在同一channel中的不同组织中的peer可以相互通信。
# 6. Orderer Chain Creation Implementation Details

排序服务创建链的实现细节
## 6.1 Orderer System Chain

排序服务系统链

The orderer network is bootstrapped with a genesis block which contains the orderer system chain id and the initial set of orderer network governance policies, including, the orderer MSPs, the orderer consensus protocol configuration, and the chain creation policies.   Although application network MSPs may also be defined here (to be referenced in the chain creation policies), they are not strictly required at bootstrap.

排序服务启动的时候加载创始块，此块中包含排序服务的系统链ID和排序网络的初始管理策略，包括排序MSP，排序共识协议配置，链创建策略。虽然应用网络的MSP也可以在这里定义（在链创建策略中引用），但它们并不是在启动的时候强制要求的。

Only the ordering organizations have permission to read or write on this chain, so from a peer consumer perspective, the existence of this chain is irrelevant.

只有排序组织有权限读写系统链，因此从peer的角度，这个链是毫不相干的。

The chain creators are enumerated in a configuration item of type Orderer, key ChainCreators, and value common.ChainCreators.  The common.ChainCreators is a repeated list of strings, each corresponding to a defined configuration item of type Policy.  Note that this is a repeated list, rather than a single policy, to accommodate the possibility of multi-tenancy.  Different consortiums may desire different chain creation policies, and although it might be possible to construct one large policy with many ‘ORs’ to accommodate this, it is more natural to specify and manage the policies on a per consortium basis. 

Orderer类型的配置中枚举了链的创建者，键为” ChainCreators“，值为通用的ChainCreator。通用的ChainCreator是一个字符串列表，每个字符串对应一个Policy类型的配置项。注意，这是一个列表，而不是只有一个策略，这样可以满足多租户的场景。 不同的联盟可能要求不同的链创建策略，尽管可以通过很多”OR“定义一个很大的策略来满足这种需求，但是更自然的方式是每个联盟指定和管理自己的策略。

An example of ordering channel genesis organization in configuration items is depicted in Table 2.

表2,描述了一个排序服务的channel初始组织的配置示例。
## 6.2 Chain Creation Request (Configuration Transaction)

创建链请求（配置交易）

When a consortium wishes to use an ordering service, the ordering service creates a chain creation policy for that consortium, typically requiring the signatures of any two participants in the consortium, but as it uses the underlying policy framework, the signature requirements may be arbitrarily complex (for instance, requiring one authoritative member, and arbitrary two others).  This chain creation policy is simply a named string, and is added internally to the chain creators of the ordering system chain.  The consortium will also be told by the ordering service admins the set of configuration items which must be present in any chain creation request (such as the ordering system MSP definitions and block validation policies).

当一个联盟希望使用排序服务时，排序服务为联盟创建一个链创建策略，典型的情况是需要联盟中任意2个参与者的签名，但是由于使用了底层的策略框架，签名条件可以制定得很复杂（例如，要求1个权威成员，和任意2个其他成员）。链创建策略是一个简单的名字字符串，并且会被添加到排序服务的系统链的创建者中。排序服务的管理员也会告诉联盟链创建请求中需要包含哪些配置项（比如配需服务系统MSP定义和块验证策略）

When members of the consortium wish to create a new chain, they simply create the set of configuration items which defines their new chain, concatenate the marshaled bytes, and compute the hash of this data.  They then create a special configuration item of type Orderer, key CreationPolicy, and type orderer.CreationPolicy, with the digest field set to the hash of the chain configuration, and the policy set to the named chain creators policy provided by the ordering service.  This ConfigurationItem is then inserted into a SignedConfigurationItem and is appropriately signed as required by the chain creation policy.   This is all wrapped into a ConfigurationEnvelope and packaged into a signed Envelope and submitted for ordering.

当联盟成员要创建新链时，它们简单地创建一组配置项，定义它们的新链，对配置项进行序列化，并计算哈希值。 然后创建一个Orderer类型的配置项，键为CreationPolicy， 类型orderer.CreationPolicy, 摘要字段设为配置的哈希值，策略设置为排序服务提供的ChainCreators 策略。将这个配置项插入SignedConfigurationItem，并且根据链创建策略进行合理的签名。所有这些封装进ConfigurationEnvelope并且打包进一个签名的Envelope，提交给排序服务。

Note then, that the signed Envelope message is exactly the contents of the new genesis block.  All of the requesting parties have their signature encoded into this genesis block, and so the only validation required by the application is to ensure that the signature on the CreationPolicy configuration item is valid, and that the digest encoded corresponds to the remaining configuration.

注意，签名的Envelope消息就是创始块的内容。 请求的所有部分都有签名信息编码进创始块，因此应用只需要验证CreationPolicy配置项的签名是合法的，并且摘要和剩余的部分相符合即可。

An example of the structure of a chain genesis transaction appears in Table 3.

表3是链创始交易的例子

When the ordering service receives a chain configuration transaction, it first checks to see if the chain ID already exists.  If it does, then it is treated as a reconfiguration transaction and accepted/rejected accordingly.  In the event that the chain does not exist, the orderer then validates that it is a well formed and currently valid chain creation request, it then wraps this request inside of an Envelope of type ORDERER_TRANSACTION bound for the ordering system chain, and submits it for consensus.  Eventually, once the transaction has been ordered, it is unwrapped, and inspected a second time for validity now that ordering has occurred.  Only in cases where the ordering system chain configuration changed or a chain creation request for the identical chain ID was submitted concurrently can this validation fail, in which case the request is logged and discarded.

当排序服务收到一个链配置交易，它首先检查链ID是否已经存在。如果已经存在，它会把交易作为一个重配置交易，并相应的做接收或者拒绝处理。如果链ID不存在，排序服务检查它的格式，验证为有效请求，然后封装这个请求到排序服务系统链绑定的ORDERER_TRANSACTION 类型的Envelope，提交给共识。最终，一旦交易被排序，它会被再解封，再次验证。只有排序服务系统链配置发生变更或者同一链ID的链创建请求同时提交的情况下，验证才会失败，这种情况下，请求会被记入日志并丢弃。

Finally, after consensus is achieved and the still valid configuration transaction is processed, new ledger resources are allocated and the configuration transaction is embedded into the genesis block for the new chain.  The created can poll for this creation via a Deliver request.

最后， 共识达成，配置请求被处理后，新的账本资源会被分配，并且配置交易会被嵌入到 新链的创始块。创建者可以通过Deliver请求获取创始块。


