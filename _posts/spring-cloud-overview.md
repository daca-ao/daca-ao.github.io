---
title: Spring Cloud 初探
mathjax: true
date: 2023-06-09 16:18:14
tags:
- 架构
- Spring
- Spring Cloud
---

Spring Cloud 和 Dubbo 都是目前主流的微服务框架。这篇主要聊聊 Spring Cloud 的一些组成情况。

<!-- more -->

# 概述

Spring Cloud 是一个基于 Spring Boot 实现的开源微服务架构开发工具。演进至今，衍生出了主流的两套框架：


## Spring Cloud Netflix

包括以下组件：

* **Spring Cloud Config**：分布式系统中统一的外部集中配置管理工具，默认使用 Git 存储；支持客户端配置信息的刷新、加密、解密操作。
* **Spring Cloud Netflix**：核心组件，对多个 Netflix OSS 开源套件进行整合
    * [Eureka](/2023/10/19/spring-cloud-service-regiscovery/#Eureka)：负责服务治理，包括服务注册与发现，已停止维护
    * [Zuul](/2023/10/05/spring-cloud-gateway/#Zuul)：服务网关，提供智能路由、访问过滤等功能；其替代品是 [Gateway](/2023/10/05/spring-cloud-gateway/#Gateway)
    * Ribbon：负载均衡
    * Hystrix：容错管理组件，实现断路器模式，提供服务熔断和限流
    * Feign：基于 Ribbon 和 Hystix 的声明式服务调用组件，负责远程服务的客户端代理
    * Tuibine：将各个服务实例上的 Hystrix 监控信息统一整合
    * Archaius：外部化配置组件
* **Spring Cloud Bus**：用于传播集群状态变化的消息总线，使用轻量级消息代理链接分布式系统中的节点，可以用来动态刷新集群中的服务配置。
* **Spring Cloud Consul**：基于 Hashicorp Consul 的服务治理组件。
* **Spring Cloud ZooKeeper**：基于 Apache ZooKeeper 的服务治理组件。
* **Spring Cloud Cluster**：针对 ZooKeeper, [Redis](/2021/07/29/redis-basics), Hazelcast, Consul 的选举算法和通用状态模式的实现。
* **Spring Cloud Security**：Spring Security 组件封装，提供用户验证和权限验证，对 Zuul 代理中的负载均衡 OAuth2 客户端及登陆认证进行支持。
* **Spring Cloud Sleuth**：分布式链路追踪组件，支持使用 HTrace, Zipkin 和基于 [Elastic Stack](/2022/02/09/es/#ELK) 的跟踪。
* **Spring Cloud Stream**：轻量级事件驱动微服务框架，使用简单的声明式模型发送接收消息。主要实现是 Kafka 和 [RabbitMQ](/2021/07/21/rabbitmq)。
* ...

优点是产品成熟，缺点是已经停止更新了，且可视化界面缺少。


## Spring Cloud Alibaba

从 2018 开始推出并迅速发展和成熟，已经在多个组织里面得到了广泛的应用。

包括以下组件：

* [Nacos](/2023/10/19/spring-cloud-service-regiscovery/#Nacos)：分布式配置中心、服务注册与发现
* RocketMQ：消息驱动组件
* [Sentinel](//#Sentinel)：服务容错组件，包括流量控制与服务降级
* Seate：分布式事务
* Dubbo：结合到 Spring Cloud 中，便是以 RPC 为框架的远程调用组件

相对于 Spring Cloud Netflix，优点是产品能够持续迭代，配置相对简单；而缺点是欠缺稳定性。


# Spring Cloud v.s. Dubbo

|   功能     |           Spring Cloud             |   Dubbo        |
| :--------  | :--------------------------------- | :------------- |
| 服务注册中心 | Eureka, Consul, ZooKeeper, Nacos   | ZooKeeper      |
| 服务调用方式 | REST，基于 HTTP                     | RPC，基于 Dubbo |
| 服务监控    | Spring Boot Admin                  | Dubbo-Monitor  |
| 熔断器      | Spring Cloud Netflix Hystrix       | 不完善          |
| 服务网关    | Spring Cloud Netflix Zuul, Gateway | 无              |
| 分布式配置  | Spring Cloud Config, Nacos         | 无              |
| 链路跟踪    | Spring Cloud Sleuth + Zipkin       | 无              |
| 数据流     | Spring Cloud Stream                 | 无              |
| 批量任务    | Spring Cloud Task                  | 无              |
| 信息总线    | Spring Cloud Bus                   | 无              |
