---
title: Java 多线程执行器（Executor 框架）
date: 2021-09-02 00:11:08
tags:
- Java
- 多线程编程
---

Java 绝不只是通过几个零散的基础类来实现多线程编程，就能给并发场景提供强大的保障的。

<!-- more -->

在并发编程的应用场景中，总不能每需要并发管理的时候就去 `new Thread()`，毕竟创建新线程有一定的代价。

如应用程序会有大量生命期很短的线程出现，我们应该使用**线程池**（thread pool）去管理线程：
* 线程池中包含很多准备运行的空闲线程
* 将 `Runnable` 对象交给线程池，就会有一个线程去调用它的 `run` 方法
* run 方法退出后，线程不会死亡，而是在池中准备为下一个请求提供服务
* 能减少并发线程数目

<br/>

Java 线程池相关的接口和类集中在 `java.util.concurrent` 包中。在介绍主要的接口之前，我们先来介绍一下 `Executors` 工厂类：


# `Executors`

Executors 工厂类提供了 `Executor`, `ExecutorService`, `ScheduledExecutorService`, `ThreadFactory` 和 `Callable` 等接口类，及其实现类的工厂方法。

主要的工厂方法概述如下：

| 方法 | 描述 |
| --- | ---- |
| `newCachedThreadPool()`     | 必要时创建新线程：空闲线程保留60秒<br/>如线程池大小超过处理任务所需要线程：回收部分空闲线程 |
| `newFixedThreadPool()`      | 该池包含固定数量的线程：空闲线程一直保留<br/>如某线程因执行异常而结束：线程池补充一个新线程 |
| `newSingleThreadExecutor()` | 只有一个线程的“池”：该线程按照顺序执行每一个提交的任务 |
| `newScheduledThreadPool()`  | 用于定时或周期性执行而构建的固定线程池，替代 `java.util.Timer` |
| `newSingleThreadScheduledThreadExecutor()` | 用于定时或周期性执行而构建的单线程执行器（“池”） |

从上述方法命名可以看出：只有一个线程的“池”，叫 `Executor`；否则都叫 `Pool`。

<br/>

下面说一下常用的接口及相关的工厂方法：


## `ExecutorService`

接口概况：
```java
package java.util.concurrent;


public interface ExecutorService extends Executor {

    // 提交 Callable 对象
    // 返回 Future 对象中包含 Callable 的执行结果
    Future<T> submit(Callable<T> task);

    // 提交 Runnable 对象
    // 通过 Future.get() 获取 Runnable 完成后指定的 result 对象
    Future<T> submit(Runnable task, T result);

    // Future.get() 完成时简单返回 null
    // 提交指定的任务并执行
    Future<?> submit(Runnable task);

    // 关闭服务
    // 先完成已提交的任务，不再接受新任务
    // 当所有任务完成：线程死亡
    void shutdown();

    // 取消尚未开始的所有任务，并试图中断正在运行的线程
    void shutdownNow();

    // 执行给定的任务列表，返回其中一个任务的结果
    // 无法确切知道返回的是哪个任务
    T invokeAny(Collection<Callable<T>> tasks);
    T invokeAny(Collection<Callable<T>> tasks, long timeout, TimeUnit unit);  // 超时则抛出 TimeoutException 异常

    // 执行给定的任务列表，以 List 方式返回所有任务的结果
    List<Future<T>> invokeAll(Collection<Callable<T>> tasks);
    List<Future<T>> invokeAll(Collection<Callable<T>> tasks, long timeout, TineUnit unit);  // 超时则抛出 TimeoutException 异常

    // 发起阻塞，直至所有 task 在请求 shutdown 后完成
    // 若在完成前超时，返回 false
    // 若线程被中断，抛出 InterruptedException
    // 调用 shutdown 之后应该调用此方法
    boolean awaitTermination(long timeout, TimeUnit unit);

    ...
}
```

使用例子：
```java
List<Callable<T>> tasks = ...;
List<Future<T>> results = executor.invokeAll(tasks);
for (Future<T> result : results)
    processFurther(result.get());
```

工厂方法：
```java
package java.util.concurrent;


/**
 * 返回带缓存（线程可重用）线程池，线程数不限
 * 必要时创建线程，在线程空闲 60 秒后终止线程
 * 相当于：new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>())
 */
public static ExecutorService newCachedThreadPool()
public static ExecutorService newCachedThreadPool(ThreadFactory factory)


/**
 * 返回一个线程池，池中的线程数由参数指定
 * 如某一时刻提交的服务数多于空闲的线程数，那将暂时未能获取服务的任务放置到等待池中，其他任务完成后再执行它们
 * 相当于：new ThreadPoolExecutor(threads, threads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
 */
ExecutorService newFixedThreadPool(int threads)
ExecutorService newFixedThreadPool(int threads, ThreadFactory factory)


/**
 * 返回一个执行器，在单个线程中依次执行各个提交进来的任务
 * 相当于：new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
 */
public static ExecutorService newSingleThreadExecutor()
public static ExecutorService newSingleThreadExecutor(ThreadFactory factory)


// 以上构造函数调用的均为 ThreadPoolExecutor 的构造方法
// 返回的是实现了 ExecutorService 接口的 ThreadPoolExecutor 类对象
```


### 实现类概况：`ThreadPoolExecutor`

`ThreadPoolExecutor` 是 Java 并发编程中最常用的线程池执行器。

```java
java.util.concurrent.ThreadPoolExecutor

// 通过扩展 AbstractExecutorService 类来实现 ExecutorService 接口

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue)

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory)

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
// corePoolSize：保留在线程池内的最小线程数（全为 idle 状态也行）
//     当前线程（无论是否在工作）若少于 corePoolSize：新建一个
//     可通过 setter 方法改变
//     可通过 preStartCoreThread() 或 preStartAllCoreThreads() 启动一个或多个线程，
// 使其进入等待状态，成功返回 true
// maximumPoolSize：线程池所允许的最大线程数
//     corePoolSize = maximumPoolSize：固定大小线程池
//     若设为最大整数值：也不是不行
//     可通过 setter 方法改变
// keepAliveTime：超过 corePoolSize 数量的其他 idle 线程所能存活的最长时间
//     可通过 setter 调整
//     也可以设置为最大长整型值（TimeUnit 为纳秒）
//     一般应用至超过 corePoolSize 数目的线程，但 allowCoreThreadTimeOut() 会采用此值
//     注：值为 0 时不会被采用
// workQueue：task 队列，提交的任务会放到这里，只保存 runnable 对象
//     若核心线程数少于 corePoolSize：直接增加一个线程
//     若多于或等于：进入队列；若不能进入队列，则在最大范围内新建
//     方式：
//         1. 直接分发：使用 SynchronousQueue 将 task 立即分发给线程池中的线程
//         2. 无界队列：线程池最多只有 corePoolSize 个线程工作，最大值无效
//         3. 有界队列
//     详见 Java 线程安全集合中的阻塞队列
// handler：因线程池或者队列空间满载导致阻塞时，所要采用的拒绝（reject） handler 对象
//     默认 ThreadPoolExecutor.AbortPolicy：丢弃任务，并抛出 RejectedExecutionException 异常
//     ThreadPoolExecutor.CallerRunsPolicy：如线程池未关闭，则直接在调用者线程完成这个任务
//     ThreadPoolExecutor.DiscardPolicy：不处理新的任务，即丢弃
//     ThreadPoolExecutor.DiscardOldestPolicy：取消队列头（最老）的 task，放入新任务再执行
//     可自定义
// threadFactory：用来创建新线程的工厂对象
//     通过传入指定工厂对象来设置线程的名称、线程组、优先级等
//     null 则采用默认的 Executors.defaultThreadFactory()

void allowCoreThreadTimeOut(boolean value)
// 设置 core 线程是否超时

int getActiveCount()
// 返回当前活跃的线程数

int getLargestPoolSize()
// 返回线程池当前最大的线程数
```

工作流程：
* 如线程池中线程小于 corePoolSize：创建新线程直接执行任务
* 如线程池中线程大于 corePoolSize：暂时将任务存储到 workQueue 中等待执行
* 如 workQueue 也满了：
    * 线程数小于最大线程池数 maximumPoolSize 时会创建新线程处理
    * 线程数大于等于最大线程池数 maximumPoolSize 时会执行拒绝策略

<br/>

## `ScheduledExecutorService`

接口概况：
```java
package java.util.concurrent;


public interface ScheduledExecutorService extends ExecutorService {

    // 预定在指定的等待时间之后执行任务
    ScheduledFuture<V> schedule(Callable<V> task, long time, TimeUnit unit);
    ScheduledFuture<?> schedule(Runnable task, long time, TineUnit unit);


    // 预定在初始延迟结束后，周期性运行给定的任务
    // 周期长度为 period，单位为 unit
    ScheduledFuture<?> scheduleAtFixedRate(Runnable task, long initialDelay, long period, TimeUnit unit);


    // 预定在初始延迟结束后，周期性运行给定的任务
    // 在前一次调用完成和下一次调用开始之间会有 delay 的延迟
    ScheduledFuture<?> scheduleWithFixedDelay(Runnable task, long initialDelay, long delay, TimeUnit unit);
}
```

实现类概况：扩展了 `ThreadPoolExecutor` 类
```java
package java.util.concurrent;

public class ScheduledThreadPoolExecutor
extends ThreadPoolExecutor implements ScheduledExecutorService {

    ...

    public ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory factory) {
        super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS, new DelayedWorkQueue(), factory);
    }

    ...
}
```

相关的工厂方法：
```java
package java.util.concurrent;


/**
 * 返回一个线程池，使用给定线程数调度任务
 * 相当于：new ScheduledThreadPoolExecutor(threads, factory)
 */
ScheduledExecutorService newScheduledThreadPool(int threads)
ScheduledExecutorService newScheduledThreadPool(int threads, ThreadFactory factory)


/**
 * 返回一个执行器，在一个单独线程中调度任务
 * 相当于：new DelegatedScheduledExecutorService(new ScheduledThreadPoolExecutor(1, factory))
 */
ScheduledExecutorService newSingleThreadScheduledExecutor()
ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory factory)
```


## 使用须知
1. 推荐用法：调用 `newCachedThreadPool()` 或 `newFixedThreadPool()`
2. 调用 `submit()` / `execute()` 提交 Runnable 或 Callable 对象
3. 如想取消任务，或如果提交 Callable 对象，则需保存好返回的 Future 对象
4. 当不再提交任何任务时调用 `shutdown()`

线程池的执行方法：
* `execute()`：异步方法，不返回运行结果，用于提交不需要返回值的任务
* `submit()`：异步方法，返回运行结果，用于提交需要返回值的任务
* `invoke()`：同步方法，返回运行结果

（待补充）
