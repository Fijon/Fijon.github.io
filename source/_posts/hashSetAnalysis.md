---
title: HashSet源码简析
tags:
  - Java
  - Collection
  - HashSet
categories: Java
date: 2016-08-15 22:21:20
keywords: Java Set
description:
---
常用Java集合类--HashSet源码简析
<!--more-->
## 前言
HashSet：实现Set接口，并继承自AbstractSet抽象类。其底层有hashMap支持。HashSet不保证Set的迭代顺序，而且集合里边的顺序并非恒久不变。鉴于HashMap支持使用null作为元素，因此HashSet也支持使用null作为元素。鉴于HashSet是由HashMap支持的，因此HashSet的很多操作都是直接调用HashMap方法。如果对HashMap较为熟悉，那么HashSet很容易就能看懂。

## 属性
`private transient HashMap<E,Object> map`：用于存储元素的对象。
`private static final Object PRESENT = new Object()`：用于管理底层map对象的一个虚值。

构造方法：
相对于`List`相关类，HashSet的构造方法有5个。由于HashSet是通过HashMap支持实现的，每一个集合的元素最终都是装入HashMap对象中，因此，其构造方法就是根据参数要求，初始化全局属性`map`。构造方法如下：
+ `public HashSet()`：无参构造方法，使用HashMap的无参构造函数初始化该对象，默认容量为16，扩展因子为0.75；
+ `HashSet(Collection<? extends E>)`：构造一个包含指定集合中的元素的新的集合。

``` java
	public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
```
构造方法中之所以进行`c.size()/.75f) + 1`处理是因为，当HashMap的元素个数达到容量的0.75倍时，就需要扩容。因此为了避免进行扩容，需要将HashMap的容量开辟为`c.size()/.75f) + 1`。如果该值小于16时，则将HashMap的容量开辟到16。因此可以知道当集合元素个数处于[0,11]范围时，集合的容量为16。
+`HashSet(int, float)`：该构造方法指定了HashMap的初始容量和扩展因子。即当集合的元素个数`size >= initialCapacity * loadFactor`时，需要对HashMap进行扩容。

``` java
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }
```
+ `public HashSet(int) `：设定集合的初始容量，其扩展因子还是0.75。
+ `default HashSet(int, float, boolean)`：`boolean`参数用来区分其他构造方法，该方法非公共方法，仅仅供给`linkedHashSet`使用。

## 成员方法
集合相对列表来说，不存在指定位置插入，或者指定位置删除以及获取指定位置的元素，这是因为集合是无序的。因此，HashSet提供`add(E)`，`remove(E)`，`contains(E)`。
+ `public boolean add(E e)`：从源码中，我们可以看到，集合的元素作为HashMap中的键存储于HashMap对象中，而`map`对象的值存放一个无意义的`Object`对象。

``` java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

+ `public boolean remove(Object o)`：删除指定的对象，直接返回`map.remove(o)==PRESENT`。
+ `public boolean contains(Object o) `：判断是否包含参数元素。直接调用`map.containsKey(o)`。
+ `public void clear()`：直接清空整个集合的元素。


--------------------------------------------------------------------------
###### 博文原创，欢迎装载！
###### 转载请附上原文链接：http://xuhuiqiang.cn/2016/08/15/hashSetAnalysis/
--------------------------------------------------------------------------
