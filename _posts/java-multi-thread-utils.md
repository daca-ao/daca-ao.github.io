---
title: Java 多线程工具类
date: 2021-09-04 23:26:01
tags:
- Java
- 多线程编程
---

本帖再介绍多几个 Java 多线程编程的工具类。

<!-- more -->

CyclicBarrier

CountDownLatch

Semaphore

```java
import java.util.concurrent.Semaphore;

acquire();  // 获取访问许可，获取时并不会加持任何锁
release();  // 释放访问许可
```

Mutex v.s. Semaphore

信号量用于保持在 0 和指定最大值之间的一个计数值
* 当线程完成一次对该信号量对象的等待（wait）时，计数值减一
* 当线程完成一次对该信号量对象的释放（release）时，计数值加一
* 当计数值为 0：线程等待该信号量对象不再能成功，直至该对象变成 signaled 状态
* 对象计数值大于 0：signaled
* 计数值等于 0：nonsignaled

适用于控制一个仅支持有限个用户的共享资源


互斥锁 Mutual exclusion (mutex)： 防止两个以上线程同时对一个公共资源进行读写的机制

互斥与信号量的区别：
1. 互斥仅允许一个线程在某段时间内进入受保护的控制块
2. 信号量则可允许有限个线程同时访问受保护的资源
3. 信号量有受线程控制的变量，互斥锁没有
4. new Semaphore(1); 即 binary semaphore，相当于互斥锁

binary semaphore 与 mutex 的区别：
1. 初始状态不同：binary semaphore 是 0，即unsignaled 状态；mutex 是 1，即可获取的状态
1. 相当于上述区别的 3：mutex 所有权不能被抢夺，而 binary semaphore 必要时会被占用

（待补充）