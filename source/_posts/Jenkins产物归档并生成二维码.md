---
title: Jenkins产物归档并生成二维码
date: 2020-04-08 09:00:00
tags: Jenkins
---

## 知识点

+ zxing二维码生成图片
+ jar包更改class文件重新打包
+ jenkins设置二维码提供扫码下载

## 前提

之前jenkins项目安装了蒲公英插件，定时打包任务完成之后会自动上传app的产物，蒲公英那边返回的下载地址也是一直是固定的，所以在邮件模板中我设定了二维码下载邮件。大概的样子

![image](https://tva1.sinaimg.cn/large/c1b251b3gy1gdm6v9cm2hj20rk0fvq46.jpg)

蒲公英那边的下载地址一直不变，下载地址中的二维码图片也是一直不会变。所以就直接使用了下载页的图片超链。

并且Jenkins作为一个简单的分发平台，我设置了任何人都有读的权限。然后把蒲公英的二维码也贴在了构建详情中，如下图

![image](https://tva4.sinaimg.cn/large/c1b251b3gy1gdm6wuxc5fj20v70khq52.jpg)

但是，问题来了。。。蒲公英是内测版本，扫码下载要登录。所以其实既然Jenkins作为我们自己的简单分发平台，点击产物也能直接下载，为何不自己在服务端生成一个二维码图片？不依赖蒲公英了。

## 使用谷歌zxing jar包命令行生成图片

网络上找到一个jar包生成没有问题，但是jar包的主函数竟然把我输入的字符串进行了一个toLowerCase的操作，导致我生成的url不对，这里需要重新去掉一点点代码，把class文件修改后重新放到jar包中。

```
package com.qihoo.test;

import com.qihoo.QRCodeUtil;
import javax.management.RuntimeErrorException;

public class QRcodeTest {
    static String image_parm = "";
    static String save_path = "";
    static String url_parm = "";

    public static void main(String[] args) throws Exception {
        if (args.length != 0) {
            String[] arrayOfString = args;
            int j = args.length;
            for (int i = 0; i < j; i++) {
                String arg = arrayOfString[i];
                if (arg.startsWith("url=")) {
                    url_parm = arg.trim().split("=")[1];
                } else if (arg.startsWith("image=")) {
                    image_parm = arg.trim().toLowerCase().split("=")[1];
                } else if (arg.startsWith("save=")) {
                    save_path = arg.trim().toLowerCase().split("=")[1];
                }
            }
            if (url_parm == "") {
                url_parm = "http://www.so.com";
            }
            QRCodeUtil.encode(url_parm, image_parm, save_path, true);
            return;
        }
        throw new RuntimeErrorException(null, "url or image save path are mandatory!");
    }
}
```

以上url_parm的赋值被我去掉了toLowerCase。然后使用eclipse重新build一个class文件出来，然后用zip工具进行一个拷贝替换。最后的可用jar包名字叫qrcode.jar

然后传到我们的服务器进行一个容器拷贝操作，把jar包拷贝到jenkins实例。

```
 docker cp qrcode.jar android_jenkins:/
```

## 更改打包脚本

我们传好了jar包之后，在构建操作增加了一些命令

![image](https://tva1.sinaimg.cn/large/c1b251b3gy1gdm79awbt3j21k80m040y.jpg)

除了assemble还有一个shell操作

```
cp app/build/outputs/apk/std/release/*.apk app/build/outputs/apk/std/release/food_safety.apk &&
java -jar /qrcode.jar url='http://218.6.70.66:30202/job/android_foodsafety/lastSuccessfulBuild/artifact/app/build/outputs/apk/std/release/food_safety.apk' image="lastapk.jpg" save="app/build/outputs/apk/std/release/"
```

因为打包的产物也就是apk名字是不固定的，这里做了一个简单的方式，直接是拷贝一份为固定名字的，然后再执行生成图片命令

> 其实这个图片地址一直是固定的，图片是固定的。是不会变的，去掉这个操作也是可以的。
>
> 另外，也可以不通过拷贝，但是每次构建我都会清空工作空间，所以才会写成固定的，不然每次重新打包都会导致以前的二维码失效

此时，我们应该保留三个东西，产物，拷贝产物，图片。所以归档命令改成

```
app/build/outputs/apk/std/release/*.apk,app/build/outputs/apk/std/release/*.jpg
```

最后改一下Jenkins的归档描述插件

```
<img src='http://218.6.70.66:30202/job/android_foodsafety/lastSuccessfulBuild/artifact/app/build/outputs/apk/std/release/lastapk.jpg' width=200px height=200px></br>
```

同时邮件中的二维码标签也可以改成这个。

就会拥有我们自己的下载二维码，摒弃蒲公英，变成了这样

![image](https://tva3.sinaimg.cn/large/c1b251b3gy1gdm7hpgntnj20y10mnwin.jpg)

































