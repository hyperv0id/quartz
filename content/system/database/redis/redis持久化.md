# 4-redis持久化

## 

持久化选择：

- 保存快照：就是现在内存的状态
- 记录日志：记录执行了 哪些命令

## 4.1-基本原理

![image-20230113001259680](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/13/image-20230113001259680-5fe935.png)

持久化文件加载：

![image-20230113001515737](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/13/image-20230113001515737-438c3b.png)

## 4.2-RDB持久化

redis database，将redis**全量数据**写到持久化文件中，系统默认开启rdb.



### 4.2.1-持久化执行

1. `save`命令手动执行，执行期间会阻塞redis服务，redis不能处理任何请求
2. `bgsave`命令，在后台保存，此时redis还能服务

### 4.2.2-RDB优化配置

在redis.conf 中的 SNAPSHOTTING 模块



设置保存点

```
save 3600 1 300 100 60 10000 
# 每3600秒内如果有一次写入执行一次bgsave
# 300 秒内写入100次，自动执行bgsave
# 60 秒内发生1W次写入，自动执行一次bgsave
```



容灾处理

```
stop-writes-on-bgsave-error yes
# 当RDB被开启，至少有一个保存点，如果RDB报错，就停止写入操作（目的是使用户意识到数据没有写到磁盘，避免其他灾难问题
# 如果此问题修复，那么写入自动恢复
# 如果有其他监控，可以设置成false
```



```
rdbcompress yes
# rdb压缩，使用LZF算法，默认开启
# 关闭可以节省CPU（拿空间换时间）
```



```
rdbchecksum yes
# 校验rdb文件
# 在rdb5 版本中使用 CRC64 算法，健壮性更强，性能低 10%
```



```
sanitize-dump-payload no
# 对ziplist 和 listpack 使用检测
```



```
rdb-del-sync-file 
# 主从集群，是否同步文件
```

### 4.2.3-rdb文件结构

使用vim打开试试？

![image-20230311195023382](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311195023382-880d9f.png)



![image-20230311195359129](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311195359129-3a6413.png)

#### SOF（start of file

常量，一个字符串（REDIS）

#### rdb_version

rdb版本号



#### EOF（end of file

常量，1字节，表示数据的结束。

后面是接下来是校验和部分



#### check_sum

使用CRC校验，判断文件是否损坏



使用 `SOF + rdb_version + 内存数据快照` 和 `check_sum` 做除法，余数为 `databases`



#### databases

![image-20230311200257640](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311200257640-4edc84.png)

- SODB：常量，占1字节，用于标识数据库的开始
- db_number：数据库编号
- key_value_pairs：当前数据库中键值数据
  - 每个pairs由以下部分构成
  - ValueType：值的类型
  - Key
  - Value
  - ![image-20230311200425240](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311200425240-acb242.png)

### 4.3-RDB持久化过程

1. fork出bgsave子进程`child`
2. `child`写入到临时文件
3. copy期间新数据会写到另一内存区域（Copy-on-Write）
4. copy完毕后，再将内存副本写到临时文件

![image-20230311200716363](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311200716363-6f5c1b.png)



## 4.3-AOF持久化

固态硬盘：适合顺序读写，不适合随机读写



## 4.4-混合持久化



## 4.5-持久化策略
