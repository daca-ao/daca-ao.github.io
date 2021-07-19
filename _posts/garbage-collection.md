---
title: JVM 垃圾回收机制
date: 2021-07-09 22:44:14
tags:
- Java
- JVM
- GC
---

Java 在发展的过程中，摒弃 C++ 中一些繁琐的容易出错的东西，引入了“计数器”的概念。
其中一个是垃圾回收（GC, Garbage Collection）机制：

<!-- more -->

* 不定时回收垃圾对象所占用的内存，内存泄漏的隐患大大降低

当对象失去所有的引用，变成了“孤魂野鬼”，也就是“垃圾”
例：垃圾对象诞生过程
```java
String str1 = new String("abc"); // 这个对象有一个引用str1指向它
String str2 = str1; // 这个对象现在有两个引用：str1，str2指向它
str1 = null; // 失去引用str1
str2 = null; // 失去引用str2，失去全部引用，变成野对象

// 由此 new String("abc"); 产生的对象失去了所有引用，将无法再被程序访问。
```

C / C++ 中没有垃圾回收机制：
* 除非程序结束，堆中申请后未被释放且无指向的内存会一直处于“三无”状态
    * 无法释放
    * 无法调用
    * 无法更改
* 因其无法再被访问，当此类内存多起来时，可利用的内存越来越少
* 导致内存泄漏（memory leak）

如下为一次内存泄漏过程：
```c++
(int *) malloc (sizeof(int)*1024);
// 内存分配函数的参数没有问题，指针的转型也没有问题，But没有任何指针指向它！
```

JVM 的垃圾回收机制包括：
* 在程序运行过程中标记垃圾对象
* 以及在 GC 过程中回收被标记的垃圾对象

Java 的垃圾回收机制是 JVM 提供的能力
* 关注的是堆内存
* 用于在空闲时间以不定时的方式动态回收无任何引用的对象占据的内存空间
* Java 语言本身没有提供释放已分配内存显示操作方法
* 所以 Java 的内存管理实际上就是对象的管理
    * 包括对象的分配和释放

注：垃圾回收回收的是无任何引用的对象占据的内存空间，而不是内存本身

方法：自动检测对象是否超过作用域从而达到自动回收内存的目的

JVM GC 算法为分代收集，设计为：
1. 频繁收集新生代
2. 较少收集老年代
3. 更少收集永久区

YG 回收（Minor GC, young GC）执行过程
* 触发：Eden 区没有足够空间——对象优先在新生代分配
1. Eden + From Survivor 区存活的对象复制到 To Survivor
2. 若 Survivor 区满，则将 Survivor 区内的存活对象转移到老年代
3. 清空 Eden 和 From Survivor
4. From Survivor 和 To Survivor 交换
5. 若老年代满，则抛出 OutOfMemoryException

OG 回收（CMS 中特有）
1. 收集老年代的垃圾
2. 在 Major GC 的过程中，至少进行了一次 Minor GC
3. 一般以 Minor GC 慢 10 倍

Full GC（Major GC）
* 对于整个堆的操作
* STOP THE WORLD
* 触发条件：
    * 晋升时，对象大小大于老年代可用大小
    * 老年代连续空间不足
    * Metaspace 达到阈值
    * System.gc()
* 老年代执行之后空间仍不足：抛出 OOM: Java heap space
* 永久区执行之后空间仍不足：抛出 OOM: Metaspace space

OG 存储对象：
* 新生代对象经过 N 次 GC 晋升到老年代。默认 N = 15
* 大对象（由 JVM 参数决定）直接存储到老年代，如数组

```java
System.gc(); // 请求运行垃圾回收器触发 Full GC，不一定真的执行了垃圾回收，且不受代码控制

Runtime.getRuntime().gc();
```

以上调用用于显式通知 JVM 可以进行一次垃圾回收

但真正的垃圾回收机制，具体在什么时间点开始发生动作，不可预料
* 这和抢占式的线程在发生作用时的原理是一样的

注：不能通过 Java 代码干预 Java 垃圾回收

标记垃圾

计数法
* 新建一个对象或有引用指向该对象时计数器 + 1
* 当一个对象引用超过生存期限或指向新对象时计数器 + 1
* 计数器对应的对象变为 0 时回收该对象
* 快、简单，但无法检测循环引用
```java
class A {public B b;}
class B {public A a;}
public static void main(String[] args) {
    A a = new A();
    B b = new B();
    a.b = b;
    b.a = a;
}
```

跟踪法/根搜索算法
* 从图论引入
* 将所有引用关系看作一张图，从根节点开始寻找对应的引用节点
* 以 GC roots 为起点形成作用链，每个对象为有向图的顶点
    * GC roots 不可达的对象将被回收
* GC roots 包括：
    * 虚拟机栈中引用的对象
User user = new User();
    * 静态属性引用的对象
private static User user = new User();
    * 常量引用的对象
private static final User user = new User();
    * 本地方法（native method）栈中引用的对象
