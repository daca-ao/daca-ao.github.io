---
title: Java 注解（Annotation）
date: 2021-07-22 00:44:36
tags:
- Java
---

从 Java 5 开始，官方开始引入注释机制，程序员们也可以通过自定义注解实现各种需求。

<!-- more -->

Java 注解是通过**反射**获取注解内容的；在编译器生成类文件时，注解可被嵌入到字节码中。
* JVM 可保留注解的内容，运行时会获得注解内容
* 类、方法、变量、参数、包都可被标注
* 支持自定义注解


# 内置注解

```java
@Override  // 检查该方法是否为重写方法
@Deprecated  // 标记过时方法
@SuppressWarnings  // 指示编译器忽略注解声明的警告
@Retention  // 标记该注解如何保存（RetentionPolicy），是只存在代码中，还是编入 class 文件中，或是在运行时通过反射访问
@Documented  // 标记注解是否包含在用户文档中
@Target  // 注解使用的目标范围（ElementType，包括类、方法、字段等），标记注解是哪种 Java 成员
@Inherited  // 标记这个注解继承自哪个注解类（默认 注解并没有继承任何子类）

@SafeVarargs  // since Java 7，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告
```

Since Java 8:
```java
@Native  // 指定字段是一个常量
@FunctionalInterface  // 标记一个匿名函数或函数式接口
@Repeatable  // 标记某注解可在同一声明使用多次
```


# 架构

![](annotation/annotation.jpg)

如上图，1 个 Annotation 和 1 个 RetentionPolicy 关联，和 1 到 n 个 ElementType 关联（图左），有很多实现类（图右）。

```java
package java.lang.annotation;

/*
 * 每个实现 Annotation 的对象都会有唯一的 RetentionPolicy 属性，有一到多个 ElementType 属性
 */
public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}
```

```java
package java.lang.annotation;

public enum RetentionPolicy {
    SOURCE,       // Annotation 信息仅存在于编译器处理期间，编译器处理完之后就没有该 Annotation 信息了

    CLASS,        // 编译器将 Annotation 存储于类对应的 .class 文件中。默认行为

    RUNTIME       // 编译器将 Annotation 存储于 class 文件中，且可由 JVM 读入
}
```

```java
package java.lang.annotation;

public enum ElementType {
    TYPE,               // 类、接口（包括注释类型）或枚举声明

    FIELD,              // 字段声明（包括枚举常量）

    METHOD,             // 方法声明

    PARAMETER,          // 参数声明

    CONSTRUCTOR,        // 构造方法声明

    LOCAL_VARIABLE,     // 局部变量声明

    ANNOTATION_TYPE,    // 注释类型声明

    PACKAGE             // 包声明
}
```


# 自定义
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
    // @interface 定义注解
    // @Target 指定 Annotation 的类型属性
    // @Retention 
}
```
