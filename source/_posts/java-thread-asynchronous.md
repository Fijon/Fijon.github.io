---
title: 具有返回值的线程：Callable与Future
tags:
  - null
categories: Java
date: 2016-09-21 09:45:13
keywords:
description:
---
一个模拟异步下载的Java例子。
<!--more-->
# 前言
我们在Java中线程通常使用的都是无返回值的线程，如果我们需要使用到具有返回值的线程时，就需要使用两个接口`Future`和`Callable`，其中，Java为我们提供了一个对`Future`的实现类`FutureTask`，通过该类我们可以实时获取线程的具体情况。而`Callable`接口是我们需要实现的线程任务逻辑。即`Future`是存储`Callable`线程的运行情况。  
我们可以通过一个实时获取下载情况的例子来说明这个类。  
# Example
这里，假设一个任务完成时间为2s，我们每个100ms就要更新当前的下载情况。由于`Future`接口仅仅提供`cancel()`,`isCancelled()`,`isDone()`,`get()`,`get(long, TimeUnit)`等接口方法，而`FutureTask`类暴露给我们使用的也是这几个方法，而我们需要的方法他们没有提供。我们可以通过适配器模式新增一个接口方法。

``` java
interface TaskStatus{
  //该方法返回当前任务的进度
   float status();
}
```
同时，新建一个类`TaskStatusImpl`，该类继承`TaskFuture`实现了接口`TaskStatus`。  

``` java
    class TaskStatus extends FutureTask<v> implements TaskStatus{
        long begin; //记录线程开始时间
		public TaskStatusImpl(Callable<v> callable) {
			super(callable);
			this.begin = System.currentTimeMillis();
		}

		@Override
		public float status() {
		//线程完成时，返回1
			if (super.isDone()) {
				return 1f;
			} else {   //未完成，这计算当前的线程完成进度
				long now = System.currentTimeMillis();
				long temp = now - begin;
				// 除以2000是因为线程设置了2000的睡眠时间。
				return temp / 2000f;
			}
		}
    }
```
测试类`Main`以上的接口和类都在该测试类的内部  

```  java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
public class Main {
	public Main() {
		System.out.println("---main begin---");
	}
	public static void main(String[] args) {
		Main main = new Main();
		Task task = main.new Task();
		TaskStatusImpl<Integer> future = main.new TaskStatusImpl<Integer>(task);
		new Thread(future).start();
		try {
			while (future.status() < 1) {
				Thread.sleep(200);
				System.out.printf("now status: %f\n", future.status());
			}
			System.out.println("Thread result: " + future.get());
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ExecutionException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	interface TaskStatus {
		float status();
	}
	class TaskStatusImpl<v> extends FutureTask<v> implements TaskStatus {
		long begin;
		public TaskStatusImpl(Callable<v> callable) {
			super(callable);
			this.begin = System.currentTimeMillis();
		}
		@Override
		public float status() {
			if (super.isDone()) {
				return 100f;
			} else {
				long now = System.currentTimeMillis();
				long temp = now - begin;
				return temp / 2000f;
			}
		}
	}
	class Task implements Callable<Integer> {
		public Integer call() throws Exception {
			System.out.println("---Thread begin---");
			Thread.sleep(2000);
			System.out.println("---Thread end---");
			return (int) (Math.random() * 10);
		}
	}
}
```
运行结果为：

``` java
---main begin---
---Thread begin---
now status: 0.100000
now status: 0.217500
now status: 0.317500
now status: 0.418500
now status: 0.519000
now status: 0.619000
now status: 0.719000
now status: 0.819000
now status: 0.919000
---Thread end---
now status: 100.000000
Thread result: 6
```

--------------------------------------------------------------------------
###### 博文原创，欢迎装载！
###### 转载请附上原文链接：http://xuhuiqiang.cn/2016/09/21/java-thread-asynchronous/
--------------------------------------------------------------------------
