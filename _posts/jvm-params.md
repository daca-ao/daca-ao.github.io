---
title: JVM 常用参数
date: 2021-07-11 20:34:26
tags:
- Java
- JVM
---

JVM 的具体实现和调优纷繁复杂，本帖主要记录一些开发和测试过程中经常用到的一些参数。

<!-- more -->

至于更详细的参数介绍和配置，可以参考[官方文档](https://docs.oracle.com/cd/E22289_01/html/821-1274/configuring-the-default-jvm-and-java-arguments.html
)来学习。

<br/>

JVM 的启动参数共分为三类：
1. 标准参数（-）：所有的 JVM 实现都必须实现这些参数的功能，而且向后兼容；
2. 非标准参数（-X）：默认的 JVM 实现这些参数的功能，但是并不保证所有 JVM 的实现都满足，且不保证向后兼容；
3. 非 Stable 参数（-XX）：此类参数各个 JVM 的实现会有所不同，将来可能会随时取消，需要慎重使用。

<br/>

# 修改虚拟机栈大小

```bash
-Xss<heap_size>[unit]  # 设置 JVM 栈的大小。每个线程设置一个值，一般设置几百 k
```


# 修改堆的大小

```bash
-Xmx<heap_size>[unit]  # 设置堆的最大值
-Xms<heap_size>[unit]  # 设置初始化堆的值

# example:
-Xmx5g
-Xms512m
```
注：JVM 启动后并不会将堆扩展到指定最大值，而是：
1. 先开辟指定的最小值；
2. 如经过数次 GC 之后仍不能满足程序执行：逐步扩容，而不是立即扩容到最大值。

具体到修改堆中某些区域的指令：
```bash
# 仅限 JDK 1.3 及 1.4 使用
-XX:NewSize=<young_gen_size>[unit]  # 设置 YoungGen 的大小，最小值默认是 1310 MB
-XX:MaxNewSize=<max_young_gen_size>[unit]  # 设置 YoungGen 的最大值，默认没有限制

-Xmn<heap_size>[unit]  # 设置新生代 YoungGen 的内存大小，JDK 1.4 及之后使用


-XX:PermSize=<perm_gen_size>[unit]  # 设置初始永久区的大小
-XX:MaxPermSize=<perm_gen_size>[unit]  # 设置永久区的最大值
# 注：因永久区在 Java 8 已被取消，上述两个参数仅适用于 Java 7 及之前版本

# 仅适用于 Java 8 及以后版本
-XX:MaxMetaspaceSize=<max_metaspace_size>[unit]  # 设置 Metaspace 元空间最大值，默认没有限制
# 设置了值之后，每当元空间达到阈值之后，方法区开始内存回收。

-XX:SurvivorRatio=<ratio>  # 设置 Survivor 区与 Eden 区的比例，默认为 0.25
# 如: -XX:SurvivorRatio=0.25

-XX:NewRatio=<ratio>  # 设置 YoungGen 区与 OldGen 区的比例，默认比例为 1:2
# 如：-XX:NewRatio=0.25
```


# GC 相关

可调用的 GC：
```bash
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+UseParallelOldGC
-XX:+USeParNewGC
-XX:+UseConcMarkSweepGC
-XX:+UseG1GC
```

GC 输出：
```bash
verbose:gc  # 启动 JVM 时，输出 JVM 里面的 GC 信息
# 输出：
# [Full GC] 178K->99K(1984K), 0.0253877 secs]

# 解释：
# Full GC：执行了一次 Full GC
# 178K, 99K：执行 GC 前后内存容量
# 1984K：内存总容量
# ... secs：执行本次 GC 耗时

-XX:+printGC  # 同上
-XX:+PrintHeapAtGC  # 每一次 GC 发生前后都打印堆的信息，格式同上
-XX:+PrintReferenceGC  # 打印对象引用信息
```

```bash
-XX:+PrintGCDetails  # 打印 GC 的详细信息

# 输出：
–Heap
– def new generation   total 13824K, used 11223K [0x27e80000, 0x28d80000, 0x28d80000)
–  eden space 12288K,  91% used [0x27e80000, 0x28975f20, 0x28a80000)
–  from space 1536K,   0% used [0x28a80000, 0x28a80000, 0x28c00000)
–  to   space 1536K,   0% used [0x28c00000, 0x28c00000, 0x28d80000)
– tenured generation   total 5120K, used 0K [0x28d80000, 0x29280000, 0x34680000)
–   the space 5120K,   0% used [0x28d80000, 0x28d80000, 0x28d80200, 0x29280000)
– compacting perm gen  total 12288K, used 142K [0x34680000, 0x35280000, 0x38680000)
–   the space 12288K,   1% used [0x34680000, 0x346a3a90, 0x346a3c00, 0x35280000)
–    ro space 10240K,  44% used [0x38680000, 0x38af73f0, 0x38af7400, 0x39080000)
–    rw space 12288K,  52% used [0x39080000, 0x396cdd28, 0x396cde00, 0x39c80000)
```
解析：
* `new generation` 是新生代
* `tenured generation` 是老年代
* `compacting perm gen` 是永久区
* 三个地址值分别为该片内存的起始点，当前使用到的地方，和最大的内存地点

```bash
-X:loggc:log/gc.log  # 指定输出 gc.log 的文件位置

-XX:+PrintGCTimeStamps  # 输出 GC 发生的时间
-XX:+PrintGCApplicationStoppedTime  # GC 产生停顿的时间
-XX:+PrintGCApplicationConcurrentTime  # GC 应用执行的时间
```


# 其他

```bash
-XX:StringTableSize=66666  # 设定 StringTable 的长度

-XX:+UseStringDeduplication  # 去掉重复字符串
# 注意：
# 只适用于 G1 收集器，且只适用于长期存活的对象
# 默认：一个字符串经过 3 次 GC 后还存活才会被列为去重的候选
# 可能会增加 GC 停顿时间
# 不会消除重复字符串的引用本身，只会替换底层的 char[]，即：aStr.value = anotherStr.value

-XX:StringDeduplicationAgeThreshold=6  # 设置经历 GC 的次数

-XX:+DoEscapeAnalysis  # 开启逃逸分析（Server 模式），默认开启
-XX:-DoEscapeAnalysis  # 关闭逃逸分析
-XX:+PrintEscapeAnalysis  # 显示分析结果
-XX:+EliminateAllocations  # 开启标量替换（默认开启）
-XX:+TraceClassLoading  # 跟踪类的加载
-XX:+TraceClassUnloading  # 跟踪类的卸载

-XX:+PrintReferenceGC  # 打印对象引用信息
-XX:+PrintVMOptions  # 打印虚拟机参数
-XX:+PrintCommandLineFlags  # 打印虚拟机显式和隐式参数
-XX:+PrintFlagsFinal  # 打印所有系统参数
-XX:+PrintTLAB  # 打印 TLAB 相关分配信息
-XX:+UseTLAB  # 打开 TLAB，默认开启
-XX:TLABSize  # 设置 TLAB 大小
-XX:+ResizeTLAB  # 自动调整 TLAB 大小
-XX:TLABWasteTargetPercent  # 设置 TLAB 所占用 Eden 空间的百分比
-XX:+DisableExplicitGC  # 禁用显式 GC（System.gc()）
-XX:+ExplicitGCInvokesConcurrent  # 使用并发方式处理显式 GC
```
