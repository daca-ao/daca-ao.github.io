---
title: Spring Cloud 网关初探
mathjax: true
date: 2023-10-05 11:27:56
tags:
- 计算机网络
- Spring
- Spring Cloud
- 架构
---

Spring Cloud 中的网关，是一个“不显山不露水”的组件。

<!-- more -->

没网关，也能访问服务。但是问题在于：如果服务多起来了，客户端需要对应着去实现访问不同服务的逻辑，这样一来，就会增加客户端的复杂度。

如果服务需要鉴权，则每个服务的鉴权处理还都需要分别实现和维护；而且流量控制也很不好做；

另外，如果服务想同时支持 HTTP 和 RPC 或另外的多种通信方式，client 要分别维护不同语言的 SDK，相当麻烦。

# 好处

网关的好处，也就是网关的功能：

* 针对所有请求进行统一鉴权、限流、缓存、日志（打点）；
* 协议转化。针对后端多种不同的协议，在网关层统一处理之后，以 HTTP 对外提供服务；
* 针对异常请求和异常处理，对外提供统一的错误码；
* 请求转发，且可以基于自身实现内网和外网的隔离。


在实现**鉴权**的时候，网关相当于是一个统一的认证中心，访客通过网关进行交互，实现身份认证和访问权限控制；

另外在实现**灰度发布**的时候，网关与分流引擎交互，将**不同比例**的流量分发到新/旧不同的服务中：

* 前期切小部分（如 10%）流量至新服务，如果效果不好或者存在 bug 就回退；
* 否则就可以逐步增大通往新服务的流量比例，以此实现平稳过渡。

# 常见实现方案

市面上常见的微服务网关实现方案有以下几种：OpenResty, Zuul, Gateway, Kong, Apisix。下面重点介绍最常用的几套方案。

## OpenResty

Nginx + Lua 的高性能 Web 服务器，集成了大量的 Lua 库和第三方模块，可在不同阶段挂载 Lua 脚本实现不同阶段的自定义行为。

OpenResty 启动的不同的阶段包括：

* **初始化**：init_by_lua 配置加载, init_worker_by_lua 健康检查
* **Rewrite / Access**：ssl_certificate_by_lua, set_by_lua 设值, rewrite_by_lua, access_by_lua 访问
* **返回内容处理**（封装访问结果）：content_by_lua 返回内容, balancer_by_lua 负载均衡, header_filter_by_lua, body_filter_by_lua
* **日志处理阶段**：log_by_lua

优点是开放、解耦。

## Zuul

属于 Netflix 解决方案的微服务网关，核心由一系列过滤器组成。

Zuul 制定了 4 种标准类型过滤器（pre filters, routing filters, post filters, error filters），对应了请求的整个生命周期：

* pre filters：鉴权、限流
* routing filters：转发、路由
* post filters：处理结果、统计、监控等，生成日志
* error filters：顾名思义，处理请求和服务器错误

Zuul 1.x 采用的是传统的 thread per connection 方式：一旦后台响应慢，线程容易阻塞，因此不适合高并发场景。

## [Gateway](https://spring.io/projects/spring-cloud-gateway)

Spring 官方团队基于 Spring Boot 2.0 + WebFlux（Spring 5.0） + Project Reactor 这套成熟的解决方案研发的网关，目的为了是提供一种简单有效且统一的 API 路由管理方式。

Gateway 的目标是取代 Zuul，因为 Zuul 直至 2.0 版本还是使用非 Reactor 模式。

不仅提供统一的路由请求方式，还能基于 **Filter 过滤链**方式提供网关最基本的功能。

### 处理流程

在理清 Gateway 处理流程之前，我们先了解下 Gateway 组件的一些定义：

1. Route：路由
2. Predicate：路由中对于具体路径的匹配，在代码中由 RoutePredicateFactory 实现
Filter：对于请求做特殊处理（路径转换、跨域等）

客户端首先向 Spring Cloud Gateway 应用发起请求。

随后，应用会在 Gateway Handler Mapping 中找到能与请求相匹配的路由，将请求发送到 Gateway Web Handler。

Handler 再通过指定的 Filter 过滤链，将请求最终发送到实际的服务执行逻辑。

### Gateway 组件

Gateway 中会部署不同的多个 route

`spring.cloud.gateway.routes.id`: 自定义路由的唯一标识
`spring.cloud.gateway.routes.predicates`: 路径匹配
`spring.cloud.gateway.routes.filters`

此外 Gateway 提供了不同的已经封装好的 RoutePredicateFactory：

* ZonedDateTime：指定时间范围，包括 BeforeRoutePredicateFactory, AfterRoutePredicateFactory, BetweenRoutePredicateFactory
* Cookie：CookieRoutePredicateFactory
* Header：HeaderRoutePredicateFactory, CloudFoundryRouteServiceRoutePredicateFactory
* Host：HostRoutePredicateFactory
* Method：MethodRoutePredicateFactory
* Path：PathRoutePredicateFactory
* Query：QueryRoutePredicateFactory
* RemoteAddr：RemoteAddrRoutePredicateFactory, WeightRoutePredicateFactory, ReadBodyPredicateFactory

Filter 分：pre 类型过滤器（请求前鉴权、限流）、post 类型过滤器（执行完毕后处理）

实现方式：
* GatewayFilter 只会应用到单个路由或一个分组的路由上
* GlobalFilter 会应用到所有路由上

不同的 RouteFilterFactory：

* GatewayFilter: AddRequestParameterGatewayFilterFactory（pre，添加查询参数，如登录态）, AddResponseHeaderGatewayFilterFactory（post，返回前在 header 添加数据）, RequestRateLimiterGatewayFilterFactory（需要引入依赖如 redis）, RetryGatewayFilterFactory
* GlobalFilter: LoadBalancerClientFilter, GatewayMetricsFilter（指标过滤器）, ForwardRoutingFilter, NettyRoutingFilter

此外还支持自定义 GatewayFilter 和 GlobalFilter，自定义需遵循命名规范。


### 实战实例

```yaml
server:
  port: 8080
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        - id: raymond-blog
          uri: https://daca-ao.github.io/
          predicates:
            - Path=/raymond
```

# 与 Spring Cloud 其它组件的集成

可集成 [Nacos](/2023/10/19/spring-cloud-service-regiscovery/#Nacos)，结合服务发现实现请求负载均衡；

还可集成 [Sentinel](/2023/10/20/spring-cloud-flow-control/#Sentinel) 实现网关限流。
