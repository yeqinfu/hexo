---
title: Android-app入口问题
date: 2019-05-17 12:00:58
tags: android
---

打开app的入口有多种

+ 从桌面app图标点击打开
+ 从系统桌面搜索app名字，打开
+ 安装该app后，安装器下面有一个打开按钮，点击打开等。

这几种不同方式打开一个app的mainactivity，比如，第一种方式打开后，进入mainactivity，然后再用第二种方式打开，会生成两个mainactivity实例。这两个实例通过发现它是在同一个栈中。如下图

![lALPBE1XYDPJByvNAQLNB0o_1866_258.png_620x10000q90g](https://wx1.sinaimg.cn/large/c1b251b3gy1g345yd629hj20h802e0sw.jpg)



而其实我们目标效果是，如果已经用某一种方式打开了。那么如果它用另一种方式打开app，那就直接进入这个mainactivity就好了。

###### 初步解决方法

把mainactivity设置为singleinstance。但是这个方式有两个缺点

> 背景：我们启动mainactivity之前是还有一个adactivity用来展示广告，展示完就关闭

+ mainactivity在一个独立的栈中，从adactivity转场到mainactivity，因为是栈转场会显得不友好

+ 而恰恰是因为有一个adactivity，当后台已经有一个mainactivity的时候再用另外一种方式启动，还是会先进入adactivity（因为adactivity是launcher activity）然后再栈转场到mainactivity

  > 这里想过把adactivity也设置为singleinstance但是呢？adactivity是广告，五秒后就自己结束了。所以，当用第二种方式进入的时候，还是会启动adactivity，而我们此时不希望启动adactivity



###### 再一个解决办法

> 我们让mainactivity保持单例是有必要的，因为没有单例打开fragment会报错

针对上main第二种一直重复启动adactivity的解决办法，在application中设置一个变量，标志着adactivity已经启动过了。如果我们app在后台活着，那么application的变量就起作用，把ad直接结束而不展示广告。

> 这里如果adactivity使用isTaskRoot来判断也可以，但是它只会判断一个栈，当我们一直点击回退，退到桌面，再点击图标进去，还是会展示广告。因为此时确实是taskroot。而如果使用变量来控制，就是所有栈都共享的了。

> 然而这种方法如果mainactivity设置为singleinstance会有各种的闪退效果，其实不是闪退，而是因为adactivity被我们手动结束掉了。这个栈清空了。把后台栈（也就是mainactivity）调用到前台，这中间有延迟，看起来像闪退了

所以，想到把mainactivity设置为singletop的启动模式，然后依然判断adactivity是否启动过，来手动关闭，这样没有闪退效果

> 虽然解决了不同入口不会重复加载广告，不会有栈转场的效果。但是当我们停留在主页面，然后一直点后退直到回到桌面。然后再点击我们的app，还是会有闪退的效果（其实还是不是闪退，是adactivity被我们结束了）。会造成这种效果，猜测是因为我们一直后退，把我们唯一的mainactivity从前台栈变成后台栈。app并没有被结束，是否结束取决于gc。然后我们点击我们的app图标，又走的是launcher activity。所以adactivity自我结束，然后系统把我们的后台栈mainactivity调用到前台。这期间才会出现这种闪退效果



##### 验证我们的猜测

+ 第一步，进入app后一直点后退，回到桌面，看看堆栈情况如下图

![lALPBE1XYDPI5FzNAe7NB_4_2046_494.png_620x10000q90g](https://wx3.sinaimg.cn/large/c1b251b3gy1g3460bb012j20h8046t94.jpg)



可以看到我们的app进入后台地址 id=12036 

然后点击app看到广告（这里改了代码，让它不结束，这样好打堆栈），停留五秒的期间看一下堆栈情况。

![lALPBE1XYDPIzUTNARrNB8g_1992_282.png_620x10000q90g](https://ws4.sinaimg.cn/large/c1b251b3gy1g3462x1ro4j20h802gglp.jpg)



看到堆栈id是12037.并且值得注意的是，这个栈中没有mainactivity，因为上一图中，我们的app只有一个栈12036

然后我们等五秒，进入主界面。看一下堆栈情况

![lALPBE1XYDPIxZTM-s0H0A_2000_250.png_620x10000q90g](https://wx4.sinaimg.cn/large/c1b251b3gy1g3465h9x84j20h8026t8u.jpg)

可以看到adactivity死后，mainactivity变成了在12037这个栈中

> 其实我这里少了一张图，就是在一直后退返回桌面之前，我们的app显示的时候，mainactivity的stackid是多少的截图。后来验证了一下，经过上述步骤，没后退桌面之前 mainactivity所在的栈id是169然后后退桌面
>
> 点击图标进入广告，广告的stackid是170，然后广告结束，mainactivity调到前台，stackid变成了170.说明一直后退这个机制，缓存了我们的mainactivity，并且把它加入了后面打开的栈中。



#### 总结

所以全局变量控制有闪退效果，不可取。最后使用singletop mainactivity

并且在adActivity使用isroottask来判断是否要展示来解决。

弊端是一直后退到桌面，再点击进去还会展示一次广告。（后期可以使用fragment来显示广告就可以解决这个问题）





### 为什么fragment的containerid找不到的问题

这个报错，是只要mainactivity有两个实例，就会报错。可以重现。

并且可以发现当有两个mainactivity然后去启动一个网页fragment，然后退其实可以发现，底部的mainactivity也启动了一样的网页fragment。

栈情况其实是mainactivity+fragment然后上面又是mainactivity+fragment 这种并行启动的问题导致白屏，

是因为用了eventbus post 了fragment导致的。比如打开x5fragment的时候，目前是把fragment post到接受事件的地方，而因为有两个mainactivity，所以两个mainactivity都会打开这个fragment。出现错乱。

所以保证了mainactivity是一个之后，这个其实可以不解决。但是最好不要把fragment当成eventbus的事件随便发送。

















