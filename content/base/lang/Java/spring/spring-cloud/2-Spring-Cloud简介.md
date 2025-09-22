---
tags: ["#java/spring #java/spring/spring-cloud"]
---
## 与微服务关系

SpringCloud是微服务的实现，是Spring家提供的微服务框架，其他的实现还有Google的`kubernetes`。

目前由三家共同维护：
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/17/20221117163835-138db5.png)

Spring_cloud提供了一系列开发组件和框架，可以简化开发

## SpringCloud版本对应关系

GA为**稳定版本**，2020年之前版本命名方式为ABCDEF开头

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/17/20221117164112-c07559.png)

使用cloud需要指定对应的`spring-boot`版本，否则会有随机debuff
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/17/20221117164524-c4df5e.png)

阿里的spring-cloud百科：[Home · alibaba/spring-cloud-alibaba Wiki (github.com)](https://github.com/alibaba/spring-cloud-alibaba/wiki)

## 2.1-常用组件表

1. 服务的注册和发现。(eureka,nacos（阿里）,consul)
2. 服务的负载均衡。(ribbon, dubbo)
3. 服务的相互调用。(openFeign, dubbo)
4. 服务的容错。(hystrix(netflix), sentinel(alibaba))
5. 服务网关。(gateway(spring)，zuul(netflix), 阿里在研)
6. 服务配置的统一管理。(config-server, nacos, apollo(携程旅游))
7. 服务消息总线。(bus)
8. 服务安全组件。(security,Oauth2.0)
9. 服务监控。(admin)(jvm)
10. 链路追踪。(sleuth+zipkin)

> 这些东西已经算少的了┗|｀O′|┛ 嗷~~

