---
title: RabbitMQ 概述
date: 2021-07-16 19:42:34
tags:
- 消息队列
- RabbitMQ
---

RabbitMQ 为 AMQP 的 Erlang 实现，将可用的接口暴露出来

<!-- more -->

集群和故障转移是构建在开放电信平台框架上

RabbitMQ 相关的重要角色：
* 生产者
* 消费者
* 代理：RabbitMQ 本身，自身不产生消息

重要组件：
* ConnectionFactory（连接管理器）：应用程序与 RabbitMQ 之间建立连接的管理器，在程序代码中使用
* Channel（信道）：消息推送使用的通道
* Exchange（交换器）：用于接收、分发消息
* Queue（队列）：存储生产者的消息
* RoutingKey（路由键）：将发布者的数据按照某种规则分配到交换器上
* BindingKey（绑定键）：将交换器的消息绑定到队列上

