[TOC]

## 0-简介

- 易于理解
  - 问题分解成三个子问题：领导者选举、日志复制、安全性
  - 状态简化：对算法限制，减少状态数量和可能的变动

- 效率和paxos等算法等同



## 1-复制状态机

复制状态机：**相同初始状态 + 相同输入 = 相同结束状态**

也就是在多个节点上，从相同的初始状态开始，执行相同的代码，其最终状态相同

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/16/20230316155303-5e6f38.png)

在raft中，leader将命令封装到logentry中，然后复制到所有follower节点中，所有follower执行相同的命令，到达一致的最终状态

一致状态扩展：只要对于相同的输入，会有相同的输出就行，内部实现可以不同。比如说有的行存，有的列存（[HTAP](../../wiki/HTAP.md)）。

## 2-状态简化

状态机：任意时刻，节点的状态在处于：`leader`, `follower`,`candidate` 之一





![image-20230316161435364](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/16/image-20230316161435364-d2013e.png)



raft将时间分成一块块任期(`term`)，每个任期最多一个leader，如果没有leader或选举失败，那么会立刻开始下一轮任期。

![image-20230316161620704](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/16/image-20230316161620704-5b4dc2.png)



节点之间采用rpc通信，

- RequestVote：请求投票，candidate在选举期间投票
- AppendEntries：追加条目，leader发起，用于复制日志和心跳检测



服务器会交换当前任期号，follower发现过期立刻变更，candidate或leader发现过期立刻变成follower。





## 3-领导者选举

### 心跳机制

- leader会周期性向所有follower发送心跳。
- follower周期内没收到心跳，自己立刻参加选举。

### 开始任期

- 开始选举后，follower自增任期号，变成candidate，投票给自己并并行的向其他服务器发送投票请求（RequestVote）
- follower收到RequestVote会立刻将票投给他，不能拒绝
- 选举超市时间：150ms-300ms

### 选举结果

- 自己成为leader
  - 条件：自己获得半数以上选票
- 其他节点选举成功
  - 收到新leader信息，发现任期号等于自己：出现新leader，已投超半数票给他。自己手慢了，没抢到。
  - 收到新leader信息，发现大于自己：火星信号不好
- 没有新leader

## 4-日志复制

- `leade`r并行地发送 `AppendEntries RPC`给`Follower`，当该条目被**超过半数**的`follower`复制后，leader在本地执行结果并返回客户端
- 提交：本地执行命令，即`leader`应用日志与状态机



![image-20230316164122666](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/16/image-20230316164122666-b69cb8.png)

7号日志超过半数，7号可以提交

### 如何知道leader是哪个节点

向随机一个节点发送请求

1. 是leader，直接拿消息
2. 不是leader，但是会返回当前leader信息（我不造啊，找他）
3. 节点宕机，重试

### 崩溃恢复

leader和follower都会宕机或者缓慢，raft需要保证有宕机的情况下继续支持日志复制

1. follower没有给leader'响应，leader会不断重发
2. 如果有follower崩溃后恢复，raft追加条目的一致性检查生效，follower按序恢复日志
3. leader宕机：新leader没有未提交的日志
   1. **没有提交**的日志没有用，直接失败
   2. 原follower采用一致性检查，强制复制新leader

#### 一致性检查

leader在每个发往follower的追加条目RPC中会放入前一个日志的**索引号和任期号**，如果follower找不到前一个日志，那么会拒绝此日志。leader会发送更前的一个日志条目，直到定位到的一个缺失的日志。

> 失败不经常发生，作者觉得优化没必要



## 5-安全性





## 6-集群成员变更





## 7-总结与性能测试





## 8-扩展：ParallelRaft



