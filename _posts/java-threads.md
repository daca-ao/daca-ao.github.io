---
title: Java 的线程（Threads）
date: 2021-08-02 23:39:40
tags:
- Java
- 多线程编程
---

工欲善其事，必先利其器。在 Java 开发中，想要做好多线程开发和并发处理，了解 Java 中的线程至关重要。

<!-- more -->

# 线程状态

**1**. **新建**（**New**）

新创建了一个线程对象，但该线程尚未运行（因为还没被调用 start() 方法）


**2**. **运行**（**Runnable**）

当一个新创建的线程对象被调用了 start() 之后：线程便进入了可运行线程池中，进入可运行（runnable）状态，等待获取 CPU 的使用权；
* 该线程可能正在运行（running），也可能没运行（ready），取决于操作系统提供的时间片。


**3**. **阻塞**（**Blocked**）

当线程试图去获取一个内部对象的同步锁（如进入 `synchronized` 块，非 `java.util.concurrent` 的锁），而该锁被其他线程持有的时候，该线程就会因某种原因放弃 CPU 的使用权，从而**暂时停止运行**，直到进入就绪状态。

JVM 会将该线程放入锁池；当所有其他线程释放了这个锁，且线程调度器允许本线程去持有它时，该线程脱离阻塞状态。


**4**. **等待**（**Waiting**）

又称等待阻塞（不知道是谁提出的这个说法）。

处于运行态的线程需要等待另一个线程去通知调度器一个条件（Condition 对象）的时候，该线程进入等待状态。JVM 将该线程放入等待池。

* 进入该状态方法：
```java
Object.wait()
Thread.join()
LockSupport.park()
```


**5**. **计时等待**（**Timed waiting**）

* 线程被调用一些超时计时方法后，会进入计时等待，JVM 将该线程放入等待池。
```java
Object.wait(millis)
Thread.sleep(millis)
Thread.join(millis)
Lock.tryLock(millis, unit)
Condition.await(millis, unit)
LockSupport.parkNanos(blocker, nanos)
LockSupport.parkUntil(blocker, deadline)
```

该状态一直保持到计时结束或接收到其他通知，随后线程会被被切换到 Runnable 状态。

```java
import java.lang.Thread;


void join()    // 等待某个指定的线程执行完毕
// 父线程等到调用该方法的子线程执行完毕才结束

void join(long millis)    // 等待指定的线程被终止或经过指定的时间

Thread.State getState()    // 获得线程状态
/**
 * NEW
 * RUNNABLE
 * BLOCKED
 * WAITING
 * TIMED_WAITING
 * TERMINATED
 */

static void sleep(long millis)    // 休眠给定的毫秒数
// sleep 方法可能抛出 InterruptedException
```

线程处于被阻塞或等待状态：暂时不活动，不运行任何代码，且消耗最少的资源。


**6**. **死亡**（Dead / **Terminated**）
* 线程执行完成（run() 正常退出）：自然死亡
* 未捕获到的异常事件终止了 run()：突然死亡
* 调用 stop() 杀死进程：`@Deprecated`

当一个线程被重新激活的时候，调度器会检查它的优先级是否比当前正在运行的线程更高；如有，则调度器从当前运行的线程中选择一个，剥夺它的运行权，转而执行新进程。

线程不可能长久处于被阻塞的状态中，必须从阻塞态中退出，且返回到就绪状态。
* 如线程处于计时等待态（timed waiting），必须经过规定的毫秒数；
* 如线程调用 `wait()`，另一线程必须调用 `notify()` 或 `notifyAll()`；
* 如线程正等待另一线程拥有的对象锁，另一线程执行完所要做的事情之后，必须放弃该锁的所有权；
* 如线程正在等待输入或输出操作完成（阻塞操作），必须等待该操作完成。

![](java-threads/java-threads.png)

线程在阻塞态和运行态之间的切换：属于线程间通信

```java
boolean isAlive()    // 确定某个线程是否活着
/**
 * 如是就绪线程或阻塞线程：返回 true
 * 如仍是新建线程或死线程：返回 false
 * 
 * 注：
 *  无法确认 alive 线程是 runnable 状态还是 blocked 状态
 *  无法确认 runnable 线程是否已经是 running 状态
 *  无法区分 new 线程和 dead 线程
 */
```

<br/>

# 单个线程的操作

## 已弃用操作

```java
@Deprecated
void suspend()    // 暂停线程
/**
 * 弃用原因：
 * 如果调用该方法挂起一个持有锁的线程，那么这个锁在线程恢复之前不可用；
 * 由此，调用该方法的线程试图获得同一个锁：导致死锁
 *    被挂起的线程等着被恢复；
 *    将其挂起的线程等着获得锁。
 */

@Deprecated
void resume()    // 调用 suspend() 方法后用于恢复线程

@Deprecated
void stop()    // 终止线程（终止所有未结束的方法，包括 run()）
/**
 * 弃用原因：
 * 线程被终止时，会立即释放其持有的所有对象的锁：这样容易导致对象不一致
 * 
 * 由此可见当线程要终止另一个线程时，不知何时调用 stop() 才是安全的
 */
```

因此有一个错误的说法：stop() 会导致某些对象被已停止的线程永久锁定。

勘误：被停止的线程通过抛出 ThreadDeath 异常退出所有同步方法，因此实际上，该线程会释放所有内部锁。


## 中断线程

```java
/**
 * 向线程发送中断请求，请求成功后，该线程中断状态被设为 true
 * 中断状态是每个线程都具有的 boolean 标志，每个线程应随时检查该状态
 * 
 * 如目前该线程正被一个 sleep()/wait() 调用阻塞了，此时调用 interrupt() 会抛出 InterruptedException
 */
void interrupt()


/**
 * 检查当前（正在执行该命令）线程是否被中断（静态方法）
 * 先获取当前线程对象（Thread.currentThread()），再调用该方法
 * 副作用：会清除当前线程的中断状态，将其重置为 false
 */
static boolean interrupted()


/**
 * 测试线程是否被中断
 * 先获取当前线程对象（Thread.currentThread()），再调用该方法
 * 该调用不改变线程中断状态
 *
 * 注：如线程被阻塞，则此时无法检测中断状态，会抛出 InterruptedException
 * 如：循环调用 sleep() —— 需捕获 InterruptedException
 */
boolean isInterrupted()

```

例：
```java
public void run() {
    try {
        ...
        while (loop) {
            doSomething();
            Thread.sleep(millis);
        }
    } catch (InterruptedException e) {
        // 线程 sleep 的时候抛出异常

    } finally {
        // cleanup
    }
}
```

处理 `InterruptedException`：
```java
void subTask() {
    ...
    try {
        sleep(delay);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}

// 或者:
void subTask() throws InterruptedException {
    ...
    sleep(delay);
    ...
}
```

<br/>

# 线程优先级

Java 中的每一个线程均有一个优先级（`int` 类型），其高度依赖于系统：
* Java 的优先级会映射到宿主机平台的优先级上；
* 因此 Sun 提供的虚拟机，线程优先级会被忽略。

每当线程调度器有机会选择新线程时：首选优先级高的线程。  
如有高优先级线程未进入 Runnable 状态，低优先级线程永远不会被执行。

```java
import java.lang.Thread;


static int MIN_PRIORITY = 1;
static int NORM_PRIORITY = 5;
static int MAX_PRIORITY = 10;

void setPriority(int newPriority)    // 设置线程优先级
/**
 * 必须在 Thread.MIN_PRIORITY 和 Thread.MAX_PRIORITY 之间
 * 默认 Thread.NORM_PRIORITY
 * 默认：线程继承父线程的优先级
 */

static void yield()    // 使当前线程处于让步状态
/**
 * 如有其他与当前线程同样优先级的可运行线程，则这些线程接下来会被调度。
 * 线程依旧是可执行（runnable）状态
 * 
 * 以竞争关系抢占资源
 */

void setDaemon(boolean isDaemon)    // 将线程转换为守护线程（daemon thread）或用户线程
/**
 * 为其他线程提供服务，如计时线程
 * 必须在线程启动（start()）前调用
 *
 * 注：守护线程应永远不要去访问固有资源：会在任何时候甚至在一个操作的中间发生中断
 */
```

<br/>

# 线程间通信

线程间通信的相关方法：

```java
import java.lang.Object;

void notifyAll()    // 解除在该对象上调用 wait() 方法的线程的阻塞状态
/**
 * 只能在同步方法或同步块内部使用
 * 如当前线程不是对象锁持有者：抛出 IllegalMonitorStateException
 * 相当于调用 Condition.signalAll()
 */

void notify()    // 随机选择一个在该对象上调用 wait() 方法的线程，解除其阻塞状态
/**
 * 只能在同步方法或同步块内部使用
 * 如当前线程不是对象锁持有者：抛出 IllegalMonitorStateException
 * 相当于调用 Condition.signal()
 */

void wait()    // 将线程置为等待状态，直到线程接到通知
/**
 * 只能在同步方法中使用
 * 如当前线程不是对象锁持有者：抛出 IllegalMonitorStateException
 * 相当于调用 Condition.await()
 */

void wait(long millis)
void wait(long millis, int nanos)
/** 
 * 将线程置为等待状态，直到接到通知或经过指定时间
 * 只能在同步方法中使用
 * 如当前线程不是对象锁持有者：抛出 IllegalMonitorStateException
 * millis：毫秒数；nanos：纳秒数
 */
```

`wait()` v.s. `sleep()`：
* wait 会释放同步锁，sleep 不释放同步锁
* **wait 是 Object 的方法**
    * 可对任意一个对象调用；调用时会导致本线程持有的对象锁被释放，进入等待此对象的等待锁定池
        * 因此其依赖于 `synchronized` 关键字
    * 调用 wait() 时会将线程挂起，直到其他线程调用同一个线程对象的 notify()（或 notifyAll()）
    * 重新被激活后，线程对象会进入对象锁定池（lock pool），准备获得对象锁并进入运行状态
    * 适用范围：等待线程、数据库连接
    * 另：wait(millis) —— 加上了超时时间，等待指定时间后会自动苏醒
* **sleep 是 Thread 的静态方法**
    * sleep(millis)：让线程休眠指定的时间，将执行机会转让给其他线程
    * 但监控状态依然保持，线程会在休眠时间结束时恢复
        * 即：sleep 的控制范围由当前线程决定，其并不依赖于 `synchronized` 关键字；况且它不是 Object 的方法，不能改变对象的内部锁状态
    * 通常用在不需要等待资源情况下的阻塞
* wait 涉及到线程之间的通信问题；而 sleep 主要是线程的运行状态控制

wait(millis) 和 sleep(millis)：
* 都是等待指定时间后自动苏醒，调用 wait 方法的当前线程会释放该同步监视器的锁定（释放锁），可以不用 notify() 或 notifyAll() 方法把它唤醒。

`wait()` / `sleep()` v.s. `yield()`：
* 调用 sleep() 或 wait() 后，线程即进入 blocked 状态
* 调用 yield() 后，线程进入 runnable 状态

`wait()`  v.s. `join()`：
* wait 方法体现线程互斥，join 方法体现线程同步
* wait 方法必须由其他线程调用 notify / notifyAll 方法进行解锁
* join 方法不需要其他线程唤醒：只要被等待线程执行完毕，当前线程自动变为 runnable 状态
* join 方法用途：子线程完成业务逻辑之前，主线程一直等待直到所有子线程执行完毕。


# Java 程序本身就是多线程的进程

只有一个 `main` 方法的 Java 类，通过 jmx 命令查看线程的时候，会有如下结果：

```bash
[4] Signal Dispatcher  # 分发处理发送 JVM 信号的线程
[3] Finalizer  # 调用对象 finalize() 的线程
[2] Reference Handler  # 清除 Reference 的线程
[1] main  # main 线程，用户程序入口
```
