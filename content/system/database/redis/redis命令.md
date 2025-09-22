# 3-redis命令

## 3.1-基本命令

### 3.1.1 ping

坚持和redis的连接是否正常

```bash
127.0.0.1:6379> ping
PONG
```

### 3.1.2-基本读写

基本写：`set key value`

基本读：`get key`

```bash
127.0.0.1:6379> set name cjj
OK
127.0.0.1:6379> set age 11
OK
127.0.0.1:6379> get name
"cjj"
127.0.0.1:6379> get age
"11"
```

### 3.1.3-切换数据库

命令：`select [数据库id]`

```bash
127.0.0.1:6379> select 5 #切换到5号数据库
OK
127.0.0.1:6379[5]> get name
(nil)
```

### 3.1.4-查看数据个数

```bash
>dbsize
(integer) 2
```



## 3.2-key操作

### 3.2.1-keys

命令：`keys [正则表达式]`

```
127.0.0.1:6379> keys *
1) "age"
2) "name"
127.0.0.1:6379> keys a*
1) "age"
127.0.0.1:6379> keys n*e
1) "name"
```

`keys` 命令非常快，但是在数据量比较大时会阻塞服务，在生产环境中使用 `scan` 命令代替

### 3.2.2-存在检测

命令格式：`exists key`

```
127.0.0.1:6379> exists name
(integer) 1
127.0.0.1:6379> exists n*
(integer) 0
```

### 3.2.3-删除key

命令格式：`del [key1 key2 ...]`，返回删除成功个数

```
127.0.0.1:6379> del a
(integer) 1
```

### 3.2.4-重命名key

命令格式：`rename old new`

```
127.0.0.1:6379> keys *
1) "bsdajiknvrieu"
2) "age"
3) "name"
127.0.0.1:6379> rename name new_name
OK
127.0.0.1:6379> keys *
1) "bsdajiknvrieu"
2) "age"
3) "new_name"
```

### 3.2.5-移动key

命令格式：`move key db_id`

```
127.0.0.1:6379> move name 3 #移动到3号数据库
(integer) 1
127.0.0.1:6379> select 3
OK
127.0.0.1:6379[3]> keys *
1) "name"
```

### 3.2.6-类型检测

命令格式：`type key`

返回：key的数据类型

```
127.0.0.1:6379[3]> type key
hash
127.0.0.1:6379[3]> type name
string
```

### 3.2.7-expire & pexpire

格式：`EXPIRE key seconds `

功能：为给定 `key` 设置生存时间。当 `key` 过期时(生存时间为 0)，它会被自动删除。 `expire `的时间单位为秒，`pexpire` 的时间单位为毫秒。在 Redis 中，带有生存时间的 `key `被称为“易失的”(volatile)。 

说明：生存时间设置成功返回 1。若 key 不存在时返回 0 。`rename `操作不会改变 key 的生存时间。



### 3.2.8-ttl & pttl

- 格式：`TTL key `
- 功能：time to live，返回给定 key 的剩余生存时间。 
- 说明：其返回值存在三种可能： 
  - 当 key 不存在时，返回 -2 。 
  - 当 key 存在但没有设置剩余生存时间时，返回 -1 。 
  - 否则，返回 key 的剩余生存时间。ttl 命令返回的时间单位为秒，而 pttl 命令 返回的时间单位为毫秒。





### 3.2.9-persist

命令格式：`PERSIST key`

去除生存时间

### 3.2.10-randomkey

获得一个随机key，数据库为空时返回nil



### 3.2.11-scan

命令格式：`SCAN cursor [MATCH pattern] [COUNT count] [TYPE type]`

迭代数据库，开始于 `cursor` 匹配 `count(默认10)` 个类型为 `type` 且满足模式 `pattern` 的`key`

## 3.3-string操作



## 3.4-hash操作



## 3.5-list操作



## 3.6-set操作





## 3.7-zset操作



## 3.8-benchmark性能测试

### 3.8.1-简介



### 3.8.2-测试



## 3.8-bitmap操作





## 3.9-HyperLogLog操作







## 3.10-GeoSpatial操作





## 3.11-发布/订阅

### 消息系统

![image-20230112234933469](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/12/image-20230112234933469-d336e0.png)

生产消费不匹配的问题

- 消费慢生产快：存储系统占满了
- 消费快：所有消息被消费完毕，消息消费者被阻塞

消息发布命令：`publish channel value`

消息订阅命令：`subscribe channel1 channel2 ...`





### redis事务

事务：

- 执行失败不会引发回滚，即不具备原子性
- 乐观锁实现简单隔离，没有复杂的隔离级别
- 执行结果写道内存，是否持久化由redis来做





redis事务处理命令：

- multi：开启事务
- exec：执行
- discard：取消



## 3.12-简单动态字符串SDS





## 3.13-redis事务
