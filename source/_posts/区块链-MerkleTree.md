---
title: 区块链-MerkleTree
date: 2018-06-05 09:00:58
tags: block chain
---



学习笔记，[参考](https://blog.csdn.net/wo541075754/article/details/54632929)

区块每一笔交易都是MerKleTree的节点，最后的Merkle根参与区块的哈希。了解一下什么是MerkleTree

#### HashList Merkle Tree的区别

Hash List可以看作一种特殊的Merkle Tree，即树高为2的多叉Merkle Tree

根据以上博客内容可以参考得知哈希树的应用可以用来P2P下载，此外比特币的交易记录也是通过该树完成打包成块，多节点下载的好处很明显，速度快。

![哈希树](https://ws1.sinaimg.cn/large/c1b251b3gy1fs0cb6uo45j20qd0rg0ty.jpg)



如图，哈希树的每相邻哈希串联成新的字符串，然后对这个字符串进行哈希得到新的根节点，以此类推最终得到root hash。

#### 时间复杂度

回顾一下时间复杂度是如何计算的。假设数据块数量为n，那么从底下的节点hash次数就分别为n，n/2,n/4一直到n=1

此时的计算次数

> sum=n+n/2+n/4....+1

很明显呢这是一个等比数列q=1/2

![等比求和公式](https://ws1.sinaimg.cn/large/c1b251b3gy1fs0ck7ptttj204c0140nu.jpg)

根据求和公式得到

> sum=2n*(1-0.5^n)

去掉常量，并且当n趋近于无穷大，0.5^n项为趋近于0，最后

> sum=n 时间复杂度为n



