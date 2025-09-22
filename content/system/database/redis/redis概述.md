# redis概述



## redis简介

redis：**re**mote **d**ictionary **s**erver，远程数据字典。是使用c语言编写的支持网络、基于内存、可持久化的内存型数据库

redis是基于 key-value 的存储系统，支持存储的value类型包括：string、List、Set、Hash、BitMap以及高级数据类型BloomFilter、HyperLogLog等



## NoSQL简介

NoSql（not-relational）通常表示非关系型数据库，其目的是为了解决大规模数据集合多种数据类型带来的挑战，特别是大数据库应用难题![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/01/20230101152529-3ed99b.png)



常见的有 列数据库、图数据库、文档型数据库、键值型数据库

### 键值型数据库

使用简单的键值方法来存储数据，可以理解成一个 **unordered_map**。典型的有redis



### 列数据库

比如说一个 `person` 数据项包括 `name` 和 `age`，列数据库会把所有`name` 存到一起，所有 `age` 存到一起，而不是把 `person` 存储到一起

典型代表：HBase

### 图数据库

主要是用于图处理的数据库，如PageRank等图机器学习算法。最有名的叫neo4j

### 文档型数据库

NoSQL与关系型的结合，把一行行记录转成文档（JSON）。

典型代表：MongoDB



## redis用途

可用于缓存，事件发布或订阅，高速队列等场景。使用C语言编写，支持网络，提供字符串，哈希，列表，队列，集合结构直接存取，基于内存，可持久化。



1. 热点数据的缓存：由于redis访问速度块、支持的数据类型比较丰富，所以redis很适合用来存储热点数据，另外结合expire，我们可以设置过期时间然后再进行缓存更新操作，这个功能最为常见，我们几乎所有的项目都有所运用。

2. 限时业务的运用：redis中可以使用expire命令设置一个键的生存时间，到时间后redis会删除它。利用这一特性可以运用在限时的优惠活动信息、手机验证码等业务场景。

3. 计数器相关问题：redis由于incrby命令可以实现原子性的递增，所以可以运用于高并发的秒杀活动、分布式序列号的生成、具体业务还体现在比如限制一个手机号发多少条短信、一个接口一分钟限制多少请求、一个接口一天限制调用多少次等等。

4. 排行榜相关问题：关系型数据库在排行榜方面查询速度普遍偏慢，所以可以借助redis的SortedSet进行热点数据的排序。

5. 分布式锁：主要利用redis的setnx命令进行，setnx："set if not exists"就是如果不存在则成功设置缓存同时返回1，否则返回0。

6. 延时操作：短信验证码等

7. 分页、模糊搜索：redis的set集合中提供了一个zrangebylex方法，语法如下：

   ZRANGEBYLEX key min max[LIMIT offset count]

   通过ZRANGEBYLEX zset-+LIMIT 0 10可以进行分页数据查询，其中-+表示获取全部数据

   zrangebylex key min max这个就可以返回字典区间的数据，利用这个特性可以进行模糊查询功能，这个也是目前我在redis中发现的唯一一个支持对存储内容进行模糊查询的特性。

8. 点赞、好友等相互关系的存储：Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以自动排重的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择

9. 队列：由于redis有list push和list pop这样的命令，所以能够很方便的执行队列操作。



## redis高性能特性

- 性能极高：读QPS11w，写QPS8W
  1. 内存操作
  2. c语言开发
  3. 代码写得好
- 简单稳定
  1. 2W行（支持集群后5W）
  2. mysql/postgresql百万行
- 持久化（RDB和AOF）
- 高可用集群
- 丰富的数据类型
- 强大的功能
- 客户端语言广泛
- 支持ACL权限控制
- 支持多线程IO模型

### redis的IO模型



### 单线程模型

![image-20230101160200167](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/01/image-20230101160200167-afe009.png)

优点：采用[**多路复用技术**](../../../wiki/多路选择算法.md)，易于维护，没有线程切换的开销，没有加解锁的问题，不存在死锁

缺点：单线程只能用一个处理器，会形成处理器浪费

### 混合线程模型

![image-20230101160854376](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/01/image-20230101160854376-0c4f94.png)



### "多"线程模型

![image-20230101160836063](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/01/image-20230101160836063-04ab40.png)

优点：结合多线程与单线程优点，

缺点：主线程限制了性能