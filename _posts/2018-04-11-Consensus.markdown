---
layout:     post
title:      "分布式共识算法"
date:       2018-04-11
author:     "K"
header-img: "img/post-bg-markdown.jpg"
tags:
    - blockchain
---

>In my whole life, I have known no wise people (over a broad subject matter area) who didn't read all the time -- none, zero.
 —— Charles T. Munger


共识算法是分布式系统最重要的研究内容之一，用以处理系统中的分歧，从而实现某种状态下的一致性。在现实的分布式系统中，服务器可能会出现延时、网络中断或者崩溃等故障。一旦出现故障就无法与其它服务器达成一致，因此需要共识算法来确保系统的容错性，即少数的故障节点不会影响系统正常运行。

传统的分布式一致性算法大多只考虑延时、宕机等非人为故障，即“非拜占庭容错”。与之对应的是“拜占庭将军问题”，同时考虑恶意节点响应和网络故障等，这也是容错性研究中最难的问题类型之一。

著名的 [FLP不可能性原理](https://yeasy.gitbooks.io/blockchain_guide/content/distribute_system/flp.html)告诉我们：在网络可靠，存在节点失效（即便只有一个）的最小化异步模型系统中，不存在一个可以解决一致性问题的确定性算法。相比同步分布式系统中消息传递时间的上限是已知的，异步分布式系统中消息可能在任何时间送达，而大部分实际的分布式系统往往是异步的。因此与其浪费时间寻找一个完美的异步系统共识算法，更应该去使用一个不那么完美却有实际意义的解法。

## 拜占庭将军问题与两军问题
[拜占庭将军问题](https://zh.wikipedia.org/wiki/%E6%8B%9C%E5%8D%A0%E5%BA%AD%E5%B0%86%E5%86%9B%E9%97%AE%E9%A2%98)（Byzantine Generals Problem），是指由Lamport, L 教授1982年首次在论文[《The Byzantine Generals Problem》](http://lamport.azurewebsites.net/pubs/byz.pdf) 中提出的分布式对等网络通信容错问题。其描述如下：

一组拜占庭将军分别各率领一支军队共同围困一座城市。为了简化问题，将各支军队的行动策略限定为进攻或撤离两种。因为部分军队进攻部分军队撤离可能会造成灾难性后果，因此各位将军必须通过投票来达成一致策略，即所有军队一起进攻或所有军队一起撤离。因为各位将军分处城市不同方向，他们只能通过信使互相联系。在投票过程中每位将军都将自己投票给进攻还是撤退的信息通过信使分别通知其他所有将军，这样一来每位将军根据自己的投票和其他所有将军送来的信息就可以知道共同的投票结果而决定行动策略。

在分布式网络中需要按照一致策略协作的计算机即为问题中的将军，而各计算机赖以进行通讯的网络链路即为信使。拜占庭将军问题描述的就是某些成员计算机或网络链路出现错误、甚至被蓄意破坏者控制的情况。与拜占庭将军问题相似的是[两军问题](https://en.wikipedia.org/wiki/Two_Generals'_Problem)，不同的是两军问题研究的是通讯信道可靠性问题。两军问题可以描述如下：

两支军队，分别由两个将军领导，正在准备攻击敌军且两方单独无法与敌军对抗，只能合作才有机会战胜敌军。两支军队都驻扎在敌军两边的山谷里。两个将军想要通讯的唯一方法就是穿过敌军阵地传送信件。问题是传送的信件可能会被守卫军截获。虽然两个将军商量好要同时对城市发起攻击，但是他们没有约定特定的攻击时间。为了保证取胜，他们必须同时发起攻击，否则任何单独发起攻击的军队都有可能全军覆没。他们必须互相通信来决定一个同时攻击时间，并且同意在那个时间发起攻击。两个将军彼此之间要知道另一个将军知道自己同意了作战计划。

假设先发出消息的一方为A军，将信息传送至B，信息到达B军，B军需要回信传达A军自己已经知道了，否则A军不敢单独行动。再一步，A军收到B军的回复，明确了B收到了信息，但是B军此时无法确认A军收到了信息，也不敢独自行动，等待消息确认；再一步，B军再次收到了A军的回复，此时A军同样陷入了困境，无法确认B军是否收到了自己发出的消息，因此最终仍旧无法达成共识。

两军问题已经证明在通讯条件不可靠的情况下是无解的，由于拜占庭将军问题更加复杂，不仅仅需要达成时间上的一致，还需要考虑到将军中可能会出现叛徒（即承诺不可靠），因此拜占庭将军问题在同样条件下依旧无解。而在不考虑信道可靠性的前提下，在拜占庭将军问题中，只要诚实节点超过2/3，那么仍旧可以使得达成的共识不受拜占庭节点干扰。


## 非拜占庭容错共识算法

### Paxos 算法

1998年 Lamport 提出 [Paxos](https://zh.wikipedia.org/zh-hans/Paxos%E7%AE%97%E6%B3%95) 算法，后续又增添多个改进版本的 Paxos 形成了 Paxos 协议家族。Paxos 的原理基于[两阶段提交](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)并进行扩展，适用于分布式的系统中存在故障（fault），但不存在恶意（corrupt）节点场景（即可能消息丢失或重复，但无错误消息）下的共识达成（Consensus）。

Paxos 是第一个被证明的共识算法，作为现在共识算法设计的鼻祖，以最初论文的难（难以理解且难以实现）出名。算法中将节点分为三种类型：
- proposer：提出一个提案，等待大家批准为结案。往往是客户端担任该角色；
- acceptor：负责对提案进行投票。往往是服务端担任该角色；
- learner：被告知结案结果，并与之统一，不参与投票过程。可能为客户端或服务端。


### Raft 

[Raft](https://raft.github.io/) 算法是 Paxos 算法的一种简化实现，其目标是提供更好理解的算法，并且证明可以提供与 Paxos 相同的容错性以及性能。Raft 同样运行在允许故障-停止的异步系统中，并且不要求可靠的消息传递，及可以容忍消息丢失、延迟、乱序以及重复。

Raft 算法主要注重协议的落地性，让分布式一致性协议可以较为简单地实现。Raft 和 Paxos 一样，只要保证(n/2+1)节点正常就能够提供服务；同时，Raft 更强调可理解性，使用了分而治之的思想把算法流程分为选举、日志复制、安全性三个子问题。可视化理解可参考 [Raft Visualization](http://thesecretlivesofdata.com/raft/)。

实现包括三种角色：leader、candidate 和 follower

缺点：任意包含 Leader 的时刻，Leader拥有完全记账权，如果此 Leader 节点是恶意的，后果不堪设想。且 leadership 的一致性算法都有个通病，吞吐量受单个节点的限制，这点在 Raft 身上体现尤甚。

优点： 容易理解和实现

无论是 Paxos 还是 Raft 算法，理论上都可能会进入无法表决通过的死循环(尽管这个概率其实是非常非常低的)，但是他们都是满足safety的，只是放松了liveness的要求，PBFT也是这样。


## 拜占庭容错共识算法

拜占庭容错共识算法解决的是网络通信可靠，但节点可能故障情况下的共识达成问题。

### PBFT 系列

1999年，卡斯托（Miguel Castro）与李斯克夫（Barbara Liskov）提出了实用拜占庭容错（PBFT）算法。该算法能提供高性能的运算，首次将拜占庭协议的复杂度从指数级降低到多项式级别，使得系统可以每秒处理成千的请求，比起旧式系统快了一些。只要叛徒不超过三分之一，忠诚的将军们就一定能达成一致结果。这个算法可以在异步网络中不保证liveness的情况下解决拜占庭容错问题。 虽然不保证Liveness，但是这个算法进入无限循环的概率非常低，在工程中是完全可用的。为此Barbara Liskov获得了2008年代图灵奖。PBFT采取两个假设提供状态复制：
1. 系统中只有一小部分数目可能是拜占庭服务器
2. 系统依赖占据主导的选举来达成共识，确保结果正确

PBFT 算法中三个阶段达成共识：Pre-Prepare、Prepare 和 Commit，整个流程如下[参考博客](https://blog.csdn.net/sxjinmingjie/article/details/77119989)： 
1. 从全网节点选举出一个主节点（Leader），新区块由主节点负责生成。 
2. 每个节点把客户端发来的交易向全网广播，主节点将从网络收集到需放在新区块内的多个交易排序后存入列表，并将该列表向全网广播。 
3. 每个节点接收到交易列表后，根据排序模拟执行这些交易。所有交易执行完后，基于交易结果计算新区块的哈希摘要，并向全网广播。 
4. 如果一个节点收到的2f（f为可容忍的拜占庭节点数）个其它节点发来的摘要都和自己相等，就向全网广播一条commit消息。 
5. 如果一个节点收到2f+1条commit消息，即可提交新区块及其交易到本地的区块链和状态数据库。 

### 区块链共识机制


### PoW 算法

相比拜占庭容错的有PBFT系列算法，一旦成共识便不可逆转，比特币网络中的POW（工作量证明）共识机制是不确定的，只能随时间推移，被推翻概率越来越小，即便是有能力作恶，其作恶成本高到远不如为善。个人拥有的算力越高，挖矿的概率也就越大，收益也越多，整个网络的安全性也正是在这种经济激励模型下得以维护和加固。

其优点是安全性高（破坏系统需要极高的成本），算法简单，容易实现；缺点是需要采购高额的硬件参与运算，且十分浪费能源，永远没有最终性，需要检查点机制来弥补最终性，区块的确认时间难以缩短等。

### PoS 算法

考虑到能耗过大，[Proof of stake](https://bitcointalk.org/index.php?topic=27787.0) 由Sunny King于2011年提出。在 POS 中，区别于依赖算力，每个验证人的投票权重取决于其持币量的大小（即股权），持有的币越多，其收益也就越多。虽然这种方式大大提高了能源效率，降低了POW的中心化风险。但是，naive POS 存在 "[Nothing at Stake](https://ethereum.stackexchange.com/questions/2402/what-exactly-is-the-nothing-at-stake-problem)" 问题，验证人为了更多的收益而选择在多条分叉上同时验证，很容易出现双重支付攻击。Pos 不像 Pow，矿工必须提前选择好一个方向进行挖矿，否则将损失收益。

针对"Nothing at Stake" 问题，Casper 中可以添加 [Slashing conditions](https://medium.com/@VitalikButerin/minimal-slashing-conditions-20f0b500fc6c)，即将验证者的余额作为安全质押金暂存，假如发现了作弊手段，验证者将失去这笔保证金。因此，可以大大提高系统的可靠性。此外，对于"[Long range attack](https://blog.ethereum.org/2014/05/15/long-range-attacks-the-serious-problem-with-adaptive-proof-of-work/)"问题也可以其它方式添加限制加以改进。

其优点是不再需要为了安全产生区块而大量消耗电能，缺点同样是永远没有最终性，需要检查点机制来弥补最终性，容易产生分叉，需要等待多个确认。

### DPOS 算法

基于POS衍生出的更专业的解决方案，股东可以将其投票权授予一名代表，类似于董事会的投票机制，选举出n个记账节点，提案者提交的提案由这些记账节点进行投票决策。
优点是减少记账节点规模，属于弱中心化，效率提高；缺点是牺牲了去中心化性质。

## Reference

- [拜占庭将军问题深入探讨](https://zhuanlan.zhihu.com/p/33275258)
- [Understanding Blockchain Fundamentals, Part 1: Byzantine Fault Tolerance](https://medium.com/loom-network/understanding-blockchain-fundamentals-part-1-byzantine-fault-tolerance-245f46fe8419)
- [From the Ground Up: Reasoning About Distributed Systems in the Real World](https://bravenewgeek.com/tag/two-generals-problem/)
- [The Paxos Protocol by Dr.TLA+ Series](https://github.com/tlaplus/DrTLAPlus/blob/master/Paxos/Paxos.pdf)
- [Paxos 与 Raft](https://yeasy.gitbooks.io/blockchain_guide/content/distribute_system/paxos.html)
- [分布式共识(Consensus)：Viewstamped Replication、Raft 以及Paxos](http://blog.kongfy.com/2016/05/%E5%88%86%E5%B8%83%E5%BC%8F%E5%85%B1%E8%AF%86consensus%EF%BC%9Aviewstamped%E3%80%81raft%E5%8F%8Apaxos/)
- [Paxos made live: an engineering perspective](http://www.read.seas.harvard.edu/~kohler/class/08w-dsi/chandra07paxos.pdf)
- [On scaling decentralized blockchains](https://fc16.ifca.ai/bitcoin/papers/CDE+16.pdf)
- [Proof of Stake versus Proof of Work - Bitfury Group](http://bitfury.com/content/5-white-papers-research/pos-vs-pow-1.0.2.pdf)
- [A (Short) Guide to Blockchain Consensus Protocols](https://www.coindesk.com/short-guide-blockchain-consensus-protocols/)
- [区块链核心技术：拜占庭共识算法之PBFT](http://blog.liqilei.com/bai-zhan-ting-gong-shi-suan-fa-zhi-pbftjie-xi/)
- [共识算法 区块链实用手册](https://wutongtree.github.io/hyperledger/consensus)
- [Where's Casper? Inside Ethereum's Race to Reinvent its Blockchain](https://www.coindesk.com/ethereum-casper-proof-stake-rewrite-rules-blockchain/)
- [Motivating Proof of Stake, a dialogue](https://hackernoon.com/motivating-proof-of-stake-a-dialogue-3a6d76bd08d)
- [Nothing at Stake, a dialogue](https://hackernoon.com/nothing-at-stake-a-dialogue-2ded91318cb9)
- [分布式一致性与共识算法](https://draveness.me/consensus)

