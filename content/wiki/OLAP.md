OLAP数据库：OLAP(Online analytical processing)数据库旨在同时**分析数据**的多个维度，帮助团队更好地理解其数据中的复杂关系

此类数据库将不同数据的同一属性连续存储，也即**列存**储。这种存储可以使得查询操作只读关心的数据属
性，而不是一整条数据，减少浪费；按列储存可以更好地支持复杂查询。

## 特点

- 大量读写，PB级别存储
- 多维分析，复杂的聚合函数
- 窗口函数自定义
- 离线、实时分析

大量读写
![大量读写](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/18/20230318222848-901514.png)

![多维分析](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/18/20230318222902-613a45.png)

- [ ] #TODO 