---
title: 测试网epoch基本使用
date: 2018-08-30 09:00:58
tags: block chain
---

### ubuntu 普通用户ulimit修改

> 1.打开/etc/security/limits.conf，在里面添加如下内容
> `* soft nofile 100000``* hard nofile 100000``其中*表示所有用户  nofile表示最大文件句柄数,表示能够打开的最大文件数目`2.编辑/etc/pam.d/common-session，添加如下内容
> `session required pam_limits.so`
> 3.编辑/etc/profile，添加如下内容
> `ulimit -SHn 100000``然后重新启动机器,再利用ulimit -n查看文件句柄数,发现文件句柄数变为100000`

以上方法适合ubuntu16，但是ubuntu18并不适用。



### 下载epoch release 包

我们下载已经编译好的epoch包之后，
































