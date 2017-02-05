---
title: 杭电2035
date: 2017-01-04 17:01:41
tags: 杭电
---
```
package oj;

import java.util.Scanner;

public class Main_2035 {

	public static void main(String[] args) {
		Scanner scanner = new Scanner(System.in);
		while (true) {
			int a=scanner.nextInt();
			int b=scanner.nextInt();
			if (a==0&&b==0) {
				break;
			}
			int result=fun(a,b);
			System.out.println(result);
			
		}

	}

	private static int fun(int a, int b) {
		a=a%1000;
		b=b%1000;
		int result;
		if (b==0) {
			result=1;
		}else{
			result=fun(a*a, b/2);
			if (b%2==1) {
				result*=a;
			}
		}
		return result%1000;
	}

}

```
快速幂递归实现，注意a,b整形溢出，每次调用mod 1000
普通递归实现a^b的话时间复杂度为O(n)快速幂可以达到O(log(2)n)
求a^b可以把b分解成b＝2^(i)+2^(i-1)...






