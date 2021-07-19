---
title: 远程过程调用（RPC）
date: 2021-07-17 20:02:32
tags:
- 计算机网络
- RPC
---

远程过程调用，全称 Remote Procedure Call Protocol，RPC。

<!-- more -->

* 实现背景：分布式
* 通过网络从远程计算机上请求调用某种服务
* 一种通过网络从远程计算机程序请求服务，而不需要了解底层网络技术的思想
    * 并不是一种规范或协议，只是一种思想
    * 不依赖于具体的网络传输协议，TCP、UDP 等都可以
    * 解决分布式系统中，服务之间的调用问题
    * 远程调用时，要能像本地调用一样方便，让调用者感知不到远程调用的逻辑
* 使用代表：
    * 应用及服务框架：Dubbo, Spring Boot
    * 远程通信协议：RMI, Socket, SOAP(HTTP with XML), REST(HTTP with JSON)
    * 通信框架：MINA, Netty