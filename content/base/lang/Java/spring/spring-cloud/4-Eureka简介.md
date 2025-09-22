---
tags: ["#java/spring #java/spring/spring-cloud"]
---
## 4.1-Eureka简介

服务注册与发现组件

## 4.2-Eureka与Zookeeper区别

### 4.2.1-CAP原则（面试）
问：为什么 zookeeper不适合做注册中心?

CAP原则又称CAP 定理，指的是在一个分布式系统中,
- C: **一致性**(Consistency)
- A: **可用性**(Availability)
- P: **分区容错性**（Partition tolerance)（这个特性是不可避免的)
CAP原则指的是，这三个要素最多只能同时实现两点，**不可能三者兼顾**。

### 4.2.2-分布式特征

- Eureka注重AP，会有不一致
- zookeeper注重CP，会不可用

## 4.3-SpringCloud其他注册中心

1. [Consul](https://www.consul.io/)
2. [nacos.io](https://nacos.io/zh-cn/)

