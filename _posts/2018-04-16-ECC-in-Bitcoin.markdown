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

椭圆曲线密码学（Elliptic Curve Cryptography，ECC），一般情况下，椭圆曲线可以用以下方程式表示（图示如下）：
```
E：y^2 = ax^3 + bx^2 + cx + d（a,b,c,d为常数）
```

\\[y^2 = x^3+ax+b \\]

![](/img/in-post/ECC/ECC-01.png)


椭圆曲线除了水平对称外，还有一个有趣的性质： 过曲线上任意两点（可重合）的直线必定与曲线相交于第三点。

下面先定义一下椭圆曲线上运算操作：

取曲线上两个点A，B，连接成直线延长与曲线相交于另一点，取该点的水平对称点，记为 (A+B) 的结果，该运算记作“椭圆曲线上的加法运算”。以特殊情况为例，若A，B为同一点，统一记为 G，取 G 点在曲线上的切线交于另一点，再取水平对称点，按照前面加法的定义得到 G+G = 2G，这样的运算定义为“椭圆曲线上的二倍运算”，依次进行可以得到4G，8G，便为乘法运算。另外，取任意点关于 x 轴对称的点，这样的运算称为“椭圆曲线上的正负取反运算”。详细可参考下图：

![](/img/in-post/ECC/ECC-03.png)

#### 椭圆曲线上的离散对数问题

椭圆曲线密码正是利用了上述运算规则中“椭圆曲线的离散对数问题”的复杂度来保证算法的安全性，其中特定乘法运算的逆运算非常困难，相比RSA，椭圆密码曲线密钥长度更短但强度更高。

椭圆曲线上的离散对数问题，其本质就是“已知xG求数x的问题”，定义为：
```
已知椭圆曲线 E，以及 E 上的一点 G，以及 G 的 x倍点 xG，求 x
```

#### 有限域上的椭圆曲线

进一步的，实际椭圆曲线密码所使用的椭圆运算，并不在实数域上，而是有限域上，有限域是指对于某个指定的质数 p，由0，1…p-1共p个元素所组成的整数集合中定义的加减乘数运算。

因此，在有限域上面的椭圆曲线并不是一条曲线，而是一些不连续的点集。

![](/img/in-post/ECC/ECC-02.png)

上图表示曲线是在素数阶p的有限域内，且p取较小值17，随着p值的增大，图形的复杂度迅速增大。

由于里面涉及到的数学知识很多，现在暂时只需了解
- 椭圆曲线上的离散对数问题就是已知G 和xG，求x
- 解椭圆曲线上的离散对数问题十分困难

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
