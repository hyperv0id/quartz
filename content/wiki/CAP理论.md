![CAP.jpg](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/08/CAP-37c35a.jpg)

#### 是什么

- C：Consistence，一致性，多个副本之间能保持严格的一致
- A：Availability，可用性，每次请求都能保证得到相应的服务，并不会有错误码。不保证数据是最新的
- P：Network Partitioning：分区容错性，在遇到任何网络分区故障时，任然能对外提供服务，除非网络发生了故障



CAP理论常用于数据库理论，同样适用于分布式存储



CAP无法同时100%满足，只能选择放弃一个

- CA系统：放弃分区容错性，加强一致性和可用性。一般用于传统单机数据库
- AP系统：放弃一致性，一般用于注重用户体验的系统
- CP系统：放弃可用性，一般用于钱财相关系统

#### 案例

![image-20230201173829779](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/image-20230201173829779-0b13c1.png)

在网络发生分区的情况下，我们必须在可用性和一致性之间作出选择。近似的进解决办法：将故障节点移交备用节点负责。

故障转移：

![image-20230201173847678](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/image-20230201173847678-095a76.png)

