---
title: Android逆向入门（一）
date: 2018-03-28 12:00:58
tags: code
---



####  如何动手？

破解Android程序通常的方法是将apk文件利用ApkTool反编译，生成Smali格式的反汇编代码，然后阅读Smali文件的代码来理解程序的运行机制，找到程序的突破口进行修改，最后使用ApkTool重新编译生成apk文件并签名，最后运行测试，如此循环，直到程序被成功破解。

#### 反编译apk文件

使用ApkTool反编译，这边选择一个没有被加固的，我自己写的一个release版本的apk。

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fpsdugbpftj20sq0p8myv.jpg)



上图是apktool的命令参数。然后开始根据格式输入命令

> apktool.bat d ZXBZW-release.apk test

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fpsdyzmb9gj21330ieace.jpg)

这边出现部分错误，目前也不知道为什么会有这个错误，先略过。

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fpse0d981ej20jy0fodh7.jpg)

test文件夹出现了一系列目录与文件。smali目录下存放了程序所欲的反汇编代码，res文件夹没有出现。

查了网上的原因，说是因为ApkTool版本太低了。[回答链接](http://www.cnblogs.com/sage-blog/p/4323049.html)

> apktool.bat -version 显示1.5.2

[apkTool 下载地址](https://ibotpeaches.github.io/Apktool/)更新ApkTool后,用jar文件反编译，没有报错

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fpsico0juqj21h70h9ta1.jpg)



![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fpsif1k51qj20ib0bwq3k.jpg)

smali目录存放了程序所有的反汇编代码，res目录则是程序的资源文件。

#### 分析问题

一般Android来说，错误提示信息通畅是指引关键代码的风向标，在错误提示附近一般是程序的核心验证代码。错误提示字符串可能是硬编码到源码中，也可能引用自“res\values”目录下的strings.xml文件。

下面打开“res\values\strings.xml”查看内容。

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fpsil8vmx2j20r00p4acv.jpg)



开发android程序时，String.xml文件的所有字符串都在“gen/<packagename>/R.java”文件的String类中被标识，每个字符串都有唯一的int类型索引值。反编译后，索引值保存在string.xml文件同目录下的public.xml文件中。

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fpsipgewd5j20kk0c8dgt.jpg)



在这里我们查找登录逻辑，在string.xml中

> <string name="token_invalid">登录过期请从新登陆！</string>

查找这个字符串，对应public.xml中

>  <public type="string" name="token_invalid" id="0x7f080210" />

得到了这个id值，然后在smali目录中搜索含有内容为0x7f080210的文件。

[常用工具包合集](https://down.52pojie.cn/Tools/Android_Tools/)

目前不知道专业人士怎么查，我用as打开这个目录ctrl+h查到了这个id

![image](http://ws1.sinaimg.cn/large/c1b251b3gy1fpsk4q5og6j211u0k9myy.jpg)





#### smali语法

```
.field private isFlag:z　　定义变量

.method　　方法

.parameter　　方法参数

.prologue　　方法开始

.line 12　　此方法位于第12行

invoke-super　　调用父函数

const/high16  v0, 0x7fo3　　把0x7fo3赋值给v0

invoke-direct　　调用函数

return-void　　函数返回void

.end method　　函数结束

new-instance　　创建实例

iput-object　　对象赋值

iget-object　　调用对象

invoke-static　　调用静态函数
条件跳转分支：

"if-eq vA, vB, :cond_**"   如果vA等于vB则跳转到:cond_**
"if-ne vA, vB, :cond_**"   如果vA不等于vB则跳转到:cond_**
"if-lt vA, vB, :cond_**"    如果vA小于vB则跳转到:cond_**
"if-ge vA, vB, :cond_**"   如果vA大于等于vB则跳转到:cond_**
"if-gt vA, vB, :cond_**"   如果vA大于vB则跳转到:cond_**
"if-le vA, vB, :cond_**"    如果vA小于等于vB则跳转到:cond_**
"if-eqz vA, :cond_**"   如果vA等于0则跳转到:cond_**
"if-nez vA, :cond_**"   如果vA不等于0则跳转到:cond_**
"if-ltz vA, :cond_**"    如果vA小于0则跳转到:cond_**
"if-gez vA, :cond_**"   如果vA大于等于0则跳转到:cond_**
"if-gtz vA, :cond_**"   如果vA大于0则跳转到:cond_**
"if-lez vA, :cond_**"    如果vA小于等于0则跳转到:cond_**

```



#### java 对比分析

```java
private boolean ifSense(){
        boolean tempFlag = ((3-2)==1)? true : false;
        if (tempFlag) {
            return true;
        }else{
            return false;
        }
    }
```



```
.method private ifSense()Z//定义方法
    .locals 2 // 指令表明了方法中非参寄存器的总数，出现在方法中的第一行

    .prologue//方法开始
    .line 22
    const/4 v0, 0x1     // v0赋值为1

    .line 24
    .local v0, tempFlag:Z
    if-eqz v0, :cond_0            // 判断v0是否等于0, 不符合条件向下走, 符合条件执行cond_0分支

    .line 25
    const/4 v1, 0x1            // 符合条件分支

    .line 27
    :goto_0
    return v1

    :cond_0
    const/4 v1, 0x0            // cond_0分支

    goto :goto_0
.end method
```

##### const语句

const语句其实就是赋值。const v1 0x0 就是把0x0赋值给v1寄存器。而const/height16不是单纯的赋值，需要将值右边零拓展为32位再赋给寄存器。如

> const/height16 v1 0x4120

其实是0x41200000赋值给v1

> const/4 vx,lit4  Puts the 4 bit constant into vx 这也就这么类似

现在查看，反编译中的id被赋值给v0这个变量































