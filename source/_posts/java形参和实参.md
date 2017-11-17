---
title: JAVA形参和实参
date: 2017-11-17 14:00:00
tags: code
---

[参考原文](https://dailycast.github.io/Java-%E5%BD%A2%E5%8F%82%E4%B8%8E%E5%AE%9E%E5%8F%82/)

```java
public class MainDe {
	
	public static void main(String[] args) {
		//int swap
		int a=1,b=2;
		swapInt(a,b);
		System.out.println("a="+a+" b="+b);
		//integer swap
		Integer c=1,d=2;
		swapInteger(c,d);
		System.out.println("c="+c+" d="+d);
		//object swap
		A e=new A(1);
		A f=new A(2);
		swapObj(e,f);
		System.out.println("e="+e.age+" f="+f.age);
		
		
	}

	private static void swapObj(A e, A f) {
		int temp=e.age;
		e.age=f.age;
		f.age=temp;
		
	}

	private static void swapInteger(Integer c, Integer d) {
		Integer temp=c;
		c=d;
		d=temp;
	}

	private static void swapInt(int a, int b) {
		int temp=a;
		a=b;
		b=temp;
	}
	
	static class A{
		A(int age){
			this.age=age;
		}
		public int age;
		
	}

}

```

运行后答案

```
a=1 b=2
c=1 d=2
e=2 f=1
```



基本数据类型在传递的时候都是传值，而对象都是传址。



##### swapInt a b

这个方法只是把main方法的两个ab的值传递给swapInt的两个形参，而main方法的ab始终指向12；swapInt是不可能改变main里面的ab的值的。



##### swapInteger cd

Integer是对象，传递给swapInteger的是cd指向的两个地址，然而swapInteger 方法中的形参地址指向的改变，依然不能影响main方法中的cd引用的地址。

##### swapObj  ef

还是传地址，main方法中的ef指向的空间地址，swapObj  更改的事这个空间地址内部的某一个变量的值，这时候main方法指向的空间还是不变，但是里面的变量值却被改变了。所以是会影响到main的



因为Integer是对象，所以按道理是可以让swapInteger 方法影响到main的。所以原博文提供了一个方法

```java
private static void swap(Integer numa, Integer numb) {
       int tmp = numa.intValue();//tmp 定义为基本数据类型
       try {
           Field field = Integer.class.getDeclaredField("value");
           field.setAccessible(true);
           field.set(numa, numb);//这个时候并不改变 tmp 的值
           field.set(numb, tmp);
       } catch (Exception e) {
           e.printStackTrace();
       }
   }
```



这个对象空间有一个value的字段里面直接存的是int的值。把它改了就可以了。

然后原文还介绍了Integer装箱的缓存，缓存的字符在-128到127之间。就是说上面呢这个方法要百分百灵敏，要么不使用自动装箱，要么自动装箱的值要超过缓存范围。









