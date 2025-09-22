---
tags: ["#java/spring #java/spring/spring-cloud"]
---
# 客户端配置

```yml
spring:
  application:
    name: eureka-client-a

server:
  port: 9999

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    register-with-eureka: true # 是否注册到eureka
    fetch-registry: true # 是否拉去注册中心（false就没法看见其他服务了）
    registry-fetch-interval-seconds: 30 # 每个多少时间去服务端拉取列表
  instance:
    hostname: localhost # 应用的主机名称，最好是ip
    instance-id: ${eureka.instance.hostname}:${spring.application.name}:${server.port} # 主机名称：应用名称：端口号
    prefer-ip-address: true # 显示ip
    lease-renewal-interval-in-seconds: 10 #续约时间
```

