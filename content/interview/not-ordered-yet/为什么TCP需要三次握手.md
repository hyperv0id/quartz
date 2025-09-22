
```
```text
Client ------SYN-----> Server
Client <---ACK/SYN---- Server
Client ------ACK-----> Server
```

1. `SYN` 是让 `目标` 知道他能从对方拿到数据包
2. `ACK` 是 `目标` 告知 对方自己能拿到数据包

那么client要与server建立TCP连接，需要通过握手确认这四件事情
1. server需要确认它可以从client接收数据包；
2. client需要确认它可以从server接收数据包；
3. client需要确认一件事：server可以从client接收数据包；
4. server需要确认一件事：client可以从server接收数据包。

一共需要 2次 `SYN` 两次 `ACK`，而 `ACK` 只能在 `SYN` 之后，所以不能是两次握手，四次握手的话会浪费一次传输时间


以及 TCP Sender 在建立连接时候，会初始化一个随机值作为起始序列号，避免恶意建立连接浪费资源。两次握手不能保证连接安全。

## 类比

分布式中拜占庭将军问题为什么要三次握手

![image-20230201105921566](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/image-20230201105921566-ec52fb.png)

两个将军都需要确认一下信息：
1. 自己的消息成功抵达
2. 对方的消息成功抵达

那么三次消息传递的作用：
1. 一次确定自己的消息未被截获
2. 一次确定对方的消息未被截获
3. 一共需要传递三次

### 拜占庭将军问题 与 TCP 的不同

1. TCP是为了传输消息
2. 两将军是为了建立共识

