---
title: Bitcoin-NG
date: 2018-08-28 09:00:58
tags: block chain
---



关于Bitcoin-NG的原理，这里结合[aeternity测试网](https://explorer.aepps.com/#/)的区块和[Bitcoin-NG白皮书](https://www.usenix.org/system/files/conference/nsdi16/nsdi16-paper-eyal.pdf)做一个说明

![img](http://wx2.sinaimg.cn/large/c1b251b3gy1fup90bt9f8j20x80f4gq6.jpg)

和比特币不同的是POW共识是用来产生一个Leader，如何标志一个Leader产生呢？如上图最新的块314。这些有序号的块被称为Key Block。Key Block的挖掘旷工在接下来的一段时期（epoch）拥有记账权，也就是产生Micro Block。也就是如上图所示，314的Micro Blocks显示为0，但是当我调用智能合约的某个方法的时候，314这个块会产生变化，如下图所示：

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fup93nei6oj20vi0eiaek.jpg)



也就是在这个记账时期还没过去的一段时间内，314的旷工拥有持续一段时间的记账。

每个块都有指向上一个块的hash值，这个值可能是Key Block的hash也可能是Key Block最后一个Micro Block的hash



![image](http://wx1.sinaimg.cn/large/c1b251b3gy1fup8u30bfhj20ct056aah.jpg)



当A拥有记账权后的一段时间内（这个时间大概是由难度值控制的时间，比如十分钟，可能多或者少）A矿机一直在打包所有的交易，也就是产生Micro Block,这个是没有难度值的块，也就是没有工作量。跟Key Block不一样。当新的KeyBlock产生时候，也就是被全网大部分节点承认后，B（产生新的Key Block的矿机）就可以获取到A的所有交易费用的60%。

### 为什么A做的工作要B闯进来46分成？

这里假设A矿机的算力为α，前一个Key Block的收益比例为r（reader）后一个旷工收益比例为1-r



假设A私自挖矿（保留最后一个Micro Block,在此基础上挖矿，如果挖到了就获得了全部交易费，如果在没挖到之前被），那么A可以获得完全的收益，但是问题是发布出去的新的Key Block要被承认的概率是多少

某种程度上，算力比例就是代表收益比例，如果你的算力α=100%那么你的平均收益就是百分百。而如果A私自挖矿，A获得的收益为α，但是A在计算的同时，全网的算力也没有闲着，它们算出的概率为1-α




























