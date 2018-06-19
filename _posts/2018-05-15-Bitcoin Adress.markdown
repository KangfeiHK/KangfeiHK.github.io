---
layout:     post
title:      "浅析比特币地址"
date:       2018-05-15
author:     "K"
header-img: "img/post-bg-markdown.jpg"
tags:
    - 比特币
---

在比特币交易中，地址是不可或缺的一部分，它用来标识网络上的用户身份，可以理解成实体银行中的账户。其本质是 ECDSA（椭圆曲线数字签名算法）中的公钥，关于椭圆曲线的介绍会单独写一篇。本文主要关注比特币地址的相关概念及产生过程，下面从私钥开始。


### 私钥

私钥是钱包中最重要的数据，通过私钥可以计算出公钥、地址等，因此必须妥善保管和备份。其本身是一个256 位长的随机数，有效私钥的范围则取决于比特币使用的 secp256k1 椭圆曲线数字签名标准。格式类似这样：5KJvsngHeMpm884wtkJNzQGaCErckhHJBGFsvd3VyK5qMZXj3hS

随机数的生成建议使用 Linux 下 /dev/urandom 驱动器。

#### 私钥的格式

RAW 形式的私钥为 256 位二进制数值（32 字节），可以导出转换为其它格式，例如 Hexadecimal Format 是由进制转换得到 64 位 16 进制串。下面介绍一下更常见的 WIF Format（Wallet Export Format） / WIF Compressed。

WIF Format 就是在 64 位 16 进制串私钥前加上前缀 0x80，后面附加校验码，（所谓附加校验码，就是对私钥经过 2 次 SHA-256 运算，取两次哈希结果的前四字节），然后再对其进行 Base58 编码，就可以得到我们常见的 WIF（Wallet import Format) 格式的私钥，结果为 51 位的 Base58Check，数字 5 开头。

WIF-compressed 就是在 64 位 16 进制串对应的字节串前加上前缀 0x80，并加上后缀 0x01， 并用 Base58Check 编码（后面多了一个压缩标志），结果为 52 位的 Base58Check，字母 K 或 L 开头。

```
// Private key (WIF, uncompressed pubkey)
5Hwgr3u458GLafKBgxtssHSPqJnYoGrSzgQsPwLFhLNYskDPyyA

//Private key (WIF, compressed pubkey)
L1aW4aubDFB7yfras2S1mN3bqg9nwySY8nkoLmJebSLD5BWv3ENZ
```

转换示例如下：

```
1. 获取在 64 位 16 进制私钥串
0C28FCA386C7A227600B2FE50B7CAE11EC86D3BF1FBE471BE89827E19D72AA1D

2. 在私钥串前加上版本号（0x80 for mainnet addresses or 0xef for testnet addresses）. 并且如果是压缩版本，则需要在末尾添加 0x01 (Also add a 0x01 byte at the end if the private key will correspond to a compressed public key.)
800C28FCA386C7A227600B2FE50B7CAE11EC86D3BF1FBE471BE89827E19D72AA1D

3. 对上述字符串计算 SHA-256
8147786C4D15106333BF278D71DADAF1079EF2D2440A4DDE37D747DED5403592

4. 对计算结果再次计算 SHA-256
507A5B8DFED0FC6FE8801743720CEDEC06AA5C6FCA72B07C49964492FB98A714

5. 获取计算结果的前 4 个字节为校验和
507A5B8D

6. 在第二步的结果后面添加 4 个字节的校验和
800C28FCA386C7A227600B2FE50B7CAE11EC86D3BF1FBE471BE89827E19D72AA1D507A5B8D

7. 转换为 base58 编码
5HueCGU8rMjxEXxiPuD5BDku4MkFqeZyd4dZ1jvhTVqvbTLvyTJ
```

> [Wallet import format](https://en.bitcoin.it/wiki/Wallet_import_format)

### 公钥

公钥是由私钥经过椭圆曲线乘法计算所得，public key = private key * G， 比特币采用 [Secp256k1 算法](https://en.bitcoin.it/wiki/Secp256k1)，椭圆曲线公式如下：

![](/img/in-post/ECC/ECC-07.png)

公钥分为压缩和非压缩格式两者格式。物理意义上，公钥是椭圆曲线上的点，并具有 x 和 y 坐标，根据椭圆曲线的方程，y 可以通过 x 求得，因此在压缩形式的公钥中舍去 y，只保留 x。压缩格式 33 字节, 以 0x2 或者 0x3 开头，非压缩 65 字节, 以 0x4 开头。即非压缩形式为 04 x y（130 位十六进制 2+64+64），压缩形式为 02 x（y is even，66 位十六进制 2+64）、或者 03 x（y is odd）。

```
// 非压缩公钥示例
04
1DC1A701EBB8EF3FC55093E25D78DCB56D21F11DD88D62714549A38539978D9D
8C953064030B72C6D468DE2676ACB46197297124FBA4F58A3ADBFF93F8D583

// 压缩公钥示例
02
1DC1A701EBB8EF3FC55093E25D78DCB56D21F11DD88D62714549A38539978D9D
```

我们可以看到压缩形式可以减小 Tx/Block 的体积，每个 Tx Input 减少 32 字节。

> [Compressed vs. Uncompresed Private Keys](https://bitcointalk.org/index.php?topic=129652.0)

### 地址

在公钥的基础上进一步转换即可得到地址，注意压缩公钥生成对应压缩公钥的地址，未压缩公钥生成对应未压缩公钥的地址。详细过程参见下图：

![](/img/in-post/Bitcoin/address_gen.png)

> [List of address prefixes](https://en.bitcoin.it/wiki/List_of_address_prefixes)

#### Base58

先从 Base64 开始说起，Base64 是一种基于 64 个可打印字符来表示二进制数据的表示方法。Base58 是一种独特的编码方式, 是 Base64 的变形（All alphanumeric characters except for "0", "I", "O", and "l"）。其中 "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz" 为比特币中 base58 的字符集 (有序)，参考博客源码实现如下：

{% highlight C %}
#include <stdio.h>
#include <stdlib.h>
#include <string>
#include <openssl/bn.h>

#define DOMAIN_CHECK(c) ('0'<=(c)&&(c)<='9'||'a'<=(c)&&(c)<='f'||'A'<=(c)&&(c)<='F')

const char * BASE58TABLE="123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz";

std::string base58encode(const std::string & hexstring)
{
    std::string result = "";
    BN_CTX * bnctx = BN_CTX_new();
    BN_CTX_init(bnctx);
    BIGNUM * bn = BN_new();
    BIGNUM * bn0= BN_new();
    BIGNUM * bn58=BN_new();
    BIGNUM * dv = BN_new();
    BIGNUM * rem= BN_new();
    BN_init(bn);
    BN_init(bn0);
    BN_init(bn58);
    BN_init(dv);
    BN_init(rem);

    BN_hex2bn(&bn, hexstring.c_str());
    //printf("bn:%s\n", BN_bn2dec(bn));
    BN_hex2bn(&bn58, "3a");//58
    BN_hex2bn(&bn0,"0");

    while(BN_cmp(bn, bn0)>0){
        BN_div(dv, rem, bn, bn58, bnctx);
        BN_copy(bn, dv);
        //printf("dv: %s\n", BN_bn2dec(dv));
        //printf("rem:%s\n", BN_bn2dec(rem));
        char base58char = BASE58TABLE[BN_get_word(rem)];
        result += base58char;
    }

    std::string::iterator pbegin = result.begin();
    std::string::iterator pend   = result.end();
    while(pbegin < pend) {
        char c = *pbegin;
        *(pbegin++) = *(--pend);
        *pend = c;
    }
    return result;
}

int main(int argc, char * argv [])
{
    std::string hexstring = "";
    FILE * fin = stdin;
    while(!feof(fin))
    {
        char c = fgetc(fin);
        if (DOMAIN_CHECK(c))
            hexstring +=c;
    }
    fprintf(stdout, "%s", base58encode(hexstring).c_str());
    return 0;
}
{% endhighlight %}

{% highlight bash %}
# 编译运行
$ g++ -o base58_encode -g base58_encode.cpp -lcrypto
$ echo -n "00010966776006953D5567439E5E39F86A0D273BEED61967F6" | ./base58_encode

$ 6UwLL9Risc3QfPqBUvKofHmBQ7wMtjvM
# 添加首部 1
$ 16UwLL9Risc3QfPqBUvKofHmBQ7wMtjvM
{% endhighlight %}

> [【比特币】base58编码](https://blog.csdn.net/hacode/article/details/37815951)

### 示例

完整的比特币地址生成过程图示如下：

![](/img/in-post/ECC/ECC-05.png)

生成过程示例如下：

```
// 0 - Having a private ECDSA key
18e14a7b6a307f426a94f8114701e7c8e774e7f9a47e2c2035db29a206321725

// 1 - Take the corresponding public key generated with it (33 bytes, 1 byte 0x02 (y-coord is even), and
// 32 bytes corresponding to X coordinate)
0250863ad64a87ae8a2fe83c1af1a8403cb53f53e486d8511dad8a04887e5b2352

// 2 - Perform SHA-256 hashing on the public key
0b7c28c9b7290c98d7438e70b3d3f7c848fbd7d1dc194ff83f4f7cc9b1378e98

// 3 - Perform RIPEMD-160 hashing on the result of SHA-256
f54a5851e9372b87810a8e60cdd2e7cfd80b6e31

// 4 - Add version byte in front of RIPEMD-160 hash (0x00 for Main Network)
00f54a5851e9372b87810a8e60cdd2e7cfd80b6e31

// 5 - Perform SHA-256 hash on the extended RIPEMD-160 result
ad3c854da227c7e99c4abfad4ea41d71311160df2e415e713318c70d67c6b41c

// 6 - Perform SHA-256 hash on the result of the previous SHA-256 hash
c7f18fe8fcbed6396741e58ad259b5cb16b7fd7f041904147ba1dcffabf747fd

// 7 - Take the first 4 bytes of the second SHA-256 hash. This is the address checksum
c7f18fe8

// 8 - Add the 4 checksum bytes from stage 7 at the end of extended RIPEMD-160 hash from stage 4. This is the 25-byte binary Bitcoin Address.
00f54a5851e9372b87810a8e60cdd2e7cfd80b6e31c7f18fe8

// Convert the result from a byte string into a base58 string using Base58Check encoding. This is the most commonly used Bitcoin Address format
1PMycacnJaSqwwJqjawXBErnLsZ7RkXUAs
```

下图是比特币站点提供的 Address map，详尽描述了关于地址生成的流程

![](/img/in-post/Bitcoin/Address_map.jpg)

### OpenSSL 实现

通过 OpenSSL 命令行实现地址生成

```
// # 0 - Private ECDSA Key
// -name 指定名称，比如 secp112r1、 secp160r1、这里比特币采用的是 secp256k1
// -genkey，生成密钥、-out 输出文件
// der A way to encode ASN.1 syntax in binary, a .pem file is just a Base64 encoded .der file
$ openssl ecparam -name secp256k1 -genkey -rand /dev/urandom -out priv.pem
$ openssl ec -in priv.pem -outform DER | tail -c +8 | head -c 32 | xxd -p -c 32

// # 1 - Public ECDSA Key
// 得到公钥（priv.pem 生成 pub_key）
$ openssl ec -in priv.pem -pubout -outform DER | tail -c 65 | xxd -p -c 65
$ 04D0D0371B4007ABE71660D95494897CD3D9C9B5B799EF8EB8AE4741158ABE56ABE8EB153928C3786D9494DC21703CF916E1EDB34E23CCF3F38B26AB1060D93A42

// # 2 - SHA-256 hash of 1
// convert (or patch) hexdump into binary first, then hash
$ echo -n "04D0D0371B4007ABE71660D95494897CD3D9C9B5B799EF8EB8AE4741158ABE56ABE8EB153928C3786D9494DC21703CF916E1EDB34E23CCF3F38B26AB1060D93A42" \ | xxd -r -p | openssl dgst -sha256
$ 6FBB27A96C11330860CAB0507E0594853860CC7C7061EB2D679C59B8C80CD7DC

// # 3 - RIPEMD-160 Hash of 2
$ echo -n "6FBB27A96C11330860CAB0507E0594853860CC7C7061EB2D679C59B8C80CD7DC" \ | xxd -r -p | openssl dgst -ripemd160
$ 022CCB7E966D8B21B020765EE0646B8FA3B34805

// # 4 - Adding network bytes to 3（比特币主网版本号 “0x00”）
$ 00022CCB7E966D8B21B020765EE0646B8FA3B34805

// # 5 - SHA-256 hash of 4
// convert (or patch) hexdump into binary first, then hash
$ echo -n 00022CCB7E966D8B21B020765EE0646B8FA3B34805 \ | xxd -r -p | openssl dgst -sha256
$ 4998860B769246F7338CFDA8DBD8FB33D842B3073DEDC1EF9BC59CAA74829817

// # 6 - SHA-256 hash of 5
// 由于是两次 sha256，所以需要再重复一次，取结果的前四个字节
$ echo -n 4998860B769246F7338CFDA8DBD8FB33D842B3073DEDC1EF9BC59CAA74829817 \ | xxd -r -p | openssl dgst -sha256
$ 3C316AC80FD73C6329D5A500F44EB5167AE008F04E7F538F59D600281934F409

// # 7 - First four bytes of 6（校验）
$ 3C316AC8

// # 8 - Adding 7 at the end of 4
$ 00022CCB7E966D8B21B020765EE0646B8FA3B348053C316AC8

// # 9 - Base58 encoding of 8
$ 1CW1nhVdpUMNFa9D7NQVTrwehFUdJZncw
```

> [How do these OpenSSL commands create a Bitcoin private/key from a ECDSA keypair](https://bitcoin.stackexchange.com/questions/59644/how-do-these-openssl-commands-create-a-bitcoin-private-key-from-a-ecdsa-keypair)