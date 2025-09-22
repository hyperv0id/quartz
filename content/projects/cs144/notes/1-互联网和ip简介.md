[TOC]

目标：

- 设计原因
- 优缺点
- 网络设计原则：
  - 分层、封装、数据包交换
- 什么是互联网
- 什么是ip
- 网络、Skype、bittorrent工作方式
- 4层网络
- 绝大多数应用使用TCP（传输层）
- Internet分包传输



## 1.1-网络应用程序

### Byte-Stream Model

双方都可以同时读写，任意一方都可以关闭连接

![image-20230209205847721](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209205847721-d61e1c.png)

### WWW（HTTP）

- `http://`表示采用http协议

一般是`get`请求，服务器会因此返回一个网页的文件



### BitTorrent

将文件分成几个区块，订阅同一个文件的群组会分到不同的区块，然后就可以互相分享文件了

![image-20230209210152765](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209210152765-1b5c6a.png)



### Skype



#### 都在公网

几乎没什么区别

![image-20230209210242547](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209210242547-c5ff7a.png)

#### 一个在私网

NAT里面的是私网，公网无法访问。

![image-20230209210304802](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209210304802-cd345e.png)

Skype会使用一个服务器能连到B，然后服务器会告诉B：A在call你，然后AB建立连接

![image-20230209210418280](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209210418280-04c566.png)

#### 都在私网

AB都在私网，A不能call B，B不能call A

Skype使用Relay（中继器），连上AB，然后转发AB之间的消息

![image-20230209210503789](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209210503789-af2ae0.png)





## 1.2-四层互联网

从上到下四层：

1. 应用层
2. 传输层
3. 网络层
4. 链接层（链路层）



链路层的工作是将数据在网络上的节点上传输。

![image-20230209210832097](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209210832097-b04cef.png)

**网络层**：作用是将数据包传输到目的地，**端到端**，相当于填写了邮件的寄件人和收件人

![image-20230209211018455](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209211018455-801be2.png)

网络层将数据交给链路层，相反的链路层为网络层服务

![image-20230209211145229](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209211145229-089fc2.png)



IP不保证数据到达，传输层会保证数据正确传输。最常见的是TCP，现在HTTP3使用QUIC协议了

TCP保证数据按顺序到达

如果用户不需要可靠的传输，可以使用UDP



应用层：成千上万的应用，使用TCP/UDP的API。比如说qq微信浏览器，邮件



IP是不得不用的，这也是为啥IPv6可能永远无法取代IPv4的原因

![image-20230209211727812](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209211727812-83bfa9.png)





## 1.3-IP服务模型

对于很多人来说，IP就是Internet



IP由`IPdata` 和 `IPHeader`组成，当传输层是要传输数据时，传输层会把数据包进`IPData`里面，IP协议会尝试通过路由器等传到目的地

![image-20230209212133921](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209212133921-4a1e69.png)

IP特性

- 数据报：要发数据时，就创建一个包
- 不可靠：可能送不到，可能超时，可能乱序
- 尽力：不会故意坑你
- 无连接：每个数据包相互独立
- 细节：
  - 不会循环，不会再一直转圈圈。IPheader里面由一个跳数，每传一次减去1当变成0时会被丢弃。
  - 太大会切割
  - 有checksum判断数据有没有被篡改
  - 带版本：IPv6，IPv4
  - 允许新功能放到头部

![image-20230209212836838](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/09/image-20230209212836838-4eced6.png)





## 1.4-包生命周期







## 1.5-



