---
layout:     post
title:      "P 与 NP 问题"
date:       2016-03-04
author:     "Kelvine"
header-img: "img/post-bg-P versus NP.jpg"
tags:
    - Study 
---


>No man has a good enough memory to be a successful liar. —— Abraham Lincoln


## 目录

1.  [时间复杂度](#section-1)<br>
2.  [多项式时间复杂度](#polynomial-time)<br>
3.  [P 类问题](#p-polynomial)<br>
4.  [NP 类问题](#np-nondeterministic-polynomial)<br>
5.  [NP-complete 问题](#np-complete--np-c--npc)<br>
6.  [NP-hardness 问题](#np-hardness-)<br>
7.  [P、NP、NP-hard、NPC 关系图](#pnpnp-hardnpc-)<br>
8.  [旅行商问题](#section-2)<br>


---

### 时间复杂度

时间复杂度是定量描述一个算法运行时间的函数。
计算时间复杂度的过程，常常需要分析一个算法运行过程中需要的基本操作，计算所有操作的数量。通常假设一个基本操作可在固定时间内完成，因此总运行时间和操作的总数量最多相差一个常量系数。

**常见时间复杂度可分为：**

常数时间、线性时间、对数时间、指数时间、阶乘时间等


**常见函数增长趋势图：**

![](http://7xo54z.com1.z0.glb.clouddn.com/NP-complexity-big-o.png)



**例：**

将 n 个数按从大至小排序，n 是其规模大小，第一轮比较（n-1）次，找出最大值，第二次比较（n-2）次，找出次大值，依次比较，直接排序结束，比较次数一共为：

![](http://7xo54z.com1.z0.glb.clouddn.com/img5.gif)

取最高次方 O(n^2)表示问题的时间复杂度。




----------

### 多项式时间复杂度（Polynomial time）

多项式时间在计算复杂度理论中，指的是一个问题的计算时间不大于问题大小 n 的多项式倍数。一般来说多项式级的复杂度是可以接受的(有效算法)，很多问题都有多项式级的解。

其中 O(n)、O(n2)、O(n3)、O(n log n) 都是，而 Ο(2n) 和 Ο(n!) 称为指数时间。


----------

### P 类问题（Polynomial）

P 类问题即为所有可以由一个确定型图灵机在多项式表达的时间内解决的问题。通俗一点说，所有能用多项式时间算法（对于一个规模是n的输入，在n^k的时间内得到结果）计算得到结果的问题的集合。


----------

### NP 类问题（Nondeterministic Polynomial）

NP 类问题指的是，能在多项式时间内检验一个解是否正确的问题或者说可以在非确定型图灵机以多项式时间解决的问题，也有另外一种说法，它是能在多项式的时间里猜出一个解的问题。

P 与 NP 的关系：

P 问题显然都是 NP 问题
NP 问题不是非 P 类问题

----------

### NP-complete 问题（缩写为 NP-C 或 NPC）

一类非常特殊的 NP 问题叫做 NP-完全问题，也即所谓的 NPC 问题，它是 NP（非决定性多项式时间）中最难的决定性问题。因此 NP 完全问题应该是最不可能被化简为 P（多项式时间可决定）的决定性问题的集合。

**正式定义：**

 - 它是一个 NP 问题，且
 - 其他属于 NP 的问题都可归约成它。

既然所有的 NP 问题都能约化成 NPC 问题，那么只要任意一个 NPC 问题找到了一个多项式的算法，那么所有的NP问题都能用这个算法解决了，NP 也就等于 P 了。

**归约：**

这里涉及一个概念，可称为问题之间的归约。可以认为各个问题的难度是不同的，表现形式为，如果我可以把问题A中的一个实例转化为问题B中的一个实例，然后通过解决问题B间接解决问题A，那么就认为B比A更难。通过对归约过程做出限制可以得到不同类型的归约。复杂度理论里经常用到的规约叫 polynomial-time Karp' reduction。其要求是转化问题的过程必须是多项式时间内可计算的。


----------

### NP-hardness 问题

NP-hardness 问题它满足 NPC 问题定义的第二条但不一定要满足第一条（NP-Hard 集合包含 NPC 问题集合）

**区别 NP-hardness、NPC：**
如果所有 NP 问题都可以多项式归约到问题 A，那么问题A就是 NP-Hard；如果问题 A 既是 NP-Hard 又是 NP，那么它就是 NP-Complete。

----------

### P、NP、NP-hard、NPC 关系图

没有什么比一张图更清晰的了。

![](http://7xo54z.com1.z0.glb.clouddn.com/7%20-%201.png)


### 旅行商问题

售货员旅行问题（甲）
如图1中，A , B , C ,…, G表示7个城市，而售货员要从 A 城出发再回到 A 城并访问 B , C ,…, G，所有的城，求最短路径。

![](http://7xo54z.com1.z0.glb.clouddn.com/mm_10_2_04_01.gif)



问题2：售货员旅行问题（乙）
与第一题之条件相同，但现在有一个给定之正整数 B，问题是是否存在一条路径其总距离不大于 B。（问题 1 与问题 2 在表面上相似，但在以后的理论上有很大的不同）

对于乙来说，只需验证是否满足条件，因此根据定义，它是一个 NP 问题，而甲是 NP-hard 问题。



----------

<br>


> 参考：<br>
> [未來數學家的挑戰 - 計算量問題](http://episte.math.ntu.edu.tw/articles/mm/mm_10_2_04/index.html) <br>
> [什么是P问题、NP问题和NPC问题](http://www.matrix67.com/blog/archives/105) <br>
> [P versus NP problem](https://en.wikipedia.org/wiki/P_versus_NP_problem) <br>

