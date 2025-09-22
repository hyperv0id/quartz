---
tags: ["#java/spring #java/spring/spring-cloud"]
---

## eureka运作原理特点

Eureka-server对外提供的是**restful风格**的服务
以http 动词的形式对url资源进行操作 `get` `post` `put` `delete`

```
http 服务 ＋ 特定的请求方式+特定的url地址
```

只要利用这些restful 我们就能对项目实现注册和发现
只不过，eureka 已经帮我们使用java 语言写了client，让我们的项目只要依赖client 就能实现注册和发现!
只要你会发起 Http 请求，那你就有可能自己实现服务的注册和发现。不管你是什么语言！

## 11.1-服务注册



### 11.1.1-eureka如何注册

当 eureka 启动的时候，会向我们指定的 serviceUrl 发送请求，把自己节点的数据以 post
请求的方式，数据以 json 形式发送过去。
当返回的状态码为 204 的时候，表示注册成功。

![image-20221119150747102](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/19/image-20221119150747102-b70a62.png)

### 11.1.2-发送注册信息

在注册时会向server发送http请求，请求信息包含自身的ip、name、port等

![image-20221119151714153](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/19/image-20221119151714153-e699ae.png)

### 11.1.3-服务存储方式

![image-20221119152235643](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/19/image-20221119152235643-de17ed.png)





## 10.2-服务续约



![image-20221119154458871](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/19/image-20221119154458871-59a26b.png)





## 10.3-服务下线（主动下线）

![image-20221119154842583](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/19/image-20221119154842583-a0058f.png)

## 10.4-服务剔除（被动下线）





