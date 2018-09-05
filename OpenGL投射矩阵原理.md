---
title: OpenGL投射矩阵原理
date: 2016-08-15 09:00:58
tags: code
---



投射矩阵是把三维图像投射到2D屏幕，在三维图像世界的坐标系采用的是左手坐标系（观察坐标），而二维相机采用的是右手坐标系（裁剪坐标），投射矩阵的作用就是相乘之后可以把观察坐标转化为裁剪坐标用于2D显示



![](https://wx4.sinaimg.cn/large/c1b251b3gy1fua65don5rj20ga070jrq.jpg)



[图片参考](https://blog.csdn.net/qq_18229381/article/details/77941325)

![](http://wx3.sinaimg.cn/large/c1b251b3gy1fu98lsnntkj21500mf477.jpg)

如图可以看到n，f在3d投影中代表的值。图中的坐标原点代表观察点（相机），角度代表视野，视野不可能超过180度。









