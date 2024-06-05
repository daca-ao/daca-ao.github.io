---
title: Spring 依赖注入概述
date: 2021-12-24 07:15:25
tags:
- Spring
- IoC
---

在 [Spring IoC 概览](/2021/12/05/spring-ioc/)中我们提到过，Spring IoC 容器主要是通过 DI 实现并管理的。

<!-- more -->

本帖基于通过**配置文件**管理 Bean 的方式，简单地介绍一下 Spring IoC 容器在不同注入方向中的具体使用。

<br/>

# 配置文件

Spring 项目需要一份 XML 文件定义容器需要管理的所有 Bean，基本格式如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="golf" class="com.volkswagen.jeta">
        <property name="jeta" ref="ea888"/>
    </bean>

    <bean id="ea888" class="com.volkswagen.ea888">
    </bean>

</beans>
```

只要能被找到，配置文件可以放在项目工程中的任意位置，且可以任意命名。一般都是命名为 `applicationContext.xml`，置于 `/src` 目录下。  
Spring Framework 利用该文件创建所有已经在配置文件中定义的 Bean，按照其中标签的定义，给分配唯一的 ID。

<br/>

# 基于 setter 的依赖注入

常用方法，用于 Bean 的无参构造函数或无参 static 工厂方法的实例化。

举个例子：

```java
package com.example;


public class TextEditor {

    private SpellChecker spellChecker;

    // a setter method to inject the dependency.
    public void setSpellChecker(SpellChecker spellChecker) {
        System.out.println("Inside setSpellChecker.");
        this.spellChecker = spellChecker;
    }

    // a getter method to return spellChecker
    public SpellChecker getSpellChecker() {
        return spellChecker;
    }

    public void spellCheck() {
        spellChecker.checkSpelling();
    }
}


// dependency
public class SpellChecker {

    public SpellChecker() {
        System.out.println("Inside SpellChecker constructor.");
    }

    public void checkSpelling() {
        System.out.println("Inside checkSpelling.");
    }
}
```

对应的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>

    <!-- Definition for textEditor bean -->
    <bean id="textEditor" class="com.example.TextEditor">
        <property name="spellChecker" ref="spellChecker"/>
    </bean>

    <!-- Definition for spellChecker bean -->
    <bean id="spellChecker" class="com.example.SpellChecker">
    </bean>

</beans>
```

```java
// 调用：
TextEditor te = (TextEditor) context.getBean("textEditor");
te.spellCheck();


// 输出：
// Inside SpellChecker constructor.
// Inside setSpellChecker.
// Inside checkSpelling.
```

另外，我们还可以使用 `p-namespace`：

```xml
<!-- encoding... -->
<beans ...
    xmlns:p="http://www.springframework.org/schema/p">

    <!-- Definition for textEditor bean -->
    <bean id="textEditor" class="com.example.TextEditor"
        p:spellChecker-ref="spellChecker"/>
    </bean>

    ...

</beans>
```

`-ref` 表明这不是一个直接的值，而是对另一个 bean 的**引用**。

或者可以将配置文件改为注入内部 bean（内部类）：

```xml
<!-- encoding... -->
<beans ...>

    <!-- Definition for textEditor bean using inner bean -->
    <bean id="textEditor" class="com.example.TextEditor">
        <property name="spellChecker">
            <bean id="spellChecker" class="com.example.SpellChecker"/>
        </property>
    </bean>

</beans>
```

## 注入集合

本质上是基于 setter 的依赖注入。

| 元素       | 描述   |
| --------  | ------ |
| `<list>`  | 线性表  |
| `<set>`   | 不重复线性表         |
| `<map>`   | 键值对，可为任何类型  |
| `<props>` | 键值对，须为字符串类型 |

先上配置文件 `Beans.xml` 看一看：

```xml
<!-- encoding... -->
<beans ...>

    <!-- Definition for javaCollection -->
    <bean id="carBrands" class="com.example.CarBrand">

        <!-- results in a setBrandList(java.util.List) call -->
        <property name="brandList">
            <list>
                <value>Mazda</value>
                <value>Honda</value>
                <value>Ford</value>
                <value>Cadillac</value>
            </list>
        </property>

        <!-- results in a setBrandSet(java.util.Set) call -->
        <property name="brandSet">
            <set>
                <value>Toyota</value>
                <value>Bentley</value>
                <value>BMW</value>
                <value>Geely</value>
            </set>
        </property>

        <!-- results in a setBrandMap(java.util.Map) call -->
        <property name="brandMap">
            <map>
                <entry key="1" value="Mini"/>
                <entry key="2" value="Trumpchi"/>
                <entry key="3" value="Nissan"/>
                <entry key="4" value="Chevrolet"/>
            </map>
        </property>

        <!-- results in a setBrandProp(java.util.Properties) call -->
        <property name="brandProp">
            <props>
                <prop key="one">Jaguar</prop>
                <prop key="two">Audi</prop>
                <prop key="three">BYD</prop>
                <prop key="four">Benz</prop>
            </props>
        </property>

    </bean>

</beans>
```

Bean 的定义：

```java
package com.example;


public class CarBrand {

    List brandList;
    Set brandSet;
    Map brandMap;
    Properties brandProp;


    // a setter method to set List
    public void setBrandList(List brandList) {
        this.brandList = brandList;
    }

    // prints and returns all the elements of the list.
    public List getBrandList() {
        System.out.println("List Elements :" + brandList);
        return brandList;
    }

    // a setter method to set Set
    public void setBrandSet(Set brandSet) {
        this.brandSet = brandSet;
    }

    // prints and returns all the elements of the Set.
    public Set getBrandSet() {
        System.out.println("Set Elements :" + brandSet);
        return brandSet;
    }

    // a setter method to set Map
    public void setBrandMap(Map brandMap) {
        this.brandMap = brandMap;
    }

    // prints and returns all the elements of the Map.
    public Map getBrandMap() {
        System.out.println("Map Elements :" + brandMap);
        return brandMap;
    }

    // a setter method to set Property
    public void setBrandProp(Properties brandProp) {
        this.brandProp = brandProp;
    }

    // prints and returns all the elements of the Property.
    public Properties getBrandProp() {
        System.out.println("Property Elements :" + brandProp);
        return brandProp;
    }
}
```

调用：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");

CarBrand jc = (CarBrand) context.getBean("carBrands");
jc.getBrandList();
jc.getBrandSet();
jc.getBrandMap();
jc.getBrandProp();
```


## 注入 Bean 引用

```xml
<!-- encoding... -->
<beans ...>

    <!-- Bean Definition to handle references and values -->
    <bean id="..." class="...">

        <!-- Passing bean reference for java.util.List -->
        <property name="brandList">
            <list>
                <ref bean="brand1"/>
                <ref bean="brand2"/>
                <value>Benz</value>
            </list>
        </property>

        <!-- Passing bean reference for java.util.Set -->
        <property name="brandSet">
            <set>
                <ref bean="brand1"/>
                <ref bean="brand2"/>
                <value>BMW</value>
            </set>
        </property>

        <!-- Passing bean reference for java.util.Map -->
        <property name="brandMap">
            <map>
                <entry key="one" value="Porsche"/>
                <entry key ="two" value-ref="brand1"/>
                <entry key ="three" value-ref="brand2"/>
            </map>
        </property>

    </bean>

</beans>
```

## 注入空值或 null

```xml
<bean id="..." class="exampleBean">
    <property name="email" value=""/>
</bean>

<bean id="..." class="exampleBean">
    <property name="email"><null/></property>
</bean>
```

小结：使用 `ref` 进行引用传递，使用 `value` 进行值传递。

<br/>

# 基于构造函数的依赖注入

```java
package com.example;


public class TextEditor {

    private SpellChecker spellChecker;

    public TextEditor(SpellChecker spellChecker) {
        System.out.println("Inside TextEditor constructor.");
        this.spellChecker = spellChecker;
    }

    public void spellCheck() {
        spellChecker.checkSpelling();
    }
}

public class SpellChecker {

    public SpellChecker() {
        System.out.println("Inside SpellChecker constructor.");
    }

    public void checkSpelling() {
        System.out.println("Inside checkSpelling.");
    }
}
```

配置文件大同小异：

```xml
<!-- encoding... -->
<beans ...>

    <!-- Definition for textEditor bean -->
    <bean id="textEditor" class="com.example.TextEditor">
        <constructor-arg ref="spellChecker"/>  <!-- “构造函数的引用” -->
    </bean>

    <!-- Definition for spellChecker bean -->
    <bean id="spellChecker" class="com.example.SpellChecker">
    </bean>

</beans>
```

```java
// 调用
TextEditor te = (TextEditor) context.getBean("textEditor");
te.spellCheck();


// 输出
// Inside SpellChecker constructor.
// Inside TextEditor constructor.
// Inside checkSpelling.
```

如果构造函数里面不止一个形参，应该怎么解析？

```xml
<!-- 如不止一个参数：需要按照形参的顺序提供参数 -->
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>
    
    <bean id="bar" class="x.y.Bar"/>
    <bean id="baz" class="x.y.Baz"/>
</beans>
```

```xml
<!-- 也可使用 type 属性显式指定构造函数的形参类型（用于基本数据类型和 String） -->
<beans>
    <bean id="exampleBean" class="com.examples.ExampleBean">
        <constructor-arg type="int" value="2001"/>
        <constructor-arg type="java.lang.String" value="Zara"/>
    </bean>
</beans>


<!-- 最好能使用 index 属性显式指定顺序 -->
<beans>
    <bean id="exampleBean" class="com.examples.ExampleBean">
        <constructor-arg index="0" value="2001"/>
        <constructor-arg index="1" value="Zara"/>
    </bean>
</beans>
```

<br/>

# Spring Bean 的装配

决定了 bean 在 Spring 容器中组合在一起的方式。  
Spring 容器需要知道需要什么 bean，以及容器应该如何使用 DI 将 bean 绑定在一起，同时将 bean 装配好。


## Bean 自动装配

自动装配能让容器在不使用 `<constructor-arg>` 和 `<property>` 的时候能够**自动装配**（auto-wire）相互协作的 bean。  
这样可以减少 xml 配置数量。

自动装配的本质是基于 setter（`byName`、`byType`）和构造函数（constructor）的依赖注入。

| 模式           | 描述   |
| ------------  | ------ |
| `no`          | 默认配置，即没有自动装配，需通过 ref 显式引用。 |
| `byName`      | 根据 `<bean>` 中某个 `<property>` 的 `name`（不是 property 的类名 class）自动装配。<br/>如：某个 bean 中有一个名为 master 的属性（对应 setMaster() 的 setter），则容器会查找名字为 master（`name="master"`）的 bean 并装配。 |
| `byType`      | 根据 property 的数据类型（type），查找与其有 IS-A 关系的 bean，并自动装配。<br/>* 多个匹配会抛出 `fatal exception`；</br>* 没有匹配的话，bean 的 property 不会被装配。 |
| `constructor` | 根据构造方法的参数的数据类型，进行 byType 模式的自动装配。 |
| `autodetect`  | 如发现默认的构造方法，则用 constructor 模式；否则使用 byType 模式。 |

基于 setter 装配的示例：

```java
public class TextEditor {

    private SpellChecker spellChecker;
    private String name;

    public void setSpellChecker(SpellChecker spellChecker) {
        this.spellChecker = spellChecker;
    }

    ...
}
```

```xml
<!-- 显式装配 -->
<!-- Definition for textEditor bean -->
<bean id="textEditor" class="com.example.TextEditor">
    <property name="spellChecker" ref="spellChecker" />
    <property name="name" value="Generic Text Editor" />
</bean>


<!-- 自动装配 -->
<!-- Definition for textEditor bean -->
<bean id="textEditor" class="com.example.TextEditor"
      autowire="byName">  <!-- 寻找名字（byName）为 spellChecker 的 bean -->
    <property name="name" value="Generic Text Editor" />
</bean>

<!-- 或： -->
<bean id="textEditor" class="com.example.TextEditor"
      autowire="byType">  <!-- 寻找类型（byType）为 SpellChecker 的 bean -->
    <!-- 注：如配置文件中存在另一个 class 为 com.example.SpellChecker 或此类的子类，会报错 -->
    <property name="name" value="Generic Text Editor" />
</bean>
```

基于构造函数装配的示例：

```java
public class TextEditor {

    private SpellChecker spellChecker;
    private String name;

    public TextEditor(SpellChecker spellChecker, String name) {
        this.spellChecker = spellChecker;
        this.name = name;
    }

    ...
}
```

```xml
<!-- 自动装配 -->
<!-- Definition for textEditor bean -->
<bean id="textEditor" class="com.example.TextEditor"
      autowire="constructor">  <!-- 寻找类型为 SpellChecker 的 bean -->
    <constructor-arg value="Generic Text Editor" />
</bean>
```

自动装配的限制：
* 自动装配不如显式装配精确；
* 自动装配随时会被 `<constructor-arg>` 或 `<property>` 覆写；
* 基本数据类型（primitive）、字符串、类等简单属性不能被自动装配。

一个比较好的选择是：用构造器参数实现强制依赖，用setter 方法实现可选依赖。
