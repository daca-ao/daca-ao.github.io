---
title: Spring Cloud 服务注册与发现
mathjax: true
date: 2023-10-19 22:29:19
tags:
- 架构
- Spring
- Spring Cloud
- 架构
- 服务注册
- 服务发现
---

服务注册和服务发现，是微服务能立足的根本。

<!-- more -->

没有服务注册和发现的功能，何来的微服务实现呢？由此服务注册和发现是微服务的重中之重，市面上也推出了大大小小很多的解决方案。

下面来介绍两款最著名的方案。

<br/>

# Eureka

Netflix Eureka 是 [Netflex](/2023/06/09/spring-cloud-overview/#Spring-Cloud-Netflix) 的核心服务，其为基于 REST 的用于定位的服务，以实现云端中间层的服务治理，包括服务注册、服务发现和故障转移。


## 原理

Eureka 的设计为经典的 C-S 架构，包括 client 端，用于处理服务的注册和发现；以及 server 端的服务注册中心。
* 客户端服务通过**注解**和**参数配置**的方式嵌入应用代码
* 应用运行时，Eureka 客户端向注册中心注册自身提供的服务，并周期性发送心跳来更新服务租约
    * 因此 Eureka 的保护机制就是心跳机制：心跳超时的服务不再注册回注册中心
* 客户端可从服务端查询当前服务信息并**缓存**到本地，周期性刷新状态。

Eureka 包括 `Region` 和 `Zone` 两个概念。  
一个 `Region` 可以对应上多个 `Zone`，`Zone` 为对应服务的分区，相当于 Nacos 的 group。


## 使用

启动类中添加 `@EnableEurekaServer` 注解：

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

```
`@EnableEurekaServer` 注解原理：

```java
@Import(EurekaServerMarkerConfiguration)
```

Eureka 唯一的自动配置类 `EurekaServerAutoConfiguration` 的条件：

```java
@Import(EurekaServerInitializerConfiguration.class)  // 初始化
@ConditionalOnBean(EurekaServerMarkerConfiguration.Marker.class)
@EnableConfigurationProperties({EurekaDashboardProperties.class})  // 控制台类，默认路径 path = "/"
```

注册集群：

```yml
-- application-cluster1.yml
eureka:
  instance:
    hostname: cluster1
  client:
    serviceUrl:
      defaultZone: http://cluster2:8003/eureka/  # 注册到另一个 Eureka 注册中心
    fetch-registry: true
    register-with-eureka: true

-- application-cluster2.yml
eureka:
  instance:
    hostname: cluster2
  client:
    serviceUrl:
      defaultZone: http://cluster1:8003/eureka/  # 注册到另一个 Eureka 注册中心
    fetch-registry: true
    register-with-eureka: true
```


# Nacos

阿里巴巴推出的王牌组件，能够充当配置中心 + 服务发现组件，实现动态配置服务、服务发现及管理、动态 DNS 等功能。

Nacos 的大体架构如下：

* **Provider APP**：服务提供者，可通过 nacos client / sidecar 实现访问 Nacos Server；
* **Consumer APP**：服务消费者
* **Name**：通过 vip / dns / address-server 实现 Nacos 的高可用服务路由
* Nacos Server，包括：
    * **OpenAPI**：服务访问入口
    * **Config Service**：配置服务模块
    * **Naming Service**：命名服务模块，建立分布式系统中所有对象、实体之间的域名与 ip 的映射关系，实现服务发现的能力
    * **Nacos core**
    * 最底层的 **Consistency Protocol**：数据一致性协议，基于 priv-raft / sync renew / rdbms based 等

使用 Nacos 需要手动配置数据库：创建 `plugins/mysql/` 目录，并放入下载好的 mysql-connector。  
随后执行 sql 创建 `nacos_config` 数据库。


## Nacos 服务注册与发现

分别基于 HTTP 和 RPC 的通信方式，有两种实现：

**一是**基于 Nacos + [Feign + Ribbon](/2023/10/19/spring-cloud-load-balancer/#Ribbon-amp-OpenFeign) 实现：

Feign 对 HTTP 进行封装，充当 provider 服务的代理；Ribbon 实现 consumer 调用 provider 服务时的负载均衡，调用 Feign Client 完成通信

```java
@FeignClient(name="nacos-provider")
```

其底层实现的原理[大同小异](/2023/06/10/micro-service/#实现原理)：

* `NacosServiceRegistry` 实现了 `spring-cloud-commons` 包 `ServiceRegistry` 的接口方法；
* `NacosAutoServiceRegistration` 实现了 `AutoServiceRegistrationAutoConfiguration` -> `AbstractAutoServiceRegistration` 以完成自动服务注册;
* 在 `AbstractAutoServiceRegistration` 的 `bind()` 中完成注册。

**二是**基于 Dubbo + Nacos：

Consumer 调用 provider 服务时，通过 Dubbo RPC 服务调用。

这种实现方式也是基于差不多的底层原理：最终也是调用 `NacosAutoServiceRegistration` 的 `register()` -> `namingService.registerInstance()` 建立调用链。  
通过 `registerInstance()` 调用中建立心跳机制（`ExecutorService.schedule()`），随后注册服务

通过 Nacos 的 OpenAPI 调用 Nacos Server 完成注册；  
调用服务时，有服务就取出来，没有的话就调用 `createServiceIfAbsent()`，完了保存到变量 `concurrentHashMap` 中做缓存；  

同时会保持集群中服务的一致性：通过调用 `list()` 接口完成服务地址查询，获得指定服务的所有健康实例的 ip。


## Nacos 配置中心

与之对标的是 Netflix 的 Spring Cloud Config。

为了配置管理，Nacos 有一个新的概念 `namespace`：

* 包括最外层 namespace（默认为 public），中间的配置分组 group，及最里层的配置集 data id，都是为了配置的隔离；
* 每一份配置集里面有对应的配置项。

因此常用 namespace 来区分不同的测试环境（dev，test，...）。

通过注解获取配置项（`@Value("${xxx.xxx}")`）的值时，需要在 controller 类层面添加注解 `@RefreshScope` 保证配置能实时更新。

配置集群：在 `cluster.conf` 中配置集群机器的 ip 和端口，同时配置每一台机器的 `application.properties` 中的 ip 和端口。

注：nacos UI 可以生成访问示例代码。


# Dubbo + Zookeeper

除了上述提到的 Nacos 和 Eureka，Dubbo + Zookeeper 也可以实现服务注册与发现。
