---
title: 记一次linux内核编译
date: 2018-10-11 09:00:58
tags: code
---



[内核下载地址](ftp://linux.cis.nctu.edu.tw/kernel/linux/kernel/v2.6/)

解压到/usr/src/kernels/目录下，进入目录看到如下内容

![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fw42sgcu31j20jm0gjgps.jpg)



#### make mrproper

保持干净源代码，删除编译过程产生的目标文件(*.o)，这个操作会将以前进行过内核功能选择文件也删除掉，第一次执行内核编译才执行这个操作。如果想删除前一次编译过程的残留数据，只要执行``` make clean ```

#### 挑选内核功能 make XXconfig

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fw4353t70pj20vf05pt9x.jpg)

```
make menuconfig
```

报错如上图，增加执行

```
sudo apt-get install libncurses5-dev
```

后面还是报错，加了sudo就图型就出来了

![image](http://wx2.sinaimg.cn/large/c1b251b3gy1fw43f7xp82j20ww0kgju4.jpg)



内核编译```[*]<*> ``` 这个代表编译进内核，``` [M] ```这个代表编译进模块

### General setup

与linux最相关的程序互动，内核版本说明都在这里，一般默认值。所以这里都没改动

### loadable module + block layer

![image](http://ws4.sinaimg.cn/large/c1b251b3gy1fw447y3lhbj20gv06ut93.jpg)



勾选了这几个

![image](http://ws4.sinaimg.cn/large/c1b251b3gy1fw449590pkj20hi05uwel.jpg)



配置项太多了，可能我稍后需要精简，但是第一次就这样吧，直接保存配置

### 内核编译和安装



```
make clean
```

编译内核

```
make bzImage
```



![image](http://ws2.sinaimg.cn/large/c1b251b3gy1fw4a7jyzumj20n809jwgs.jpg)

报错，说是没有这个文件，据说是版本太新了才没有，想办法拷贝[一个进去](https://raw.githubusercontent.com/torvalds/linux/v4.0/include/linux/compiler-gcc5.h)。

把链接的代码写入文件，拷贝到include/linux/目录下面

但是还是有很多其他的错误

所以，这里打算放弃了，换一下不同版本的源码

























