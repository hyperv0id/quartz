---
tags: ["#java/spring #java/spring/spring-cloud"]
---
## 背景假设

假设你需要手写一个注册中心，这时候你的注册中心需要有哪些功能？

![image-20221117190217895](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117190217895-59eabf.png)

1. 服务列表，保存应用数据
2. 应用下线、挂掉需要整理
3. 主动上、下线，如何处理
4. 被动上下线，如何处理
5. 上下线会对现有程序产生什么影响？



### 心跳机制

应用和注册中心之间需要建立联系，每次联系就是一次请求，表示服务还活着，代表一次心跳。



### AP原则

如果一段时间 90% 的服务都失去心跳，那么这是仲裁中心要剔除服务吗？

不会，可能是自己挂了。



## 配置

编写配置文件：

```yaml
server:
  port: 8761 #eureka 默认端口

spring:
  application:
    name: eureka-server-01
eureka:
  server:
    eviction-interval-timer-in-ms: 10000 # 服务端每间隔多少毫秒定期删除
    # 85% 的服务下线，eureka不会剔除任何服务
    renewal-percent-threshold: 0.85 # 的服务失去心跳 超过阈值不会剔除服务
  instance: # 实例配置
    instance-id: ${eureka.instance.hostname}:${spring.application.name}:${server.port} # 主机名称：应用名称：端口号
    hostname: localhost
    prefer-ip-address: true # 以ip形式显示具体服务信息
    lease-renewal-interval-in-seconds: 5 # 服务实例续约时间间隔

  #
```

重新启动服务端

![image-20221117192331877](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117192331877-36898a.png)



![image-20221117193240185](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117193240185-347f21.png)