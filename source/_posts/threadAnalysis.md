---
title: Thread源码简析
tags:
  - Java
  - Thread
categories: Java
date: 2016-08-28 15:41:52
keywords: Java Thread
description: Thread源码简单分析
---
Java线程系列（二）---Thread源码简析
<!--more-->
## 前言
通常，Java Web开发的时候，很少用到线程的知识。在学校各种Java web开发项目中，大部分用的是三大框架。但是，在笔试或者面试题中，常常可以看到各种相关线程的知识。因此我们需要对线程重视起来。

## 成员变量与代码块
### 静态代码块
但我们打开Thread源码时，首先看到的是一个成员变量的声明与一个静态代码块。静态的代码是在类被加载时运行的，在Thread开头的代码为：

``` java
private static native void registerNatives();
    static {
        registerNatives();
    }
```
以上代码用来确认`registerNatives()`方法是在<clint>方法中第一个做的事情。`registerNatives()`是本地方法，用于将本地函数向Java虚拟机进行登记，可以使虚拟机更有效率的找到该类登记的本地函数。这是因为Thread类中有很多本地方法。

__< cilnt >方法：所有类变量的初始化语句和类型的静态初始化语句都会被Java编译器收集到一起放入到一个特殊的方法中，该方法就是<clint>，该方法用于初始化静态的类变量，在类初始化时被调用。__
### 成员变量

``` java 
private char        name[]; //用于存储线程的名称的字符数组。
private int         priority;  //线程的优先级
private Thread      threadQ;
private long        eetop;
    /* Whether or not to single_step this thread. */ 
    private boolean     single_step;
    /* Whether or not the thread is a daemon thread.  当前线程是否为守护线程（后台线程）*/
    private boolean     daemon = false;
    /* JVM state */
    private boolean     stillborn = false;
    /* What will be run. 当前正在运行的线程任务*/
    private Runnable target;
    /* The group of this thread 当前线程所在线程组*/
    private ThreadGroup group;
    /* The context ClassLoader for this thread 当前线程的类加载器*/
    private ClassLoader contextClassLoader;
    /* The inherited AccessControlContext of this thread 当前线程所继承的AccessControlContext  */
    private AccessControlContext inheritedAccessControlContext;
  /* For autonumbering anonymous threads. 对匿名函数进行自动编号*/
private static int threadInitNumber;
    private static synchronized int nextThreadNum() {
        return threadInitNumber++;
    }
/* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class.
     *用于关联当前线程的ThreadLocal值，有ThreadLocal类维护当前的映射。
     *ThreadLocal在可以实现在线程内实现一个局部变量，在线程的任何地方都可以访问，能够减少线程的传递。
     *该变量不能被子线程所继承。
    ThreadLocal.ThreadLocalMap threadLocals = null;
/*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class. 
     *用于关联当前线程的InheritableThreadLocal值，有InheritableThreadLocal类维护当前的映射.
     *InheritableThreadLocal使子线程可以从父线程获取值。
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
 /*
     * The requested stack size for this thread, or 0 if the creator did
     * not specify a stack size.  It is up to the VM to do whatever it
     * likes with this number; some VMs will ignore it. 
     *用于确定当前线程栈的大小，如果创建时没有指定栈大小值的话，会置为0。
     *他的值取决于虚拟机所给定的大小。一些虚拟机会忽略这个值。
     */
    private long stackSize;
    /*
     * JVM-private state that persists after native thread termination. 虚拟机的私有状态，该状态一直持续到本地线程终止。
     */
    private long nativeParkEventPointer;
    /*
     * Thread ID 当前线程的Id
     */
    private long tid;
    /* For generating thread ID自动生成的线程ID */
    private static long threadSeqNumber;
    /* Java thread status for tools,
     * initialized to indicate thread 'not yet started'
     *为工具提供的Java线程状态，该状态会被初始化为“未启动状态”
     */
    private volatile int threadStatus = 0;
    /**
     * The argument supplied to the current call to
     * java.util.concurrent.locks.LockSupport.park.
     * Set by (private) java.util.concurrent.locks.LockSupport.setBlocker
     * Accessed using java.util.concurrent.locks.LockSupport.getBlocker
     */
//用于记录当前线程阻塞资源
    volatile Object parkBlocker; 
    /* The object in which this thread is blocked in an interruptible I/O
     * operation, if any.  The blocker's interrupt method should be invoked
     * after setting this thread's interrupt status.
     */
    private volatile Interruptible blocker;
    private final Object blockerLock = new Object();
    /* Set the blocker field; invoked via sun.misc.SharedSecrets from java.nio code
     */
//初始化block成员变量，该方法会在可中断的I/O操作上被调用。
    void blockedOn(Interruptible b) {
        synchronized (blockerLock) {
            blocker = b;
        }
    }
/* 用于表示当前线程的优先级，数值越大，优先级越大。默认的优先级为NORM_PRIORITY */
    public final static int MIN_PRIORITY = 1; 
    public final static int NORM_PRIORITY = 5; 
    public final static int MAX_PRIORITY = 10;
```
_AccessControlContext用于封装线程运行的上下文，可以根据该上下文做出的系统资源访问决定。线程是基于它所封装的上下文而不是当前执行线程的上下文作出访问决定。如果为默认时，则其上下文继承自父亲线程。_
## 构造方法
Thread的构造方法有多个，但最终都是调用`init`方法。`init`方法设置相关参数的默认参数，比如所在线程组，是否为守护线程，以及优先级等。下边所提到的父线程即为创建该线程的线程。以下了解一下`init`的实现。
### init
``` java
   private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }
        this.name = name.toCharArray();
       //获取当前正在运行的线程对象，任务线程在哪个线程被new Thread();那么currentThread就返回哪个线程（创建者线程）。
        Thread parent = currentThread();
      /*安全管理器是一个允许应用程序实现安全策略的类。
       *它允许应用程序在执行一个可能不安全或敏感的操作前确定该操作是什么，
       *以及是否是在允许执行该操作的安全上下文中执行它。应用程序可以允许或不允许该操作。*/
        SecurityManager security = System.getSecurityManager();
      //如果线程组为空，用于判断是否是applet程序。
        if (g == null) {
            /* Determine if it's an applet or not */
            /* If there is a security manager, ask the security manager what to do. 
               *如果安全管理器存在，则从安全管理器获取线程组作为需要运行的线程的所在线程组。*/
            if (security != null) {
                g = security.getThreadGroup();
            }
            /* If the security doesn't have a strong opinion of the matter use the parent thread group. 
               *如果不存在安全管理器或者无法从安全管理器获取线程组，
               *则以当前正在执行的线程所在线程组作为将要执行的线程的线程组。*/
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }
        /* checkAccess regardless of whether or not threadgroup is explicitly passed in. *判断当前的线程组是否可以传递，当如果不能被传递时，会抛出异常，该方法没有返回值。 */
        g.checkAccess();
        /*
         * Do we have the required permissions?判断是否具有允许请求权限。
         */
        if (security != null) {
        /*该方法用于验证子类对象可以在不违反安全约束的情况下进行实例化：
          *子类不能复写非final的安全敏感方法。否则抛出异常*/
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }
        /*同步方法，用于为线程组中未执行的线程数加1，由于未开始执行的线程是不会加入运行线程组的，
         *所以如果他们一直没有开始执行的是可以被GC的。
         *但是他们必须记录起来，以保证包含未执行的守护线程组不会被GC。*/
        g.addUnstarted();
        this.group = g;
        //是否是守护线程和以及其线程的优先度，继承自父线程。
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        /*在设置priority时，会对当前的优先级进行判断，
         *如果当前的优先级优于父线程优先级的话，会讲当前优先级改为父线程优先级。*/
        setPriority(priority);
        if (parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares 存储栈空间大小*/
        this.stackSize = stackSize;
        /* Set thread ID 设置线程的id值*/
        tid = nextThreadID();
    }
```
`init`方法具有多个参数，分别是`ThreadGroup`是当前所在线程组，`Runnable`是需要线程运行的任务，`String`是线程的名称，`long`是线程的栈空间大小，`AccessControlContext`是用继承的线程的上下文，即线程运行的环境。
### 构造方法
所有的构造方法均是通过调用init方法来实现，区别仅在于构造方法中明确了多少个参数，有哪些参数是默认参数。
1. 无参构造方法：`public Thread() `。无参的构造方法是线程的一切参数均使用默认值，采用安全管理的所在的线程组或父线程所在线程组作为当前线程的线程组，并且无运行任务，相当于run()无运行逻辑代码。，名字是“Thread-线程的序号“，栈空间大小为默认值0。
2. 有运行任务和特定封装好的运行上下文的构造函数： `Thread(Runnable target, AccessControlContext acc) `。
3. 有明确线程组和运行任务的构造函数：public Thread(ThreadGroup group, Runnable target)。
4. 有明确的线程运行名：`public Thread(String name)`。
5. 有明确的线程组和线程名称： `public Thread(ThreadGroup group, String name)`。
6. 有明确的执行任务和线程名称：`public Thread(Runnable target, String name)`。
7. 有明确的线程组，执行任务和线程名称：`public Thread(ThreadGroup group, Runnable target, String name)`。
8. 具有明确的线程组，执行任务，线程名称以及栈空间大小： `public Thread(ThreadGroup group, Runnable target, String name,   long stackSize)`。
## 常用方法
在Thread中方法大多都是本地方法。在码代码时，常用的方法有start()，run(),exit(),interrupt(),等等。

###   start
`start`方法是有synchronized关键字修饰，保证了在同一时刻内，只有一个线程在执行当前任务。并且一个线程仅仅只能被start一次。

``` java
 public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
           //判断线程的状态，任何一个线程在start前的状态都是0，如果start前的状态不为0,那么意味着该线程是执行过，因此会有异常。这也意味着，一个线程仅能start一次。
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
//将线程加入线程组中，当没有start之前时，线程在线程组中，是属于没有执行的部分。start方法执行时，线程组，未执行的线程数减一。
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            t ry {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
```
### run
`run`方法：线程运行的任务。当run方法没有被复写时，或者实例化thread没有明确指明线程的执行任务时，run方法无实际意义。

``` java
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```
### sleep
`public static void sleep(long millis, int nanos)`：1毫秒 = 1000000纳秒。精确到纳秒级别的阻线程塞时间实际上仅仅精确到毫秒。如果nanos>= 500000或者millis为0，当nanos>0时，millis++。线程阻塞millis毫秒。

``` java
    public static void sleep(long millis, int nanos)
    throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }
        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }
        sleep(millis);
    }
```
`sleep`具有另外一个重载方法`sleep(long)`，精确到毫秒。
### interrupt
`interrupt()`：该线程的主动行为。如果当前线程并非是可中断线程，则仅仅将线程的标志位设为中断，而不是立即中断该线程。分为很多种情况：

``` java 
    public void interrupt() {
//如果并发当前线程主动中断自己，则该线程会判断当前的运行的线程是否有权限修改该线程。可能会抛出SecurityException相关线程
        if (this != Thread.currentThread())
            checkAccess();
//同步操作，确定在一个时间段内只有一个线程尝试中断目标线程。
        synchronized (blockerLock) {
//判断当前线程是否是可中断的I/O操作线程。如果是，这直接中断该线程的操作，如果不是，仅仅是将将线程设上中断标志位。真正使线程中断的操作是b.interrupt();
            Interruptible b = blocker;
            if (b != null) {
                interrupt0();           // Just to set the interrupt flag
                b.interrupt(this);
                return;
            }
        }
        interrupt0();
    }

```
可以看到，当我们的线程如果为非可中断的I/O线程时，我们调用`interrupt`方法时，线程仅仅是将中断标志位标志为中断状态，但是线程实际上还是处于一个运行状态。因此，如果我们需要在线程的状态处于中断状态时，立即中断的话，需要一直循环判断线程是否处于中断状态。
### isInterrupted与interrupted
二者均是判断线程是否中断的方法，一个是判断当前线程是否处于中断状态（静态方法），一个是用于判断目标线程是否处于中断状态（非静态方法）。

判断线程是否处于中断状态的方法，

``` java
public static boolean interrupted(){ 
	return currentThread().isInterrupted(true);
 }
public boolean isInterrupted(){ 
	return isInterrupted(false);
}
```
二者的均是通过本地方法` isInterrupted(boolean ClearInterrupted)`实现，参数表示是否重置线程的状态，当参数为`true`时，需要将线程的中断状态重置为`false`。

``` java
private native boolean isInterrupted(boolean ClearInterrupted);
```
当参数为true时，需要清除当前的线程状态，并重置。
### join
`join`方法使当前线程等待指定线程执行结束后或者执行一定时间后才继续执行，此时当前线程的状态为`WAITING`或者`TIMED_WAITING`。

``` java
public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
//如果等待时间为0，表示需要一直等待，直到线程结束。
        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
//计时操作开始，如果超时，则立即退出，如果线程在规定时间内执行完毕，也立即返回，
            while (isAlive()) {
                long delay = millis - now;
//超时
                if (delay <= 0) {
                    break;
                }
//休眠
                wait(delay);
//刷新等待时间
                now = System.currentTimeMillis() - base;
            }
        }
    }
```
该方法具有重载方法`public final void join()`，该方法没有指定执行时间，需要等待指定线程执行结束后，才能执行当前线程。
### setPriority
`setPriority` 用来设置Java线程的优先级。Java线程优先级总共有10个级别，1～10，数值越大，优先级越高。通常默认的线程优先级别为5。如果需要对线程设置优先级，则必须在线程启动前进行设置。

``` java
public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            setPriority0(priority = newPriority);
        }
    }
```
在设置目标程序的的优先级时，是判断当前运行线程是否有权向修改目标程序的优先级。线程的优先度设置范围为[1,10]，当超过这个范围时，便会抛出异常。

## 本地方法
在Thread中有很多本地方法，由这些本地方法为Thread提供底层实现。
+`public static native Thread currentThread();`：用于返回当前正在执行线程的状态。
+`public static native void yield();`：让出当前处理机，但依然持有资源锁。
+`public static native void sleep(long millis)；`：在指定时间内，线程处于阻塞状态，然后又回到就绪状态。
+`private native void setPriority0(int newPriority);`:设置优先级。
+`private native void stop0(Object o);`：终止线程。
+`private native void suspend0();`：挂起线程。
+`private native void resume0();`：重新开始挂起的线程。
+`private native void interrupt0();`：中断线程
+`private native void setNativeName(String name);`

## PS：1、线程状态
线程的状态由一个枚举类来实现。

``` java 
public enum State{
    NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED;
}
```
+`NEW`：线程还未启动前的状态，即调用start方法之前的线程状态。
+`RUNNABLE`：运行状态，通俗的理解就是线程在执行run方法时的线程状态。当前Java虚拟机正在执行该线程。
+`BLOCKED`：阻塞状态，表示当前线程正在等待一个监控锁，即线程因获取不到资源锁而处于阻塞状态。
`WAITING`：等待状态，由于线程调用了没有指定时间的join方法，资源对象的object.wait方法，和锁的park方法。该状态+表示本线程正在等待其他线程执行一些指定的动作才能改变当前的状态。
+`TIMED_WAITING`：具有有限等待时间的状态，该状态有以下操作导致：sleep，具有指定时间的wait方法，具有等待时间的join方法，和locksupport的parkNanos以及parkUntil。
+`TERMINATED`：结束状态，线程已经执行完毕。
当线程处于`WAITING`，`TIMED_WAITING`状态时，如果去中断该线程，会导致该线程抛出异常。
## PS 2、线程优先级
上面提到线程的优先级主要用数值[1,10]表示，其各个意思表示：
+`10`：Crisis management（应急处理）
+`7-9` ：Interactive, event-driven（交互相关，事件驱动）
+`4-6`：IO-bound（IO限制类）
+`2-3`：Background computation（后台计算）
+`1`：Run only if nothing else can（仅在没有任何线程运行时运行的）
线程的优先级仅仅是保证了在多个线程进行竞争时，优先级高的线程可能优先获取CPU。
## PS 3、Thread的静态方法
Thread中的静态方法其针对对象均是当前对象。如：`yield`

--------------------------------------------------------------------------
参考链接：
+ http://stackoverflow.com/questions/1010645/what-does-the-registernatives-method-do
+ http://techbook.blog.163.com/blog/static/304885102012235613945/
+ http://blog.csdn.net/jamse19860909/article/details/7210244
--------------------------------------------------------------------------

--------------------------------------------------------------------------
###### 博文原创，欢迎装载！
###### 转载请附上原文链接：http://xuhuiqiang.cn/2016/08/28/threadAnalysis/
--------------------------------------------------------------------------
