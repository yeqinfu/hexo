---
title: 自定义view-网红时钟
date: 2019-05-27 14:00:58
tags: android
---

写了一个网红时钟，代码不完善，进位有bug，但是不想改。

主要记录这个view的实现思路。效果图

![效果图](http://wx2.sinaimg.cn/large/c1b251b3gy1g3bc7r0fabj209r0aut9k.jpg)

都是文字，所以第一步想的是定义文字工具类

```java
static {
        for (int i = 0; i < hours.length; i++) {
            hours[i] = numberArray[i] + "点";
        }

        for (int i = 0; i < months.length; i++) {
            months[i] = numberArray[i] + "月";
        }
        minutes[0] = "零分";
        for (int i = 1; i < minutes.length; i++) {
            minutes[i] = numberArray[i] + "分";
        }
        seconds[0] = "零秒";
        for (int i = 1; i < seconds.length; i++) {
            seconds[i] = numberArray[i] + "秒";
        }


    }

    public static String[] getSeconds() {
        return seconds;
    }

    public static String[] getMinutes() {
        return minutes;
    }

    public static String[] getHours() {
        return hours;
    }

    /**
     * 获取月份数组
     *
     * @return
     */
    public static String[] getDays() {
        return getDays(null);
    }

    public static String[] getDays(Calendar calendar) {
        if (calendar == null) {
            calendar = Calendar.getInstance();
        }
        DateUtils du = new DateUtils();

        int dayOfMonth =  du.getDays(du.getYear(), du.getMonth());
        String[] result = new String[dayOfMonth];
        for (int i = 0; i < result.length; i++) {
            result[i] = numberArray[i] + "日";
        }

        return result;
    }

    /**
     * 月份数组
     *
     * @return
     */
    public static String[] getMonths() {
        return months;
    }
```

这样，文字内容生成就抽出来，就不会感觉乱糟糟啦

整体实际上是把它分为6个圆形，半径根据如下文字作为长度，每个半径之间定义一个小间隙变量radiusOffset

```
 private String year = "2019年";
    private String month = "十二月";
    private String day = "三十一号";
    private String dayhelf = "下午";
    private String hour = "十二点";
    private String minute = "五十九分";
    private String second = "五十九秒";
    private float radiusOffset = 10;
```

整体画笔只有一根，所以初始化的时候可以确定半径长度

```java

    private void init() {
        yearRadius = getTextLength(year) / 2 + radiusOffset;
        monthRadius = yearRadius + radiusOffset + getTextLength(month);
        dayRadius = monthRadius + radiusOffset + getTextLength(day);
        dayHalfRadius = dayRadius + radiusOffset + getTextLength(dayhelf);
        hourRadius = dayHalfRadius + radiusOffset + getTextLength(hour);
        minuteRadius = hourRadius + radiusOffset + getTextLength(minute);
        secondRadius = minuteRadius + radiusOffset + getTextLength(second);
        commonTextHeight = -getTextPaint().ascent() + getTextPaint().descent();
       // startAnimator();
        setDateAndStart(null);
    }
```

这种对称图，第一步先平移画布到中心位置

```
  // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);
```

把年画进去，目前写死2019，因为不相信有明年

```java
   /**
     * 一个帮助你把文字写在指定点并居住在指定点位置的方法
     * 这里的偏移直角坐标系采用默认指教坐标系
     */
    private void drawTextOnMiddle(Canvas canvas, String targetText, float x, float y, Paint paint) {
        /*文字测量*/
        Rect bounds = new Rect();
        paint.getTextBounds(targetText, 0, targetText.length(), bounds);
        float width = bounds.right - bounds.left;
        float height = bounds.bottom - bounds.top;
        canvas.drawText(targetText, x - width / 2, y + height / 2, paint);
    }
```

抽出一个方法，这个方法只要你告诉它中心点，它就会把文字画出来，文字的中心点会是你指定的。

##### 画月份

```
  double initAngle = 0;
        canvas.save();
        canvas.rotate(monthRepeat * (360.0f / 12) + (float) monInitAngle);
        for (int i = 0; i < 12; i++) {
            Polar polar = new Polar(monthRadius, initAngle);
            drawTextToPointLeft(canvas, polar.toPoint(), CommonNumber.getMonths()[i], getTextPaint());
            canvas.rotate(-360 / 12);
        }
        canvas.restore();
```

这个rotate是因为动的时候需要偏移，如果我们不需要动，那就没他什么事了。每次偏移都是通过角度来计算的。

所以这里定义一个极坐标类，用来辅助偏移便于理解

```
 /**
     * 极坐标类
     */
    static class Polar {
        double radius;
        double angle;

        public Polar(double radius, double angle) {
            this.radius = radius;
            this.angle = angle;
        }

        public double getRadius() {

            return radius;
        }

        public void setRadius(double radius) {
            this.radius = radius;
        }

        public double getAngle() {
            return angle;
        }

        public void setAngle(double angle) {
            this.angle = angle;
        }

        public Point toPoint() {
            Point point = new Point();
            point.x = (int) (Math.cos(angle) * radius);
            point.y = (int) (Math.sin(angle) * radius);
            return point;
        }
    }
```

然后不断偏移画文字，文字是重力靠外围的，所以定义了一个方法

```

    private void drawTextToPointLeft(Canvas canvas, Point toPoint, String targetText, Paint paint) {
        Rect bounds = new Rect();
        paint.getTextBounds(targetText, 0, targetText.length(), bounds);
        float width = bounds.right - bounds.left;
        toPoint.x = (int) (toPoint.x - width / 2);
        drawTextOnMiddle(canvas, targetText, toPoint.x, toPoint.y, paint);

    }
```

它可以找到靠外的文字中心位置，然后再调用drawTextOnMiddle完成绘图。

其他时分秒都是类似的省略。

### 动画部分

动画生成一个值

```
  final ValueAnimator valueAnimator= ValueAnimator.ofFloat(0, 1);
```

比如秒，每次动画过程偏移6度。

```
  secondInitAngle = ((float) animation.getAnimatedValue() * 360.0 / 60);
```

所以这里动画执行过程一直改这个偏移度数。而当到59秒的时候会导致分发生动画

```java
  if (secondRepeat == 59) {
                    minuteInitAngle = secondInitAngle;
                    if (minuteRepeat == 59) {
                        hourInitAngle = ((float) animation.getAnimatedValue() * 360.0 / 12);
                        if (hourRepeat == 11) {
                            dayHalfInitAngle = ((float) animation.getAnimatedValue() * 360.0 / 2);
                            int dayLength=CommonNumber.getDays(null).length;
                            if (dayHalfRepeat == 1) {
                                dayInitAngle = ((float) animation.getAnimatedValue() * 360.0 /dayLength);
                                if (dayRepeat==1){
                                    monInitAngle=((float) animation.getAnimatedValue() * 360.0 / 12);
                                }
                            }

                        }

                    }

                }
```

所以分也开始计算偏移角度。而当秒59，分也59这两个条件同时成立时，时针也要动，这样一直驱动向高位动画。

大概就这么多，动画进位还有问题。真的用到这个view的时候再解决。





