---
title: Spring Cloud 负载均衡
mathjax: true
date: 2023-10-19 23:57:36
tags:
- 架构
- Spring
- Spring Cloud
- 负载均衡
---

负载均衡，是微服务能够健壮持续运行的关键。

<!-- more -->

在 Spring Cloud 中，通常结合 Ribbon 和 OpenFeign 实现微服务之间的通信和负载均衡。

# Ribbon & OpenFeign

Spring Cloud Ribbon 基于 Netflix Ribbon 实现，需要配合 RestTemplate, Feign 等通信组件共同完成请求负载均衡


## RestTemplate

RestTemplate 是 Spring 3.0 支持的 HTTP 请求工具。

```java
class RestTemplate extends InterceptingHttpAccessor implements RestOperations
```

提供常见的 REST 请求方案的模板，以及一些通用的请求执行方法 `exchange()` 和 `execute()`，可以平替比较繁琐的 Apache HttpClient。


## 实现

1. `DiscoveryClient`（[Nacos](/2023/10/19/spring-cloud-service-regiscovery/#Nacos) 实现的 `NacosDiscoveryClient`） + 自实现负载均衡
2. `LoadBalancerClient`（Ribbon 实现的 `RibbonLoadBalancerClient`）
3. `@LoadBalanced`

Ribbon 负载均衡策略：

* RoundRobin
* Random
* AvailabilityFiltering
* WeightedResponse
* Retry
* BestAvailable

`OpenFeign` 组件 = Feign，是声明式的伪 http 客户端，底层使用的是 RestTemplate，默认实现了负载均衡。

Feign 与 Dubbo 的调用区别：

|         | Dubbo | Feign |
| ------  | ----- | ----- |
| 传输协议 | TCP   | TCP   |
| 开发语言 | Java  | 不限   |
| 性能     | 好    | 一般   |
| 灵活性   | 一般   | 好    |
