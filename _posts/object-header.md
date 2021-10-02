---
title: 对象头
date: 2021-07-16 19:42:34
tags:
- Java
- JVM
---

在 Hotspot 虚拟机中，除了对象本身的实例数据（Instance Data）之外，其在内存中的布局还包括了**对象头**（Object Header）和对齐填充（Padding）。

<!-- more -->

对齐填充的字节保留主要是出于 JVM 对 Java 对象内存占用的要求来考虑的。每个对象在 JVM 内存占用的大小应为 1 个字长（8 bits）的倍数，因此每个对象最后会有几个字节用于补全，没有特别的功能。  
因此本帖不再赘述对齐填充，就专门来聊聊对象头。

Hotspot 虚拟机的对象头包括两（三）部分信息：
1. 对象自身的运行时数据
2. 类型指针
3. 数组长度（前提是该对象是数组）

<br/>

# 运行时数据（Mark Word）
包括哈希码（hash code）、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等。  
数据长度在 32 位和 64 位的虚拟机（未开启压缩指针）中分别是 32 bits 和 64 bits。

对象需要存储的运行时数据很多，其实已超出了 32、64 位 Bitmap 结构所能记录的限度；但是，对象头信息是**与对象自身定义数据无关**的额外存储成本。  
基于这一点，再考虑到虚拟机的空间效率，因此 Mark Word 被设计成一个**非固定的数据结构**，以便在极小的空间内存储尽量多的信息，且会根据对象的状态复用自己的存储空间。

比如：在 32 位 Hotspot 虚拟机中，如果对象未被锁定，那么 Mark Word 的 32 bits 的空间将被如下分配：
* 25 bits 被用于存储对象的 hash code
* 4 bits 用于存储对象分代年龄
* 1 bit 固定为 0（因其未被锁定，不是偏向锁状态）
* 2 bits 用于存储锁标志位

如下表的第一行所示。

而在其他状态下，对象的运行时数据存储内容如下：

<table>
	<tr>
	    <th>场景（状态）</th>
	    <th colspan="5">存储内容占用的空间</th>
	</tr>
	<tr>
	    <th rowspan="2" bgcolor="#CCCCCC">锁状态</th>
	    <td colspan="2">25 bits</td>
        <td rowspan="2">4 bits</td>
        <td>1 bit</td>
	    <td>2 bits</td>
	</tr>
	<tr>
	    <td>23 bits</td>
	    <td>2 bits</td>
        <td>是否是偏向锁（biased_lock）</td>
        <td>锁标志位</td>
	</tr>
	<tr>
	    <th bgcolor="#CCCCCC">无锁</th>
        <td colspan="2">对象的 hash code</td>
        <td>age 分代年龄</td>
        <td>0</td>
        <td>01</td>
	</tr>
	<tr>
	    <th bgcolor="#CCCCCC">偏向锁</th>
        <td>偏向锁记录的线程 ID</td>
        <td>epoch</td>
        <td>age 分代年龄</td>
        <td>1</td>
        <td>01</td>
	</tr>
	<tr>
        <th bgcolor="#CCCCCC">轻量级锁</th>
        <td colspan="4">pointer_to_lock_record 指向栈中锁记录的指针</td>
        <td>00</td>
	</tr>
	<tr>
	    <th bgcolor="#CCCCCC">重量级锁</th>
        <td colspan="4">pointer_to_heavyweight_monitor 指向互斥量（重量级锁）的指针</td>
        <td>10</td>
	</tr>
	<tr>
	    <th bgcolor="#CCCCCC">GC 标记</th>
        <td colspan="4">空</td>
        <td>11</td>
	</tr>
</table>

<br/>

在 64 位环境中，对象的运行时数据存储分布如下：

<table>
	<tr>
	    <th>场景（状态）</th>
	    <th colspan="6">存储内容占用的空间</th>
	</tr>
	<tr>
	    <th rowspan="2" bgcolor="#CCCCCC">锁状态</th>
	    <td colspan="2" rowspan="2">56 bits</td>
        <td rowspan="2">1 bit</td>
        <td rowspan="2">4 bits</td>
        <td>1 bit</td>
	    <td>2 bits</td>
	</tr>
	<tr>
        <td>是否是偏向锁（biased_lock）</td>
        <td>锁标志位</td>
	</tr>
	<tr>
	    <th bgcolor="#CCCCCC">无锁</th>
        <td>未使用空间（25 bits）</td>
        <td>对象的 hash code（31 bits）</td>
        <td>cms_free</td>
        <td>age 分代年龄</td>
        <td>0</td>
        <td>01</td>
	</tr>
	<tr>
	    <th bgcolor="#CCCCCC">偏向锁</th>
        <td>偏向锁记录的线程 ID（54 bits）</td>
        <td>epoch（2 bits）</td>
        <td>cms_free</td>
        <td>age 分代年龄</td>
        <td>1</td>
        <td>01</td>
	</tr>
	<tr>
        <th bgcolor="#CCCCCC">轻量级锁</th>
        <td colspan="5">pointer_to_lock_record 指向栈中锁记录的指针</td>
        <td>00</td>
	</tr>
	<tr>
	    <th bgcolor="#CCCCCC">重量级锁</th>
        <td colspan="5">pointer_to_heavyweight_monitor 指向互斥量（重量级锁）的指针</td>
        <td>10</td>
	</tr>
	<tr>
	    <th bgcolor="#CCCCCC">GC 标记</th>
        <td colspan="5">空</td>
        <td>11</td>
	</tr>
</table>
<br/>

一个对象在一个时间点总是处于其中的一个状态，只是状态之间可能会切换。

**参数说明**：

`biased_lock`：对象第一次被线程索取锁的时候，线程 ID 会写入 Mark Word 内（32 位系统的前 23 bits，或 64 位系统的前 54 bits）
* 并且将 Mark Word 的“偏向锁”那一位设置为 1
* 下次该线程想要获取锁的时候，直接检查对象头中保存的 ID 是否等于自身线程 ID，如一致则认为当前线程获取了锁
* 这样就不需要再次获取锁了，略过了轻量级和重量级两种锁的加锁阶段，提高效率。

`epoch`：验证偏向锁有效性的时间戳。

`cms_free`：说起来就跟内存分配策略有关系了。
* CMS 是基于清理算法的收集器（Concurrent Mark-Sweep），相应的内存分配策略就是空闲列表 free list；
* 内存碎片问题即是将不可达对象维护到一个列表 free list 里面，这些对象占用的空间为可用空间，随时分配给线程来使用；
* 因此，可以大概推断该占位是用来标记对象是否在列表中。

`age`：一共 4 位，最大值也就是 15：这就决定了为什么**晋升到老年代的年龄**设置（`-XX:MaxTenuringThreshold`，也是已完成的 GC 次数）不能超过 15 的原因。

`pointer_to_heavyweight_monitor`：指向互斥量（重量级锁）的指针。
* 在 Java 中，每个对象都持有一个属于自己的 Monitor 对象（在 Hotspot 虚拟机中由 C++ 类 `ObjectMonitor` 实现），指向它的指针就是 pointer_to_heavyweight_monitor
* 这也正好说明了：在 Java 中，任何一个对象都可以作为锁的存在
* 参数的名字带了 monitor，也是锁被称为“**监视器锁**”的原因。


接下来聊聊 Mark Word 里面很重要的锁标志位。


## 锁标志位
锁标志位 和 是否为偏向锁 两者共同对应到唯一的锁状态。

| 对象状态                |  组合    | 是否为偏向锁 | 锁标志位 |
| :-----:                | :-----: | :-----:    | :-----: |
| 无锁                    | 001     | 0          | 01      |
| 偏向锁                  | 101     | 1          | 01      |
| 轻量级锁（CAS + 失败尝试） | -00     | -          | 00      |
| 重量级锁（互斥）          | -10     | -          | 10      |
| GC 标记                 | -11     | -          | 11      |

所以说 `synchronized` 锁的是**对象**，而且分 4 种程度不同的锁。

锁只能升级不能降级，但是偏向锁可以被重置为无锁状态（也就是改一个位的值的事）。

<br/>

# 类型指针（Klass pointer）
你没看错，就是这个 "klass"。

它用来存储对象的类型指针，该指针指向它所属类的元数据。JVM 通过这个指针来确定这个对象是哪一个类的实例。

举个例子：
```java
Person p = new Person();
// p 是实例，保存在栈中，是对新的 Person() 对象的引用
// Person() 对象在堆中，保存了一个自己的类型指针 klass pointer
// Person() 对象要在元空间 metaspace 找到元数据 instanceKlass，靠的就是 klass pointer
```

该指针的位长度为 JVM 的一个字大小，跟 Mark Word 的大小是一样的：32 位 JVM 的长度位 32 bits，64 位的则是 64 bits。  
如果应用对象过多，使用 64 位的指针会浪费大量的内存。此时为了节约内存，可以开启指针压缩 `-XX:+UseCompressedOops` 将长度压缩为 32 位。

<br/>

# 数组长度
如果对象是一个数组，那么对象头还需要有额外空间用于存储数组长度。

具体长度随 JVM 架构差别而不同，跟 Mark Word 和 klass pointer 的大小也是一样的：32 位 JVM 的长度位 32 bits，64 位的则是 64 bits。  
如果在 64 位 JVM 上开启了指针压缩的话，该区域长度也会被压缩为 32 位。

<br/>

最后问个小问题：

32 位的 Hotspot JVM 中，`Integer` 对象的大小是拆箱 `int` 的几倍？  
解：Mark Word 32 bits，4个字节；类型指针同样 4 个字节；至于实例数据，`Integer` 只有一个 `int` 类型的成员变量 value，大小 4 字节：以上合起来 12 字节，加上补齐，一共 16 字节，是 int 的 4 倍。
