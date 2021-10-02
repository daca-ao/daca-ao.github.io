---
title: Java 线程安全集合类
date: 2021-10-02 00:17:13
tags:
- Java
- Collection
- 多线程编程
---

线程安全集合类是 Java 除了并发工具类和多线程执行器之外，另外一种成熟的并发编程解决方案。

<!-- more -->

在还没有推出线程安全集合的早期，开发任务使用 `Collections` 工具类中的**同步包装器**来实现线程安全的集合操作：

```java
static <E> Collection<E> synchronizedCollection(Collection<E> c)
static <E> List synchronizedList(List<E> l)    // 所有方法都带同步锁
static <E> Set synchronizedSet(Set<E> s)
static <E> SortedSet synchronizedSortedSet(SortedSet<E> s)
static <K, V> Map<K, V> synchronizedMap(Map<K, V> m)
static <K, V> SortedMap<K, V> synchronizedSortedMap(SortedMap<K, V> s)
```

Java 在后来的 JDK 中推出了两个线程安全的集合类：
```java
Hashtable
Vector
```

至今，在 JCF 中的同步接口包括：
```java
BlockingDeque
BlockingQueue
ConcurrentMap
ConcurrentNavigableMap
TransferQueue
```

以上都是能够避免 `ConcurrentModificationException` 的类和方法。

我们先从阻塞队列开始说起。

<br/>

# 阻塞队列

阻塞插入：当队列满的时候，队列会阻塞插入元素的线程，直到队列有空缺为止；  
阻塞移除：当队列为空的时候，获取元素的线程会等待至队列非空为止。

| 方法 / 特殊情况处理方式  | 抛出异常    | 返回特殊值   | 超时退出               | 一直阻塞  |
| -------------------- | ---------- | ---------- | --------------        | -------- |
| 插入方法              | add(e)     | <font color="red">o</font>ffer(e)  | offer(e, time, unit)  | pu<font color="blue">t</font>(e)   |
| 移除方法              | remove()   | p<font color="red">o</font>ll()    | poll(time, unit)      | <font color="blue">t</font>ake()   |
| 检查方法              | element()  | peek()      | -         | -        |

<br/>

| 方法 | 正常动作 | 特殊情况动作 |
| --  | -----   | ---------- |
| add     | 添加元素（添加 null 值非法） | 队列满时抛出 `IllegalStateException` |
| offer   | 添加元素并返回 true  | 队列满时返回 false      |
| put     | 添加元素            | 队列满时**阻塞**        |
| remove  | 移除并返回队列头元素  | 队列空时抛出 `NoSuchElementException`       |
| poll    | 移除并返回队列头元素  | 队列空时返回 null       |
| take    | 移除并返回队列头元素  | 队列空时**阻塞**        |
| element | 返回队列头元素       | 队列空时抛出 `NoSuchElementException`       |
| peek    | 返回队列头元素       | 队列空时返回 null       |

<br/>

在 JCF 中，阻塞队列对应着 `BlockingQueue`、`BlockingDeque` 和 `TransferQueue` 接口。

```java
import java.util.concurrent.BlockingQueue;

BlockingQueue<E>  // 阻塞队列


void put(E element)    // 添加元素，在必要时阻塞

boolean offer(E element, long time, TimeUnit unit)
// 添加给定元素，成功则返回 true，队列满时返回 false
// 必要时阻塞，直至元素已经被添加，或添加操作超时

E poll(long time, TimeUnit unit)
// 移除并返回队列头元素
// 必要时阻塞，直至元素可用，或移除操作超时
// 失败时返回 null

E take()    // 移除并返回头元素，必要时阻塞
```

```java
import java.util.concurrent.BlockingDeque;

BlockingDeque<E>  // 双向阻塞队列


void putFirst(E element)
void putLast(E element)
// 添加元素，必要时阻塞

boolean offerFirst(E element, long time, TimeUnit unit)
boolean offerLast(E element, long time, TimeUnit unit)
// 添加给定元素，成功时返回 true，队列满时返回 false
// 必要时阻塞，直至元素被添加，或添加操作超时

E pollFirst(long time, TimeUnit unit)
E pollLast(long time, TimeUnit unit)
// 移除并返回头元素或尾元素
// 必要时阻塞，直至元素可用，或移除操作超时
// 失败时返回 null

E takeFirst()    // 移除并返回头元素，必要时阻塞
E takeLast()     // 移除并返回尾元素，必要时阻塞
```

```java
import java.util.concurrent.TransferQueue;

TransferQueue<E>

/**
 * 传输一个值，或尝试在给定时间内传输这个值
 * 阻塞至另一线程将元素删除
 */
void transfer(E element)

/**
 * 尝试在给定时间内传输这个值
 * 阻塞至另一线程将元素删除
 * 调用成功时返回 true
 */
boolean tryTransfer(E element, long time, TimeUnit unit)
```


## 实现类

`ArrayBlockingQueue`：由数组结构组成的有界阻塞队列

```java
import java.util.concurrent.ArrayBlockingQueue;


ArrayBlockingQueue<E>(int capacity)
ArrayBlockingQueue<E>(int capacity, boolean fair)
// 构造一个带有指定的容量和公平性设置的阻塞队列
// 使用循环数组实现
```


`LinkedBlockingQueue`：由链表结构组成的有界阻塞队列

`LinkedBlockingDeque`：由链表结构组成的双向阻塞队列

```java
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.LinkedBlockingDeque;


/**
 * 构造一个无上限的阻塞队列或双向队列
 * 采用锁来保持同步
 * 链表实现
 */
LinkedBlockingQueue<E>()
LinkedBlockingDeque<E>()

/**
 * 构造有限的阻塞队列或双向队列
 * 链表实现
 */
LinkedBlockingQueue<E>(int capacity)
LinkedBlockingDeque<E>(int capacity)
```


`DelayQueue`：使⽤优先级队列实现的⽆界阻塞队列

```java
import java.util.concurrent.DelayQueue;
import java.util.concurrent.Delayed;

DelayQueue<E extends Delayed>

/**
 * 构造一个包含 Delayed 元素的无界的、阻塞时间有限的阻塞队列
 * 使用堆（priority heap）实现
 * 基于时间调度的序列，只有延迟超过时间的元素才可从队列中移出
 */
DelayQueue()


interface Delayed {}

// 得到该对象的延迟
// 用给定时间单位度量
long getDelay(TimeUnit unit)
```

`PriorityBlockingQueue`：⽀持优先级排序的⽆界阻塞队列

```java
import java.util.concurrent.PriorityBlockingQueue;


/**
 * 构造一个无边界阻塞优先队列
 * 使用堆（priority heap）实现
 * initialCapacity 初始值为 11
 * comparator 为用来对元素进行比较的比较器，如未指定则元素应实现 Comparable 接口
 */
PriorityBlockingQueue<E>()
PriorityBlockingQueue<E>(int initialCapacity)
PriorityBlockingQueue<E>(int initialCapacity, Comparator<? super E> comparator)
```

`LinkedTransferQueue`：由链表结构实现的 TransferQueue，⽆界阻塞队列

`SynchronousQueue`：不存储元素的阻塞队列，简单实现了阻塞机制


## 应用

生产者线程向队列插入元素，消费者线程从队列取出元素
* worker（工作者）线程周期性地将中间结果存储到阻塞队列
* 其他 worker 线程移除中间结果作进一步修改
* 阻塞队列会自动做负载均衡
    * 搜索或移除元素时，会等待队列变为非空
    * 添加元素时，会等待队列有可用空间
* 会创建专门的线程去访问各对象，而不是各种有可能的工作线程

<br/>

# 线程安全集合

JDK `concurrent` 包下还有许多线程安全的集合类，使用起来跟普通的名字对应的集合类几乎没有区别。

它们的使用特点是：size 不必在常量时间内操作
* 确定集合的当前大小需要遍历
* 集合返回弱一致性（weak consistent）的迭代器，因此不一定反映出构造后所有的修改
* 不会抛出 ConcurrentModificationException

## ConcurrentHashMap & ConcurrentSkipListMap

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentSkipListMap;


/**
 * 基于 HashTable 及 ConcurrentMap 的高性能同步实现，支持 HashTable 所有旧方法
 * 构造一个可被多线程安全访问的散列映射表
 *
 * initialCapacity 默认为16
 * loadFactor：如每个桶平均负载超过该因子，表大小会被重新调整（默认为0.75）
 * concurrencyLevel：并发写线程的估计数目
 */
ConcurrentHashMap<K, V>()
ConcurrentHashMap<K, V>(int initialCapacity)
ConcurrentHashMap<K, V>(int initialCapacity, float loadFactor, int concurrencyLevel)

/**
 * 构造一个可被多线程安全访问的有序映射表
 * TreeMap 的同步实现，实现了 ConcurrentNavigableMap 接口
 * 第一个构造器要求键实现 Comparable 接口
 */
ConcurrentSkipListMap<K, V>()
ConcurrentSkipListMap<K, V>(Comparator<? super K> comp)


V putIfAbsent(K key, V value)
// 如该键不在映射表中，则将给定键值对关联，并返回 null
// 否则返回与该键关联的现有值


boolean remove(K key, V value)
// 如给定的键与给定值关联，删除给定键值对并返回 true
// 否则返回 false


boolean replace(K key, V oldValue, V newValue)
// 如给定键当前与 oldValue 相关联，则将 newValue 与键相关联
// 否则返回 false
```

实现：

Java 7 & before：分段锁实现并发更新
* 底层采用数组 + 链表
* 核心静态类：
    * Segment：继承 ReentrantLock充当锁的角色，每个 Segment 对象守护每个散列映射表的若干个桶
    * HashEntry：封装映射表的键值对
* 每个桶由若干个 HashEntry 对象链接起来组成链表



Since Java 8：Node + CAS + Synchronized 保证并发安全
* 取消 Segment，直接用 table 数组存储键值对
* 当 HashEntry 对象组成的链表长度超过 TREEIFY_THRESHOLD 时：链表转换为红黑树



<br/>

## ConcurrentLinkedQueue

```java
import java.util.concurrent.ConcurrentLinkedQueue;


/**
 * 构造一个可被多线程安全访问的无边界非阻塞的队列
 * 
 * 非阻塞的 FIFO 并发队列
 * 采用 CAS（Compare and Set）操作保证同步：在 set 之前先 compare 该值有没有变化，没有变化才对其赋值
 * 底层采用链表实现
 */
ConcurrentLinkedQueue<E>()
```

<br/>

## 其它线程安全集合

`ConcurrentSkipListSet`
* 线程安全的有序集合
* 可以视为 `TreeSet` 的同步实现，实现了 NavigableSet 接口

```java
import java.util.concurrent.ConcurrentSkipListSet;


ConcurrentSkipListSet<E>()
ConcurrentSkipListSet<E>(Comparator<? super E> comp)
// 构造一个可被多线程安全访问的有序集
// 第一个构造器要求元素实现 Comparable 接口
```

`CopyOnWriteArrayList`

先说一下概念：**CopyOnWrite**, COW，复制再写入
* 在添加元素的时候，先将原集合列表复制一份，再添加新的元素
* 只适合**读多写少**的情况

该集合类参考了延时懒惰策略和读写分离的思想
* 在读取时，多线程正常读取内容
* 写内容时，将当前对象进行复制，往**新的对象进行修改**，完成后再将引用指向新的对象
* 将读写分离，属于读写锁的实现（读同步，写互斥）

```java
// 添加元素时，先加锁，再进行复制替换操作，最后释放锁
public boolean add(E e) {
    ReentrantLock lock = this.lock;
    lock.lock();

    boolean added;
    try {
        Object[] elements = this.getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        this.setArray(newElements);
        added = true;
    } finally {
       lock.unlock();
    }

    return added;
}


private E get(Object[] a, int index) {
    return a[index];
}

public E get(int index) {
    return this.get(this.getArray(), index);
}
```

类似的实现：`CopyOnWriteArraySet`
```java
// 添加元素：调用 CopyOnWriteArrayList 的 addIfAbsent()
public boolean addIfAbsent(E e) {
    Object[] snapshot = this.getArray();
    return indexOf(e,snapshot, 0, snapshot.length) >= 0 ? false : this.addIfAbsent(e,snapshot);
}
```
