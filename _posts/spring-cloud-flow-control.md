---
title: Spring Cloud 流量控制
mathjax: true
date: 2023-10-20 21:21:23
tags:
- 架构
- Spring
- Spring Cloud
- 流量控制
---

流量控制，是监控应用流量的 QPS 或者并发线程数等指标；当达到指定阈值时，对流量进行限制，必要时进行服务降级、甚至服务熔断。

<!-- more -->

# Sentinel

面向分布式服务架构的轻量级流量控制组件，主要以流量为切入点，从限流、流量整形、服务降级、系统负载保护等多个维度保障微服务稳定。

主要特征：实时监控、机器发现、规则配置

* 提供实时监控功能，包括查看单机秒级数据，设置 500 台以下规模的集群汇总运行情况
* 适用场景丰富
* 配置简单

其组成为 Java 类库 + Dashboard，将配置中心/数据库作为动态规则源并与之结合。


## 使用

使用 `SphU` 或 `SphO` 进行限流：引入 maven 依赖之后，定义资源 `SphU.entry()` / `SphO.entry()`，定义规则，查看执行结果

除此之外，还可以引入注解 `@SentinelResource` 标记在服务类的业务方法上：

* 引入依赖后，编写 `Configuration` 配置类，再编写服务类
* 需要另外添加 `processBlockHandler` 和 `fallbackHandler`

限流异常和资源清洗：

* 通过实现 `UrlBlockHandler` 接口重写 `blocked()` 方法，从而自定义 URL 限流异常
* 通过实现 `UrlCleaner` 接口重写 `clean()` 方法，进行 URL 资源清洗

集成 Spring Cloud 后可以通过 dashboard 方式启动。

可集成 [Nacos](/2023/10/19/spring-cloud-service-regiscovery/#Nacos) 保存流控规则，以免每次重启 Sentinel 后流控规则会被清空。
