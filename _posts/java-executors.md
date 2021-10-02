---
title: Java 多线程执行器（Executors）
date: 2021-09-02 00:11:08
tags:
- Java
- 多线程编程
---

Java 绝不只是通过几个零散的基础类来实现多线程编程，就能给并发场景提供强大的保障的。

<!-- more -->

创建新线程是有一定代价的
如程序有大量生命期很短的线程，应使用线程池（thread pool）
* 线程池中包含很多准备运行的空闲线程
* 将 Runnable 对象交给线程池，就会有一个线程调用 run 方法
* run 方法退出后，线程不会死亡——在池中准备为下一请求提供服务
* 能减少并发线程数目

# ExecutorService

```java
java.util.concurrent.ExecutorService


Future<T> submit(Callable<T> task)
// 提交 callable 对象
// 返回 Future 对象中包含 callable 的执行结果

Future<T> submit(Runnable task, T result)
// 提交 runnable 对象
// 通过 Future.get() 获取 runnable 完成后指定的 result 对象

Future<?> submit(Runnable task)
// get 方法完成时简单返回 null
// 提交指定的任务执行

void shutdown()
// 关闭服务
// 先完成已提交的任务，不再接受新任务
// 当所有任务完成：线程死亡

void shutdownNow()
// 取消尚未开始的所有任务，并试图中断正在运行的线程

T invokeAny(Collection<Callable<T>> tasks)
T invokeAny(Collection<Callable<T>> tasks, long timeout, TimeUnit unit)
// 执行给定的任务，返回其中一个任务的结果
// 无法知道返回的是哪个任务
// 第二个方法若发生超时，抛出 TimeoutException 异常

List<Future<T>> invokeAll(Collection<Callable<T>> tasks)
List<Future<T>> invokeAll(Collection<Callable<T>> tasks, long timeout, TineUnit unit)
// 执行给定的任务，以 List 方式返回所有任务的结果
// 第二个方法若发生超时，抛出 TimeoutException 异常

boolean awaitTermination(long timeout, TimeUnit unit)
// 发起阻塞，直至所有 task 在请求 shutdown 后完成
// 若在完成前超时，返回 false
// 若线程被中断，抛出 InterruptedException
// 调用 shutdown 之后应该调用此方法

/*
例：
List<Callable<T>> tasks = … ;
List<Future<T>> results = executor.invokeAll(tasks);
for (Future<T> result : results)
    processFurther(result.get());
*/
```


# Executor

```java
import java.util.concurrent.Executors;

public static ExecutorService newCachedThreadPool();
public static ExecutorService newCachedThreadPool(ThreadFactory factory);
// 返回带缓存（线程可重用）线程池
// 该池在必要时创建线程，在线程空闲 60 秒后终止线程
// new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());

ExecutorService newFixedThreadPool(int threads)
ExecutorService newFixedThreadPool(int threads, ThreadFactory factory)
// 返回一个线程池，池中线程数由参数指定
// 如提交服务数多于空闲线程数，那将得不到服务的任务放置到队列中，其他任务完成后再执行它们
// new ThreadPoolExecutor(threads, threads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());

public static ExecutorService newSingleThreadExecutor()
public static ExecutorService newSingleThreadExecutor(ThreadFactory factory)
// 返回一个执行器，在单个线程中依次执行各个提交上来的任务
// new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());


// 注：以上方法调用的均为 ThreadPoolExecutor 的构造方法
// 返回实现了 ExecutorService 接口的 ThreadPoolExecutor 类对象
```

```java
// 以下为预定执行（Scheduled Execution）

ScheduledExecutorService newScheduledThreadPool(int threads)
ScheduledExecutorService newScheduledThreadPool(int threads, ThreadFactory factory)
// 返回一个线程池，使用给定线程数调度任务
// new ScheduledThreadPoolExecutor(threads, factory)


ScheduledExecutorService newSingleThreadScheduledExecutor()
ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory factory)
// 返回一个执行器，在一个单独线程中调度任务
// new DelegatedScheduledExecutorService(new ScheduledThreadPoolExecutor(1, factory))

ScheduledThreadPoolExecutor
Extends ThreadPoolExecutor

ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory factory)
// super(threads, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS, new DelayedWorkQueue<Runnable>(), factory)

java.util.concurrent.ScheduledExecutorService （接口）

ScheduledFuture<V> schedule(Callable<V> task, long time, TimeUnit unit)
ScheduledFuture<?> schedule(Runnable task, long time, TineUnit unit)
// 预定在指定的时间之后执行任务


ScheduledFuture<?> scheduleAtFixedRate(Runnable task, long initialDelay, long period, TimeUnit unit)
// 预定在初始延迟结束后，周期性运行给定的任务
// 周期长度为 period


ScheduledFuture<?> scheduleWithFixedDelay(Runnable task, long initialDelay, long delay, TimeUnit unit)
// 预定在初始延迟结束后，周期性运行给定的任务
// 在一次调用完成和下一次调用开始之间有 delay 的延迟

```

使用线程池须知：
1. 推荐用法：调用 Executors 类中静态的方法 newCachedThreadPool 或newFixedThreadPool
2. 调用 submit / execute 提交 Runnable 或 Callable 对象
3. 如想取消任务，或如果提交 Callable 对象，则需保存好返回的 Future 对象
4. 当不再提交任何任务时调用 shutdown

线程池的执行方法：
* submit: 异步方法，返回运行结果
* execute: 异步方法，不返回运行结果
* invoke: 同步方法，返回运行结果

（待补充）
