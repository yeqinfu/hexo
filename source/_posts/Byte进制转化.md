---
title: Byte进制转化
date: 2018-07-23 09:00:58
tags: code
---



### 进制转化 Integer.toBinaryString(i);

因为看到Integer有这个转化二进制的方法，而Byte之类的却没有，所以看看它是怎么实现进制转化的。并且完成我需要的Byte进制转化。

```java
public static String toBinaryString(int i) {
        return toUnsignedString0(i, 1);
    }
private static String toUnsignedString0(int val, int shift) {
        // assert shift > 0 && shift <=5 : "Illegal shift value";
        int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
        int chars = Math.max(((mag + (shift - 1)) / shift), 1);
        char[] buf = new char[chars];

        formatUnsignedInt(val, shift, buf, 0, chars);

        // Use special constructor which takes over "buf".
        return new String(buf, true);
    }
public static int numberOfLeadingZeros(int i) {
        // HD, Figure 5-6
        if (i == 0)
            return 32;
        int n = 1;
        if (i >>> 16 == 0) { n += 16; i <<= 16; }
        if (i >>> 24 == 0) { n +=  8; i <<=  8; }
        if (i >>> 28 == 0) { n +=  4; i <<=  4; }
        if (i >>> 30 == 0) { n +=  2; i <<=  2; }
        n -= i >>> 31;
        return n;
    }
static int formatUnsignedInt(int val, int shift, char[] buf, int offset, int len) {
        int charPos = len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            buf[offset + --charPos] = Integer.digits[val & mask];
            val >>>= shift;
        } while (val != 0 && charPos > 0);

        return charPos;
    }
```

主要是以上四个方法做了转化。

numberOfLeadingZeros 这个方法主要是可以得出一个int类型前面有多少个零。就是长度为32bit的int有多少个0

这个方法有两个小知识点，一个是右移运算法，无符号右移运算符。还有n -= i >>> 31;的执行顺序。

#### 为什么先无符号右移，再左移回来？

```java
int j = -4 >>>1;
System.out.println(j);
//output
2147483646
```

我们知道负数是补码形式在计算机中的

> -4真值：1000 0000 0000 0000 0000 0000 0000 0100
>
> -4反码：1111 1111 1111 1111 1111 1111 1111  1011
>
> -4补码：1111 1111 1111 1111 1111 1111 1111  1100
>
> 无符号对补码进行右移一位空位补0。最后一个0没了，前面多了一个0，此时变成了正数，就是比正数最大值少一
>
> 0111 1111 1111 1111 1111 1111 1111  1110



numberOfLeadingZeros 这个方法的思想是通过二分法来判断有几个前置零。

![image](http://ws4.sinaimg.cn/large/c1b251b3gy1ftjv3mr3sij210g0en0us.jpg)

这就类似当位数为32位的时候，通过四次二分法就可以确定它有几个零了。

而这四个if就很巧妙得把四次二分法都完全执行了。

#### toUnsignedString0 转化成无符号字符串

知道前面有多少个零之后，就可以按需声明一个char数组来存储非零部分的数字。后续比较简单。

值得注意的是toUnsignedString0参数表中的shift。当传入1就代表转化的是二进制，因为后续是用shift来右移。如果传入4代表就是16进制。传入3代表八进制。

#### 写一个Byte的二进制转化

我的需求是写个Byte的进制转化，并希望签名补0就是完整的8bit长度。根据以上思想完善如下：

```
	byte n=1;
	n = i >>> 7;
```

发现这段会报错，位移运算符接收的必须是整形

```java
public static byte numberofLeadingZerosByte(byte i){
		if (i==0) {
			return 8;
		}
		byte n=1;
		if (i>>>4==0) {
			n+=4;
			i<<=4;
		}
		if (i>>>6==0) {
			n+=2;
			i<<=2;
		}
		n -= i >>> 7;
		return n;
		
	}
public static void main(String[] args) {
		System.out.println(numberofLeadingZerosByte((byte) 1));
		System.out.println(numberofLeadingZerosByte((byte) 2));
		System.out.println(numberofLeadingZerosByte((byte) 4));
		System.out.println(numberofLeadingZerosByte((byte) 8));
		System.out.println(numberofLeadingZerosByte((byte) 16));
		System.out.println(numberofLeadingZerosByte((byte) 64));
		System.out.println(numberofLeadingZerosByte((byte) -1));
		System.out.println(numberofLeadingZerosByte((byte) -2));
		System.out.println(numberofLeadingZerosByte((byte) -4));
		System.out.println(numberofLeadingZerosByte((byte) -8));
		System.out.println(numberofLeadingZerosByte((byte) -16));
		System.out.println(numberofLeadingZerosByte((byte) -64));
	}
//output
7
8
5
6
3
1
2
2
2
2
2
2
```

但是以上代码是不报错的，估计是被向上转型了	n -= i >>> 7;这段代码被向上转型了。

```java
public static void main(String[] args) {
		byte i0=-1;
		short i1 = -1;
		int i2 = -1;
		long i3 = -1;

		System.out.println(i0 >>> 7);
		System.out.println(i1 >>> 7);
		System.out.println(i2 >>> 7);
		System.out.println(i3 >>> 7);

	}
//output
33554431
33554431
33554431
144115188075855871
```

说明byte，short会被转型为int。导致不准确的问题。

也就是说当-1传入numberofLeadingZerosByte这个方法执行	n -= i >>> 7;的时候。i先向上转型变成

1111 1111 1111 1111 1111 1111 1111 1111

然后无符号右移7位

0000 0001 1111 1111 1111 1111 1111 1111

上面这个数33554431然后1-33554431得到-33554430它的二进制

11111110000000000000000000000010

然后再被强制转型赋值给n

取低位八位

0000 0010

也就是2

=============为什么我这么猜是对的呢？====================================

因为传入-1，-2，-8都是返回2.我们再试一下-8按照上诉流程得出的结果，先向上转型变成

1111 1111 1111 1111 1111 1111 1111 1000

然后无符号右移7位

0000 0001 1111 1111 1111 1111 1111 1111

发现跟上面是一样的。就不用再试剩下的步骤了。

### 现在如何让他（numberofLeadingZerosByte）返回正确的数？

改成这样

```java
public static byte numberofLeadingZerosByte(byte i){
		if (i==0) {
			return 8;
		}
		byte n=1;
		if (i>>>4==0) {
			n+=4;
			i<<=4;
		}
		if (i>>>6==0) {
			n+=2;
			i<<=2;
		}
		if (i>>>7==0) {
			return n;
		}else{
			return --n;
		}
		
	}
public static void main(String[] args) {
		System.out.println(numberofLeadingZerosByte((byte) 1));
		System.out.println(numberofLeadingZerosByte((byte) 2));
		System.out.println(numberofLeadingZerosByte((byte) 4));
		System.out.println(numberofLeadingZerosByte((byte) 8));
		System.out.println(numberofLeadingZerosByte((byte) 16));
		System.out.println(numberofLeadingZerosByte((byte) 64));
		System.out.println(numberofLeadingZerosByte((byte) -1));
		System.out.println(numberofLeadingZerosByte((byte) -2));
		System.out.println(numberofLeadingZerosByte((byte) -4));
		System.out.println(numberofLeadingZerosByte((byte) -8));
		System.out.println(numberofLeadingZerosByte((byte) -16));
		System.out.println(numberofLeadingZerosByte((byte) -64));
	}
//output
7
6
5
4
3
1
0
0
0
0
0
0
```

### Byte转String

```java
public static String byteToBinaryStr(byte b){
		int zeroSize=numberofLeadingZerosByte(b);
		char[] buf = new char[8];
		for(int i=0;i<zeroSize;i++){
			buf[i]='0';
		}
		System.out.println(b
				);
		int mag=Byte.SIZE-zeroSize;
		int charPos=7;
		
		while(mag> 0) {
			System.out.println(b&1);
			mag--;
			buf[charPos--] = (b&1)==1?'1':'0';
		
			b=(byte) (b>>>1);
		} 
		return new String(buf);
	}
```

完结。



​	


















