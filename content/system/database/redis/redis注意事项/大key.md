## 定义

| 数据类型                         | 大key标准                            |
| -------------------------------- | ------------------------------------ |
| String类型                       | value字节数量达到10KB                |
| Hash/Zset/Set/list等复杂数据结构 | 元素个数大于5000，或者总字节大于10MB |



## 危害

1. 读取成本高
2. 容易导致慢查询（过期、删除
3. 主从复制异常，导致服务阻塞，无法正常响应请求
4. 业务侧请求redis超时报错

![image-20230318220036453](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/18/image-20230318220036453-b3ec5e.png)



## 解决方案

### 拆分大key

将大key拆分成多个小key。

![image-20230318220121180](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/18/image-20230318220121180-bae577.png)

集合类数据采用hash取余，或者位掩码的方式决定放在哪个key当中

### 压缩

使用压缩算法压缩数据，读取时解压。



压缩算法选择：gzip、snappy、lz4

如果是JSON，采用MessagePack序列化



### 冷热分离

榜单列表只缓存10页，后续列表走数据库

