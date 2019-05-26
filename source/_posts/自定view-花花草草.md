---
title: 自定义view-花花草草
date: 2019-05-27 14:00:58
tags: android
---

先上效果图

<img src="https://wx4.sinaimg.cn/large/c1b251b3gy1g3f6vh94cqj20ct0m6ju1.jpg" alt="GMW}Z~6ZCYV6T}@~[JLEAMM" width="461" data-width="461" data-height="798">

我不知道这个有什么用，就是想画一画。本来想画像一颗海草海草，然后随风飘摇。

目前这个，还没能飘起来，没写动画。

具体思路如下

#### 画叶子

先定义一个叶子的类。专门负责画叶子的

```java
  /**
     * 定点和根部 只要确定这两个点就可以画出叶子了
     */
    Point top;
    Point buttom;
    /*两点的距离*/
    float distance;
    /*两点的斜率*/
    float a;
    /*atan值*/
    double atan;
    double rotateValue;
```

如题，画叶子，如果知道叶子的根部和顶部的点的位置，那么画叶子也就画出来了。

distance是根部和顶部的距离。a是以这两点为必经点的一个一次函数的斜率。斜率知道了那么atan的值也就知道了，那么它和x轴的偏移角度也就知道了。

```java
  /**
     * 把叶子画出来吧！
     *
     * @param canvas
     */
    public void drawSelf(Canvas canvas) {
        canvas.save();
        a = (buttom.y - top.y) * 1.0f / (buttom.x - top.x);
        atan = Math.atan(a);
        rotateValue = (float) (atan / (2 * Math.PI) * 360);
        if (atan<0){
            rotateValue=180+rotateValue;
        }
        /*移动到根部*/
        canvas.translate(buttom.x, buttom.y);
        canvas.rotate((float) rotateValue);
        Path path=new Path();
        path.moveTo(0,0);
        path.quadTo(distance/2,distance/4,distance,0);
        path.moveTo(0,0);
        path.quadTo(distance/2,-distance/4,distance,0);
        canvas.drawPath(path,getTextPaint());
        canvas.restore();

    }

```

这里为了方便画图，要记得保存图层来画图

偏移的角度atan可能是负数，也就是叶子在左边的情况下。反过来计算出需要旋转的角度rotateValue

知道了角度之后，把画布旋转过来，画布原点在叶子根部，叶子定点在x轴方向，这样画二阶贝塞尔曲线就很容易啦。贝塞尔曲线的控制点目前定义在中点，高度为距离的四分之一。画完记得恢复画布

### 画一颗小草

因为可能有很多的小草，需要定义小草类。

```
   /**小草高度，默认是2，一个单位高度是一个二阶贝塞尔曲线弧*/
    int height=5;
    /**每一个单位高度的真实高度*/
    float radianHeight=300;
    /**小草扭动幅度，也就是控制点偏移量*/
    float radianOffset=radianHeight/4;
    /**小草的根部位置*/
    Point rootPostion=null;
    /**
     * 叶子和茎的偏移角度
     *
     * */
    float angle=60;
    /**
     * 叶子的数量，不包括顶部一个
     * 目前先暂定，一个弧度只有一个叶子，其实可以控制更多
     * TODO 自定义多个叶子
     * */
    int leafNumber=height;
```

小草类还待继续完善。小草是婀娜多姿的，那么其实是很多二阶贝塞尔曲线拼起来的。一个二阶曲线这里定义为一个单位高度，也就是height，每个height具体高度radianHeight ，小草扭动的幅度，也就是二阶控制点距离中心轴的位置。还有就是小草的根部位置。

```
canvas.save();
/*移动到根部*/
canvas.translate(rootPostion.x,rootPostion.y);
Path path=new Path();
path.moveTo(0,0);
for (int i=1;i<height+1;i++){
    Point p1=new Point((int)radianOffset,(int)((i-1)*radianHeight+radianHeight/2));
    Point p2=new Point(0, (int) (radianHeight*i));
    path.quadTo(p1.x,p1.y,p2.x,p2.y);

    Point p0=new Point(0, (int) ((i-1)*radianHeight));
    Point buttom=calculateBerzierTwo(p0,p1,p2,0.5f);
    Point top=new Point();
    top.x= (int) (buttom.x+radianOffset);
    top.y= (int) (buttom.y+radianHeight/3);
    radianOffset=-radianOffset;
   // canvas.drawLine(0,0,0,radianHeight*height,getTextPaint());
   // canvas.drawCircle(buttom.x,buttom.y,20,getTextPaint());
    new Leaf(top,buttom).drawSelf(canvas);
}
canvas.drawPath(path,getTextPaint());
canvas.restore();
```

把画布移动到根部，然后开始画二阶曲线。二阶曲线的起点终点控制点都在y轴网上附近，都很容易找到。但是这里有一个叶子生成的位置需要注意一下。目前叶子生成位置都是在扭动的拐弯处。

所以拐弯处的点，定义为叶子的根部，根部的位置确认是通过二阶贝塞尔曲线的函数去确认的。

```
  public float getBerzierTwo(float p0,float p1,float p2,float t){
        return (1-t)*(1-t)*p0+2*t*(1-t)*p1+t*t*p2;
    }
    public Point calculateBerzierTwo(Point p0,Point p1,Point p2,float t){
        Point bPoint=new Point();
        bPoint.x= (int) getBerzierTwo(p0.x,p1.x,p2.x,t);
        bPoint.y= (int) getBerzierTwo(p0.y,p1.y,p2.y,t);
        return bPoint;

    }
```

这两个函数是二阶函数的原理函数，其实直接用这个函数去画二阶线也行，我之前用过，但是考虑到系统已经提供了方法，就用系统的，性能肯定更好。然后我们是用这个函数用来确定点位置的。

确定了根部位置，在确定叶子的顶部位置其实只是根绝根部进行了一个小偏移。没什么可说

然后再循环过程中就画了一下叶子，最后画小草。

### 一堆草

```
/*y轴翻转*/
canvas.scale(1,-1);
/*画布向下平移*/
canvas.translate(0,-mHeight);
Random random=new Random();
for (int i=0;i<20;i++){
    Point point=new Point();
    point.y=0;
    point.x=random.nextInt(10)*mWidth/10;

    new Grass(random.nextInt(10),150+random.nextFloat()*150f,point).drawSelf(canvas);
}
```

最外层最简单，随机生成不同高度的小草

[源码位置](https://github.com/yeqinfu/ClockView)