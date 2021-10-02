---
title: 数据库事务
date: 2021-07-25 22:15:36
tags:
- 数据库
---

我们常说到的数据库**事务**（Transaction），指的是访问并可能更新数据库中各种数据项的一个程序执行单元。

<!-- more -->

事务包括一系列的数据库操作，管理着 INSERT, UPDATE, DELETE 等语句。它可以是一条或一组 SQL 语句，或者是整个程序。

按照作用域来看，数据库事务主要可分两种：
* 局部事务
    * 特定于一个单一的事务资源，如一个 JDBC 连接
    * APP 组件与资源位于同一单元
* 全局事务
    * 跨多个事务资源的事务，比如在一个分布式系统中的事务

特性：须具备 **ACID**

**A**: 原子性（**A**tomicity）
* 一个事务中的操作，要么全部完成，要么全部不完成（不做）
* 不会结束在某个中间环节
* 如果事务执行发生了错误：回滚（**rollback**）到事务开始前的状态，就如事务从未被执行过一样

**C**: 一致性（**C**onsistency）
* 事务开始前和开始后，数据库的完整性没有被破坏
* 表示写入的资料必须完全符合所有的预设规则
* 包含资料的精确度、串联性以及后续数据库可以自发性完成预定的工作

**I**: 隔离性（**I**solation)
* 数据库应拥有允许多个并发事务同时对其数据进行读写和修改的能力
* 并发事务之间不能互相干扰
* 可防止多个事务并发执行时，由于交叉执行而导致数据的不一致

**D**: 持续性（**D**urability）
* 事务处理结束后，对数据的修改是永久的，即使系统故障也不会丢失

<br/>

# 并发对数据库事务隔离性的破坏

| 并发下的问题                   | 描述   |
| :---------                   | :---- |
| 脏写                          | 事务回滚了其它事务对数据项的已提交修改   |
| 丢失更新                      | 事务覆盖了其它事务对数据的已提交修改     |
| 脏读 Dirty Read               | 一个事务读取了另一个**未提交**的事务中的数据 |
| 不可重复读 Non-repeatable Read | 对于数据库中某个数据，在同一个事务范围内多次查询却返回不同数据值<br/>因为在查询间隔，该数据被另一个事务修改并提交了 |
| 幻读（多行的不可重复读）Phantom  | 一个事务多次查询之间的数据的条数不一致<br/>针对**一批数据整体**出现的问题 |

<br/>

## 脏写

| 脏写示意  | Session A | Session B |
| :-----:  | :-------  | :-------  |
| T1       | begin;    |           |
| T2       | select * from car where id = 1; | |
|          | （比如此时读出来的是 mazda）        | |
| T3       |           | begin;    |
| T4       |           | update car set name='honda' where id = 1; |
| T5       |           | commit;（脏写）|
| T6       | rollback; |           |

会话 A rollback 后，car 记录会被回滚到 "mazda"，而不是已经被 B 会话提交的 "honda"，这就造成了脏写。

<br/>

## 丢失更新

| 丢失更新示意  | Session A | Session B |
| :-----:     | :-------  | :-------  |
| T1          | begin;    |           |
| T2          |           | begin;    |
| T3          |           | select * from car where id = 1; |
| T4          | select * from car where id = 1;   | |
|             | （比如此时两个会话读出来的都是 mazda） | |
| T5          | update car set name='honda' where id = 1;  | |
| T6          |           | update car set name='mitsubishi' where id = 1; |
| T7          | commit;   |           |
| T8          |           | commit;   |

会话 B 的 "mitsubishi" 更新会因为会话 A 的提交而丢失。

<br/>

## 脏读

| 脏读示意  | Session A | Session B |
| :-----:  | :-------  | :-------  |
| T1       | begin;    |           |
| T2       |           | begin;    |
| T3       |           | update car set name='audi' where id = 1; |
| T4       | select * from car where id = 1; | |
| T5       | commit;   | |
| T6       |           | rollback; |

T4 的时候读取到的结果如果是 "audi"，就出现了脏读，因为会话 B 将事务回退了。

<br/>

## 不可重复读

| 不可重复读示意  | Session A | Session B |
| :-----:       | :-------  | :-------  |
| T1            | begin;    |           |
| T2            | select * from car where id = 1; | |
|               | （比如此时会话读出来的是 mazda）    | |
| T3            |           | begin;    |
| T4            |           | update car set name='mitsubishi' where id = 1; |
| T5            |           | commit;   |
| **T6**        | select * from car where id = 1; | |
|               | （此时会话读出来的是 mitsubishi）   | |
| T7            |           | begin;    |
| T8            |           | update car set name='honda' where id = 1; |
| T9            |           | commit;   |
| **T10**       | select * from car where id = 1; | |
|               | （此时会话读出来的是 honda）        | |

T6 和 T10 都是不可重复读。

脏读 v.s. 不可重复读
* 脏读读取的是其它事务**未提交**的脏数据
* 不可重复读读取的是其它事务**已经提交**的数据

<br/>

## 幻读

| 幻读示意  | Session A | Session B |
| :-----:  | :-------  | :-------  |
| T1       | begin;    |           |
| T2       | select * from car where id > 0; | |
|          | （比如此时会话读出来的是 mazda）    | |
| T3       |           | begin;    |
| T4       |           | insert into car(name) values('audi') |
| T5       | select * from car where id > 0; | |

如果 T5 读取出来的值是 ('mazda', 'audi')，就出现了幻读。

幻读 v.s. 不可重复读
* 不可重复读针对确定的**某一行**数据
* 幻读针对的是不确定的**多行**数据（Result Set）

<br/>

因此我们才设定了事务之间的隔离级别：

# 事务隔离级别

逐级提高：

<big>**读未提交**</big>（**Read Uncommitted**）：所有事务都可看到其它事务修改过但未提交的内容数据。
* 可能发生脏读、不可重复读和幻读问题
* 没有解决任何并发问题，不常用

| 读未提交示意  | Session A | Session B |
| :-------:   | :-------  | :-------  |
| T0          | （比如此时 id = 1 的是 honda） | |
| T1          | begin;    |           |
| T2          | update car set name='mazda' where id = 1; | begin; |
| T3          |           | select * from car where id = 1;        |
|             |           | （此时会话读出来的是 mazda）               |
| T4          | commit;   |           |
| T5          |           | select * from car where id = 1;        |
|             |           | （此时会话读出来的是 mazda）               |

<br/>

<big>**读已提交**</big>（**Read Committed**）：一个事务只能读取其它事务已经提交的内容数据
* 如：事务 B 只能在事务 A 修改过并且提交之后，才能读取到事务 B 修改的数据
* 解决了脏读问题，但没有解决不可重复读和幻读，也不常用
* Oracle 的默认级别

| 读已提交示意  | Session A | Session B |
| :-------:   | :-------  | :-------  |
| T0          | （比如此时 id = 1 的是 honda）               |        |
| T1          | begin;    |           |
| T2          | update car set name='mazda' where id = 1; | begin; |
| T3          |           | select * from car where id = 1;        |
|             |           | （此时会话读出来的是 honda）               |
| T4          | commit;   |           |
| T5          |           | select * from car where id = 1;        |
|             |           | （此时会话读出来的是 mazda）               |

<br/>

<big>**可重复读**</big>（**Repeatable Read**）：保证一个事务之间的多个实例在**并发**下能读取同一数据
* 对同一字段的多次读取结果是一样的，除非数据被自身事务修改
* MySQL InnoDB 的默认级别

| 可重复读示意  | Session A | Session B | Session C |
| :-------:   | :-------  | :-------  | :-------  |
| T0          | （比如此时 id = 1 的是 honda）               |        |
| T1          | begin;    |           | begin;    |
| T2          | update car set name='mazda' where id = 1; | begin; |        |
| T3          | select * from car where id = 1;           | select * from car where id = 1;        | select * from car where id = 1;        |
|             | （此时会话读出来的是 mazda）                  | （此时会话读出来的是 honda）               | （此时会话读出来的是 honda）               |
| T4          | commit;   |           |           |
| T5          |           | commit;   | commit;   |
| T6          |           | select * from car where id = 1;        | |
|             |           | （此时会话读出来的是 mazda）               | |

<br/>

<big>**串行化**</big>（**Serializable**）：事务之间只能顺序执行，使之没有任何冲突
* 最高级别
* 通过加锁实现（读锁和写锁）

串行化规范对于不同的情景有不同的阻塞操作：

|              | 事务 A 读操作 | 事务 A 写操作 |
| :---------:  | :---------  | :---------   |
| 事务 B 读操作  | 不阻塞       | 阻塞         |
| 事务 B 写操作  | 阻塞        | 阻塞          |

<br/>

**读读操作不阻塞**：

|       | Session A | Session B |
| :---: | :-------  | :-------  |
| T0    | （比如此时 id = 1 的是 honda）     |           |
| T1    | begin;    |           |
| T2    | select * from car where id = 1; | begin;    |
|       | （此时会话读出来的是 honda）        |           |
| T3    |           | select * from car where id = 1; |
|       |           | （此时会话读出来的是 honda）        |

事务 A 的读操作不会阻塞事务 B 的读操作。

<br/>

**读写操作阻塞**：

|       | Session A | Session B |
| :---: | :-------  | :-------  |
| T1    | begin;    |           |
| T2    | select * from car where id = 1; | begin;    |
|       | （此时会话读出来的是 honda）        |           |
| T3    |           | update car set name='mazda' where id = 1; |
|       |           | （会话 B 修改操作被阻塞）           |
| T4    | commit;   |           |
| T5    |           | 此时写操作才会被执行               |

事务 A 的读操作会阻塞事务 B 的写操作。

<br/>

**写读操作阻塞**：

|       | Session A | Session B |
| :---: | :-------  | :-------  |
| T0    | （比如此时 id = 1 的是 honda）     |           |
| T1    | begin;    |           |
| T2    | update car set name='mazda' where id = 1;   | begin;    |
| T3    |           | select * from car where id = 1; |
|       |           | （会话 B 读取操作被阻塞）           |
| T4    | commit;   |           |
| T5    |           | 此时读操作才会被执行，查到的是 mazda  |

事务 A 的写操作会阻塞事务 B 的读操作。

<br/>

**写写操作阻塞**：

|       | Session A | Session B |
| :---: | :-------  | :-------  |
| T0    | （比如此时 id = 1 的是 mitsubishi）     |           |
| T1    | begin;    |           |
| T2    | update car set name='mazda' where id = 1;   | begin;    |
| T3    |           | update car set name='honda' where id = 1;   |
|       |           | （会话 B 修改操作被阻塞）           |
| T4    | commit;   |           |
| T5    | select * from car where id = 1; | 此时写操作才会被执行      |
|       | （此时会话读出来的是 mazda）        |           |
| T6    |           | select * from car where id = 1; |
|       |           | （此时会话读出来的是 honda）        |
| T7    |           | commit;   |

事务 A 的写操作会阻塞事务 B 的写操作。

<br/>

级别越高，越能保证数据完整性和一致性，对并发的效率也越低。

| 对象状态  | 脏读  | 不可重复读 | 幻读   |
| :-----  | :---- | :-----   | :----- |
| 读未提交  | <font color="red">可能</font>   | <font color="red">可能</font>   | <font color="red">可能</font>  |
| 读已提交  | <font color="green">不会</font> | <font color="red">可能</font>   | <font color="red">可能</font>  |
| 可重复读  | <font color="green">不会</font> | <font color="green">不会</font> | <font color="red">可能</font>  |
| 串行化   | <font color="green">不会</font> | <font color="green">不会</font> | <font color="green">不会</font> |
