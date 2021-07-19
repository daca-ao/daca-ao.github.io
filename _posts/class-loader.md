---
title: JVM Class Loader 类加载器
date: 2021-07-08 00:07:22
tags:
- Java
- JVM
---

Java 程序的基本单元，就是一个个的类（Class）及类实例（Class Instance）。在使用类之前，必须先将类加载进 JVM 的内存中。

<!-- more -->

类加载器作用：将类加载到 Java 虚拟机，并且检查类的完整性。


类加载机制：
* 将描述类的数据从 `.class` 文件加载到内存
* 对数据进行校验、转换解析和初始化
* 最终构造成可被虚拟机直接使用的 Java 类型

好处：
* 类的加载、连接和初始化过程都在程序的运行时完成，令 Java 具备高度的灵活性
* 避免类被重复加载：即每个 JVM 中，只会有一个拥有同样全路径的 .class 文件
* 沙箱安全机制：防止核心类库的核心类被篡改

JVM 只会加载程序执行时所需要的类文件（.class 文件），**暂时不使用的类则不会被加载**。

<br/>

# 类加载过程
三大类，五步：包括**加载** -> **验证** -> **准备** -> **解析** -> **初始化**。

**1. 加载 Load**：加载类到内存，在堆区建立一个 class 对象
1. 通过类的全限定名 ( com.xxx.xxxx…) 获取该类的二进制字节流
2. 从磁盘读取文件，或请求 Web 上的文件等
3. 将该字节流代表的静态存储结构转换为运行时（方法区）的数据结构
4. 在静态区（内存）生成 java.lang.Class 对象，作为方法区该类的各种数据结构的访问入口

**2. 链接 Link**：把 class 的二进制数据合并到 JRE，具体包括**验证**、**准备**和**解析**三步：
1. 验证：class 文件中类结构、语义、操作等的合法性，包括：
    1. 文件格式验证
    2. 元数据验证（语义分析，类与类的继承关系等）
    3. 字节码验证（数据流和控制流分析）
    4. 符号引用验证（对类自身以外的信息进行匹配校验）
2. 准备：为类的静态变量分配内存，并将非 final 的静态变量初始化默认值（零值），final 的静态变量初始化为其指定值
3. 解析：将常量池中的符号引用替换成直接引用
    * 符号引用：类名、方法名、字段名，具有人类可读性，类似于 OS 中的逻辑地址
    * 直接引用：内存偏移量，指示内存真实地址，类似于 OS 中的物理地址
    * 符号引用指向的目标不一定加载到了内存中，而直接引用的目标一定已经加载到了内存中

**3. 初始化 Initialization**：赋予静态变量程序员指定的值，初始化静态代码块
1. 真正执行类中定义的 Java 程序代码
2. 不可逆

例：程序从 HelloWorld.class 开始运行——
1. 虚拟机从磁盘读取文件或请求 Web 上的文件等方法，加载 HelloWorld 类文件（`.class` 文件）的内容；
2. 如 HelloWorld 类拥有来自另一个类的域，或拥有超类，那么这些类文件也会被加载（类的解析）；
3. 虚拟机执行 HelloWorld 中 `main()` 方法；
4. 如 `main()` 或 `main()` 调用的方法需要用到更多的类：就加载这些类。

与以下行为相关的类，如没有被加载的话，此时将被加载到 JVM 内存中：
1. 创建类实例，包括：
    1. 使用 new 关键字创建实例
    2. 使用 Class.forName() 反射方法来创建实例
2. 调用类的静态方法
3. 给类的静态变量赋值
4. 访问非 final 的静态变量
    * 常量在被写进类的字节码之前，会被编译器优化为 value 而不是 field
    * 因此编译器并不会生成字节码来从实例中载入 field 的值，而是直接把这个 value 插入到字节码中
5. 初始化类时，若其父类尚未加载，则先初始化其父类
    * 这种情形会递归加载所有之前未被加载的父类
6. JVM 启动时，用户指定的包含 main 方法的类

发生以下情况时，类不会被加载：
1. 子类调用父类的静态方法，子类不会被加载，只有父类会被初始化
    * 对于静态字段，只有直接定义这个字段的类才会被初始化
2. 作为数组元素被引用的类
3. 访问类的 final 静态变量，即常量

<br/>

# 类加载器类型
每个 Java 程序至少拥有三类加载器。

**1. 引导类（启动类）加载器**（**Bootstrap Class Loader**）
* 最顶层的类加载器，是 Java 虚拟机不可分割的一部分
* 负责加载 JDK 的**核心类库**，如 `rt.jar`、`resources.jar`、`charsets.jar` 等，包括：
    * `$JAVA_HOME/lib` 中的类库
    * JVM 的 `-Xbootclasspat`h 参数指定的类库
* C++ native code 编写，随 JVM 启动而启动
* 注：没有对应的 ClassLoader 对象
* 在构造的时候会同时构造 Extension Class Loader

**2. 扩展类加载器**（**Extension Class Loader**）
* 默认从 `$JAVA_HOME/jre/lib/ext` 目录加载“标准的扩展”
* 还包括 java.ext.dirs 系统变量指定的类库
* Java 实现，为 URLClassLoader 实例（继承自 ClassLoader）
* 注：并不使用类路径
* 在构造的时候同时构造 Application Class Loader

**3. 系统类加载器**（**System Class Loader**）或**应用类加载器**（**Application Class Loader**）
* 用于加载应用类
* 加载应用程序的 classpath 目录下所有 jar 和 class 文件
* Java 实现，为 URLClassLoader 实例（继承自 ClassLoader）

注：还可从 `/jre/lib/endorsed` 目录加载——将某个标准 Java 类库替换为更新的版本。

<br/>

除了上述三类必备的类加载器之外，用户还可以自定义类加载器：  
**4. Custom Class Loader**
* 用户自定义的类加载器，常用于代码的加解密

<br/>

# 类加载顺序：双亲委派模型
类加载器有一种“父/子关系”，它们是**组合**而非继承关系：
* 当某个类加载器收到类加载请求，首先将请求递归委派给“父类”；
* 当它的父类无法完成加载时，加载工作才由它自身完成
* 除了 Bootstrap 引导类加载器外，每个类加载器都有一个父类加载器

以上即为“双亲委派模型”
* 父与子之间不是继承关系，是包含关系
* 好处：避免重复加载某个类

具体描述如下：
* 某 ClassLoader 亲自搜索需要加载的某个类前，将加载任务委托至父类加载器
    * 检查这个类是否已被加载，否则委派 App ClassLoader 加载
    * 检查这个类是否已被加载，否则委派 Extension ClassLoader 加载
    * 检查这个类是否已被加载，否则委派 Bootstrap ClassLoader 加载
    * 从 Bootstrap ClassLoader 试图加载
    * 如没加载到：任务转交给 Extension ClassLoader
    * 如没加载到：任务再转交给 App ClassLoader 
    * 如没加载到：任务再转交给该 ClassLoader 自身，或到网络或指定文件系统 URL 加载该类
    * 如没加载到：抛出 `ClassNotFoundException`

如：某些具有插件结构的程序通过 URLClassLoader 类实例加载类
```java
URL url = new URL("file://path/to/plugin.jar");
URLClassLoader pluginLoader = new URLClassLoader(new URL[] {url});
Class<?> cl = pluginLoader.loadClass("mypackage.MyClass");

// URLClassLoader 构造器没有指定父类加载器：pluginLoader 父类就是系统类加载器
```
