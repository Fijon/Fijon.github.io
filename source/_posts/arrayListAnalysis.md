---
title: ArrayList源码简析
date: 2016-08-13 10:46:56
tags: 
- Collection
- List
- Original
categories: Java 
keywords: 集合 Collection List ArrayList
description: 常用的Java集合类源码简析
---
常用Java集合---ArrayList
<!--more-->
## 前言
ArrayList是我们常用的集合之一，它继承自抽象类AbstractList，实现了List接口。其底层使用数组实现，并可以放入null元素。ArrrayList支持泛型，因此底层数组为一个Object数组。以下先从ArrayList的属性开始说起。
## 属性简析
``` java
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] EMPTY_ELEMENTDATA = {}; 
private transient Object[] elementData; 
private int size; 
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8; 
```
其中： 
`DEFAULT_CAPACITY`属性是默认的数组长度列表。当我们在初始化ArrayList时，如果没有给定默认的空间，ArrayList会在第一次添加元素的时候，直接实将elementData实例化为长度为10的Object数组。
`EMPTY_ELEMENTDATA`属性是一个空的常量数组，会在第一次实例化ArrayList对象时便声明初始化，并一直在内存中存在。
`elementData`属性是最终存储元素的变量。
`size`属性是ArrayList的元素个数，并不是elementData数组的长度。
`MAX_ARRAY_SIZE`常量表示装载数据数组的最大长度。但是elementData的长度可以超过该数值。

实际上，在ArrayList源码中，我们还可以看到一个从父类继承下来的变量`modCount`，该变量记录每一次更新ArrayList的次数，在迭代器用于判断ArrayList的结构是否更改。

## 构造方法
ArrayList提供了三个构造方法，分别默认无参构造方法`public ArrayList()`,  具有初始化长度的构造方法`public ArrayList(int initialCapacity)`，和将一个集合转换为ArrayList的构造方法`public ArrayList(Collection<? extends E> c)`。

``` java
public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
}
public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }
public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
```
无参构造函数，自动将elementData指向EMPTY_ELEMENTDATA。建议使用具有初始化长度的构造方法来实例化ArrayList对象。即使需要将一个集合转换为ArrayList，也不太建议使用`public ArrayList(Collection<? extends E> c)`方法。因为将集合转换为数组的过程中可能会出错。

## 更新ArrayList结构
更新ArrayList结构指的是对ArrayList进行增删改的操作，体现在ArrayList方法中便是：`add(E)`,`remove(E)`,`clear()`,`set(int, E)`等等。每次更新ArrayList的结构，都需要记录其操作次数。

### 增
增的方法，主要有多个：`add(E)`，`add(int, E)`，`addAll(Collection)`, `addAll(int, Collection)`
#### add(E)
在ArrayList的末尾添加一个元素。

``` java
 public boolean add(E e) {
        ensureCapacityInternal(size + 1); 
        elementData[size++] = e;
        return true;
 }
 ```
可以看到`add(E)`方法及其简单，仅仅是对`elementData`数组进行赋值操作。而`ensureCapacityInternal(int)`方法相对来说比较复杂，该方法用于判断是否需要对底层数组`elementData`进行扩容，并对相应结果进行处理。

``` java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;  
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```
上面代码可以看到，带有初始化长度的ArrayList对象与无参函数实例化ArrayList对象在数组添加第一个元素上的区别。在构造函数部分我们可以看到，无参构造函数实例化的ArrayList对象的`elementData`数组指向常量空数组对象`EMPTY_ELEMENTDATA`。第一次往一个空ArrayList对象添加元素时，`size`变量值为0，因此`minCapacity`值为1。从` minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);`这行代码可以看到，无参函数最后默认底层数组长度为`DEFAULT_CAPACITY`,即为10。
方法`ensureExplicitCapacity(minCapacity);`判断是否需要扩容的方法，并且记录更新结构次数的记录，其真正实现扩容的方法为`grow(int)`方法。当`minCapacity - elementData.length > 0`为true时，其数组参数过短需要进行扩容。这个扩容条件与HashMap不同，HashMap是在当前到达当前容量的0.75时进行扩容。
在需要扩容时，按照1.5倍大小扩容（`int newCapacity = oldCapacity + (oldCapacity >> 1)`)。需要判断`newCapacity`是否小于`minCapacity`，如果小于，则以`minCapacity`为扩充容量，并赋值给`newCapacity`。继续判断`newCapacity`是否大于数组链表的最大容量`MAX_ARRAY_SIZE`。如果大于，则需要判断`minCapacity`是否越界。在不越界的情况下，如果如果当前的数组长度为`MAX_ARRAY_SIZE`，则将数组长度扩充到`Integer.MAX_VALUE`。

``` java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
}
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```
为什么需要判断`newCapacity`与`minCapacity`的大小？按照理解：`minCapacity = size + 1`,`size <= elementData.length`,而`newCapacity = 1.5 * elementData.length`,因此在`elementData.length >= 10`的情况下`minCapacity`永远小于`newCapacity`。在我的理解中，需要排除我们设置ArrayList的长度为1，2，3的情况。

#### add(int,E)
在ArrayList的指定位置，添加一个元素，将在原有的位置上的元素往后移一个位置。

```java 
public void add(int index, E element) {
    rangeCheckForAdd(index);
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```
首先先坚持index是否在(0, size)范围内，然后记录更新结构次数，并判断是否进行扩容，然后调用系统函数进行数组复制。

数组复制：
Arrays.copyOf(t[] original, int length)与Arrays.copyOfRange(t[] original, int from， int two);
二者都是最终调用System.arraycopy(Object src,  int  srcPos,  Object dest, int destPos, int length);src是原数组，srcPos是开始复制的下标位置，dest是目标数组，destPos是开始装载的位置，length是复制长度，即被复制数组长度为：srcPos + length -1。
Arrays.copyOf(t[] original, int length)：original是源数组，length是从下表为0开始，总共复制length个元素。
Arrays.copyOfRange(t[] original, int from， int to)：original是源数组，从下标为from开始复制，直到下表为to结束，但是不包括下标to元素,复制长度为to - from。

``` java
int [] one = {0,1,2,3,4};
int [] two = new int[5];
two = Arrays.copyOf(one, 2);
System.out.println(Arrays.toString(two)); //[0, 1]

two = new int[5];
two = Arrays.copyOfRange(one, 1, 4);
System.out.println(Arrays.toString(two)); //[1, 2, 3]

two = new int[5];
System.arraycopy(one, 1, two, 2, 2); 
System.out.println(Arrays.toString(two)); //[0, 0, 1, 2, 0]
```

### 改
ArrayList的改主要体现在`set(int, E)`方法上，该方法返回下标为index的原有元素，并将当前位置的值置为element；还有截取子序列的方法`subList(int fromIndex, int toIndex)`，`subListRangeCheck(int fromIndex, int toIndex, int size)`。
#### set(int, E)
`set(int, E)`方法返回下标为index的原有元素，并将当前位置的值置为element；

``` java
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```
#### subList(int fromIndex, int toIndex)与subListRangeCheck(int fromIndex, int toIndex, int size)
这两个方法是用于截取子序列的方法，直接返回了SubList对象。
### 删
删除ArrayList元素的方法有多个，分别为：`remove(int)`，`remove(Object)`，`removeAll(Collection)`，`retainAll(Collection)`，`clear()`。
#### remove(int)
`remove(int)`:删除下标为参数的元素，并返回该元素。

``` java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                        numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```
先检查是否越界，获取该下标元素，最后进行数字元素移动处理，将该下标之后的元素往前移动一位，并将最后一位元素置空。
#### remove(Object)
`remove(Object)`:判断ArrayList对象是否包含该对象，一旦检测到包含该对象，着删除该对象，并返回true，如果未找到，则返回false。该方法仅删除排在最前的相同元素。

``` java
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```
源码是通过`if (o.equals(elementData[index]))`来判断是否存在目标元素，因此我们需要根据需求来复写装载对象的`equals(object)`方法。
#### removeAll(collection)与retainAll(Collection)
二者的作用相反，一个是移除集合所包含的元素，一个是仅保留集合中包含的元素。二者均是通过`batchRemove(Collection, boolean)`方法实现。`removeAll(Collection)`调用``batchRemove(Collection, false)`,`retainAll(Collection)`调用`batchRemove(Collection, boolean)`

``` java
    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```
源码中使用缓存数组装载ArrayList对象中与Collection对象相同的元素，通过`if (c.contains(elementData[r]) == complement)`语句来判断（好精彩的思维）。
当`complement`为false时：
如果集合中包含该元素，`c.contains(elementData[r]`为true,因此条件不成立，跳过if支线。
如果集合中不包含该元素，`c.contains(elementData[r]`为false,因此条件成立，数组`elementData`前面装载的是Collection对象与ArrayList对象的差集。
当`complement`为true时：
如果集合中包含该元素，`c.contains(elementData[r]`为true,因此条件成立，`elementData`前面装载的是Collection对象与ArrayList对象的交集。
如果集合中不包含该元素，`c.contains(elementData[r]`为false,因此条件不成立，跳过if支线。
之所以需要try是因为AbstractCollection的包含是通过迭代器来判断的，而迭代器操作时有可能抛出异常，因此需要try来捕获异常。由finally处理可以知道，一旦异常发生，这将未判断的元素一并视为不需要移除或者需要保留的元素。
#### clear()
`clear()`方法循环将数组的元素置为`null`，`size`置为0，并将更新次数`modCount`自增1。

### 查
对ArrayList的查操作主要包括：通过下标获取元素，查看元素的下标，查看元素最后出现的下标，判断是否包含该元素，以及ArrayList对象是否包含元素。分别为`get(int)`，`indexOf(E)`，`lastIndexOf(E)`，`contains(E)`，`isEmpty()`
#### get(int)
方法`get(int)`非常简单，仅仅是检查index是否合法，之后直接返回下标为index的数组元素。
``` java
    public E get(int index) {
        rangeCheck(index);
        return elementData(index);
    }
    E elementData(int index) {
        return (E) elementData[index];
    }
```
#### indexOf(E)与lastIndexOf(E)
`indexOf(E)`与`lastIndexOf(E)`的区别在于一个从0开始，一个从数组的结尾开始查起。

``` java
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
`lastIndexOf(E)`方法仅仅需要将`for (int i = 0; i < size; i++)`改为`for (int i = size-1; i >= 0; i++)`
#### contains(E)与isEmpty()
这两个方法较为简单，`contains(E)`方法通过调用`indexOf(E)`方法实现，而`isEmpty()`是直接返回`size == 0`的结果。

``` java
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```
### ArrayList迭代器
ArrayList迭代器提供了一个可以遍历ArrayList对象，允许按照任一方向遍历ArrayList，并可在迭代期间修改ArrayList对象。ArrayList的迭代器有两个，分别是直接实现Iterator<E>接口的`Itr`内部类和继承`Itr`类实现ListIterator接口的`ListItr`内部类。这两个迭代器均是ArrayList的内部类。
获取ArrayList迭代器的方法有三种：`ListIterator(int)`，`ListIterator()`，`Iterator()`。
`ListIterator(int)`：返回从所给参数开始的ListItr迭代器。
`ListIterator()`：返回从0开始迭代器。
`Iterator()`：返回从0开始的Itr的迭代器。
#### Itr
##### 属性
`int cursor`:游标，记录当前元素的下一个元素，cursor == size时，证明已经遍历到ArrayList的结尾了。
`int lastRet = -1`:记录最近一次被返回的元素的下标，如果没有的话，则为-1；
`int expectedModCount = modCount`:记录该ArrayList对象结构的更新次数，需要与ArrayList对象的`modCount`变量值一样。当有异时，抛出`ConcurrentModificationException`异常。

在迭代器的每次操作之前，都需要判断ArrayList结构是否被更新过（即判断`expectedModCount == modCount`）。一旦二者不相等，则会抛出异常。
##### 方法
Itr直接实现Iterator<E>接口的三个方法，`hasNext()`，`next()`，`remove()`。
每个方法均需要检测`expectedModCount == modCount`。

#### ListItr
`ListItr`继承`Itr`，实现了`ListIterator`接口，具有`hasPrevious()`，`nextIndex()`，`previousIndex()`，`previous()`，`set(E)`，`add(E)`方法。

### SubList
SubList继承AbstractList并标记RandomAccess接口。其方法实现大多调用ArrayList方法。
#### 属性
`private final AbstractList<E> parent`:被截取的ArrayList对象，父序列。
`private final int parentOffset`：相对于父序列的相对偏移量。
`private final int offset`：子序列的偏移量。
`int size`：子序列的元素个数。
具体的属性意思，可以查看其构造方法：

``` java
SubList(AbstractList<E> parent, int offset, int fromIndex, int toIndex) {
            this.parent = parent;
            this.parentOffset = fromIndex;
            this.offset = offset + fromIndex;
            this.size = toIndex - fromIndex;
            this.modCount = ArrayList.this.modCount;
}
```

--------------------------------------------------------------------------
###### 博文原创，欢迎装载！
###### 转载请附上原文链接：http://xuhuiqiang.cn/2016/08/13/arrayListAnalysis/
--------------------------------------------------------------------------

