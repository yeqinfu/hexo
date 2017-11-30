---
title: 学习https
date: 2017-11-21 09:00:58
tags: code
---

[学习参考](http://blog.csdn.net/clh604/article/details/22179907)

##### HTTPS

由HTTP+SSL/TLS组成。服务端配置证书可以自己制作或者权威机构购买。证书传送其实就是公钥传送，里面包含颁发机构，过期时间等。

##### 初始化

浏览器将自己支持的加密规则发送给服务端

##### 客户端解析证书

服务端选一组加密算法和hash算法，把信息征收发送给客户端。客户端TLS验证公钥是否有效，比如颁发机构，过期时间，如果异常会弹出警告。如果没有异常，就生成一个随机值。用证书对改随机值进行加密。

##### 传送加密信息

服务端得到随机值，服务端和客户端用这个随机值来进行加密解密。

##### 服务端解密信息

服务端用私钥解密之后，得到随机值（私钥），然后把内容通过该值进行对称加密。

##### 传输加密后的信息

服务端用随机值（私钥对称加密）传输内容

##### 客户端解密

客户端用对称加密（随机值）进行解密



#### ssl位置

ssl在应用层和tcp层之间。通过加入ssl头部再传到下一层。



![三次握手](http://wx2.sinaimg.cn/mw690/c1b251b3gy1flzvdmhlonj20w90yrq55.jpg)



上图梳理来自[博文代码](http://blog.csdn.net/u014386474/article/details/51669098)



[证书参考](https://www.cnblogs.com/fron/p/https-20170111.html)

## 生成服务器的密匙文件casserver.keystore



> ### keytool -genkey -alias casserver -keypass cas123 -keyalg RSA -keystore casserver.keystore -validity 365



-alias指定别名为casserver；

-keyalg指定RSA算法；

-keypass指定私钥密码；

-keystore指定密钥文件名称为casserver.keystore；

-validity指定有效期为365天。

另外提示输入密匙库口令应与-keypass指定的cas123相同.您的名字与姓氏fron.com是CAS服务器使用的域名（不能是IP，也不能是localhost），其它项随意填。

## 生成服务端证书casserver.cer

> keytool -export -alias casserver -storepass cas123 -file casserver.cer -keystore casserver.keystore

-alias指定别名为casserver；

-storepass指定私钥为liuqizhi；

-file指定导出证书的文件名为casserver.cer；

-keystore指定之前生成的密钥文件的文件名。

注意：-alias和-storepass必须为生成casserver.keystore密钥文件时所指定的别名和密码，否则证书导出失败

## 导入证书文件到cacerts 密钥库文件

接下来就是把上面生成的服务器的证书casserver.cer导入到cacerts密钥库文件中(后面的客户端会用到这些)

## 服务端Tomcat配置





服务端，android客户端https配置待续~~





















