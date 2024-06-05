---
title: Java 常用关键字（keywords）整理
date: 2021-07-14 00:14:48
tags:
- Java
---

本帖用来整理一些常见的 Java 语言中的关键字（保留字）。

<!-- more -->

# abstract
* 修饰抽象类或方法

<br/>

# assert
断言（Since Java 4），用于查找内部程序错误

<br/>

# final
final 关键字在 Java 中被称为“完结器”，表示最终的意思。  
它能声明类、方法、属性（方法参数和变量）。

    Java classes declared as final cannot be extended. 
final 声明的类不能被继承。

    Methods declared as final cannot be overridden. 
final 声明的方法不能被重写，但不影响被继承。

    The variable declared as final should be initialized only once and cannot be changed. 

final 声明的变量**只能被赋值一次**，不可被修改（Java 虚拟机为变量设定的默认值，并不计作一次赋值）。
* 修饰基本类型：值不可变，即常量
* 修饰对象：该变量被赋予的引用不可变

对于被 final 修饰的对象，不可改变的只是该变量所保存的引用，并不是该引用指向的对象：
```java
final StringBuilder s = new StringBuilder();  // final 修饰的是 s，即 s 保存的地址不能变
s.append("a");    // 无误，s 指向的对象是可以被修改的
s = null;         // 报错，s 的值不能被修改
```

被 final 修饰的变量必须要被初始化：
* 定义时初始化
* 初始化块中初始化
* 类的构造器（Constructor）中初始化
* 静态变量也可在静态初始化中初始化

注：使用 final 修饰的共享数据结构，**在它的对象构造函数完成后才能对其他线程可见（其他线程在其初始化完成后才能看到这个对象）**。

一个小的知识点：一个被 final 定义并同时指定了初始值，且初始值为编译时就被确定的变量，被称作**宏变量**。编译器会将程序所有用到该变量的地方直接替换成该变量的值，这叫做**宏替换**。
```java
final String a = "hello";    // 宏变量
final String b = a;          // 非宏变量，编译时会替换成 b = "hello"
```

<br/>

## final 的内存语义

限定了一些指令重排序：
* 在构造函数里面对 final 域的写操作，和随后将这个被构造出来的对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
* 初次读一个包含 final 域的对象的引用，与随后初次读这个 final 域，这两个操作之间不能重排序。

```java
class FinalExample {
    int i;
    final int j;
    static FinalExample obj;

    public FinalExample() {
        i = 1;  // 写普通域
        j = 2;  // 写 final 域
    }

    public static void writer() {
        obj = new FinalExample();  // 先创建，再赋值
    }

    public static void reader() {
        FinalExample object = obj;  // 1
        int a = object.i;  // 2. 不会和 object 被赋值重排序：有数据依赖
        int b = object.j;  // 3. 不会和 object 被赋值重排序：有数据依赖
    }
}
```

对于 final 域，JMM 禁止编译器将 final 域的写操作重排序到构造函数之外。编译器通过在 final 域的写之后，构造函数 return 之前，插入一个 StoreStore 屏障来实现。

假设有线程 A 首先去执行 `writer()`，紧接着线程 B 执行 `reader()` 方法。时间片如下：

| 时间片 Time  | 线程 A             | 线程 B              |
| :---------: | -----             | -----              |
| T1          | 构造函数开始执行     |                    |
| T2          | 写 final 域 j = 2; |                    |
|             | StoreStore 屏障    |                    |
|             | 构造函数执行结束     |                    |
| T3          | 将构造函数执行所返回对象的那个引用，赋值给引用对象 obj |  |
| T4          |                   | 该对象引用 obj       |
| **T5**      |                   | 读对象的普通域 i      |
| T6          |                   | 读对象的 final 域 j  |
| **T7**      | 写普通域 i = 1;    |                     |

如上，因为变量 i 不是 final 域，编译的时候会被优化到构造函数之外，线程 A 在执行的过程中就会出问题。  
而对于 final 域 j，因为刚刚提到，需要等到它**所属的对象构造函数完成后，final 域才能对其他线程可见**。所以 final 域的写操作不会被重排序到构造函数之外。

讨论读 final 域的内存语义时，我们再来举同样的例子，假设时间片如下：

| 时间片 Time  | 线程 A             | 线程 B              |
| :---------: | -----             | -----              |
| T1          | 构造函数开始执行     |                    |
| **T2**      |                   | 读对象的普通域 i      |
| **T3**      | 写普通域 i = 1;    |                     |
| T4          | 写 final 域 j = 2; |                    |
|             | StoreStore 屏障    |                    |
|             | 构造函数执行结束     |                    |
| T5          | 将构造函数执行所返回对象的那个引用，赋值给引用对象 obj |  |
| T6          |                   | 该对象引用 obj       |
|             |                   | LoadLoad 屏障       |
| T7          |                   | 读对象的 final 域 j  |

对于线程 B 访问 reader()，绝大多数处理器不会对 1 和 2，或者 1 和 3 重排序，但是极少数处理器仍然会进行重排序。  
对于 final 域的读操作，JMM 在读引用与读 final 域之间**插入 LoadLoad 屏障**。即读一个 final 域不能被重排序到读对象引用之前。  
而非 final 域就容易出问题。


## 实现 final 的可见性

要实现 final 域初始化后才能被其他线程可见，需要保证构造函数内部的对于被构造对象的引用不能被其他线程看到；也就是说，对象引用（`this`）不能在构造函数中“逸出”。

举一个错误的例子：
```java
class FinalReferenceEscapeExample {
    final int i;
    static FinalReferenceEscapeExample obj;

    public FinalReferenceEscapeExample() {
        i = 1;  // 1. 写 final 域
        obj = this;  // 2. this 引用在此逸出
    }

    public static void writer() {
        new FinalReferenceEscapeExample();
    }

    public static void reader() {
        if (obj != null) {  // 3
            int temp = obj.i;  // 4
        }
    }
}
```

还是一样的假设，线程 A 首先去执行 `writer()`，紧接着线程 B 执行 `reader()` 方法。时间片如下：

| 时间片 Time  | 线程 A       | 线程 B             |
| :---------: | -----       | -----              |
| T1          | 构造函数开始  |                    |
| **T2**      | obj = this;<br/>2. 被构造对象的引用在这里“逸出” |                     |
| T3          |             | if (obj != null)<br/>3. 读取不为 null 的对象引用      |
| T4          |             | int temp = obj.i;<br/>4. 读取了 final 域初始化之前的值 |
| T5          | i = 1;<br/>1. 对 final 域初始化              |                     |
|             | 构造函数结束  |                    |

因此 obj 会在对象还没完成初始化的时候被线程 B handle 了，导致应该被其他线程可见的 final 域初始化反而不可见。

小结：只要对象是正确构造的（被构造对象的引用在构造函数中没有“逸出”），则不需要使用同步（synchronized 或 volatile）就可以保证任意线程都能看到这个 final 域在构造函数中被初始化之后的值。

这是经过 JSR-133 增强过的语义，因为在旧的 JMM 中，一个比较严重的问题是，线程可能会看到 final 域的改变，这会导致 String 对象的值可能会被改变。
```java
public final class String implements ... {
    private final char value[];  // 在旧的 JMM 中可能会被改变
    ...
}
```

<br/>

# finally

只能被用在 `try` / `catch` 语句中

    The finally block always executes when the try block exits. This ensures that the finally block is executed even if an unexpected exception occurs. 

`return`, `continue`, `break` 都不能阻止 finally 语句块的执行
* `return`, `continue`, `break` 所在语句块的内容会先输出，然后轮到 finally
* 但 finally 块是在他们被执行之前执行的

<br/>

# finalize

`finalize()` 方法为 Object 类的方法：

    Before an object is garbage collected, the runtime system calls its finalize() method. 

对象在被回收之前，其 finalize() 方法会被调用，但只会被调用一次。

<br/>

# static

* 修饰静态类、类成员方法、类成员变量以及 static 静态代码块
* 不能修饰局部变量

static 变量与普通变量区别：也就是**类**与**对象**层面的区别
* 所属目标不同：静态变量属于类的变量，普通变量属于对象的变量
* 存储区域不同：静态变量存储在方法区的静态区，普通变量存储在堆
* 加载时间不同：
    * 静态变量随着类的加载而加载，随着类的消失而消失
    * 普通变量随着对象的加载而加载，随着对象的消失而消失
* 调用方式不同：静态变量只能通过类名调用，普通变量只能通过对象调用

<br/>

# synchronized 

用来修饰一个方法或者一个代码块，能够保证在同一时刻最多只有一个线程执行这段代码，相当于让线程拥有了一把“锁”。

使用 synchronized 去同步方法，利用的是 Java 对象的对象锁（`Object.lock()`）来保护整个方法。
* 可声明静态方法

应用背景（多线程修改同一变量）：
```java
public class Test {
    int x = 0;
    public void inc() {
        x++;
    }
}


// 测试代码
Test test = new Test();
new Thread(() -> {
    for (int i = 0; i < 10000; i++)
        test.inc();
}).start();

new Thread(() -> {
    for (int i = 0; i < 10000; i++)
        test.inc();
}).start();

new Thread(() -> {
    for (int i = 0; i < 10000; i++)
        test.inc();
}).start();
```

注：三个线程执行完毕后：`test.x` 的值不一定就是 `30000`，有可能是 `[10000, 30000]` 之间的一个值。

这是因为，由于三个线程在同时访问一个变量，每个线程中 x 的自增操作**并不是原子性的**，它会被分解为：
1. 从主内存中将 `x` 的值读取到该线程的工作内存中；
2. `x++`；
3. 将 `x++` 的值写回工作内存及主内存。

然而，实际的情况可能是：（以 `x = 0` 为例）
1. `x = 0`
2. T1 从主内存读取 `x` 的值并写入其工作内存
3. T1 计算 `x` 的值为 1
4. T2 从主内存读取 `x` 的值并写入其工作内存，此时 T2 读取到的 `x` 的值**仍为 0**
5. T2 计算 `x` 的值仍为 1
6. T1 将 `x = 1` 写入工作内存及主内存
7. T2 将 `x = 1` 写入工作内存及主内存
8. 事实上 T1 和 T2 一共计算了**两次**，但 `x` 的值只增加了 1

<br/>

## 关键字的应用


### 同步某个方法

```java
public synchronized 数据返回类型 方法名(参数) { ... }
```
使用 synchronized 修饰的某个方法，被称为**同步方法**。

同步方法无需显式地指定同步监视器（Monitor），因为同步方法的 Monitor 是 this：也就是调用该同步方法的对象。
* 可将某类变成线程安全的类 —— 该类相当于 Monitor
* 线程需获得内部对象锁，才能真正调用这个方法；而且锁只会加在访问同一个对象的这个方法的那些线程上
    * 比如类 Klass 有一个同步方法，分别有两个对象 klassA 和 klassB，调用它们俩的那个同步方法所获得的对象锁就不是一回事。


所以，上述应用背景可作如下改进：
```java
public class Test {
    int x = 0;
    public synchronized void inc() {
        x++;
    }
}

// 测试代码
Test test = new Test(), test2 = new Test();

Thread t1 = new Thread(() -> {
    for(int i = 0; i < 10000; i++)
        test.inc();
});

Thread t2 = new Thread(() -> {
    for(int i = 0; i < 10000; i++)
        test.inc();
});

Thread t3 = new Thread(() -> {
    for(int i = 0; i < 10000; i++)
        test2.inc();
});

Thread t4 = new Thread(() -> {
    for(int i = 0; i < 10000; i++)
        test2.inc();
});

t1.start();
t2.start();
t3.start();
t4.start();
// t1, t2 共享 test.inc() 的锁，t3, t4 共享 test2.inc() 的锁
```

* 所有访问同一 `test` 对象 inc() 方法的线程共用一把锁
* 其他线程如果访问其他对象的 inc() 方法，使用的是另外的锁


`synchronized` 修饰静态方法：
```java
public class Test {
    static int x = 0;
    public static synchronized void inc() {
        x++;
    }
}

// 测试代码
Thread t1 = new Thread(() -> {
    for(int i = 0; i < 10000; i++)
        Test.inc();
});

Thread t2 = new Thread(() -> {
    for(int i = 0; i < 10000; i++)
        Test.inc();
});

Thread t3 = new Thread(() -> {
    for(int i = 0; i < 10000; i++)
        Test.inc();
});

Thread t4 = new Thread(() -> {
    for(int i = 0; i < 10000; i++)
        Test.inc();
});

t1.start();
t2.start();
t3.start();
t4.start();
// 以上，所有线程共享 Test.inc() 的锁
```

因为静态方法是独立于对象的存在，所以所有访问该静态方法的线程共享一把锁。

<br/>

### 同步代码块（同步阻塞）

```java
synchronized (同一个数据) { ... } 
```

这里说的“同一个数据”，是指多线程并发访问的数据，暂且称为**锁芯**。

通过锁住某一块代码，比锁住整个方法更有效率；因为这样能同时允许多个线程进入到方法中，进行那些无需上锁的操作。

锁芯可以为固定的 this 对象或类的 class 对象，也可更换为其他对象，但不能为 null。
* 原因：锁芯应该是所有访问该方法的线程均能访问到的同一个对象，这样才能确保其线程互斥
* 所以选用 `this`、`class` 及其他 `final` 对象能防止锁芯被更换，从而避免各线程获得的对象锁不一样。


**1. 普通方法中的同步代码块：**
```java
private static int x = 0;
public void inc() {
    synchronized (this) {
        x++;
    }
}
// 这是代码块范围为整个方法里的代码，且锁住 this 对象
// 等价于 public synchronized void inc() { x++; }

private static int x = 0;
public void inc() {
    synchronized (Test.class) {
        x++;
    }
}

private final Object lock = new Object();  // 注意这个 final
private static int x = 0;
public void inc() {
    synchronized (lock) {
        x++;
    }
}
```

**2. 静态方法中的同步代码块：**

静态方法通常只与**类**有关，凌驾于该类的任何对象之上；这时，锁芯通常由所有直接或间接调用该类**静态方法**的线程共享。

所以，此时的锁芯显然**不能是 this 对象**或任何非静态对象，只能是静态对象或类的 class 对象。
* 通常添加 `final` 关键字

```java
private static int x = 0;
public static void inc() {
    synchronized (Test.class) {
        x++;
    }
}
// 等价于 public static synchronized void inc() { x++; }


private final static Object lock = new Object();
private final static int x = 0;
public static void inc() {
    synchronized (lock) {
        x++;
    }
}
```

内部对象锁只有一个相关条件，其局限性为：
* 不能中断一个正在试图获得锁的线程
* 试图获得锁时不能设定超时
* 每个锁仅有单一条件


**另：客户端锁定（client-side locking）**：锁定某对象，将读写操作放入同一代码块中
```java
public void transfer(Vector<Double> accounts, int from, int to, int amount) {
    synchronized (accounts) {
        accounts.set(from, accounts.get(from) - amount);
        accounts.set(to, accounts.get(to) + amount);
    }
    System.out.println("...");
}
```
十分脆弱，不推荐使用。

注：`synchronized` 修饰的方法在子类中不会继承同步性
* 即：多个线程调用该子类所继承的父类的方法时，不会加锁
* 除非重写并加上关键字

线程同步代码块通用规则：  
（以下规则以 synchronized 关键字为例，对其他对象锁同样适用）
1. 两个并发线程访问同一对象 object 中的 `synchronized(this)` 代码块时：**一个时间内只能有一个线程得到执行**。另一线程必须等待当前线程执行完这个代码块以后才能执行该代码块。
2. 然而，当一个线程访问 object 的一个 `synchronized(this)` 代码块时，其他线程仍可以访问该 object 中的**非** synchronized(this) 代码块。
3. 关键：当一个线程访问 object 的一个 `synchronized(this)` 代码块时，其他线程对 object 中所有其他 synchronized(this) 代码块的访问将被**阻塞**。
4. 一个线程访问 object 的一个 `synchronized(this)` 代码块，便获得了该 object 的对象锁；其他线程对该 object 所有同步代码部分的访问都被暂时阻塞。

<br/>

## synchronized 原理：**可重入性**

基本的实现原理是基于进入和退出管程（Monitor）对象来实现同步的，monitor 对象通过计数器实现可重入性。在实现同步方法和同步代码块这两者之间有一些实现细节不一样。

**同步方法**：
```java
public synchronized void method() { doSomething(); }
```

等同于
```java
public void method() {
    this.intrinsicLock.lock();  // this 对象的锁
    try {
        doSomething();
    } finally {
        this.intrinsicLock.unlock();
    }
}
```

编译后产生的指令集中，该同步方法对应指令中的方法表结构里面会设置 `ACC_SYNCHRONIZED` 标志：
```
public synchronized void syncTask();
    descriptor: ()V
    flags: (0x0021) ACC_PUBLIC, ACC_SYNCHRONIZED
    ...

```

当方法被调用时，调用指令会去检查 `ACC_SYNCHRONIZED` 标志，有的话，执行到这里的这个线程就会持有同步方法所在对象的 monitor 对象，monitor 对象的计数器加 1；执行完了方法之后再释放，计数器减 1.

<br/>

**同步代码块**的原理，在于编译之后，字节码文件会对应生成 `monitorenter` 和 `monitorexit` 两条指令，两者所在行号之间是代码块所要执行的方法被编译过后的指令：
```
...
 3: monitorenter
 4: getstatic     #2
 7: 1dc           #3
 9: invokevirtual #4
12: aload_1
13: monitorexit
...
```

某个线程执行同步代码块的时候，就需要获取锁，也就是获得该代码块所在对象所持有的 Monitor 对象（监视器锁）的持有权。  
根据以上例子可知，`monitorenter` 是线程获得锁的指令，`monitorexit` 是线程释放锁的指令。

了解了 synchronized 的原理，我们就能知道，一个线程从一个对象的 synchronized 方法调用相同对象的另一个 synchronized 方法，线程在执行两个方法时获得的是**一把**对象锁：
```java
synchronized void m1(String t) {
    System.out.println(t + "m1");
    m2(t);
}

synchronized void m2(String t) {
    System.out.println(t + "m2");
}
```
* 即：调用同一对象的 m1 或 m2 方法时，所有线程共享一把锁
* 锁被获得（进入）了两次，monitor 对象被持有了两次，记数 2
* 进入该对象 m1 的线程在进入 m2 时可直接获得该锁，这也体现了“可重入性”
* 锁被释放一次，即 monitor 对象被释放一次，计数器就减 1，直到计数器为 0，锁就被释放掉了，先到先得。

注意：
* 非持有锁的线程在计数器大于 0 的时候访问该对象，线程只能等待
* 可重入的代码不能使用静态或全局变量作为锁芯，同时不会调用不可重用的代码
* 当 synchronized 方法 A 调用某方法 B，B 没有上锁，且可被其他线程直接调用
    * 等于开了免锁操作 B 方法的后门，造成线程不安全
* 调用不同对象的 synchronized 方法：线程不共享对象锁
```java
public class SyncTest {
    synchronized void m1(String t) {
        System.out.println(t + "m1");
        m2(t);
    }

    synchronized void m2(String t) {
        System.out.println(t + "m2");
    }

    public static void main(String[] args) {
        SyncTest t = new SyncTest();
        SyncTest t2 = new SyncTest();

        new Thread(() -> {
            t.m1(Thread.currentThread().getName());
        }).start();

        new Thread(() -> {
            t.m1(Thread.currentThread().getName());
        }).start();

        new Thread(() -> {
            t.m1(Thread.currentThread().getName());
        }).start();

        new Thread(() -> {
            t2.m1(Thread.currentThread().getName());
        }).start();
    }
}

/* 
   可能的情况：
    Thread-0m1
    Thread-0m2
    Thread-1m1
    Thread-1m2
    Thread-3m1    // t2对象的锁：这是另一把锁
    Thread-3m2    // 另一把锁
    Thread-2m1
    Thread-2m2
 */
```


## synchronized 锁的级别

在 Java 6 之前，synchronized 使用的是重量级锁。  
在此版本之后进行了优化，拥有了**无锁** -> **偏向锁** -> **轻量级锁** -> **重量级锁**的升级过程。

锁的升级是针对于不同同步场景进行的优化：
1. 在不存在锁竞争的时候，线程进入同步方法或代码块，使用偏向锁
2. 存在竞争时升级为轻量级锁，其采用自旋锁的实现
3. 如果同步方法或代码块执行时间很短的话，采用轻量级锁虽然会占用 CPU 资源，但是相对比使用重量级锁还是更高效的；
4. 但是如果同步方法或代码块执行的时间很长，则使用轻量级锁自旋带来的性能消耗就远比使用重量级锁要大，这时候就需要升级为重量级锁了。


## synchronized 在 JMM 的语义

通过锁的**释放**和**获取**而建立起来的 happens-before 关系（监视器锁规则）。

假如有如下代码，俩线程 A 和 B 分别访问 writer() 和 reader()：
```java
class MonitorExample {
    int a = 0;

    public synchronized void writer() {  // 1
        a++;  // 2
    }  // 3

    public synchronized void reader() {  // 4
        int i = a * a;  // 5
        ... 
    }  // 6
}
```

| 时间片 Time  | 线程 A             | 线程 B            |
| :---------: | -----              | -----            |
| T1          | 1. 获得锁           |                  |
| T2          | 2. 执行临界区中的代码 |                  |
| T3          | 3. 释放锁           |                  |
| T4          |                    | 4. 获得同一个锁    |
| T5          |                    | 5. 执行临界区的代码 |
| T6          |                    | 6. 释放锁         |

程序顺序规则（1 + 2 + 3，4 + 5 + 6）和监视器锁规则结合之后，提供的 happens-before 保证，就是 2 happens-before 5。

释放锁对应的内存语义：将加锁操作（临界区代码）对本地内存中共享变量的修改**写回**主内存的共享变量中。  
实质上是线程向接下来将要获取这个锁的某个线程**发出了消息**，消息是该线程对共享变量所做的修改。

获取锁对应的内存语义：先将本地内存中的共享变量**置为无效**，再去主内存中**读取**加锁操作所需要的共享变量到本地内存的副本中。  
实质上是线程**接收**了先前某个线程发出的消息，消息是先前线程对共享变量所做的修改。

结合以上解释，我们可以得知，线程 A 释放锁，线程 B 获得锁，实质上就是线程 A 通过主内存向线程 B 发送消息的过程。

<br/>

# transient

`transient` 只能用于修饰变量，用来标识非永久性（不能被序列化）的数据。
* 当对象被序列化/反序列化时，被 transient 关键字修饰的变量不会被持久化/恢复
    * 如：类中有一个 InputStream 对象
    * 对其序列化/反序列化是不合适的，因为序列化和反序列化的位置可能不同，从而导致恢复时对象不可用

<br/>

# volatile

在了解 [Java 内存模型](/2021/07/10/jmm)的时候，我们知道，编译器为了加快程序运行速度，对一些变量的写操作会先在寄存器或 CPU 缓存进行，在高速缓存中拥有一份抽象的工作内存，操作完成后最后再写入主内存。
 
工作内存中的变量是从主内存中读取拷贝，被修改后再写回主内存的；在这个过程中，变量新的值，即某次操作的工作内存变量值对其他操作线程是不可见的。

多个线程并发操作想要正常工作，所必须满足的三个条件如下：

**1. 可见性**
* 一个线程修改了变量，其他线程要立即看得到
* 一般情况下，工作内存的变量被修改后不会立即写入主内存，这就会产生脏读

**2. 原子性**
* do it all OR don't do it at all
* 只有最基本的赋值是原子操作；变量互相赋值、自增减均非原子操作

**3. 有序性**
* 指令重排序：处理器为提高效率，会对代码的执行顺序进行优化
* 但会根据 as-if-serial 和 happens-before 来保证执行结果和顺序执行的结果相同

如：
```java
int a = 10;    // 1
int r = 2;     // 2
a = a + 3;     // 3
r = a * a;     // 4
// 由于 3 和 4 对 1 有依赖，所以编译期在进行指令重排序的时候，会确保 3 和 4 在 1 后面执行
// 可能的顺序 1234, 1324, 2134
```

* 但是多线程时，当依赖某个变量来判断语句的执行时，可能会因为指令重排序而出错
```java
// Thread1
context = loadContext();    // 1
boolean inited = true;      // 2

// Thread2
while(!inited) sleep();     // 1
postInit(context);          // 2

// 出错：Thread1.2 -> Thread2.1 -> Thread2.2 -> Thread1.1（即有可能 context 还没被初始化，为空值）
```


## volatile 与并发

volatile 的特性：把单个 volatile 变量的读和写，看作是使用同一个锁对单次读写操作做的同步。

**1. 可见性**

volatile 可以保证变量的可见性
* 被修饰的成员变量在每次被线程访问时，都强迫其从共享内存中直接重读该成员变量的值
    * 且：当成员变量发生变化时，强迫操作线程将变化值写回共享内存
* 由此可知，在该机制下，任何时刻的两个不同的线程总是会看到某个成员变量的同一个值（看到其他线程对某个变量的写入）
* 只能用来修饰变量

例 1：
```java
class VolatileExample {
    volatile long v1 = 0L;

    public void set(long l) {
        v1 = l;
    }

    public void getAndIncrement() {
        v1++;  // 复合多个变量的读写
    }

    public long get() {
        return v1;
    }
}
// 无需给方法加锁
```

相当于以下实现：
```java
class VolatileExample {
    long v1 = 0L;

    public synchronized void set(long l) {  // 写加锁
        v1 = l;
    }

    public void getAndIncrement() {
        long temp = get();    // 调用已同步的读方法
        temp += 1L;           // 普通写操作
        set(temp);            // 调用已同步的写方法
    }

    public synchronized long get() {  // 读加锁
        return v1;
    }
}
```
注意：`getAndIncrement()` 并没有加锁，因为就如下面将要说到的：volatile **并不能保证操作的原子性**。

上述例子采用 `long` 来举例，是想通过 `long`（64 位）数据的赋值来说明：如果程序在 32 位系统执行，则变量是会（Java 鼓励，不强求）分成高 32 位和低 32 位写入的，而且不是原子性的操作。


**2. 原子性**
* volatile 变量不能提供原子性，不会主动发起阻塞
* 操作对于非原子操作，会将单句代码拆解，分别执行

```java
volatile int inc = 10;
for(int i = 0; i < 1000; i++) {
    inc++;
}
// 10 个线程均执行以上循环，结果可能小于 1000*10
```

解析：因为 inc++ 并不是原子性操作，包括三步：
1. 主内存读取 inc
2. 计算 inc` = inc + 1
3. 写回主内存 inc = inc`

出错的情况：
1. T1 从主内存读 inc = 10，写入 T1 工作内存
2. T2 从主内存读 inc = 10，写入 T2 工作内存
3. T1 计算 inc` = 11 并将其写进工作内存和主内存
4. T2 计算 inc` = 11 并将其写进工作内存和主内存

注意：
* 加锁才保证操作的原子性，volatile 只保证操作的可见性。
* 对 volatile 变量的赋值和取值是原子性的，复合操作并不保证原子性


**3. 有序性**
* volatile 保证有序性：JMM 会**限制指令重排序**
* 确保本条指令不会因为编译器和虚拟机的优化而被省略

原则：  
指令优化时，不能把对 volatile 写的前置操作放在它的后面，也不能把 volatile 读的后续操作放在它的前面：
* 当第⼆个操作是 volatile 写时，不管第⼀个操作是什么，都不能重排序。
* 当第⼀个操作是 volatile 读时，不管第⼆个操作是什么，都不能重排序。
* 当第⼀个操作是 volatile 写，第⼆个操作是 volatile 读时，不能重排序。

<table>
	<tr>
	    <th>是否能重排序</th>
	    <th colspan="3">第二个操作</th>
	</tr>
	<tr>
	    <th>第一个操作</th>
	    <th>普通读/写</th>
        <th>volatile 读</th>
	    <th>volatile 写</th>
	</tr>
	<tr>
	    <td>普通读/写</td>
	    <td></td>
        <td></td>
        <td>NO</td>
	</tr>
	<tr>
	    <td>volatile 读</td>
        <td>NO</td>
        <td>NO</td>
        <td>NO</td>
	</tr>
	<tr>
	    <td>volatile 写</td>
        <td></td>
        <td>NO</td>
        <td>NO</td>
	</tr>
</table>

```java
// x、y 为非 volatile 变量
// flag 为 volatile 变量
int x, y;
volatile boolean f;

x = 2;       // 1
y = 0;       // 2
f = true;    // 3
x = 4;       // 4
y = -1;      // 5

// 不可能的顺序：1或2 在 3 后面；4或5 在 3 前面
// 可能的顺序只有：12345, 12354, 21345, 21354
```

保证有序性的前提下，对前面的例子作修改：
```java
volatile boolean inited;

// T1
context = loadContext();  // 1
inited = true;            // 2

// T2
while(!inited) sleep();   // 1
postInit(context);        // 2
// T1.1 必在 T1.2 前执行，以此来避免出错
```


## 实现：内存屏障

volatile 所能保证的可见性，是根据它所能提供的有序性而来；而volatile 的有序性的实现，靠的是内存屏障（Memory Barrier）。

对于处理器重排序，JMM 的重排序规则会要求 Java 编译器生成指令序列时，在指令之间插入特定类型的内存屏障指令，由此来禁止特定类型的处理器重排序。
* 通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证

以下为不同的屏障类型：

| 屏障类型           | 指令示例              | 说明             |
| :-------:         | :-----:              | :-----------    |
| LoadLoad Barriers      | Load1;*LoadLoad*;Load2      | 确保 Load1 数据的装载先于 Load2 及所有后续装载指令的执行  |
| StoreStore Barriers    | Store1;*StoreStore*;Store2  | 确保 Store1 数据对其他处理器的可见（刷新到内存）先于 Store2 及所有后续存储指令的存储  |
| LoadStore Barriers     | Load1;*LoadStore*;Store2    | 确保 Load1 数据的装载先于 Store2 及所有后续的存储指令刷新到内存  |
| **StoreLoad Barriers**   | Store1;*StoreLoad*;Load2    | 确保 Store1 数据对其他处理器可见（刷新到内存）先于 Load2 及所有后续装载指令的装载。StoreLoad Barriers 会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。 |

说明：“指令示例”里面，分号中间的就是屏障。

StoreLoad Barriers 是全能型屏障，同时具有其他三种内存屏障的效果；但是执行的代价很大，因为需要将所有工作内存变量刷新回主内存。  
绝大多数处理器都支持 StoreLoad Barriers，不一定支持其它屏障。

通过最小化内存屏障数目来实现有序性是很难的，几乎不可能。  
JMM 在实现指令排序的时候先会保证正确性，在此基础上才考虑效率。由此有 volatile 在 JMM 的**保守**的屏障策略：
* 在每个 volatile 写操作的前⾯插⼊⼀个 StoreStore 屏障
    * （即：当第⼆个操作是 volatile 写时，不管第⼀个操作是什么，都不能重排序）
* 在每个 volatile 写操作的后⾯插⼊⼀个 StoreLoad 屏障
    * （即：当第⼀个操作是 volatile 写，第⼆个操作是 volatile 读时，不能重排序）
* 在每个 volatile 读操作的后⾯插⼊⼀个 LoadLoad 屏障 & LoadStore 屏障
    * （即：当第⼀个操作是 volatile 读时，不管第⼆个操作是什么，都不能重排序）

| 时间片 Time  | 操作线程             | 说明          |
| :---------: | -----               | -----        |
| T1          | 普通读               |              |
| T2          | 普通写               |              |
| T3          | StoreStore 屏障     | 禁止上面的普通写和下面的 volatile 写重排序 |
| T4          | **volatile 写**     |              |
| T5          | StoreLoad 屏障      | 禁止上面的 volatile 写与下面可能有的 volatile 读/写重排序 |

T5 插入 StoreLoad 屏障的原因：以防万一，为了能正确实现语义。

| 时间片 Time  | 操作线程             | 说明          |
| :---------: | -----               | -----        |
| T1          | **volatile 读**     |              |
| T2          | LoadLoad 屏障     | 禁止下面所有的普通读操作和上面的 volatile 读重排序 |
| T3          | LoadStore 屏障      | 禁止下面所有的普通写操作和上面的 volatile 读重排序 |
| T4          | 普通读               |              |
| T5          | 普通写               |              |

同时，JMM 还会对一些内存屏障作优化处理。

比如以下的例子：
```java
class VolatileBarriersExample {
    int a;
    volatile int v1 = 1;
    volatile int v2 = 2;

    void readAndWrite() {
        int i = v1;  // 第一个 volatile 读
        int j = v2;  // 第二个 volatile 读
        a = i + j;   // 普通写
        v1 = i + 1;  // 第一个 volatile 写
        v2 = j * 2;  // 第二个 volatile 写
        ...
    }
}
```

在保证 volatile 读/写内存语义的前提下，可以省略一些不必要屏障：

| 时间片 Time  | 操作线程            | 说明         |
| :---------: | -----             | -----        |
| T1          | 第一个 volatile 读  |             |
| T2          | LoadLoad 屏障      | 禁止上面的 volatile 读和下面的 volatile 读重排序                    |
|             |                   | （省略了 LoadStore 屏障：下面的普通写不可能越过在它上面的 volatile 读） |
| T3          | 第二个 volatile 读  |             |
|             |                   | （省略了 LoadLoad 屏障：下面没有普通读操作）                         |
| T4          | LoadStore 屏障     | 禁止下面的普通写操作和上面的 volatile 读重排序                       |
| T5          | 普通写             |              |
| T6          | StoreStore 屏障    | 禁止上面的普通写和下面的 volatile 写重排序 |
| T7          | 第一个 volatile 写  |             |
|             |                   | （省略了 StoreLoad 屏障：下面紧接着一个 volatile 写）                |
| T8          | StoreStore 屏障    | 禁止上面的 volatile 写和下面的 volatile 写重排序                    |
| T9          | 第二个 volatile 写  |             |
| T10         | StoreLoad 屏障     | 禁止上面的 volatile 写与下面可能有的 volatile 读/写重排序            |

<br/>

## volatile 内存语义

还是拿上面一段读写代码做例子，用 volatile 改一下，俩线程 A 和 B 分别访问 writer() 和 reader()：
```java
class ReorderExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1;  // 1
        flag = true;  // 2
    }

    public synchronized void reader() {
        if (flag) {  // 3
            int i = a * a;  // 4
            ... 
        }
    }
}
```

之前提过，volatile 能禁止指令重排序，因此 1 和 2 之间不会重排序，3 和 4 之间不会重排序。因此根据分析可知：

| 时间片 Time  | 线程 A              | 线程 B                     |
| :---------: | -----               | -----                    |
| T1          | 1. 修改共享变量 a     |                          |
| T2          | 2. 写 volatile 变量  |                          |
| T3          |                     | 3. 读取 volatile 变量      |
| T4          |                     | 4. 读取共享变量 a，并写入 i  |

由程序顺序规则（1 在 2 之前，3 在 4 之前）和 volatile 变量规则（volatile 写在读之前，即 2 在 3 之前）结合之后，提供的 happens-before 保证，就是 1 happens-before 4。

volatile 写对应的内存语义：写 volatile 变量时，JMM 将本地内存中共享变量的值直接刷新到主内存的共享变量中。  
实质上是这个操作线程向接下来将要读取这个 volatile 变量的某个线程**发出了消息**，消息是该线程对共享变量所做的修改。

volatile 读对应的内存语义：先将本地内存中的共享变量**置为无效**，再去主内存中**读取**共享变量的值到本地内存的副本中。  
实质上是这个操作线程**接收**了先前某个线程发出的消息，消息是先前线程对 volatile 变量所做的修改。

结合以上解释，我们可以得知，线程 A 写一个 volatile 变量，线程 B 读取这个 volatile 变量，实质上就是线程 A 通过主内存向线程 B 发送消息的过程。

<br/>

## 使用场景：状态标记

* 利用有序性及可见性，规避非原子性操作

例：多线程环境开发下，单例模式的 double check
```java
class Singleton {
    private volatile static Singleton instance = null;
    private int x;

    private Singleton() { x = 1; }

    public static Singleton getInstance() {
        if(instance == null) {
            synchronized (Singleton.class) {
                if(instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}

/* 由于instance = new Singleton() 非原子操作，会被拆成三步，利用 volatile 避免指令重排序避免出错
 三步分别是：
     1. 为对象分配内存
     2. 调用构造函数初始化成员变量（如有）
     3. 将 instance 指向新对象
 可能的顺序：123, 132，若 instance 为 volatile 则只能为 123

 出错的情况（instance 不为 volatile）：
 1. T1 调用，instance == null, 通过判断拿到锁，开始初始化对象
 2. 指令重排序为 132
     a. 为对象分配内存
     b. 将 instance 指向新对象, instance != null
     c. T1 阻塞
     d. 调用 instance 构造方法，初始化成员变量，等
 3. T2 在 2.b 与 2.d 之间调用，由于 instance != null，直接返回 instance，指向已经分配内存但未调用构造方法的对象，此时成员变量未初始化，可能出错
 */
```
