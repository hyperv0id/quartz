---
tags:
  - java/spring/spring-cloud
  - java/spring
Category: spring-cloud
---



## 10.1-服务注册

当项目**启动时**（eureka的客户端），就会向eureka-server **发送**自己的**元数据**（原始数据)(运行的ip，端口port，健康的状态监控等，因为使用的是http/ResuFul请求风格），eureka-server会在自己内部保留这些元数据（内存中）。（有一个服务列表)(restful风格，以 http动词的请求方式，完成对url资源的操作)

## 10.2-服务续约

服务**定时**向server汇报，称作**心跳**

## 10.3-服务下线（主动下线）

告知server自己要关机了

## 10.4-服务剔除（被动下线）

server感觉你出问题了，主动踢了你
