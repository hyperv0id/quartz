[toc]

## 基本概念

see：[RPC](../../../dev/backend/RPC.md)

## RPC分层设计

### 例子：Apache Thrift

![image-20230125142551800](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/25/image-20230125142551800-5577e2.png)



### 编解码层

#### 生成代码

IDL文件描述了接口的规范，这样即使不同语言编写的程序都可以使用。

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/25/20230125142650-2f11c7.png)

#### 数据格式

- 语言特定格式：编程语言将内存数据编码成字节序列，其他语言无法读取
- 文本格式：具有人类可读性的格式，性能比较差，如Json、XML、YAML
- 二进制编码：把数据转换成二进制流跨语言、高性能，常见的有`Thriift` 的 `BinarProtocol` 等



TLV编码：

- Tag：类型
- Length：长度
- Value：值，Value也可以是一个TLV
- tag和length占用了额外的空间

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/25/20230125223746-041323.png)



### 协议层

#### 概念

特殊结束符：过于简单，对于一个协议单元必须要全部读入才能够进行处理，除此之外必须要防止用户传输的数据不能同结束符相同，否则就会出现紊乱。

HTTP 协议头就是以回车(CR)加换行(LF)符号序列结尾。

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/20230128140914-0d5972.png)



变长协议：一般都是自定义协议，有 header 和 payload 组成，会以定长加不定长的部分组成，其中定长的部分需要描述不定长的内容长度，使用比较广泛

![image-20230128163755240](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128163755240-b93e9d.png)

#### 协议构造

![image-20230128164017202](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128164017202-64cc79.png)

#### 协议解析

![image-20230128164357528](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128164357528-6f2d0a.png)



### 网络通信层

#### Sockets API

位于传输层和应用层之间

![image-20230128164439194](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128164439194-ffd7c1.png)



#### 网络库

- 对上层提供API
  - 封装底层Socket API
  - 连接管理和事件分发
- 协议支持
  - TCP
  - UDP
  - UDS
- 优雅退出
- 异常处理
- 性能
  - 应用层buffer减少copy
  - 高性能定时器、对象池

### 小结

- RPC三个核心层
- 二进制编解码原理、选型
- 协议构造解析流程
- socket api调用流程
- 网络库指标

## RPC关键指标

### 稳定性

#### 保障策略

![image-20230128200814282](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128200814282-99981f.png)

**熔断**：保护调用方，防止调用服务出现问题而影响到整个链路

A调用B，B调用C。如果C超时了，那么B也会超时，那么A就会频繁的调用B，B会因为堆积大量请求导致服务宕机

**限流**：限制流量，防止大流量把线路压垮了

**超时控制**：被调用端响应过慢，调用端会主动停止不重要的请求，即使释放资源。

#### 请求成功率

- **负载均衡**：避免单个服务的负载过大
- **重试**：调用失败会重试几次，超过次数才算真正的失败

![image-20230128201233447](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128201233447-abb01c.png)

#### 长尾请求

长尾请求：响应时间明显高于平均响应时间的请求

pk99：按请求长度从小到大排列，超过99%的长度的会被认为长尾请求

处理长尾请求：在返回之前重新发送一次。按照过往经验来看99%的请求能在`t3`内返回，这时发送备份请求`Req2`，`Req2`会很快返回。

![image-20230128201451609](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128201451609-136e1b.png)

#### 注册[中间件](../../../wiki/中间件.md)（拦截器）

在创建时将这些功能可选地加上

![image-20230128201858783](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128201858783-eee4c7.png)

### 易用性

- 开箱即用：合理的默认配置、丰富的文档
- 周边工具：生成代码、脚手架

### 扩展性

client发送消息会依次通过中间件处理，server亦然。

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128202046911-3f36c0.png)



### 观测性

观测手段

- Log、Metric、Tracing
- 内置观测服务：Linux中的`top`命令

![image-20230128202249294](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128202249294-4c80a1.png)



### 性能

场景：

- 单机、多机
- 单连接、多连接
- 单/多    client/server
- 请求大小
- 请求类型：pingpong，streaming

目标：

- 高吞吐
- 低延迟

手段：

- 连接池
- 多路复用
- 高性能编解码协议
- 高性能网络库





## 企业实践——kitex

[Kitex | CloudWeGo](https://www.cloudwego.io/zh/docs/kitex/)

### 整体架构

- `kitex core`：核心组件
- `kitex byted`：字节内部设施基础
- `kitex tool`：代码生成工具

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/20230128210410-a501a2.png)



### 自研网络库

原生网络库缺点：

- 无法感知连接状态：使用连接池时，池中存在失效连接，影响连接池复用
- 存在goroutine暴涨风险：一个连接一个goroutine，利用率低下

#### Netpoll

- 解决无法感知连接状态的问题：使用epoll主动监听
- 解决goroutine暴涨风险：建立goroutine池，复用goroutine
- 提升性能：引入`Nocopy Buffer`，向上层提供`Nocopy`调用接口，减少拷贝
- 



### 扩展性设计

![image-20230128211131835](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/28/image-20230128211131835-e6fc2f.png)





### 性能优化

#### 调度优化

- 优化`epoll_wait`调度上的控制
- gopoll重用goroutine，降低同时运行的协程数量

#### LinkBuffer

- 读写并行无锁，支持nocopy流式读写
- 高效扩容缩容
- Nocopy Buffer池化，减少GC

#### Poll

引入对象池和内存池，减少GC开销





### 合并部署







## 参考

[‍⁣⁤⁢‌﻿⁤⁢⁢⁣﻿‬⁤⁡⁡⁢⁢⁤‍⁢﻿‌‬⁢‍⁣⁢⁣‌⁣‬⁡‌⁣‬﻿深入浅出 RPC 框架 副本.pptx - 飞书云文档 (feishu.cn)](https://bytedance.feishu.cn/file/boxcn5DUtKdJDDitx8NHShv2xZd)

[RPC 框架分层设计 - 掘金 (juejin.cn)](https://juejin.cn/course/bytetech/7142811324462923783/section/7142809631831228429)

[RPC 关键指标分析与企业实践 - 掘金 (juejin.cn)](https://juejin.cn/course/bytetech/7142811324462923783/section/7142812225919516680)