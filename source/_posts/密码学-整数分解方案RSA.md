---
title: 密码学-整数分解方案RSA为例
date: 2018-06-11 15:00:00
tags: cryptography
---



### 公钥算法家族

整数分解方案：有效公钥方案是基于分解大整数是非常困难的，代表例子RSA

离散对数方案：基于有限域内的离散对数问题，例子 Diffied-Hellman密钥交换，Elgamal加密，数字签名DSA

椭圆曲线EC方案：离散对数算法的一个推广就是椭圆曲线公钥方案，例子椭圆需要Diffie-Hellman密钥交换，椭圆曲线数字签名算法ECDSA



### 整数分解方案

##### 欧几里德算法

![image](http://ws4.sinaimg.cn/large/c1b251b3gy1fs7ardwc85j20wc07c418.jpg)



这段话中，如何证明x-y和y也互为素数？辗转相减法或者辗转相除法也是运用这个结论去求得两数的最大公约数，而互为质数的两数，最大公约数为1。

>  证明x-y与y互为素数
>
> 

























































