---
title: Android编译兼容性
date: 2019-02-13 14:00:58
tags: android
---

#### 区别minSdkVersion，targetSdkVersion，compileSdkVersion

minSdkVersion好理解，略。

targetSdkVersion这个决定了应用在实际的操作系统中是否开启兼容模式，比如targetSdkVersion=23就是android6.0中引入了动态权限申请，也就是我们如果把targetSdkVersion设置为>=23，举例拍照，我们就要更改写法，不只是在清单中声明了拍照权限就可以，而是要在使用的地方去动态申请这个权限，而如果我们把targetSdkVersion设置为22，那如果设备的操作系统是23，那么当前的android设备会启动兼容模式运行你的程序，换句话说，你不用动态申请牌照权限，只要清单声明就可以。然而，在调用相机的地方，兼容模式会帮你弹出需要相机权限这个请求。

compileSdkVersion就是你的代码需要用哪个sdk去编译，编译过程会检测出错误，警告或者新api特性等等

#### 举例

我们假设一个app它三个值

```
 minSdkVersion 15
 targetSdkVersion 28
 compileSdkVersion 28
```

目前28是最新的android9.0

也就是要兼容到15，用28编译，如果设备超过28会开启兼容模式，我们不用处理。我们要处理的是，如果调用一个函数是28才有的，那么运行在15的设备会怎样？

[点击这里查看每个版本新增的api](https://developer.android.com/reference/android/view/package-summary)

![image](https://wx4.sinaimg.cn/large/c1b251b3gy1g05r35gahpj219a0lfadi.jpg)

可以看到我们把level设置为15，部分api为灰色，代表15这个版本设备没有该api，那么我们的minSdkVersion设置为15，就不能用ViewAnimationUtils，因为这个added in [API level 21]

![image](https://wx3.sinaimg.cn/large/c1b251b3gy1g05r5f67wtj20ye038wep.jpg)

调用的时候出现警告，如果我们无视这个警告会怎样，我电脑模拟器android版本为5.1.1（level 22），一台手机8.0（level 26）

![image](https://ws4.sinaimg.cn/large/c1b251b3gy1g05smjjkr2j20wq0lpmzj.jpg)

如图，打印了模拟器的版本是22，但是

![image](https://wx1.sinaimg.cn/large/c1b251b3gy1g05snmuzxfj20mw03rglx.jpg)

这个类需要23，这里没做任何处理，跑到模拟器，发现并没有崩溃。换了一个模拟器也没有崩溃。看一下安卓添加快捷方式的代码

![image](https://ws1.sinaimg.cn/large/c1b251b3gy1g05tykslxaj20tf0e6q4m.jpg)

![image](https://ws2.sinaimg.cn/large/c1b251b3gy1g05txnsphpj20um06c75b.jpg)

这次终于奔溃了，我不知道没奔溃的原因，但是低版本调用高版本的api是不能忽视它在编译期间的警告的，会导致在低版本设备奔溃。

#### 处理这些警告

第一种按照提示，根据版本代码判断来执行相应的逻辑

![image](https://ws2.sinaimg.cn/large/c1b251b3gy1g05u15z1ckj20qh0adgmv.jpg)

第二种，根据提示添加注解

![image](https://ws2.sinaimg.cn/large/c1b251b3gy1g05u35ah51j20v30ci0ua.jpg)

试一试第二种会不会一样奔溃，当然，结果是奔溃的，也就是说这几个注解没啥卵用。那么它是干嘛的？

```
@SuppressLint("NewApi"）屏蔽一切新api中才能使用的方法报的android lint错误
用@TargeApi($API_LEVEL) 使可以编译通过
```

也就是说其实，这几个注解只是帮你度过编译期间的lint检查而已，并不能影响运行时。所以最好的做法还是第一种。

#### targetSdkVersion，compileSdkVersion

知道targetSdkVersion是是否开启兼容模式的根本之后，想到正常情况下，这两个值是相等的。如果这两个值不相等会如何，很明显compileSdkVersion是作用于编译期间。也就是使用哪个android.jar来编码。我这里突然改成

compileSdkVersion=15，那编辑器直接报红色，因为压根没找到这个类。而如果compileSdkVersion>targetSdkVersion会怎样？我猜可能你使用高版本api的时候编译器不会报错，但是运行会报错。总之就是targetSdkVersion=compileSdkVersion比较不会有问题，别瞎搞。

![image](https://wx4.sinaimg.cn/large/c1b251b3gy1g05um9tnbtj20dg04emxh.jpg)