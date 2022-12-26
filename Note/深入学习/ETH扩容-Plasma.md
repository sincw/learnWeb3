# ETH-L2扩容方案Plasma

### 什么是Plasma

目前的主流区块链系统普遍存在可扩展性（Scalability）问题。解决该问题一般有下面两种思路：

- Layer1技术：直接修改底层区块链，比如修改区块大小和结构（2MB区块/SegWit等）、修改共识机制（PoS/DPos/PBFT等）、分片（Sharding）等方案

- Layer2技术：在底层区块链之上构造新的交易环境，分担主链的负载，代表方案包括状态通道和各种侧链子链技术。Layer2技术基于以下假设：**并非每个人都需要知道网络上发生的每一笔交易**。事实上，主链只需要记录交易的最终结果，交易的过程可以在更高效更经济的交易环境中完成。

Plasma属于Layer2技术中的一种，即**侧链（sidechain）**技术。具体来说，就是在主链上锁定一部分资产，然后在侧链上创建新资产，后续交易都在侧链上完成。当用户退出侧链时，销毁侧链上的资产，然后在主链上解锁剩余的资产。

那么由谁来创建新资产呢？共识机制。当然，侧链一般会选择效率更高的共识算法，比如PBFT/PoS/DPoS等。根据不可能三角法则，在可扩展性提高的同时，去中心化和安全性必然会出现一定程度的削弱。因此，Plasma需要解决一个重要的问题：**在侧链安全性无法保证时，所有人都可以安全地回退（fallback）的主链上**。

为了实现以上目标，Plasma框架包含以下三个主要组成部分：

- 链下执行：即在主链之外执行交易

- 状态提交：侧链状态的压缩版本（即交易的Merkle根），需要提交到主链上

- 退出机制：用户通过提供凭证（Merkle proof），可以随时、安全地退出侧链。

### Plasma MVP

MVP即Minimal Viable Plasma，是Plasma的一个最小实现版本。

Plasma MVP基于UTXO模型，只支持转账，不支持脚本或智能合约的执行。侧链共识机制采用POA（Proof-of-Authority），依赖一个被称为operator的节点生产侧链区块，然后把侧链状态提交到主链上。用户首先通过主链的智能合约充值，之后就可以完全在侧链上交易了，如果想要提现或者侧链出现安全性问题，用户可以向主链智能合约发起退出申请。

#### 充值（deposit）

目前只支持ETH充值，后续会支持ERC20代币

![](D:\Learn\web3\note\深入学习\img\aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzY0OS80YjA1ZmJmNDY0Nzk2ZWUwNjQ1NGM4ZWU3ZWY3NjNlMS5wbmc=.webp)

首先用户（紫色）向主链智能合约发起一个deposit调用，交易执行后会产生一个DepositCreated的event。Operator（红色）监测到这个event后，**创建一个只包含一笔交易的区块**，为用户创建侧链资产。侧链上的交易由两个输入UTXO和两个输出UTXO组成，在充值情况下，只有第一个输出有值，其他都为0。

随后，operator把该区块的Merkle根提交到主链智能合约，合约会发送BlockSubmitted的event。用户可以监听到这个event，并验证其有效性。

#### 退出（exit）

如果用户想要把侧链资产重新转回主链，则需要启动退出流程。  
![](D:\Learn\web3\note\深入学习\img\aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzQ2NS8zZDBlODY2Mjk1NTY3Y2I5MGRlNDE2NGNmZmNiNmQ0OS5wbmc=.webp)

合约会把该请求放入一个优先队列中，优先级等于**区块高度 * 1000000000 + 区块中交易index * 10000 + 交易中UTXO的index**。这样设计的目的是保证出现非法交易时，之前的所有合法UTXO都可以安全退出。

退出请求需要等待一段时间才能生效，可以理解为“公示”。等待时间长度等于**Max(UTXO创建时间+2 weeks，当前区块时间+1 week)**，也就是说，最快7天、最慢14天可以完成提现操作。

那么发起退出调用需要提供哪些材料呢？我们看一下startExit()的函数原型：

```js
    function startExit(
        uint256 _utxoPos,
        bytes _txBytes,
        bytes _proof,
        bytes _sigs
    )
```

- _utxoPos：UTXO的具体位置，以“blknum * 1000000000 + txIndex * 10000 + oIndex”的形式传递进来
- _txBytes：UTXO对应的原始交易数据(RLP编码)，用于和merkle proof一起验证该交易是否存在于区块中
- _proof：即merkle proof，算出原始交易的哈希值后，可以通过merkle proof算出区块的Merkle根，验证该笔交易的合法性
- _sigs：包含两个签名：交易签名和确认签名，用于确认用户是否有权提取该UTXO（确认签名后面会讲到）

最后，当等待期结束后，用户可以向主链智能合约发起一个finalizeExit调用，把资金连同EXIT_BOND真正转入主链账号中。

#### 挑战退出（challengeExit）

如果用户发现，有人给他转了一笔钱，但是又发起了退出申请想把这笔钱提走，那就必须阻止他，这被称为challengeExit。 

之前提到，提现会有7～14天的等待期，就是为了让用户有足够的时间发起挑战。如果挑战成功，则把退出请求从优先队列中删除，同时把EXIT_BOND作为奖励转给发起挑战的用户。挑战退出需要提供的材料参见以下函数原型：

```js
function challengeExit(
        uint256 _cUtxoPos,
        uint256 _eUtxoIndex,
        bytes _txBytes,
        bytes _proof,
        bytes _sigs,
        bytes _confirmationSig
    )
```

其中_cUtxoPos代表发起挑战的UTXO的位置，_eUtxoIndex代表被挑战的UTXO是交易中的第几个输入。

#### 抵御“扣块攻击”

“扣块攻击“，即Block Withholding Attack，指的是operator生成区块后不广播给其他用户，同时把该区块的Merkle根提交到主链上。这样就造成了信息不对称：如果operator先伪造一笔交易把UTXO转给自己，然后把你消费这笔UTXO的交易**放在后面**，一起打包进区块（不广播）。在这之后，operator向主链申请提现，用户无法对他发起挑战，因为无法提供需要的材料。当然，用户也可以向主链申请提现这笔UTXO，但是operator可以出示被扣留块中你的交易签名，对你发起挑战，导致你提现失败。

为了抵御这种攻击，MVP增加了一个确认签名（Confirmation Signature）的设计：

![](D:\Learn\web3\note\深入学习\img\aHR0cHM6Ly9waWFuc2hlbi5jb20vaW1hZ2VzLzY0OS80YjA1ZmJmNDY0Nzk2ZWUwNjQ1NGM4ZWU3ZWY3NjNlMS5wbmc=.webp)

举个例子：用户A（紫色）在侧链上给用户B（黄色）发了一笔转账交易，此时他会对该交易进行签名。只有当用户A确认该笔交易已经被包含在一个合法区块中（即他收到了这个块），并且operator也已经把Merkle根提交到主链上了以后，才会给用户B发送一个确认签名。之前提到过，用户发起退出申请时必须提供确认签名信息，所以如果发生扣块攻击，operator将无法得到用户的确认签名，也就无法成功提现。

operator的最大作恶就是使用户无法从侧链提取自己额资金。

### 总结

Plasma是一个侧链扩容框架，属于一种Layer2技术。Plasma MVP是Plasma的一个最小实现，基于UTXO模型，用户可以通过主链智能合约充值到侧链，在需要提现或者侧链出现安全问题时通过“exit/challenge”机制安全退出侧链。

在Plasma内存块链和以太坊链进行交互时，`是没有包含Plasma链的所有交易数据`，其实也就是没有包含Plasma链每一次状态变更的数据，会导致以太坊链如果脱离了Plasma圈，是没有办法复原每一次的数据，所以它非常依赖Plasma保护。

退出期比较长，因为每个节点都定期把相关的数据提交到以太坊链上，但因为使用`欺诈证明`的机制，用户在进入以太坊链的时候是非常快的速度，但要退出Plasma内存块链的时候最少可能需要一周时间，才能保证比较高的安全性，不然可能有人在其中短时间作恶，资金可能就会产生风险。

不过Plasma本身还是会有些优势，因为不会把所有数据都提交到主链上，所以潜在的扩容效果会很高，可能远比之后Rollup的方案更高。

为了抵御扣块攻击，采用了双签名设计，确保了侧链数据的可见性（data availability）。

### Plasma方案缺点

1.向主链提交的是 Merkle Tree 的根哈希。因此子链中的实际交易情况被隐藏，在主链上只能看到子链区块的哈希值。

2.由于只有一个operator矿工参与挖矿，Plasma operator可以作恶修改子链的交易细节，锁定了用户的资金。

3.提取资金需要7-14天。（虽然Plasma有多种实现方案让operator无法作恶，保证了用户的资金安全，但是各种实现都需要有一个争议期，这个用户无法接收的）

### 一些有用的链接

- https://ethresear.ch/t/plasma-cash-plasma-with-much-less-per-user-data-checking/1298
- https://en.wikipedia.org/wiki/Non-fungible_token
- http://erc721.org/
- https://github.com/ethsociety/learn-plasma
- https://medium.com/@kelvinfichter/whats-a-sparse-merkle-tree-acda70aeb837
- https://karl.tech/plasma-cash-simple-spec/
- https://github.com/loomnetwork/plasma-paper/blob/master/plasma_cash.pdf
- https://github.com/loomnetwork/plasma-cash
- https://github.com/omisego/plasma-cash
- https://github.com/wolkdb/deepblockchains/tree/master/Plasmacash
- https://github.com/luciditytech/lucidity-plasma-cash
- https://www.likecs.com/show-203343060.html
- https://blog.csdn.net/TurkeyCock/article/details/84565660
- https://www.chaineye.info/27/course_article_detail
- https://abmedia.io/chainnews-introducing-plasma-and-rollup
