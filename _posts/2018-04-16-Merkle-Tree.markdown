---
layout:     post
title:      "Merkle Tree 理解与实现"
date:       2018-04-16
author:     "K"
header-img: "img/post-bg-markdown.jpg"
tags:
    - Blockchain
---

### Merkle Tree 结构

Merkle Tree（默克尔树）的结构如下所示，它允许将大量数据压缩成单个哈希值，常常用于数据的完整性校验。默克尔树的叶子节点是文件的哈希值，非叶子节点是由直接下层的两个哈希值拼接后再次哈希得到。循环计算，直至得到根节点（Merkle root）。

![](/img/in-post/Merkle-Tree/MerkleTree.png)

在网络传输后，对于小文件的验证当然不用这么麻烦，只要直接进行哈希即可验证是否完整。但是一旦文件变大，就得切分成小的数据块，这样就算发生下载中断，也只需要重新下载缺失的部分文件。

Merkle Tree 的特点就是底层任意一个节点发生变化，其父节点直至根节点经过的所有节点，都会发生变化。与 Hash list 相比，虽然两者都能校验最终文件的完整性，且能在验证的情况下花费最小代价重新下载文件块，但是默克尔树能够在下载单个分支的文件后立即对其进行验证。


### Merkle root 计算实现

在比特币中，Merkle root 是在所有交易数据哈希值的基础上多次哈希得到，并且最终保存在 block header 中。下面解释一下具体的计算：

由于在比特币中使用的是小端字节序（little endian），因此需要进行转换，简单的说，计算机读取数据时，依次按顺序读取字节。如果是大端字节序，先读高位字节，反之则先读低位字节。详细可阅读阮一峰的 - [理解字节序](http://www.ruanyifeng.com/blog/2016/11/byte-order.html)，转换如下所示：

```
3a459eab5f0cf8394a21e04d2ed3b2beeaa59795912e20b9c680e9db74dfb18c
8cb1df74dbe980c6b9202e919597a5eabeb2d32e4de0214a39f80c5fab9e453a
```

在 bitcointalk 上有一个很好的回答，[Re: Merkle Tree example](https://bitcointalk.org/index.php?topic=44707.msg534605#msg534605) 解释了这个问题。

下面以两笔交易为例，实现代码，可以看到在比特币中使用了[两次哈希](https://bitcoin.stackexchange.com/questions/6037/why-are-hashes-in-the-bitcoin-protocol-typically-computed-twice-double-computed)。

{% highlight python %}
import hashlib

def hash256(s):
  return hashlib.new('sha256', s).digest()

def dhash256(s):
  return hash256(hash256(s))

tx1="3a459eab5f0cf8394a21e04d2ed3b2beeaa59795912e20b9c680e9db74dfb18c".decode('hex')[::-1]
tx2="be38f46f0eccba72416aed715851fd07b881ffb7928b7622847314588e06a6b7".decode('hex')[::-1]

mrkl_tree = dhash256(tx1+tx2)
result=mrkl_tree[::-1].encode('hex')

print result
{% endhighlight %}

此外，[BitCoin technology Merkle tree](http://java-lang-programming.com/en/articles/29) 一文用 Java 较完整地实现了Merkle tree，可加深理解。

### Merkle tree 应用

除了区块链和torrent，还有Certificate authorities、Highly scalable databases like Apache Cassandra and Dynamo DB 以及Digital signature alternatives to RSA 等等，详细可查看文章 - [Merkle Tree Introduction](https://medium.com/@evankozliner/merkle-tree-introduction-4c44250e2da7)。


> 参考：<br>
> [Merkle Trees](https://hackernoon.com/merkle-trees-181cb4bc30b4) <br>
> [Understanding Merkle Trees - Why use them, who uses them, and how to use them](https://www.codeproject.com/Articles/1176140/Understanding-Merkle-Trees-Why-use-them-who-uses-t) <br>