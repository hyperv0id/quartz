# 热key

## 定义

某个key的QPS特别高，导致Server实例出现CPU突增



一般认为 $QPS > 500$ 就是热key



![image-20230318220609501](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/18/image-20230318220609501-b08e6c.png)



## 解决方案

### LocalCache-缓存的缓存

设置LocalCache，业务方位LocalCache，未命中就走redis



![image-20230318220720928](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/18/image-20230318220720928-1cffe5.png)



### 拆分

将热key复制写入多份，分散在不同实例上，更新时也需要一并更新。

![image-20230318220925954](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/18/image-20230318220925954-91c95c.png)

### redis代理

发现热key，并保存在local cache中

![image-20230318220951891](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/18/image-20230318220951891-c93f74.png)