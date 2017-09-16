---
title: java线程基础
tags:
  - Java
  - Thread
  - Runable
categories: Java
date: 2016-08-15 15:13:40
keywords: 
description:
---
Java线程基础系列（一）
Thread与Runnable
<!--more-->
## 前言
Java线程一直是我的薄弱部分，平时很少用到它。就算是用到，也是基于别人封装好的基础上进行开发操作。因此有必要对Java线程进行一次梳理总结。
简略图谱看这里[点我跳转哦！](http://xuhuiqiang.cn/2016/08/14/threadAbstract/)。

## 定义
`并发`：在周期时间段内，程序同时执行；
`并行`：同一时刻内，程序同时执行。
进程的引入是为了程序能够并发执行，并且可以对并发的程序加以描述和控制。进程是程序的一次执行，各个进程之间按各自独立的、不可预知的速向前推进（以异步的方式运行）。对进程控制仅仅是激活，挂起，调度等操作，而我们在进行这些操作时并不知道进程已经执行到何种程度。在进程没有有引入线程的概念前，进程是系统进行资源分配和调度分派的基本单位。

线程：线程的引入是为了减少程序在并发执行时所付出的时空开销，从而使OS具有更好的并发性。线程具有进程的许多特征，因此也被称为轻型进程。引入了线程的进程不再是一个可执行的实体，也不再是调度分派的基本单位。线程称为OS独立调度和分派的基本单位，进程下的线程共享进程资源。而进程成为OS分派资源的基本单位。

## Java线程
Java线程的创建有多种：
+ 实现Runnable接口
+ 继承Thread类
+ 通过线程池

### Runnable接口
通过Runnable接口实现线程极其简单，仅仅需要实现`run()`方法就可了。`run()`方法是线程的执行体，我们的线程执行逻辑便是在该方法实现。当`run()`执行完毕，线程结束。

### Thread类
通过实现`Runnable`接口创建的线程极其简单粗暴，所有的线程的运行参数都是系统默认的。因此Java提供了一个可以设置线程运行参数的`Thread`类。`Thread`实现了`Runnable`接口，并向外暴露了极多的方法。常用的`Thread`方法：
+ `sleep()`：使当前正在执行的线程休眠让出cpu，其它线程获取CPU，但并不释放其所持有的资源锁。
+ `join()`：当线程调用该方法时，宿主线程需要等待该线程执行结束才能继续，即宿主线程会被阻塞。调用该方法可使异步线程变为同步线程。
+ `interrupt()`：中断调用者线程，被中断的线程会抛出InterruptedException。当抛出异常时，线程并不会状态并不会立刻改变，并且可以通过`isInterrupted()`方法检测线程状态。
+ `yield()`：暂停当前正在执行的线程对象，执行其他线程。等同于sleep()；线程并不阻塞，而是由运行状态转为就绪状态。

`sleep()`与`Object.wait()`方法的区别：
`wait()`：此方法让当前持有该对象的线程进入阻塞状态，并将该线程放入对象的等待池中，然后该线程放弃此资源对象上的所有同步请求，即释放资源锁。当该线程重新获取到该资源对象时，该对象和当前线程的同步状态与调用`wait()`方法时的情况完全相同。注意：在调用`wait()`方法时，需要对该资源加锁，或者使用`synchronized`关键字括起来。

与`wait()`配套使用的方法还有：`notify()`，`notifyAll()`。`notify()`用于唤醒在对象监视器上等待的单个线程。如果有多个线程，则任意唤醒其中的一个线程。而`notifyAll()`一次唤醒此对象上等待的所有线程，所有线程以常规方式在该对象上主动同步的其他所有线程进行竞争。

#### Thread与Runnable的区别
直接实现`Runnable`接口的仅仅有一个`run()`方法，并且在获取线程对象后，一旦调用`run()`方法，线程立即启动执行。由于`Thread`类实现`Runnable`接口，可以直接查看`Thread`的`run()`。

``` java
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```
`target`为Runnable型成员变量，在初始化方法中`init()`完成其初始化。因此我们在自定义的线程类中，如：

``` java
public class ThreadOne extends Thread {
	@Override
	public void run() {
	//逻辑代码
	}
}
```
在`ThreadOne`中,我们复写了`run()`方法，由于Java多态的性质，我们是直接调用了`ThreadOne`方法，而不会调用父类方法。因此可以利用该特点，写出共享资源的线程类来。

```java
public class ThreadTemplate implements Runnable{
	private int count;
	@Override
	public void run() {
		try {
			Thread.sleep(500);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.printf("%d ", count++);
	}
}
public class TestThread {
	Runnable run = new ThreadTemplate();
		Thread one = new Thread(run);
		Thread two = new Thread(run);
		
		one.start();
		two.start();
}
```
执行以上代码：输出结果不定，有可能为`0 1`，也有可能为`1 0`，甚至有可能为`0 0`;
修改`TestThread`的主方法为：

```java
Thread one = new Thread(new ThreadTemplate());
Thread two = new Thread(new ThreadTemplate());
		
one.start();
two.start();
```
则其输出结果永远为：`0 0`；

_TIPS:_

_我们在实现一个线程的时候，应该考虑使用实现Runnable接口的方法，并使用`new Thread(Runnable)`的调用方式。_

ps：线程池做的笔记比较乱，估计花的时间比较久。。。

--------------------------------------------------------------------------
###### 博文原创，欢迎装载！
###### 转载请附上原文链接：http://xuhuiqiang.cn/2016/08/15/threadBegin/
--------------------------------------------------------------------------
