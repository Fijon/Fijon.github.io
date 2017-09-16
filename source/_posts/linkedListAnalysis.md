---
title: LinkedList源码简析
tags:
- Collection
- List
- Original
categories: Java
date: 2016-08-13 22:34:28
keywords: 集合 Collection List LinkedList
description: Java线性链表源码简单解析

---
常用Java集合--线性双向链表LinkedList<E>源码简析
<!--more-->
## 前言
LinkedList<E>常用集合之一，是一个线性双向链表，继承自AbstractSequentialList类，实现了List接口和Deque接口。LinedkList的结点元素可以为null值，为一个线程不安全的链表。
## 属性简析
``` java
	transient int size = 0;
	transient Node<E> first;
	transient Node<E> last;
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

```
其中：
`int size`:记录链表的大小。
`Node<E> first, last`:first首结点指针，last尾结点指针。
`Node<E>`的构造方法不对当前结点的头前驱结点和后继结点进行判空，因此需要在获取新的Node对象之后，需要对其前驱与后继结点进行判空处理。这也是与ArrayList的不同之处。ArrayList在底层是通过数组下标来对ArrayList对象进行遍历，而LinkedList是通过Node结点的前驱指针和后继指针进行遍历操作。
## 构造方法
LinkedList<E>的构造方法分别为无参构造方法`LinkedList<E>`与`LinkedList<E>(Collection)`，两个构造方法及其简单，无参构造方法是一个空方法，没有执行代码。而有参构造方法是调用`addAll(Collection)`方法实现。与ArrayList类不同，LinkedList没有提供固定长度的构造方法。
当链表中只有一个结点时，`first`和`last`指针均指向该结点。

## 成员方法
LinkedList针对Node封装了多个方法，涉及到插入结点、删除结点和查询结点操作。大部分方法为私有或者同包之类可调用的方法。
### Node结点
对于Node结点插入操作分为三种：
`private void LinkedFirst(E)`:将所给元素插入到链表的头部，私有方法。
`default void LinkedLast(E)`:将所给元素插入到链表的尾结点，默认权限的方法。
`default void LinkedBefore(E, Node)`：将所给元素插入指定结点的前面。
`private E unlinkFirst(Node<E> f)`：在链表中，删除头节点First结点，返回删除头节点节点的元素值。（需要满足条件 (f == first && f !=null)
`private E unlinkLast(Node<E> l)`:删除链表的尾结点。
`default E unlink(Node<E> x)`：删除链表中的节点，x ！= null
以上操作都会影响到链表的结构，因此`modCount`变量都会自增。

`default Node<E> node(int index)` ：按照所给定的下标返回结点。

``` java
Node<E> node(int index) {
        // assert isElementIndex(index);
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```
`node(int)`是一个查询操作，针对参数`index`的大小，判断是从头结点或从尾结点开始查。当index小于`size/2`时，从头结点查起，当大于时从尾结点开始查起。

_当将整数除以2时，提倡使用移位处理，而不是使用除法运算。_

### 查
LinkedList查询方法包括：`indexOf(Object)` ，`getFirst()`，`getLast()`，`contains(Object)`，`get(int)`，`lastIndexOf(Object)`等等。
`public int indexOf(Object o)`:判断链表中是否存在元素o，如果存在则返回该元素的索引位置，否则返回-1。

``` java
public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```
`indexOf(Object)`：我们所说的LinkedList可以装载空值，是将Node结点中的Item变量值为空，而不是将Node结点置为空值。另一方面，判断二者是否相同，是通过调用Object的equals(Object)方法。因此我们需要自定义的类型对象OverWrite该方法。
`public int lastIndexOf(Object o)`：与indexOf类似，但是是从尾结点开始，两个方法都是返回第一次查询到的元素值。一旦匹配成功则立即返回。
`public boolean contains(Object o)` ：查看`indexOf()`方法的返回值，返回其与-1比较结果。
`public E getFirst()`:返回first结点的元素值，当first == null时，抛出异常NoSuchElementException异常。
`public E getLast()`：返回last结点的元素值，当last == null时，抛出异常NoSuchElementException异常。
`public E element()`：等同于getFirst()；
`public E peek()`，`public E peekFirst()`，`public E peekLast()`：peek方法仅仅返回头结点或尾结点的元素，如果头结点或尾结点为空指针时(first == null, last == null)，直接返回null。
`public E get(int index)`：在检查索引是否合法后，直接调用`node(int index)`方法。

以上关于查方法，不影响LinkedList的结构。在查询之后，删除查到的方法有：
`public E poll()`，`public E pollFirst()`，`public E pollLast()`：poll方法返回头结点或尾结点并删除该结点，当删除结点为空时，返回null。
`public E pop()`：删除并返回头结点，如果头结点为null，则直接返回null。

### 增
LinkedList插入元素的方法有：`add(E)`，`addAll(Collection)`，`addAll(int, Collection`，`push(E)`，`add(int, E)`，`offer(E)`等等方法。插入元素的方法均需要更新`modCount`的值。
`public void add(E element)`：往链表结尾插入一个新元素。直接调用`linkLast(E)`。
`public void add(int index, E element)`：往指定位置插入一个新的元素。当index == size时，直接调用`linkLast(E)`。否则直接调用`default void LinkedBefore(E, Node)`方法，参数Node = `node(index)`。
`public boolean addAll(int index, Collection<? extends E> c)`：将集合插入在index处开始的位置内。

``` java
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```
第一次if分支判断`if (index == size)`，当判断条件为true时，链表当前的尾结点索引为size - 1，因此尾结点的后继指针为null，入位置的前驱结点是链表尾结点。所以` succ = null; pred = last;`
当判断条件为false，则获取索引为index的结点，将其当作最后的插入结点的后继结点，并将其前驱结点当作第一个插入结点的前驱结点。
最后循环插入集合的所有元素。

`public void push(E e)`：等同于addFirst方法。
`public boolean offer(E e)`，`public boolean offerFirst(E e)`，`public boolean offerLast(E e)`：往链表插入头结点或者尾结点，直接调用`add***(E)`方法。

### 改
改是指修改结点的元素值，并返回结点的原有元素值。使用`set(int, E)`方法。

``` java
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
```
可以看到修改结点的元素值，不改变`modCount`的值，因此，在获取迭代器之后，通过该方法修改链表的元素值时，不会抛出异常。

### 删
LinkedList针对删除方法提供了：`removeFirst()`，`removeLast()`，`clear()`，`remove(int)`，`removeFirstOccurrence(Object)`，`removeLastOccurrence(Object)`。这些方法可以移除头结点，尾结点，按照指定索引移除结点。针对多个相同元素的结点，则移除第一次出现或者最后一次的结点。
在判断链表是否存在该元素时，是调用`equals(Object)`方法，因此需要注意该方法的实现逻辑。

## 迭代器
LinkedList具有两个迭代器，分别为ListItr（我称为正向迭代器），DescedningIterator（反向迭代器）。分别通过`public ListIterator<E> listIterator(int index)`和` public Iterator<E> descendingIterator()`方法获得。DescedningIterator本质上还是ListItr，通过持有ListItr对象来实现各个功能。

``` java
    private class DescendingIterator implements Iterator<E> {
        private final ListItr itr = new ListItr(size());
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
    }
```
### ListItr<E>
我们可以通过`listIterator()`或`listIterator(int)`获取具有全部结点的迭代器或者从指定索引位置开始的迭代器。

#### 属性
`private Node<E> lastReturned = null;`：记录最近一次返回的结点
`private Node<E> next;`：游标指针，记录当前结点的下一结点。
`private int nextIndex;`：游标索引，记录当前结点的下一结点的索引值。
`private int expectedModCount = modCount;`：更新次数标记。

#### 构造方法
``` java
    ListItr(int index) {
        // assert isPositionIndex(index);
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }
```

--------------------------------------------------------------------------
###### 博文原创，欢迎装载！
###### 转载请附上原文链接：http://xuhuiqiang.cn/2016/08/13/linkedListAnalysis/
--------------------------------------------------------------------------
