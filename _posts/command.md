---
title: 命令模式（Command）
date: 2021-06-19 11:12:20
tags:
- 设计模式
---

命令模式别名为动机（Action）、事务（Transaction），属于对象行为型模式。

<!-- more -->

在《设计模式：可复用面向对象软件的基础》中的介绍：

    “将一个请求封装为一个对象，从而可以用不同的请求对客户进行参数化、对请求排队或者记录请求日志，以及支持可取消的操作”

即：方便于使用不同的请求、队列或日志来参数化其它对象，同时支持可撤销操作。

<br/>

# 意图
有时候需要向某个对象发送一个请求，但：
* 不知道接收该请求的对象（接收者）是谁；
* 不知道具体处理过程，被请求到的操作是哪（几）个。

在命令模式中，设计好的一堆命令会放到一个“编排器”中，“编排器”规定了命令的执行顺序。  
按照命令模式的开发流程，只需要知道程序运行中指定的具体请求接收者即可。


# 动机
* 可使得请求的发送者和接收者之间实现完全的解耦；
* 发送者和接收者之间没有直接联系：发送者只需要知道如何发送请求命令即可，其它一概不管。


# 结构
![](command/commanduml.jpeg)

以上，可见命令模式包括以下角色：

`Command`
* 抽象命令类
* 声明执行操作的接口（execute()）

`ConcreteCommand`
* 具体命令类
* 将一个接收者 Receiver 对象绑定于一个动作，调用 Receiver 相应的操作，以实现 execute()
* 如没有绑定 Receiver 对象：则操作的逻辑在该具体命令类中定义

`Invoker`
* 调用者
* 定义某一个请求过来的时候，对应有一个命令执行该请求

`Receiver`
* 接收者
* 定义了多个实际的与不同的请求相关的操作
* 任何类都可能成为一个接收者
* **可有可无**：视乎业务逻辑；如不定义接收者类：将操作逻辑上提到具体命令类

`Client`
* 客户类
* 通过调用调用者 Invoker 执行命令

<br/>

# 实现
* Client 和 Receiver 需要同时开发
* 在开发过程中，随着具体命令 ConcreteCommand 的变化，Client 和 Receiver 分别需要不停地重构、改名
* Receiver 是实际的执行类，执行与请求相关的操作
    * concreteCommand.execute() 调用 Receiver 对应的操作，然后 Client 通过调用 Involker 实例，执行 concreteCommand.execute()
* Invoker 对象一般由一个或多个具体命令类构造而成
* 如要实现可撤销功能（redo & undo），需要命令不按照调用执行，而是按照执行时的情况排序并执行

<br/>

# 示例代码

调用者：
```java
public class Invoker {

    private Command commandOne;
    private Command commandTwo;

    // 该调用者仅接收两个命令
    public Invoker(Command commandOne, Command commandTwo) {
        this.commandOne = commandOne;
        this.commandTwo = commandTwo;
    }

    public void actionOne() {
        commandOne.execute();
    }

    public void actionTwo() {
        commandTwo.execute();
    }
}
```

接收者：具体做事情的类
```java
public class Receiver {

    public Receiver() {
        //
    }

    public void actionOne() {
        System.out.println("ActionOne has been taken.");
    }

    public void actionTwo() {
        System.out.println("ActionTwo has been taken.");
    }
}
```

命令类：间接或直接做事情的类
```java
public interface Command {
    void execute();
}

public class ConcreteCommandOne implements Command {

    private Receiver receiver;

    public ConcreteCommandOne(Receiver receiver) {
        this.receiver = receiver;
    }

    public void execute() {
        receiver.actionOne();
    }
}

public class ConcreteCommandTwo implements Command {

    private Receiver receiver;

    public ConcreteCommandTwo(Receiver receiver) {
        this.receiver = receiver;
    }

    public void execute() {
        receiver.actionTwo();
    }
}
```

客户类：
```java
public class Client {

    public static void main(String[] args) {
        Receiver receiver = new Receiver();
        Command commandOne = new ConcreteCommandOne(receiver);
        Command commandTwo = new ConcreteCommandTwo(receiver);
        Invoker invoker = new Invoker(commandOne, commandTwo);

        // 客户完全不需要管执行到的具体命令和顺序
        invoker.actionOne();
        invoker.actionTwo();
    }
}
```

Redo & Undo：
```java
public class ConcreteCommandOne implements Command {

    private Receiver receiver;
    private Receiver lastReceiver;

    public ConcreteCommandOne(Receiver receiver) {
        this.receiver = receiver;
    }

    public void execute() {
        record();
        receiver.actionOne();
    }

    public void record() {
        // 记录状态
    }

    public void undo() {
        // 恢复状态
    }

    public void redo() {
        lastReceiver.actionOne();
    }
}
```
具体可用 List，Queue，图等实现方式。

<br/>

# 优缺点
优点：
* 降低系统的耦合度
* 新的命令可以很容易地加入到系统中
* 可以比较容易地设计一个命令队列和宏命令（组合命令），即解决好一系列命令的安排
* 可以方便地实现对请求的 Undo 和 Redo

缺点：
* 可能会导致某些系统有过多的具体命令类：因为针对每一个命令都需要设计一个具体命令类
* 因此某些系统可能需要大量具体命令类，这将影响命令模式的使用
