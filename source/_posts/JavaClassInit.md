---
title: Java类的初始化顺序
tags:
  - Java
categories: Java
date: 2016-08-15 10:21:25
keywords: Java Class ClassLoader
description: Java类初始化顺序
---
一篇由师弟引发的关于Java类初始化顺序的博文~~~~
<!--more-->
## 前言
本篇博文讲的是Java类的实例化顺序。
## 起源
昨天晚上在整理个人笔记时，师弟直接发了一张StackOverFlow异常的图片，问如何解决该问题。。。瞬间就哔了某种动物了。。。没有具体信息你让我怎么去定位问题。从之后异常的详细信息上看，是两个类不断的实例化导致的栈溢出。问题可抽象为以下代码：

``` java
public class TestSecond{
	TestFirst a = new TestFirst();
}
```
``` java
public class TestFirst{
	TestSecond b = new TestSecond();

	public static void main(String[] args){
		new TestFirst();
	}
}
```
通过`javac TestB.java`，`javac TestA.java`编译后，通过`java TestA`运行TestA，程序运行抛出异常，是不断的循环
`at TestA.<init>(TestA.java:3)`，`at TestB.<init>(TestB.java:2)`两个错误。从`init`可以知道是在实例化`TestA`和`TestB`的过程中出错。其出错的原因在于不明确类的实例化过程。

## 类的实例化过程
``` java

public class TestA{
	static int num = 0;
	static String staName = "one";
	String dynName = "two";
	String dynNameSec ;
	static String strNameSec;

	//第一个静态代码块
	static{
		System.out.printf("num: %d, staName: %s, strNameSec: %s ,strAfter: %s\n", num, staName, strNameSec,strAfter);
		num++;
	}
	//第二个静态代码块
	static{
		strNameSec = "static after"
		System.out.printf("num: %d, staName: %s, strNameSec: %s ,strAfter: %s\n", num, staName, strNameSec,strAfter);
		num++;
	}

	// 第一个代码块
	{
		dynNameSec = "second";
		System.out.printf("num: %d, staName: %s, dynName: %s\n", num++, staName, dynName);
	}
	//// 第二个代码块
	{
		System.out.printf("num: %d, dynNameSec: %s, dynName: %s\n", num++, dynNameSec, dynName);
	}
	static  String strAfter = "after";
	public static void main(String[] args){
		//new TestA();
	}
}
```
先把`new TestA()`注释，重新编译执行后可以看到输出结果（截图麻烦，直接写结果）：
`num: 0, staName: one, strNameSec: null ,strAfter: null`
`num: 1, staName: one, strNameSec: static after ,strAfter: null`
调换两个静态代码块的位置，输出结果：
`num: 0, staName: one, strNameSec: static after ,strAfter: null`
`num: 1, staName: one, strNameSec: static after ,strAfter: null`
由此可以看到静态的代码块的执行顺序与其位置有关。并且静态代码块会在加载类的Class代码的时候便开始声明和初始化静态变量并执行静态代码块。

换回代码块的位置，取消注释，重新编译执行后结果为：
`num: 0, staName: one, strNameSec: null ,strAfter: null`
`num: 1, staName: one, strNameSec: static after ,strAfter: null`
`num: 2, staName: one, dynName: two`
`num: 3, dynNameSec: second, dynName: two`
将两个静态代码块的位置放在非静态代码块的后边，重新编译执行结果为：
`num: 0, staName: one, strNameSec: null ,strAfter: null`
`num: 1, staName: one, strNameSec: static after ,strAfter: null`
`num: 2, staName: one, dynName: two`
`num: 3, dynNameSec: second, dynName: two`
可以发现调换位置后的结果与调换前的结果相同。因此静态代码块的执行比非静态代码要先。

互换两个非静态代码块的位置，重新编译执行，结果为：
`num: 0, staName: one, strNameSec: null ,strAfter: null`
`num: 1, staName: one, strNameSec: static after ,strAfter: null`
`num: 2, dynNameSec: null, dynName: two`
`num: 3, staName: one, dynName: two`
从以上代码可以看出，类在实例化的顺序是：
+ *_先声明静态成员变量，并随后执行静态代码块。_*
+ *_声明非静态成员变量，并执行非静态代码块。_*
+ *_最后执行构造方法。_*

## 问题
回到最开始的问题：`TestFirst`在实例化时，需要声明并初始化`TestSecond`，因此，`new TestFirst()`时，会首先去执行
`TestSecond b = new TestSecond();`，而在初始化`TestSecond`的过程时，又要先获取`TestFirst()`对象。因此造成一个循环调用的结果，最后导致栈空间溢出。
简单的解决方法：延迟两个变量的初始化，仅仅对两个变量进行声明。改为：

``` java 
public class TestSecond{
	TestFirst a;

	public void setA(TestFirst a){
		this.a = a;
	}
}
```
``` java
public class TestFirst{
	TestSecond b;

	public void setB(TestSecond b){
		this.b = b;
	}

	public static void main(String[] args){
		TestSecond b = new TestSecond();
		TestFirst a = new TestFirst();
		a.setB(b);
		b.setA(a);
	}
}
```
## 遗留问题：

``` java
public class TestC{
	static TestB B = new TestB();
	public static void main(String[] args){
		new TestC();
		System.out.println("one");
	}
}

public class TestB{
	TestC c = new TestC();
}
```
最后的输出是：one。

对于类的初始化顺序还是有点疑惑！！！！

--------------------------------------------------------------------------
###### 博文原创，欢迎装载！
###### 转载请附上原文链接：http://xuhuiqiang.cn/2016/08/15/JavaClassInit/
--------------------------------------------------------------------------
