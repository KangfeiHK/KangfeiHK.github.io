---
layout:     post
title:      "从椭圆曲线加密算法到比特币"
date:       2018-04-17
author:     "K"
header-img: "img/post-bg-markdown.jpg"
tags:
    - 比特币
---

### 非对称密码学

公钥密码（Public-key cryptography），也称非对称密码学（asymmetric cryptography）中，密钥分为公钥（public key）和私钥（private key），前者用于加密，后者用于解密，两者一一对应，统称密钥对（key pair）。

举个例子，双方通信时，其中一方将自己的公钥给对方，要求对方用这个公钥加密后发消息给自己，这样一来，只有拥有的私钥的自己才能完成消息的解密，实现了消息的安全传输。


RSA 算是最为普及的一种公钥密码算法，此外还有一些其它加密算法，例如椭圆曲线密码学也备受关注，本文简单介绍两者。然后介绍一下椭圆曲线在比特币中的应用案例。


### RSA

RSA 算是最为普及的一种公钥密码算法，利用了质因数分解的复杂度来保证其安全性，其加解密计算公式如下：

```
密文 = 明文 ^ E mod N（加密过程），其中公钥是{E，N}
明文 = 密文 ^ D mod N（解密过程），其中私钥是{D，N}
```
RSA 加解密过程可以分为以下几步，首先用伪随机生成器生成512位大小的两个质数，记为p，q，那么记：
```
N = p*q
L = lcm(p-1,q-1) （最小公倍数）
```
然后还是通过伪随机生成器生成 E，使得满足以下条件：
```
1. 1 < E < L
2. gcd(E,L)，E 和 L 的最大公约数为1
```
接着求数 D，使其满足：
```
1. 1 < D < L
2. E * D mod L = 1
```
一旦发现了对大整数进行质因分解的高效算法，就可以对RSA进行破解。


### 椭圆曲线密码学

椭圆密码曲线（Elliptic Curve）完整的定义可参考 [Wolfram MathWorld](http://mathworld.wolfram.com/EllipticCurve.html)，一般情况下，可以用以下方程式表示：

![](/img/in-post/ECC/ECC-08.gif)

图示如下：

![](https://kangfeihk.com/img/in-post/ECC/ECC-01.png)

我们知道 RSA 正是利用了“**大数质因数分解**”的难度，从而保证了其安全性。同样地，Diffie-Hellman 密钥交换则是利用了“**有限域上的离散对数问题**”的复杂度。在 ECC 中，则是由“**椭圆曲线离散对数问题**”来保证其安全性。在介绍问题之前，先看一下相关预备知识：

#### 群

群是一个集合G，连同一个运算 "·"，它结合任何两个元素 a 和 b 而形成另一个元素，记为 a·b。符号"·"是具体的运算，例如加法。要具备成为群的资格，这个集合和运算(G,·)必须满足叫做群公理的四个要求：
1. 封闭性：对于所有 G 中 a, b，运算 a·b 的结果也在 G 中。
2. 结合性：对于所有 G 中的 a, b 和 c，等式 (a·b)·c = a· (b·c)成立。
3. 单位元：存在 G 中的一个单位元 0，使得对于所有 G 中的元素 a，等式 0·a = a·0 = 0 成立。
4. 逆元：对于每个 G 中的 a，存在 G 中的一个元素 b 使得 a·b = b·a = 0，这里的 0 是单位元。
如果加上第五条要求，
5. 要求符合交换律 a·b = b·a

这样的群我们称之为 **阿贝尔群（Abelian group）** 也叫 **交换群（commutative group）**。

举个例子，整数集合 Z 不但满足群的定义，还符合阿贝尔群的要求。而自然数集合 N 由于第 4 条的限制，就不满足群的定义。

#### 椭圆曲线上的群公理

我们在椭圆曲线上定义群公理的几个要求：
1. 群中的元素都在椭圆曲线上
2. 单位元指向无穷远处 0
3. 曲线上任意一点 P 的逆元关于 x 轴对称
4. 加法定义如下：一条直线上与曲线相交地三个非零点 P，Q，R，满足加法运算规则，P + Q + R = 0.（如下图所示，重要）

![](http://andrea.corbellini.name/images/three-aligned-points.png)

这里的加法，我们不要求运算的顺序，P + (Q + R) = Q + (P + R) = 0，因此也满足阿贝尔群的定义。下面我们可以逐渐了解到 **ECC 的数学基础**正是由椭圆曲线上的点构成的阿贝尔加法群中 ECDLP 问题的复杂度。

下面我们看一下任意两个数的加法运算，以 P + Q 为例，我们知道 P + Q + R = 0，又因为阿贝尔群的定义，式子可以转换为 P + Q = -R, -R 和 R 关于 x 轴对称，直观的结果如下图所示

![](http://andrea.corbellini.name/images/point-addition.png)

下面看一下特殊情况的运算：
- 如果 P = 0 或者 Q = 0，由于定义了无穷远处 0 为单位元，因此可以得知，对于曲线上任意的点 P 或者点 Q，P + 0 = P 或者 Q + 0 = Q 都是成立的
- 如果 P = -Q，则根据逆元的定义可知，P 和 Q 关于 x 轴对称，P + Q = P +（-P） = 0
- 如果 P = Q，那么过 P 点的切线与椭圆曲线相交于另一点 R，得 P + P = -R

作者提供了[可视化计算加法](https://cdn.rawgit.com/andreacorbellini/ecc/920b29a/interactive/reals-add.html?px=-1&py=4&qx=1&qy=2)的在线工具，有兴趣可以手动尝试。

除了加法，我们定义乘法如下：*n*P 即 *n* 个 P 相加的和，可在[这里](https://cdn.rawgit.com/andreacorbellini/ecc/920b29a/interactive/reals-mul.html)进行手动查看计算过程。以 *n* = 151 为例，如果按照正常运算，对于 *k* 位的二进制数字，计算时间复杂度位 $$ O(2^{k}) $$，效率太低。结合上面介绍过的运算规则，我们可以把 151 转换成二进制 10010111，

\\[ 151P = 2^{7} P + 2^{4} P + 2^{2} P + 2^{1} P + 2^{0} \\]

可以大大提高运算次数，时间复杂度下降至 $$ O(\mathrm{log}n) $$ 或者 $$ O(k) $$

椭圆曲线上的二倍运算过程可以参见下图：

![](/img/in-post/ECC/ECC-03.png)

我们定义上述运算的逆运算为对数运算，即已知 P 和 Q，求出 n 的值。

#### 有限域上的椭圆曲线

前面的描述以及计算都是在实数域范围内进行的，在 ECC 中，将椭圆曲线严格限制到有限域（finiite fields）中来提升难度。

有限域，是一个有限数字元素的集合，下面统一记为 F𝑝 ，包括了所有 0 到 p-1 的整数。有限域最常见的例子是当 *p* 为素数时，整数对 *p* 取模。其中有限域的阶（有限域中元素的个数）是一个素数的幂。有限域上的加减乘除相对比较简单，多一步模操作，这里不展开，简单提一下，除法运算需要先找到一个倒数再进行乘法运算。

前面我们定义曲线的点集为：

![](/img/in-post/ECC/ECC-08.gif)

下面切换到有限域 Fp 上，

![](/img/in-post/ECC/ECC-09.gif)

其中 0 依旧认为是无限远的点，以及 a 和 b 是在 Fp 上的两个整数。可以证明椭圆曲线在有限域 Fp 上还是一个阿贝尔群。

在有限域上面的椭圆曲线并不是一条曲线，而是一些不连续的点集。

![](/img/in-post/ECC/ECC-02.png)

上图表示曲线是在素数阶p的有限域内，且 *p* 取较小值17，随着 *p* 值的增大，图形的复杂度迅速增大。

在同一条直线的三个点 P，Q，R，同样满足 P + Q + R = 0，只不过直线的定义需要更改为点集：$$ ax + by + c = 0(\mathrm {mod} \; p) $$

给定一个非零点 Q，其相反数可以表示为：$$ \left ( x_{Q},-y_{Q}\; \mathrm {mod} \; P \right ) $$，例如在 $$ \mathrm{F}_{29} $$上，记 Q 为 (2，5)，那么 -Q = (2，-5 mod 29) = (2,24)

在实数域上，2P 的计算需要得到曲线在点 P 的切线，但在不连续的域上，不再适用这种方式。

好了，在了解了以上基础知识之后，回到椭圆曲线离散对数问题，本质即是

- 已知
    - 椭圆曲线E
    - 椭圆曲线E上一点G（基点）
    - 椭圆曲线E上的一点xG（x倍的G）
- 求解
    - x

这个问题的难度保证了椭圆曲线密码的安全性。

##### 椭圆曲线在比特币上的应用

在比特币中，公钥是由私钥通过椭圆曲线算法计算得到 `public key = private key * G`，使用 [Secp256k1](https://en.bitcoin.it/wiki/Secp256k1) 参数，其定义公式如下：

![](/img/in-post/ECC/ECC-07.png)

比特币中私钥、公钥、公钥哈希和钱包地址的关系如下图所示：
![](/img/in-post/ECC/ECC-04.png)

另外，更完整的生成过程可参考图：
![](/img/in-post/ECC/ECC-05.png)


> 参考/更多阅读：<br>
> [图解密码技术](https://hackernoon.com/merkle-trees-181cb4bc30b4) <br>
> [【比特币技术系列】 椭圆曲线加密算法 - ECDSA](http://www.ehcoo.com/Bitcoin_ECDSA.html) <br>
> [A (relatively easy to understand) primer on elliptic curve cryptography](https://arstechnica.com/information-technology/2013/10/a-relatively-easy-to-understand-primer-on-elliptic-curve-cryptography/) <br>
> [椭圆曲线算法：入门（1）](https://www.jianshu.com/p/2e6031ac3d50) <br>
> [椭圆曲线加密算法](https://juejin.im/post/5a67f3836fb9a01c9b661bd3) <br>
