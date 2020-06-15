---
title: Jenkins打包Android
date: 2020-03-26 09:00:00
tags: Jenkins
---

## 实现目标

+ 每天下班定时打包
+ 打包后的产物归档
+ 打包后收集，从上一次打包到本次打包的commit记录（也就是要求我们写好commit）
+ 把产物和打包更新日志（commit日志）作为邮件模板内容发送给指定人

## 脚本步骤

+ 前面添加描述，gitbucket账号配置，定时策略就略过了

+ 执行android脚本

  ![image](https://tva3.sinaimg.cn/large/c1b251b3gy1gd76p1giv5j20yv0lw75x.jpg)

  > 能够执行成功的前提是 环境已经下了Android SDK，SDK tools。并且配置好了ANDROID_HOME变量

+ 产物归档

  ![image](https://tvax1.sinaimg.cn/large/c1b251b3gy1gd7a4gd0ufj20yc0843yw.jpg)

  > 通过正则提取指定产物

## 邮件配置

+ 系统管理员配置

![image](https://tva2.sinaimg.cn/large/c1b251b3gy1gd7a7zf08tj20tv05st8w.jpg)

> 设置系统管理员邮箱

+ [腾讯邮件SMTP服务配置](https://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=371)

  ![image](https://tva3.sinaimg.cn/large/c1b251b3gy1gd7afov1moj20x20izdgx.jpg)

+ 填写默认的邮件模板

[构建邮件模板HTML](https://www.jianshu.com/p/67aedb3582c7)

> 这个模板内容就包括了最近提交的日志，所以我们提交代码应该以功能开发或修复为提交单元，规范自己的提交格式。提交的记录也是作为阐明我们更新日志的内容

> 具体的构建邮件通知主要靠这个Extended e-mail notification插件，需要jenkins下载这个插件
>
> 这个插件可以帮忙定义发送邮件的内容，和构建结果不同而触发不同的发送方式

+ 构建触发

  ![image](https://tva4.sinaimg.cn/large/c1b251b3gy1gd7dhvd0vmj21l00mddi2.jpg)

  > 邮件发送的触发，这里设置了两个，党打包失败的时候发送邮件给开发，打包成功则发送给默认接受列表
  >
  > 这个列表在系统配置那边添加，可以添加多个。

## 发送邮件

构建成功后

![image](https://tvax4.sinaimg.cn/large/c1b251b3gy1gd7dlj4mskj211k0oxac0.jpg)

![image](https://tva1.sinaimg.cn/large/c1b251b3gy1gd7dlunsh7j20uf0i8my8.jpg)

可以大致看到此次构建的一些增量功能信息和产物

## 后记

构建频率设置为每天构建一次，后续大版本更新应该还需要以下功能

+ 构建成功自动发布到蒲公英平台，并邮件通知产品等

+ QQ或者钉钉构建通知机器人

+ 单元测试用例加入构建

  

