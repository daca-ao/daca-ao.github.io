---
title: Spring IoC 
date: 2021-12-05 13:08:48
tags:
- Spring
- IoC
---

首先我们要明确的是，IoC 并不是最近才兴起的说法，更不是只有 Spring 才能应用的逻辑，而是自 OOP 面世以来就有的概念。

<!-- more -->

# 什么是 IoC？

**Inversion of Control**，**控制反转**，是 OOP 的一种设计原则和架构模式（算不上是设计模式）。

一句经典的解释，就是：  
**"Don’t call me, I’ll call you. "**

<br/>

## 问题与解决

针对“控制反转”这个概念，我们先提出三个问题：
1. （**Q1**）什么的控制？
2. （**Q2**）为何反转？
3. （**Q3**）如何反转？

首先，我们来看一下传统 Java 代码中的 HAS-A 结构：
```java
public class B {
    private A a;
    public B(A a) {
        this.a = a;
    }
}

// somewhere else in code
A a = new A();
B b = new B(a);
```

这种耦合度较强的代码，至少有一处需要：
1. 显式或隐式地创建 A 对象
2. 将 A 对象传入 B 类的构造方法

对于每一次的 HAS-A，都需要这样的程序代码进行上述控制（**Q1**），当复杂度升高时，便难以掌控了（**Q2**）。

针对 HAS-A 的典型问题，IoC 代替程序代码包办了这层控制，从而降低耦合度：
* 为了解耦，需要使用一个统一的对象依赖控制器
* 程序代码只需向 IoC 控制器“索要”依赖的对象，而不是自己“造”
    * 即：**依赖对象的获得被反转了**（**Q3**），所反转的“控制”是对象间的依赖。

所以 IoC 是利用**反射**编程实现控制的反转。


## 特点（**Q2**）

* 体现软件工程原则之一：解耦
* 对实例的控制由程序转移到了控制器/容器/框架，随后便可以注入到引用中
* 为模块化提供支撑
* 实现“热插拔”


## IoC 的实现

IoC 仅为一种设计原则，通常还结合 DIP（Dependency Inversion Principle）使用。

其实现并不是非 Spring 不可，可以结合多种设计模式实现，如策略模式 + 工厂模式、模板方法模式、依赖注入（DI）或依赖查找（DL）实现。

实际应用中有三种方式可以实现控制反转：
1. 构造方法注入
2. setter 注入
3. 成员变量注入


### 具体实现：DI

Dependency Injection，依赖注入，属于设计模式的一种，是 IoC 的一种具体实现。
* 被反转的控制是对象依赖的注值
    * 即：被注入对象依赖 IoC 容器配置依赖对象
* 以往程序代码主动为对象的依赖赋值，如今由 DI 控制器统一处理
    * 例：在类 A 实例创建过程中即创建了依赖的 B 对象
* 实现方式：
    * 基于接口（Interface Injection）
        * 实现特定接口以供外部容器注入所依赖类型的对象
    * 基于 setter（Setter Injection）
        * 让外部容器调用传入所依赖类型的对象
    * 基于构造函数（Constructor Injection）
        * 在新建对象时传入所依赖类型的对象
    * 基于参数（Parameter Injection）
    * 基于注解
    * ...

举一个简单的例子：

```java
public class TextEditor {

    private SpellChecker checker;

    public TextEditor() {
        this.checker = new SpellChecker();  // TextEditor 依赖于 SpellChecker
    }
}
```

```java
public class TextEditor {

    private IocSpellChecker checker;

    public TextEditor(IocSpellChecker checker) {  // 改过后：在构造函数注入依赖 IocSpellChecker
        this.checker = checker;
    }
}
```

如此一来，再调用 TextEditor 的时候就可以：

```java
SpellChecker sc = new SpellChecker;  // dependency
TextEditor textEditor = new TextEditor(sc);  // get the dependency (inject the dependency)
```


### 具体实现：DL
Dependency Lookup，依赖查找
* 比依赖注入更加主动
* 在需要时通过调用框架提供的方法主动索取相对应类型的对象
* 获取时需要提供相关的配置文件路径、key 等信息来确定获取对象的状态



较为知名的 Java IoC 容器：
* 轻量级：Pico Container，Avalon，Spring，HiveMind
* 较重量级：JBoss，Jdon
* 超重量级：EJB

各种概念的层次：


某个具体的实现流程：


# Spring IoC

将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理


Spring IoC 定义以下概念：
* Bean
    * 等待被注入的依赖对象
    * 不一定非得要 JavaBean，可以是 POJO 类
* 容器
    * 控制反转的控制器
    * 负责创建、管理、装配、配置 bean
        * 并管理所有 bean 的生命周期以及它们之间的依赖链
        * 支持加载服务时的饿汉式初始化和懒加载
    * 主要通过 DI 进行管理
好处：
* 使应用代码量降到最低，容易测试，UT 不再需要单例和 JNDI 查找机制
* 使松散耦合得以实现
* 支持即时实例化和懒加载

Spring 容器根据配置文件或注解获取 metadata，进行 Bean 依赖的管理
主要完成三个任务：
1. 根据配置文件或注解，解析出 Bean 之间完整的依赖图，取得其类名
2. 利用 Java 反射，用适当的方式创建出 Bean，保存到一个容器内
3. 再次利用反射，在适当时机取得将被依赖的 Bean，将其作为成员变量注入到依赖其的 Bean 中
    1. 可通过构造函数、setter 或其他方法注入



两种不同类型的容器：

1. BeanFactory（org.springframework.beans.factory.BeanFactory 接口）
* 包含 bean 集合的工厂类
* 最简单，为 DI 提供基本支持
* 其与其相关接口在 Spring 中仍存在大量与 Spring 整合第三方框架的反向兼容性的目的
* 相对于 ApplicationContext 更轻量级，数据量和速度可观
    * 用于移动设备或基于 applet 的应用

实现类：
```java
XmlBeanFactory // 从 XML 文件读取配置元数据
// @Deprecated 应使用 ApplicationContext

XmlBeanFactory factory = new XmlBeanFactory(new ClassPathResource("Beans.xml"));
HelloWorld obj = (HelloWorld) factory.getBean("helloWorld");
```


2. ApplicationContext（org.springframework.context.ApplicationContext 接口）
* Spring 的核心，负责管理所有 bean 完整生命周期
* BeanFactory 的子接口，在 BeanFactory 基础上提供额外功能
* 添加更多企业特定功能
    * 从属性文本中解析文本
    * 将事件传递给所指定的监听器
* 包含 BeanFactory 容器所有功能
实现类：
```java
FileSystemXmlApplicationContext // 提供配置文件 XML 的完整路径
ClassPathXmlApplicationContext // 提供正确配置的 CLASSPATH 环境变量，容器便会从 CLASSPATH 搜索 bean 配置文件


ApplicationContext context = new FileSystemXmlApplicationContext("C:/Users/raymond.ao/workspace/HelloSpring/src/Beans.xml");
HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
```


3. WebApplicationContext
XmlWebApplicationContext // 在 Web application 范围内加载在 XML 文件中已被定义的 bean



通过配置文件完成管理的例子：
```java
package pojo;

public class Source {
    private String fruit;  // 类型
    private String sugar;  // 糖分描述
    private String size;  // 大小杯

    /* setter and getter */
    ...
}
```

配置文件管理：src/applicationContext.xml（位置和名称任意，此处为例子）
* Spring Framework 利用该文件创建所有已定义的 Bean，按照标签定义分配唯一的 ID
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="source" class="pojo.Source">  <!-- name 用于之后从 Spring 容器获得实例时使用；class 中配置全限定名称 -->
        <property name="fruit" value="橙子"/>
        <property name="sugar" value="多糖"/>
        <property name="size" value="超大杯"/>
    </bean>

</beans>
```

调用：
```java
// 该 API 创建应用上下文
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
Source source = (Source) ctx.getBean("source");  // 直接从 Spring 获取一个对象

System.out.println(source.getFruit() + " " + source.getSugar() + " " + source.getSize());


/*
 * 输出结果：
 *
 * 橙子 多糖 超大杯
 */
```

区别：

| BeanFactory          | ApplicationContext |
| -------------------- | ------------------ |
| 使用懒加载             | 使用即时加载         |
| 使用语法显式提供资源对象 | 自己创建和管理资源对象 |
| 不支持国际化           | 支持国际化           |
| 不支持基于依赖的注解    | 支持基于依赖的注解     |
