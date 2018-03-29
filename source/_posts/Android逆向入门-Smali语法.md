---
title: Android逆向入门-smali语法
date: 2018-03-28 14:00:58
tags: code
---



### smali语法

[出处](https://blog.csdn.net/wdaming1986/article/details/8299996)

android编译工具编译成dex，android系统的DVM会执行该文件，smali/baksmali则是DVM可执行文件的汇编器/反汇编器。反汇编Dalvik可执行文件后（dex），将会的带smali后缀文件，smali代码拥有特定语法。

其中dex文件和smali文件可以通过smali，baksmali工具相互转换，而smali文件无法完整转化成java文件，可能是dx工具将java文件的字节码的class文件转化成文件的过程中，进行了重新排列，取出了多余的信息。

```
.class public Lcom/cn/daming/activity/HelloWorldAppActivity;
.super Landroid/app/Activity;
.source "HelloWorldAppActivity.java"


# instance fields
.field private mTextView:Landroid/widget/TextView;


# direct methods
.method public constructor <init>()V
    .locals 0

    .prologue
    .line 7
    invoke-direct {p0}, Landroid/app/Activity;-><init>()V

    return-void
.end method


# virtual methods
.method public onCreate(Landroid/os/Bundle;)V
    .locals 2
    .parameter "savedInstanceState"

    .prologue
    .line 12
    invoke-super {p0, p1}, Landroid/app/Activity;->onCreate(Landroid/os/Bundle;)V

    .line 13
    const/high16 v0, 0x7f03

    invoke-virtual {p0, v0}, Lcom/cn/daming/activity/HelloWorldAppActivity;->setContentView(I)V

    .line 14
    const/high16 v0, 0x7f05

    invoke-virtual {p0, v0}, Lcom/cn/daming/activity/HelloWorldAppActivity;->findViewById(I)Landroid/view/View;

    move-result-object v0

    check-cast v0, Landroid/widget/TextView;

    iput-object v0, p0, Lcom/cn/daming/activity/HelloWorldAppActivity;->mTextView:Landroid/widget/TextView;

    .line 15
    iget-object v0, p0, Lcom/cn/daming/activity/HelloWorldAppActivity;->mTextView:Landroid/widget/TextView;

    const/high16 v1, 0x7f04

    invoke-virtual {v0, v1}, Landroid/widget/TextView;->setText(I)V

    .line 16
    return-void
.end method
```

对应java代码

```java

package com.cn.daming.activity;

import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;

public class HelloWorldAppActivity extends Activity {
	private TextView mTextView;
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        mTextView = (TextView)findViewById(R.id.text_view);
        mTextView.setText(R.string.hello);
    }
}
```



Dalvik 字节码有两种类型：原始类型；引用类型（包括对象和数组）

原始类型：V  void只能用于返回值类型

​                    Z boolean类型

​                     B byte

​                      S short

​                      C char

​                      I int

​                      J long（64位）

​                      F float

​                      D double（64bit）

对象类型：Lpackage/name/ObjectName;相当于java中的package.name.ObjectName

​                   L: 表示这个是一个对象类型

​                    package/name:该对象所在的包

​                    ;  :表示对象名称结束



数组的表示形式：

​                   [I :表示一个整形数组的以为数组，相当于java的int[]

​                   对于多维数组，只要增加[就可以了。[[I =int\[][]; 注，每一维最多255个

对象数组的表示形式：

​                   [Ljava/lang/String 表示String的对象数组；

方法的表示形式：

​                   Lpackage/name/ObjectName;->methodName(III)Z

​                   Lpackage/name/ObjectName表示类型

​                   methodName 表示方法名

​                    III表示参数，这里表示三个整形参数，方法参数是一个接一个的，中间没有隔开

字段的表示形式：

​                   Lpackage/name/ObjectName;->FieldName:Ljava/lang/String

​                   表示包名，字段名和各字段类型

有两种方式指定一个方法有多少个寄存器是可用的：

​                  .register 指令指定了方法中的寄存器的总数

​                  .locals指令表明了方法中非参寄存器的总数，出现在方法的第一行

方法的传参：

​                  当一个方法被调用的时候，方法的参数被置于最后N个寄存器中；

​                  例如，一个方法有两个参数，那么参数将至于最后两个寄存器，非静态方法中的第一个参数总是调用改方法的对象；对于静态方法除了没有隐含的this参数以外，其他都一样；

寄存器的命名方式：

​                  V命名

​                  P命名 第一个寄存器就是方法中的第一个参数寄存器

​                  比较：使用P命名是为了防止以后如果在方法中增加寄存器，需要对参数寄存器重新进行编号的缺点

​                  特别说明：Long和Double类型的64bit需要两个寄存器

例如：对于非静态方法

​                  LMyObject->myMethod(IJZ)V;

​                  有四个参数：LMyObject,int,long,boolean;需要五个寄存器来存储参数

​                  P0 this

​                  P1 I(int)

​                  P2,P3 J(long)

​                  P4 Z(boolean)

现在回头看smali代码就一目了然了













