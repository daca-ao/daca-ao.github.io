---
title: 观察者模式（Observer）
date: 2021-06-22 14:22:58
tags:
- 设计模式
---

观察者模式属于对象行为型模式。

<!-- more -->

《设计模式：可复用面向对象软件的基础》中的介绍：

    “定义对象之间的一种一对多的依赖关系，以便当一个对象的状态发生改变时，所有依赖它的对象都得到通知并自动更新”


# 结构图
![](observer/observer-diagram.gif)

以上，可见观察者模式包括：

`Subject`
* 观察目标（主题），发生改变的对象
* 将所有对观察者对象的引用保存到一个集合里
* 每一个主题都可以有多个观察者

`ConcreteSubject`
* 具体主题
* 将有关状态存入具体观察者对象
* 在具体主题发生改变时，给所有观察者发出通知

`Observer`
* 观察者，被通知的对象
* 为所有的具体观察者定义一个接口
* 在得到主题的通知时能及时更新自己

`ConcreteObserver`
* 具体观察者
* 实现抽象观察者角色所要求的更新接口
* 以便使本身状态与主题状态相协调

一个观察目标对应多个观察者，而观察者之间没有相互联系。


# 示例代码
观察者抽象类：
```java
public abstract class Observer {

    protected Subject subject;

    public abstract void update();
}
```

具体观察者的实现：
```java
public class BinaryObserver extends Observer {

    public BinaryObserver(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

    @Override
    public void update() {
        System.out.println("Binary String: " + Integer.toBinaryString(subject.getState()));
    }
}

...
```

目标实现：
```java
public class Subject {

    private List<Observer> observers = new ArrayList<>();
    private int state;

    public void attach(Observer observer) {
        observers.add(observer);
    }

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
        notifyAllObservers();
    }

    void notifyAllObservers() {
        for (Observer observer : observers) {
            observer.update();
        }
    }
}
```

调用：
```java
Subject subject = new Subject();

new HexaObserver(subject);
new OctalObserver(subject);
new BinaryObserver(subject);

System.out.println("First state change: 15");
subject.setState(15);
System.out.println("Second state change: 10");
subject.setState(10);
```

<br/>

# 优缺点
优点
1. 观察者和被观察者是**抽象耦合**的。
2. 建立一套触发机制。

缺点
1. 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
2. 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。
3. 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。
