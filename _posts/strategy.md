---
title: 策略模式（Strategy）
date: 2021-06-19 09:20:09
tags:
- 设计模式
---

现实中，一件事可能会有很多种方式实现，但总有一种最高效的方式。

<!-- more -->
为了让问题得以解决，我们将可以解决问题的一个或多个方法定义成一个“算法群”；其中的每一个方法对应**一组具体的算法**，我们将其称之为“**策略**”。

<br/>

<big>策略模式别名为政策（Policy），属于对象行为型模式。</big>

《设计模式：可复用面向对象软件的基础》中的介绍：

    “定义一系列算法，将他们一一封装，并使他们可以相互替换”
    “本模式使得算法能独立于使用它的客户而变化”

实现的时候把不同组的算法封装在不同的类里。

<br/>

# 结构图
![](strategy/strategy_pattern_diagram.jpeg)

以上，可见策略模式包括：

`Context`
* 环境类，上层访问策略的**入口**
* 维护一个对抽象策略 Strategy 对象的引用
* 通过一个 ConcreteStrategy 来配置
* 可定义一个接口让 Strategy 访问它的数据

`Strategy`
* 抽象策略类
* 定义所有所支持的算法的公共接口，Context 通过该接口调用某个具体策略类 ConcreteStrategy 定义的算法

`ConcreteStrategy`
* 具体策略类
* 封装具体的算法实现

例图：
![](strategy/strategy_pattern_uml_diagram.jpeg)

<br/>

# 代码示例

角色代码

```java
// Context 持有对 Strategy 的引用，并且提供了调用策略的方法
public class Context {

    // 通过抽象类接口接收不同类型的具体策略类
    private Strategy strategy;

    /**
     * 传进一个具体的策略实例
     * @param strategy
     */
    public Context(Strategy strategy) {
        this.strategy = strategy;
    }

    /**
     * 调用策略
     * 通过多态调用具体策略类的方法
     */
    public void contextInterface(int value1, int value2) {
        strategy.algorithmLogic(value1, value2);
    }
}
```

```java
// 抽象策略类，定义了策略的方法
public interface Strategy {
    public void algorithmLogic(int value1, int value2);
}
```

```java
// 具体策略角色类
public class ConcreteStrategyA implements Strategy {

    @Override
    public void algorithmLogic(int value1, int value2) {
        // 具体的算法逻辑
    }
}
```

调用代码

```java
// 客户端
public class Client {

    public static void main(String[] args) {
        // 调用某一组策略 A
        Context context = new Context(new ConcreteStrategyA());
        context.contextInterface(3, 4);
    }
}
```

<br/>

# 特点
* 一系列算法组成一个策略
* 这些算法都实现了相同的接口或继承了相同的抽象类

<br/>

# 优缺点
优点：
1. 算法可以自由切换。
2. 避免使用多重条件判断，一定程度杜绝了 if-else 带来的代码不美观。
3. 扩展性非常良好；如需新的策略，只需扩展原有抽象策略接口即可。

缺点：
1. 策略类比较容易增多。
2. 所有策略类都需要对外暴露

<br/>

# 策略模式结合简单工厂模式的应用
应用于：商店促销活动

策略上下文 Context：
```java
public class CashContext {
    CashSuper cs = null;

    public CashContext(CashSuper csuper) {
        this.cs = csuper;
    }

    public double getResult(double money) {
        return cs.acceptCash(money);
    }
}
```

策略：
```java
public interface CashSuper {

    /**
     * 抽象策略，计算实收的费用
     * @param money 应收金额
     * @return 实收金额
     */
    double acceptCash(double money);
}


// 正常收费策略
public class CashNormal implements CashSuper {

    @Override
    public double acceptCash(double money) {
        return money;
    }
}

// 打折策略
public class CashRebate implements CashSuper {

    private double moneyRebate;
    
    /**
     * @param moneyRebate 折扣
     */
    public CashRebate(double moneyRebate) {
        this.moneyRebate = moneyRebate;
    }

    @Override
    public double acceptCash(double money) {
        return money * (moneyRebate / 10);
    }
}

// 满减策略
public class CashRefund implements CashSuper {

    /**
     * 应收金额
     */
    private final double moneyCondition;

    /**
     * 返利金额
     */
    private double moneyRefund;

    public CashRefund(double moneyCondition, double moneyRefund) {
        this.moneyCondition = moneyCondition;
        this.moneyRefund = moneyRefund;
    }

    @Override
    public double acceptCash(double money) {
        if (money >= moneyCondition) {
            money -= moneyRefund;
        }
        return money;
    }
}
```

简单工厂：
```java
public class CashFactory {

    public static CashSuper createCashAccept(String cashType) {

        CashSuper cs = null;
        switch (cashType.toLowerCase()) {
            case "20discount":  // 八折
                cs = new CashRebate(8);
                break;
            case "refund100for300":  // 满 300 减 100
                cs = new CashRefund(300, 100);
                break;
            case "normal":  // 正常收费
            default:
                cs = new CashNormal();
                break;
        }
        return cs;
    }
}
```

然后就是简单的调用：
```java
CashSuper cs = CashFactory.createCashAccept("20discount");  // 打八折
double result = cs.acceptCash(300);
System.out.println(result);  // 240
```

<br/>

策略的高级玩法：**反射实现**

```java
public class CashContextReflect {

    Class<?> clazz = null;
    Object obj = null;

    public CashContextReflect(String className, Class[] paramsType, Object[] params) {
        try {
            clazz = Class.forName(className);
            Constructor con = clazz.getConstructor(paramsType);
            obj = con.newInstance(params);
        } catch (Exception e) {
            // 异常处理
        }
    }

    public double getResult(double money) {
        return ((CashSuper) obj).acceptCash(money);
    }
}


// 调用：
String type = "com.example.CashRebate";  // 调用“打八折”策略
Class[] paramTypes = {double.class};
Object[] params = {8.0};

CashContextReflect context = new CashContextReflect(type, paramTypes, params);
double money = context.getResult(300);
System.out.println(money);  // 240
```