

| 原文 | 作者 | 审核修正 |
|---|---|---|
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/readwrite.html) | Linsheng Yu | Dijun Liu |



## 交易模拟和read-write set

在背书节点上的交易模拟期间会产生一个交易的read-write set。`read set`包含在模拟期间交易读取到的唯一key及对应version。`write set`交易改写的唯一key（可能与`read set`中的key重叠）及对应的新value。如果交易的更新操作是删除一个key，则在`write set`为该key设置一个delete标记。

此外，如果交易中对一个key改写多次，则只保留最后的修改值。如果交易中读取一个key的值，即使交易在读取之前更新了该key的值，读取到的也会是之前提交过的而不是刚更新的。换句话说，不能读取到同一交易中修改的值。

如前所述，key的version只记录在`read set`；`write set`只包含key及对应新value。

对于`read set`的version的实现有很多种方案，最基本要求就是为key生成一个非重复标识符。例如用递增的序号作为version。在目前的代码实现中我们使用了blockchain height作为version方案，就是用交易的height作为该交易所修改的key的version。交易height由一个结构表示（见下面Height struc），其中TxNum表示这个tx在block中的height（译注：也就是交易在区块中的顺序）。该方案相较于递增序号有很多优点--主要是这样的version可以很好地利用到诸如statedb、交易模拟和校验这些模块中。

此外，如果模拟交易中执行了批量查询（range query），批量查询结果会被放到read-write set中的`query-info`。

	// 译注
	
	// read-write set 结构
	type TxReadWriteSet struct {
		NsRWs []*NsReadWriteSet
	}
	type NsReadWriteSet struct {
		NameSpace        string
		Reads            []*KVRead
		Writes           []*KVWrite
		RangeQueriesInfo []*RangeQueryInfo
	}
	type RangeQueryInfo struct {
		StartKey     string
		EndKey       string
		ItrExhausted bool
		Results      []*KVRead
		ResultHash   *MerkleSummary
	}
	type MerkleSummary struct {
		MaxDegree      int
		MaxLevel       MerkleTreeLevel
		MaxLevelHashes []Hash
	}
	type MerkleTreeLevel int
	type Hash []byte

	// read set 结构
	type KVRead struct {
		Key     string
		Version *Height
	}
	type Height struct {
		BlockNum uint64
		TxNum    uint64
	}
	
	// write set 结构
	type KVWrite struct {
		Key      string
		IsDelete bool
		Value    []byte
	}
	
下面是一个假设的交易模拟生成的read-write set示例，简单起见，示例中使用了递增序号作为version。
		
	<TxReadWriteSet>
	  <NsReadWriteSet name="chaincode1">
	    <read-set>
	      <read key="K1", version="1">
	      <read key="K2", version="1">
	    </read-set>
	    <write-set>
	      <write key="K1", value="V1">
	      <write key="K3", value="V2">
	      <write key="K4", isDelete="true">
	    </write-set>
	  </NsReadWriteSet>
	<TxReadWriteSet>

## 使用read-write set 验证交易和更新worldState
提交节点（committer）利用`read set`部分校验交易的有效性；用`write set`部分更新key的version和value。

在验证阶段，如果`read set`中每个key的version都与stateDB中对应worldState（假设所有之前的有效交易，包括同一个block中的交易，都已经提交完成，即已更新ledger）的version相匹配，则认为此交易有效。

如果read-write set中包含`query-info`，则还要对此执行额外的校验。该校验确保在此批量查询的结果范围内没有key被增删改。换句话说，如果在验证阶段重新执行该批量查询（模拟期间执行的交易）应该产生与模拟交易期间相同的结果。此校验确保交易在提交时出现幻读会被认为无效。注意，这个幻读保护仅限于Chaincode的`GetStateByRange`和`GetStateByPartialCompositeKey`两个方法***（译注：此处文档上提到的是`GetStateByRange`和`GetQueryResult`两个方法，但在代码里的注释却不是这样，此处以代码为准。详见fabric/examples/chaincode/go/marbles02/marbles_chaincode.go）***。而其他批量查询方法（如：`GetQueryResult`）会有幻读风险，因此这种查询应该只用于不会被提交到ordering的`只读交易`，除非app能保证交易模拟和交易验证提交两阶段之间结果集稳定。

如果交易验证通过，committer就会用`write set`更新worldState。在更新阶段，`write set`中的每个key在worldState中对应的value都会被更新，然后worldState中这些key的version也会随着更新。

## 交易模拟与交易验证 示例

本节通过示例场景帮助理解read-write set。存在一个key设为`k`，在worldState中由元组`(k,var,val)`表示，其中`ver`是`k`的最新的version，`val`是`k`的value。

现在有五个交易，分别是`T1,T2,T3,T4,T5`，这五个交易的模拟过程是针对相同的worldSate快照，下面的代码片段显示了模拟交易的worldState快照以及每个交易执行读写的顺序。

	World state: (k1,1,v1), (k2,1,v2), (k3,1,v3), (k4,1,v4), (k5,1,v5)
	T1 -> Write(k1, v1'), Write(k2, v2')
	T2 -> Read(k1), Write(k3, v3')
	T3 -> Write(k2, v2'')
	T4 -> Write(k2, v2'''), read(k2)
	T5 -> Write(k6, v6'), read(k5)

假设这些交易的顺序是T1,...T5（可能在同一个block或者不同block）

1. `T1`验证成功，因为它没有read操作。之后在worldState中的`k1`和`k2`会被更新成`(k1,2,v1'), (k2,2,v2')`
2. `T2`验证失败，因为它读取的`k1`在之前的交易`T1`中被修改了**（译注：需要特别注意一个前提，即这五个交易的模拟过程是对于相同的worldState快照，而且T2又有write操作，所以T2会进入commit阶段进行验证，这样T2的k1.ver=1，T1完成后实际的k1.ver=2了，然后T2在commit校验是就会失败。也就是上文提到的一个交易的模拟和提交期间，某key的值被修改。。。*但是有个疑问，正常使用中应该会经常出现T1、T2这种顺序的情况，难道会经常发生交易校验失败？？如果如此，那对于用户来说岂不很难用?暂有此疑，有待研究*）**
3. `T3`验证成功，因为它没有read操作。之后在worldState中的`k2`会被更新成`(k2,3,v2'')`
4. `T4`验证失败，因为它读取的`k2`在之前的交易`T1`中被修改了
5. `T5`验证成功，因为它读取的`k5`没有在之前的任何交易中修改

**译注：**

原文示例交易需要进一步阐述

  1. 交易中，示例里面忽略了在读取key的时候，是需要带有这个key的版本信息的。

  2. 值得注意的是，T1...T5在同一个区块和不同区块的处理方式不同。
    - 如果读取的key在此区块前面的交易中已经有update，则直接置此交易为失效
    - 如果读取的key在本区块前面的交易没有做update，则需要判断state中的版本和commit的版本是否一致，如果不一致，则置为失效交易

    为了尽量避免失效交易，application和chaincode需要进行精心设计，避免同一个资产交易信息尝试在一个区块上进行update操作。如果避免不到，可以适度重试交易。

**注意：** 交易不支持多read-write set
