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


# assert
断言（Since Java 1.4），用于查找内部程序错误


# final
final 关键字在 Java 中被称为“完结器”，表示最终的意思。  
它能声明类、方法、属性（方法参数和变量）。

    Java classes declared as final cannot be extended. 
final 声明的类不能被继承。

    Methods declared as final cannot be overridden. 
final 声明的方法不能被重写，但不影响被继承。

    The variable declared as final should be initialized only once and cannot be changed. 

final 声明的变量只能被赋值一次，不可被修改。
* 修饰基本类型：值不可变，即常量
* 修饰对象：该变量被赋予的引用不可变

对于被 final 修饰的对象，不可改变的只是该变量所保存的引用，并不是该引用指向的对象。
```java
final StringBuilder s = new StringBuilder();  // final 修饰的是 s，即 s 保存的地址不能变
s.append("a");    // 无误，s 指向的对象是可以被修改的
s = null;         // 报错，s 的值不能被修改
```
“被赋值一次”：Java 虚拟机为变量设定的默认值不计作一次赋值

被 final 修饰的变量必须被初始化：
* 定义时初始化
* 初始化块中初始化
* 类的构造器（Constructor）中初始化
* 静态变量也可在静态初始化中初始化

注：一个被 final 定义并同时指定了初始值，且初始值为编译时就被确定的
* 该 final 变量为宏变量
* 编译器会将程序所有用到该变量的地方直接替换成该变量的值：宏替换
```java
final String a = "hello";    // 宏变量
final String b = a;          // 非宏变量
```


# finally
只能被用在 `try` / `catch` 语句中

    The finally block always executes when the try block exits. This ensures that the finally block is executed even if an unexpected exception occurs. 

`return`, `continue`, `break` 都不能阻止 finally 语句块的执行
* `return`, `continue`, `break` 所在语句块的内容会先输出，然后轮到 finally
* 但 finally 块是在他们被执行之前执行的


# finalize
finalize() 方法为 Object 类的方法

    Before an object is garbage collected, the runtime system calls its finalize() method. 


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


# synchronized 

* 用来修饰一个方法或者一个代码块
* 能够保证在同一时刻最多只有一个线程执行这段代码
* 进入这块代码的线程相当于拥有了一把“锁”

因 Java 中每个对象均有对象锁（`Object.lock()`）
* synchronized 同步方法：对象锁将保护整个方法
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

这是因为，由于三个线程同时访问一个变量，而每个线程中 x 的自增操作会分解为：
1. 从主内存中将 `x` 的值读取到该线程的工作内存中
2. `x++`
3. 将 `x++` 的值写回工作内存及主内存

然而，实际的情况却是：（以 `x = 0` 为例）
1. `x = 0`
2. T1 从主内存读取 `x` 的值并写入其工作内存
3. T1 计算 `x` 的值为 1
4. T2 从主内存读取 `x` 的值并写入其工作内存，此时 T2 读取到的 `x` 的值**仍为 0**
5. T2 计算 `x` 的值仍为 1
6. T1 将 `x = 1` 写入工作内存及主内存
7. T2 将 `x = 1` 写入工作内存及主内存
8. T1 和 T2 一共计算了**两次**，但 `x` 的值只增加了 1


## 应用

### 同步某个方法

```java
public synchronized 数据返回类型 方法名() { ... }
```
* 使用 synchronized 来修饰某个方法，则该方法称为**同步方法**
* 对于同步方法而言，无需显式指定同步监视器（Monitor）
    * 同步方法的 Monitor 是 this：即调用该同步方法的对象
    * 可将某类变成线程安全的类 —— 该类相当于 Monitor
* 若要调用该方法：线程需获得内部对象锁
    * 锁只会加在访问同一个对象该方法的那些线程上


根据上述，改进上述应用背景：
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

静态方法
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
* 静态方法独立于对象存在：因此所有访问该静态方法的线程共享一把锁


### 同步代码块（同步阻塞）

```java
synchronized (同一个数据) { ... } 
```
* 同一个数据：N 条线程同时访问一个数据
* 比锁住整个方法更有效率：因为同时可以有多个线程进入到方法中，进行那些无需上锁的操作

我们暂且称这“同一个数据”为**锁芯**
* 锁芯可为固定的 this 对象或类的 class 对象，也可更换为其他对象
* 几乎任意对象，但不能为 null
* 原因：锁芯应为所有访问该方法的线程均能访问的同一个对象，这样才能确保其线程互斥
* this、class 及其他 final 对象能防止锁芯更换，从而避免各线程的锁不一样


**1. 普通方法代码块：**
```java
private static int x = 0;
public void inc() {
    synchronized (this) {
        x++;
    }
}
// 等价于 public synchronized void inc() { x++; }
// 代码块范围为整个方法里的代码，且锁住 this 对象

private static int x = 0;
public void inc() {
    synchronized (Test.class) {
        x++;
    }
}

private final Object lock = new Object(); // 注意这个 final
private static int x = 0;
public void inc() {
    synchronized (lock) {
        x++;
    }
}
```

**2. 静态方法代码块：**
* 静态方法通常只与**类**有关，与类的任何对象无关
* 此时锁芯通常由所有直接或间接调用该类静态方法（类名.方法）的线程共享
    * 因此显然**不能是 this 对象**或任何非静态对象
    * 只能是静态对象或类的 class 对象
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
十分脆弱，不推荐使用

注：`synchronized` 修饰的方法在子类中不会继承同步性
* 即：多个线程调用该子类继承父类的方法时不会加锁
* 除非重写并加上关键字


## synchronized 原理：**可重入性**

使用 synchronized 关键字同步的方法与锁的关系：
```java
public synchronized void method() { doSomething(); }
```

等同于
```java
public void method() {
    this.intrinsicLock.lock();
    try {
        doSomething();
    } finally {
        this.intrinsicLock.unlock();
    }
}
```
* 即：synchronized 关键字提供的是一个可重入锁

从一个对象的 synchronized 方法调用该对象的另一个 synchronized 方法：调用线程在执行这两个方法的时候，获得的是**一把**对象锁
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
* 锁被获得（进入）了两次，记数 2
* 进入该对象 m1 的线程在进入 m2 时可直接获得该锁，这也体现了“可重入性”

注意：
* 可重入的代码不能使用静态或全局变量作为锁芯，同时不调用不可重用的代码
* 当 synchronized 方法 A 调用某方法 B，B 没有上锁的同时可被其他线程直接调用
    * 这会导致其绕过了锁直接进行 B 方法的操作，造成线程不安全
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

/* 可能的情况
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


# transient

`transient` 只能用于修饰变量，用来标识非永久性（不能被序列化）的数据。
* 当对象被序列化/反序列化时，被 transient 关键字修饰的变量不会被持久化/恢复
    * 如：类中有一个 InputStream 对象
    * 对其序列化/反序列化是不合适的，因为序列化和反序列化的位置可能不同，从而导致恢复时对象不可用


# volatile

在了解 Java 内存模型的时候，我们知道，编译器为了加快程序运行速度，对一些变量的写操作会先在寄存器或 CPU 缓存进行，最后再写入主内存。

也就是说，每个线程在处理器的高速缓存中都拥有一份工作内存，保存它用到的共享变量的副本。
* 工作内存中的变量是从主内存中读取拷贝，在被修改后再写回主内存的
    * 在这个过程中，变量新值，即工作内存变量值对其他线程不可见


并发正常工作所必须满足的三个条件：

**1. 可见性**
* 一个线程修改了变量，其他线程要立即看得到
* 一般情况下，工作内存的变量被修改后不会立即写入主内存，这就会产生脏读

**2. 原子性**
* do it all OR don't do it at all
* 只有最基本的赋值是原子操作；变量互相赋值、自增减均非原子操作

**3. 有序性**
* 指令重排序：处理器为提高效率，会对代码的执行顺序进行优化
* 但会根据 happens-before 来保证执行结果和顺序执行的结果相同

如：
```java
int i = 0;
int f = false;
i = 1;        // 1
f = true;     // 2
// 1 和 2 无依赖，执行顺序可能为：12 或 21

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

// 出错：Thread1.2 -> Thread2.1 -> Thread2.2 -> Thread1.1（有可能 context = null）
```


## volatile 与并发

**1. 可见性**

volatile 保证可见性
* 被修饰的成员变量在每次被线程访问时，都强迫其从共享内存中直接重读该成员变量的值
    * 且：当成员变量发生变化时，强迫操作线程将变化值写回共享内存
* 由此可知，在该机制下，任何时刻的两个不同的线程总是会看到某个成员变量的同一个值
* 只能用来修饰变量

例：
```java
private boolean done;
public synchronized boolean isDone() { return done;}
public synchronized void setDone() { done = true;}
```

使用 volatile：
```java
private volatile boolean done;
public boolean isDone() {return done;}
public void setDone() { done = true;}
// 无需给方法加锁
```

**2. 原子性**
* volatile 变量不能提供原子性，不会主动发起阻塞
* 对于非原子操作：单句代码会被拆解执行

```java
volatile int inc = 10;
for(int i = 0; i < 1000; i++) {
    inc++;
}
// 10 个线程均执行以上循环，结果可能小于 1000*10
```

解析：因为 inc++ 并不是原子性操作，其包括三步：
1. 主内存读取 inc
2. 计算 inc` = inc + 1
3. 写回主内存 inc = inc`

出错的情况：
1. T1 从主内存读 inc = 10，写入 T1 工作内存
2. T2 从主内存读 inc = 10，写入 T2 工作内存
3. T1 计算 inc` = 11 并将其写进工作内存和主内存
4. T2 计算 inc` = 11 并将其写进工作内存和主内存


**3. 有序性**
* volatile 保证有序性：**禁止指令重排序**
* 确保本条指令不会因为编译器和虚拟机的优化而被省略

原则：
1. 执行对 volatile 的读/写操作前，前面的更改一定全部执行，且结果对后面可见；后面的更改一定还没有被执行
2. 指令优化时，不能把对 volatile 的读/写的前置操作放在它的后面，也不能把 volatile 读/写的后续操作放在它的前面
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
// T1.1 必在 T1.2 前执行，由此来避免出错
```


## 使用场景：状态标记

* 利用有序性及可见性，规避非原子性操作

例：单例模式的 double check
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

注：使用 final 修饰的共享数据结构，其他线程在对象构造函数完成后才能看到该对象

另：`java.util.concurrent.atomic` 包中有很多类方法保证了操作的原子性
