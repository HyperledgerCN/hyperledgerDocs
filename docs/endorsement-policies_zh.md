
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html) | Linsheng Yu |  |


Endorsement policies are used to instruct a peer on how to decide whether a transaction is properly endorsed. When a peer receives a transaction, it invokes the VSCC (Validation System Chaincode) associated with the transaction’s Chaincode as part of the transaction validation flow to determine the validity of the transaction. Recall that a transaction contains one or more endorsement from as many endorsing peers. VSCC is tasked to make the following determinations: 

- all endorsements are valid (i.e. they are valid signatures from valid certificates over the expected message) 
- there is an appropriate number of endorsements 
- endorsements come from the expected source(s)

节点通过背书策略来确定一个交易是否被正确背书。当一个peer接收一个交易后，就会调用与该交易Chaincode相关的VSCC（Chaincode 实例化时指定的）作为交易验证流程的一部分（还有RW版本验证）来确定交易的有效性。为此，一个交易包含一个或多个来自背书节点的背书。VSCC的背书校验包括：

* 所有的背书是有效的（即，有效证书做的有效签名）
* 恰当的（满足要求的）背书数量
* 背书来自预期的背书节点

Endorsement policies are a way of specifying the second and third points.

背书策略就是用来定义上边的第二和第三点。

## Endorsement policy design - 背书策略设计

Endorsement policies have two main components: 

- a principal 
- a threshold gate

A principal `P` identifies the entity whose signature is expected.

A threshold gate `T` takes two inputs: an integer `t` (the threshold) and a list of `n` principals or gates; this gate essentially captures the expectation that out of those `n` principals or gates, `t` are requested to be satisfied.

背书策略有两个主要组成部分：

* 主体（principal）：`P` 定义了期望的签名来源实体
* 门阀阈值（threshold gate）：`T` 有两个参数：整数`t`（阈值）和`n`个主体，表示从这`n`个主体中获取`t`个签名

For example: - `T(2, 'A', 'B', 'C')` requests a signature from any 2 principals out of `A`, `B` or `C`; - `T(1, 'A', T(2, 'B', 'C'))` requests either one signature from principal `A` or 1 signature from `B` and `C` each.

例如：

* `T(2, 'A', 'B', 'C')`表示需要`A`、`B`、`C`中任意2个主体的签名背书
* `T(1, 'A', T(2, 'B', 'C'))`表示需要来自主体`A`的签名或者来自`B`和`C`两者的签名背书

## Endorsement policy syntax in the CLI - CLI中背书策略语法

In the CLI, a simple language is used to express policies in terms of boolean expressions over principals.

在CLI中，用一种简单的布尔表达式来表示背书策略。

A principal is described in terms of the MSP that is tasked to validate the identity of the signer and of the role that the signer has within that MSP. Currently, two roles are supported: **member** and **admin**. Principals are described as `MSP`.`ROLE`, where `MSP` is the MSP ID that is required, and `ROLE` is either one of the two strings `member` and `admin`. Examples of valid principals are `'Org0.admin'` (any administrator of the `Org0` MSP) or `'Org1.member'` (any member of the `Org1` MSP).

Fabric使用MSP来描述主体，MSP用于验证签名者的身份和签名者在MSP中的角色/权限。目前支持两种角色：**member**和**admin**。主体的描述形式是`MSP.ROLE`，其中`MSP`是MSP ID，`ROLE`是**member**或**admin**。比如一个有效的主体可表示为`'Org0.admin'`（MSP`Org0`的管理员）或`'Org1.member'`（MSP`Org1`的成员）。

The syntax of the language is:

CLI语法是：

`EXPR(E[, E...])`

where `EXPR` is either `AND` or `OR`, representing the two boolean expressions and `E` is either a principal (with the syntax described above) or another nested call to `EXPR`.

其中`EXPR`是`AND`或`OR`，表示布尔表达式；`E`是上面语法所描述的主体或者是另一个嵌套进去的`EXPR`。

For example: 

- `AND('Org1.member', 'Org2.member', 'Org3.member')` requests 1 signature from each of the three principals 
- `OR('Org1.member', 'Org2.member')` requests 1 signature from either one of the two principals 
- `OR('Org1.member', AND('Org2.member', 'Org3.member'))` requests either one signature from a member of the `Org1` MSP or 1 signature from a member of the `Org2` MSP and 1 signature from a member of the `Org3` MSP.

例如：

* `AND('Org1.member', 'Org2.member', 'Org3.member')`表示需要三个主体共同签名背书
* `OR('Org1.member', 'Org2.member')`表示需要两个主体之一的签名背书
* `OR('Org1.member', AND('Org2.member', 'Org3.member'))`表示需要`Org1`的签名背书或者`Org2`和`Org3`共同的签名背书

## Specifying endorsement policies for a chaincode - 为chaincode指定背书策略

Using this language, a chaincode deployer can request that the endorsements for a chaincode be validated against the specified policy. NOTE - the default policy requires one signature from a member of the `DEFAULT` MSP). This is used if a policy is not specified in the CLI.

部署Chaincode时可以指定背书策略。注意：如果没有指定背书策略就使用默认的背书策略，即需要MSP`DEFAULT`的一个成员的签名背书。

The policy can be specified at deploy time using the `-P` switch, followed by the policy.For example:

部署Chaincode时用`-P`指定背书策略，例如：

	peer chaincode instantiate -C testchainid -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a","100","b","200"]}' -P "AND('Org1.member', 'Org2.member')"

This command deploys chaincode `mycc` on chain `testchainid` with the policy `AND('Org1.member', 'Org2.member')`.
	
此命令会以背书策略`AND('Org1.member', 'Org2.member')`在链`testchainid`上部署chaincode`mycc`。

## Future enhancements - 未来计划

In this section we list future enhancements for endorsement policies: 

- alongside the existing way of identifying principals by their relationship with an MSP, we plan to identify principals in terms of the *Organization Unit (OU)* expected in their certificates; this is useful to express policies where we request signatures from any identity displaying a valid certificate with an OU matching the one requested in the definition of the principal. 
- instead of the syntax `AND(., .)` we plan to move to a more intuitive syntax `. AND . `
- we plan to expose generalized threshold gates in the language as well alongside `AND` (which is the special `n`-out-of-`n` gate) and OR (which is the special `1`-out-of-`n` gate)

本节列举了背书策略的未来计划增强功能：

* 除了通过与MSP的关系确定principals身份的现有方式外，我们计划根据证书中的组织单位(OU)来标识principals；这样就可以请求与背书策略中定义的principal的OU相匹配（同一组织单位内）的任意实体的签名作为背书
* 以更直观的语法`. AND .`取代语法`AND(., .)`
* 还计划将阈值放到`AND`（`n`-out-of-`n`）和 `OR` (`1`-out-of-`n`)的语法中

