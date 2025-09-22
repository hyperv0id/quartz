---
tags: ["#java/spring #java/spring/spring-cloud"]
---
![image-20221117193426852](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117193426852-871101.png)



## 集群方案

### 主从模式

主机挂了就全挂了

例：nginx挂载多个tomcat

### 去中心化模式

更加高可用

raft算法

## Eureka的集群模式

去中心化模式，没有主从的概念

eureka会将数据广播、扩散

server之间两两互相联系，任意一个server添加服务都会通知另外的server

![image-20221117194032921](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117194032921-bd7fb2.png)





三个服务端都出现在列表里了：

![image-20221117195220328](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117195220328-285d35.png)

这时添加客户端到 8761 端口会导致其他服务器也注册

![image-20221117195347762](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117195347762-3fdfa5.png)



但现在还不是集群

修改host文件，骗eureka是peer1，2，3这三个机器。

在配置文件中修改`localhost`为peer1，2，3

![image-20221117195623600](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117195623600-0ec0c4.png)

修改后重新启动，可以看到几个数据分片

<img src="https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/17/image-20221117195922889-dfbb6d.png" alt="image-20221117195922889" style="zoom:50%;" />



### 集群最终解决方案

1. 注册到包括自己的所有服务器
2. 不再包含`hostname`
3. 以不同端口启动

