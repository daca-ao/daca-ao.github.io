---
title: Java 内存异常处理
date: 2021-07-15 22:46:15
tags:
- Java
- JVM
---

Java 程序中常见的内存毛病就是**内存溢出**和**内存泄漏**了。本帖专门来聊聊这俩毛病，以及做一下比较。

<!-- more -->

# 内存溢出

内存溢出（Out Of Memory, **OOM**），对应的是 `java.lang.OutOfMemoryError` 这一非运行时异常。
* 继承自 `java.lang.VirtualMachineError`。

常见原因：
* 内存中加载的数据量过于庞大，如一次过从数据库取出过多数据
* 集合类中有对对象的引用，使用完之后未清空，使得 JVM 不能回收
* 代码中存在死循环或循环产生过多重复的对象实体
* 启动参数内存值设定得过小
* 使用的第三方软件中的 BUG

常见的错误提示：
```bash
# tomcat: 
java.lang.OutOfMemoryError: PermGen space  # Java 8-
java.lang.OutOfMemoryError: Java heap space

# web logic:
Root cause of ServletException java.lang.OutOfMemoryError

# resin: 
java.lang.OutOfMemoryError

# java:
java.lang.OutOfMemoryError
```


## 需要重点排查的原因
* 检查代码中是否有死循环或者递归调用
* 检查是否存在重复产生新对象实体的大循环
* 检查对数据库查询中，是否有一次获得全部数据的查询：
    * 一般来说，如果一次取 10 万条数据到内存，就可能引起内存溢出。这属于比较隐蔽的问题，在上线前数据较少，不容易出问题，上线后数据增加，就可能会有问题
    * 对于数据库查询，尽量使用分页的方式查询
* 检查 List、Map 等集合对象是否有使用完之后，未清除的问题
    * List、Map 等集合对象会始终存有对对象的引用，使得这些对象不能被 GC 回收

## 主要的解决方法

**1. 增加 JVM 内存大小**
* 执行某个 .class 文件之后，可以使用 `java -Xmx256M xxx.class` 来设置运行该 .class 时 JVM 所允许占用的最大内存是 256M
* 对于 Tomcat：启动时对 JVM 设置内存限度，在 `catalina.bat`（`catalina.sh`）中添加
```bash
set CATALINA_OPTS=-Xms128M -Xmx256M
set JAVA_OPTS=-Xms128M -Xmx256M

# 或者将 `%CATALINA_OPTS%` 和 `%JAVA_OPTS%` 代替为 `-Xmx128M -Xmx256M`
```
* 对于 Resin 容器：启动时对 JVM 设置内存限度，在 `bin` 文件夹下创建一个 `startup.bat`
```bat
@echo off
call "httpd.exe" "-Xms128M" "-Xmx256M"
:end
```

**2. 优化程序、释放垃圾（避免死循环）**


# 内存泄漏

Memory Leak，指程序在申请内存后，无法释放已经申请的空间。

## 分类

内存泄漏可分为以下四类：

**1. 常发性内存泄漏**：发生内存泄漏的代码会被多次执行，每次被执行的时候都会导致一块内存泄漏

**2. 偶发性内存泄漏**：发生内存泄漏的代码只在特定环境或操作过程下才发生
* 常发性和偶发性是相对的：对于特定环境，偶发性也许会变成常发性
* 测试环境和测试方法对检测内存泄漏至关重要

**3. 一次性内存泄漏**：发生内存泄漏的代码只被执行一次
* 可能性：算法缺陷
* 如：在类的构造函数分配内存，在析构函数没有释放内存 —— 泄漏一次

**4. 隐式内存泄漏**：程序在运行过程中不停分配内存，但直到结束时才释放内存
* 严格说并没发生内存泄漏：因为最终释放了所有申请的内存
* 但对于服务器程序：运行几周、几天或几个月，不及时释放可能会导致最终耗尽系统所有内存

内存泄漏最终会导致内存溢出。


## 经典场景

**1. 静态集合类中的元素失去了外部引用：**
```java
static HashMap<String, Object> map;
Object o = new Object();
map.put("o", o);

o = null;  // map 仍然持有该对象的引用，不能被 GC
System.out.println(null == map.get("o"));  // false
```

**2. Hash 集合添加元素引发的内存泄漏：**
```java
Map<Person, String> map = new HashMap<>(1000);

while (true) {
    Person dummyKey = new Person("zhangsan", 18);
    
    // 可重复添加，导致内存泄漏
    map.put(dummyKey, "value");
}

class Person {

    private String name;
    private int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 加入 Hash 集合的类如果不实现 hashCode()，集合每次将该类的对象视为新的对象，因此导致可以重复添加
    @Override
    public int hashCode() {
        // 如示例：Person 类需实现 hashCode() 方法，且 hashCode() 的返回值与属性相关
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        return result;
    }
}
```

**3. 监听器/各种连接：**
* 调用 addListener() 后忘记释放
* 建立 DB Connection，Socket Connection，I/O Stream 后忘记调用相应的 close() 方法
    * 不仅需要调用 Connection::close()，还需关闭查询到的 ResultSet 和 Statement
* ...


**4. 单例对象 HAS-A 另外一个对象的引用：**
```java
class A {...}

public class B {

    private B() {}
    private static B INSTANCE = new B();
    public static B getInstance() {
        return INSTANCE;
    }

    private A a;
    public void setA(A a) {this.a = a;}
    // 对象 a 不能被垃圾回收
}
```


# 内存溢出和内存泄漏的区别

**内存溢出**是指程序在申请内存时，**没有足够内存空间**供其使用
* 如申请了一个 Integer 的内存，却存了 Long 才能存下来的数

**内存泄漏**则是指程序在申请内存后，**无法释放**已经申请的空间
* 一次内存泄露或许可以忽略，但内存泄漏堆积后果很严重，内存迟早会被占光
