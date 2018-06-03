---
title: java自动装箱
date: 2018-06-03 12:16:58
tags: code
---

闲来无事，搞一下自动装箱，想知道它是编译期完成的还是如何如何

### 装箱

上一段代码

```java

public class Main_Interger {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Integer integer=1;
		System.out.println(integer.intValue());

	}

}

```

这就是最简单的自动装箱，一个1被负值给一个对象。然后执行javac去编译这个文件，得到的字节码再去查看编译后的代码是怎样的

```java
import java.io.PrintStream;

public class Main_Interger
{
  public static void main(String[] paramArrayOfString)
  {
    Integer localInteger = Integer.valueOf(1);
    System.out.println(localInteger.intValue());
  }
}
```

所以很明显自动装箱是在编译期完成的，装箱的方法调用的是Integer的valueOf方法

所以另外一个例子

```java

public class Main_Interger {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Integer integer=1;
		Integer integer2=1;
		
		System.out.println(integer==integer2);
		
		Integer integer3=150;
		Integer integer4=150;
		
		System.out.println(integer3==integer4);
	}

}
//output
true
false
```

两次答案不一致，这个就是因为valueof方法搞的鬼，下面查看这个方法。

```java
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
//  static final int low = -128;
 static final int high;
  int h = 127;
   high = h;
```

源码看出，对integer进行了缓存，才导致这些问题。



### 拆箱

```java

public class Main_Interger {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Integer integer=1;
		int sum=integer;
		System.out.println(sum);
		
		
	}

}
//output
1
```

编译后

```java
import java.io.PrintStream;

public class Main_Interger
{
  public static void main(String[] paramArrayOfString)
  {
    Integer localInteger = Integer.valueOf(1);
    int i = localInteger.intValue();
    System.out.println(i);
  }
}
```

调用了intValue方法获得该值。就这么多





















