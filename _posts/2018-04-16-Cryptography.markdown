---
layout:     post
title:      "浅析密码学基础"
date:       2018-04-16
author:     "K"
header-img: "img/post-bg-markdown.jpg"
tags: 
    - 密码学
---

密码学技术通常具有机密性、完整性、认证、不可否认性中的一个或多个特性，现实应用中往往需要组合使用以发挥各自优势。举例来说，机密性是为了确保原始信息不会被窃听，因此进行加密操作，对应的技术是对称加密和非对称加密算法。完整性是确保消息不被篡改，因此需要对信息进行完整性校验，对应的技术有单向散列函数、消息验证码、数字签名。认证是为了防止攻击者伪装成真正的发送者，因此需要检查消息是否来自合法的发送者，对应的密码技术有消息验证码，数字签名。不可否认性是为了防止信息发送者事后否认发布的行为，因此需要签名证明消息的来源，对应的密码技术为数字签名。常见[技术图谱](https://xiaoxueying.gitbooks.io/graphic-cryptology/content/cryptogram_world.html)如下所示，本文将整理一些相关笔记（更新中）。

![](https://xiaoxueying.gitbooks.io/graphic-cryptology/content/cryptogram.png)

## 单向函数

哈希函数在日常中较为常见，比如在下载大文件时，为了供用户验证完整性常常会提供一串哈希值，一般为 MD5. MD5 和 SHA 也是最常见的单向散列函数，SHA 其实代表的是 SHA 家族算法，并不特指某一个。有 SHA-1、SHA-2、SHA-3 系列，其中 SHA-1 已经不再安全，SHA-2 中具体细分为 SHA-224、SHA-256、SHA-384、SHA-512 等算法，最常见的就是 SHA-256，目前仍广泛使用。更新版本的 SHA-3 不是 SHA-2 的直接改进版，两者没有太大的关联。下面针对几个代表进行介绍。

哈希函数具备以下特点：输入任意大小的字符串，产生固定大小的输出，计算简单，且重复计算具有确定性，只有修改了输入，尽管变动很小，输出也会发生很大变化。

哈希函数的安全一般满足以下三条性质：

- 原像稳固性：给定一个哈希值，在H(x)的映射下，找不到对应的 x，其实就是单向性的保证，无法逆运算
- 第二原像稳固性：给定一个 x 和 H(x) 的映射，找不到一个 x'，使得 y 相同，弱抗碰撞性
- 碰撞稳固性：给定 H（x），找不到不同的 x，使得它们对应的 y 相同，抗强抗碰撞性

大多数 Hash 函数都是通过 Merkle-Damgard (MD)迭代结构对压缩函数进行迭代实现的。Damgard 和 Merkle 定义了所谓 “压缩函数(compression function)”，就是将一个固定长度输入，变换成较短的固定长度的输出，这对密码学实践上 Hash 函数的设计产生了很大的影响。Hash 函数就是被设计为基于通过特定压缩函数的不断重复“压缩” 输入的分组和前一次压缩处理的结果的过程，直到整个消息都被压缩完毕，最后的输出作为整个消息的散列值。尽管还缺乏严格的证明，但绝大多数业界的研究者都同意，如果压缩函数是安全的，那么以上述形式散列任意长度的消息也将是安全的。这就是所谓 Damgard/Merkle 结构。任意长度的消息被分拆成符合压缩函数输入要求的分组，最后一个分组可能需要在末尾添上特定的填充字节，这些分组将被顺序处理，除了第一个消息分组将与散列初始化值一起作为压缩函数的输入外，当前分组将和前一个分组的压缩函数输出一起被作为这一次压缩的输入，而其输出又将被作为下一个分组压缩函数输入的一部分，直到最后一个压缩函数的输出，将被作为整个消息散列的结果。

不同于 SHA-2 使用的 MD 结构，美国国家标准与技术研究院（NIST）发布的第三代安全散列算法 Keccak 使用的是 [海绵建构](https://zh.wikipedia.org/wiki/%E6%B5%B7%E7%B6%BF%E5%87%BD%E6%95%B8)，也叫海绵函数。（这一部分深入解释待补充）

> [Hash 算法及其应用](http://www.iwms.net/n923c43.aspx)

## 对称密码

### 密钥配送问题

在对称密码中，如 AES，由于加解密使用的是相同的密钥，因此一定会遇到密钥配送问题（key distribution problem）。常见的解决方案中有事先共享密钥、通过密钥分配中心分配、Diffie-Hellman 密钥交换以及通过公钥密码来解决，下面介绍 Diffie-Hellman 密钥协商。

#### Diffie-Hellman 密钥协商

大多数密码技术的安全由数学难题保证，如 Diffie-Hellman 密钥协商算法的安全性依赖于有限群的离散对数问题。如下等式中，已知 g，x，p 求 y 十分容易，但是反向计算却非常难。在实际运算中，为了保证难度，通常对 g，x，p 有一定要求，例如建议 p 为 512 位的大素数，具体细节这里暂时忽略。

\\[ y \equiv g^{x} \  (\textrm{mod} \  p) \\]

以 Alice 和 Bob 为例，他们各自生成一个随机数 a，b，分别计算得到 A，B，并将计算的结果发送给对方，对方在接收到后进行二次计算得到最终的共享密钥，计算公式如下：

\\[ A \equiv g^{a} \  (\textrm{mod} \  p) \\]
\\[ B \equiv g^{b} \  (\textrm{mod} \  p) \\]
\\[ K \equiv B^{a} \  (\textrm{mod} \  p) = (g^{b})^{a} \ (\textrm{mod} \  p) \\]
\\[ K \equiv A^{b} \  (\textrm{mod} \  p) = (g^{a})^{b} \ (\textrm{mod} \  p)\\]

由此 Alice 和 Bob 得到了只有他们两个人知道的共享密钥，其中 g、p 两个素数公开，A 和 B 也公开，只有两个随机数 a 和 b 需要保密。为了更形象地理解，可以参考下面的 [示例](https://l2x.gitbooks.io/understanding-cryptography/docs/chapter-3/diffie-hellman%E5%AF%86%E9%92%A5%E4%BA%A4%E6%8D%A2.html)：

![](https://l2x.gitbooks.io/understanding-cryptography/assets/dh.png)

> [Diffie-Hellman Protocol](http://mathworld.wolfram.com/Diffie-HellmanProtocol.html)

#### ECDH

ECDH 密钥交换算法是 ECC 和 Diffie-Hellman 的结合，具体过程类似，只不过 ECDH 的安全性依赖于椭圆曲线上点构成的群所对应离散对数问题的复杂性。椭圆曲线密码学（ECC）具有高强度的安全性，可以在更短密钥的情况下达到相同的安全性。具体的会在下文单独介绍。下面通过 OpenSSL 密码库举个例子：

{% highlight bash %}
# 双方各自生成密码对
openssl ecparam -name secp256k1 -genkey -noout -out alice_priv_key.pem
openssl ec -in alice_priv_key.pem -pubout -out alice_pub_key.pem

openssl ecparam -name secp256k1 -genkey -noout -out bob_priv_key.pem
openssl ec -in bob_priv_key.pem -pubout -out bob_pub_key.pem

# 双方根据自身私钥和对方公钥生成 共享密钥
openssl pkeyutl -derive -inkey alice_priv_key.pem -peerkey bob_pub_key.pem -out alice_shared_secret.bin
openssl pkeyutl -derive -inkey bob_priv_key.pem -peerkey alice_pub_key.pem -out bob_shared_secret.bin

# 对比查看密钥是否一致
base64 alice_shared_secret.bin
base64 bob_shared_secret.bin

# 自定义传输内容
echo "Hi, Bob, this is secret message from Alice" > plain.txt

# 传送方加密传输内容
openssl enc -aes256 -base64 -k $(base64 alice_shared_secret.bin) -e -in plain.txt -out cipher.txt

# 接收方解密
openssl enc -aes256 -base64 -k $(base64 bob_shared_secret.bin) -d -in cipher.txt -out plain_again.txt

# 读取验证传输内容
cat plain_again.txt
{% endhighlight %}

虽然上述算法可以解决保密性问题，但由于 ECDH 密钥交换协议不验证公钥发送者的身份，因此无法阻止中间人攻击。如果监听者 Mallory 截获了 Alice 的公钥，就可以替换为他自己的公钥，并将其发送给 Bob。Mallory 还可以截获 Bob 的公钥，替换为他自己的公钥，并将其发送给 Alice。这样，Mallory 就可以轻松地对 Alice 与 Bob 之间发送的任何消息进行解密。他可以更改消息，用他自己的密钥对消息重新加密，然后将消息发送给接收者。为了解决此问题，可以使用公共证书颁发机构 (CA) 向双方提供可信数字签名密钥。

> [ECDH 算法概述](https://docs.microsoft.com/zh-cn/previous-versions/visualstudio/visual-studio-2008/cc488016(v=vs.90))

## 非对称密码

公钥密码（Public-key cryptography），也称非对称密码学（asymmetric cryptography）中，密钥分为公钥（public key）和私钥（private key），前者用于加密，后者用于解密，两者一一对应，统称密钥对（key pair）。举个例子，双方通信时，其中一方将自己的公钥给对方，要求对方用这个公钥加密后发消息给自己，这样一来，只有拥有的私钥的自己才能完成消息的解密，实现了消息的安全传输。对比可知，相比对称加密，非对称加密不存在密钥配送问题。

### RSA

RSA 算是最为普及的一种公钥密码算法，其可靠性由质因数分解的复杂度保证，即对两个质数相乘容易，而将其合数分解很难。一旦发现了对大整数进行质因分解的高效算法，就可以对 RSA 进行破解。为了安全起见，实际应用中 RSA 密钥长度可选择为 2048位，安全性要求更高也可以选择 4096位。RSA 生成密钥对的步骤如下：

1. 确定 N，首先用伪随机生成器生成两个大质数，记为 p，q，计算 N = *p* * *q*
2. 确定 L，L = *φ* (N) = *φ* (*p* ) * *φ* (*q*) = (*p* -1) * (*q*-1)
3. 确定 E，选择一个小于 L 的整数 E，并满足 E 和 L 互质
4. 确定 D，在求得 E 的基础之上，选择一个小于 L 大于 1 的整数 D，并满足 $$  E* D  \equiv 1 \  (\textrm{mod} \  L) $$

由上步骤计算得到 RSA 私钥对为(D，N)，公钥对(E，N)，进一步加解密计算公式如下：

\\[ \textrm{加密过程：} \ \textrm{Decrypted}= \textrm{Plain}^{E} \ mod \ N \\]
\\[ \textrm{解密过程：} \ \textrm{Plain}= \textrm{Decrypted}^{D} \ mod \ N \\]

上面只是对 RSA 做一个最基本的介绍，对数学基础和推导暂时不进行展开。如果想要深入了解可以看这篇文章：[Prime Number Hide-and-Seek: How the RSA Cipher Works](http://www.muppetlabs.com/~breadbox/txt/rsa.html)

### ECC

椭圆曲线密码学（ECC) 是一种高效的非对称加密算法，其中椭圆密码曲线（Elliptic Curve）完整的定义可参考 [Wolfram MathWorld](http://mathworld.wolfram.com/EllipticCurve.html)，一般情况下，可以用以下方程式表示：

![](/img/in-post/ECC/ECC-08.gif)

图示如下：

![](https://kangfeihk.com/img/in-post/ECC/ECC-01.png)

我们知道 RSA 正是利用了“**大数质因数分解**”的难度，从而保证了其安全性。同样地，Diffie-Hellman 密钥交换则是利用了“**有限域上的离散对数问题**”的复杂度。在 ECC 中，则是由“**椭圆曲线离散对数问题**”来保证其安全性。在介绍问题之前，先看一下相关预备知识：

#### 群

群是一个集合G，连同一个运算 "·"，它结合任何两个元素 a 和 b 而形成另一个元素，记为 a·b。符号"·"是具体的运算，例如加法。要具备成为群的资格，这个集合和运算 (G,·) 必须满足叫做群公理的四个要求：
1. 封闭性：对于所有 G 中 a, b，运算 a·b 的结果也在 G 中。
2. 结合性：对于所有 G 中的 a, b 和 c，等式 (a·b)·c = a· (b·c)成立。
3. 单位元：存在 G 中的一个单位元 0，使得对于所有 G 中的元素 a，等式 0·a = a·0 = 0 成立。
4. 逆元：对于每个 G 中的 a，存在 G 中的一个元素 b 使得 a·b = b·a = 0，这里的 0 是单位元。
如果加上第五条要求，
5. 要求符合交换律 a·b = b·a

这样的群我们称之为 **阿贝尔群（Abelian group）** 也叫 **交换群（commutative group）**。举个例子，整数集合 Z 不但满足群的定义，还符合阿贝尔群的要求。而自然数集合 N 由于第 4 条的限制，就不满足群的定义。

#### 椭圆曲线上的群公理

我们在椭圆曲线上定义群公理的几个要求：
1. 群中的元素都在椭圆曲线上
2. 单位元指向无穷远处 0
3. 曲线上任意一点 P 的逆元关于 x 轴对称
4. 加法定义如下：一条直线上与曲线相交地三个非零点 P，Q，R，满足加法运算规则，P + Q + R = 0.（如下图所示，重要）

![](/img/in-post/ECC/three-aligned-points.png)

这里的加法，我们不要求运算的顺序，P + (Q + R) = Q + (P + R) = 0，因此也满足阿贝尔群的定义。下面我们可以逐渐了解到 **ECC 的数学基础**正是由椭圆曲线上的点构成的阿贝尔加法群中 ECDLP 问题的复杂度。

下面我们看一下任意两个数的加法运算，以 P + Q 为例，我们知道 P + Q + R = 0，又因为阿贝尔群的定义，式子可以转换为 P + Q = - R, - R 和 R 关于 x 轴对称，直观的结果如下图所示

![](/img/in-post/ECC/point-addition.png)

下面看一下特殊情况的运算：
- 如果 P = 0 或者 Q = 0，由于定义了无穷远处 0 为单位元，因此可以得知，对于曲线上任意的点 P 或者点 Q，P + 0 = P 或者 Q + 0 = Q 都是成立的
- 如果 P = - Q，则根据逆元的定义可知，P 和 Q 关于 x 轴对称，P + Q = P +（- P） = 0
- 如果 P = Q，那么过 P 点的切线与椭圆曲线相交于另一点 R，得 P + P = - R

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

在同一条直线的三个点 P，Q，R，同样满足 P + Q + R = 0，只不过直线的定义需要更改为点集：$$ ax + by + c = 0 \ (\mathrm {mod} \; p) $$

给定一个非零点 Q，其相反数可以表示为：$$\left ( x_{Q},-y_{Q}\; \mathrm {mod} \; P \right )$$，例如在 $$\mathrm{F}_{29}$$ 上，记 Q 为 (2，5)，那么 - Q = (2，- 5 mod 29) = (2,24)

在实数域上，2P 的计算需要得到曲线在点 P 的切线，但在不连续的域上，不再适用这种方式。

好了，在了解了以上基础知识之后，回到椭圆曲线离散对数问题，本质即是

- 已知
    - 椭圆曲线 E
    - 椭圆曲线 E 上一点 G（基点）
    - 椭圆曲线 E 上的一点 xG（x 倍的 G）
- 求解
    - x

这个问题的难度保证了椭圆曲线密码的安全性。

> [A (relatively easy to understand) primer on elliptic curve cryptography](https://arstechnica.com/information-technology/2013/10/a-relatively-easy-to-understand-primer-on-elliptic-curve-cryptography/)
