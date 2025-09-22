---
aliases: "Hybrid transactional/analytical processing"
---

- **偏重事务处理（online transactional processin, [OLTP](OLTP.md)）**：此类数据库将不同属性连续存储，也即按**行存**储。按行存储可以使得插入/更新/删除更快，毕竟一条数据的所有属性是连续存储的。这种存储模型也叫做 N-Ary Storage Model (NSM)。
- **偏重数据分析（online analytical processing, [OLAP](OLAP.md)）**：此类数据库将不同数据的同一属性连续存储，也即**列存**储。这种存储可以使得查询操作只读关心的数据属性，而不是一整条数据，减少浪费；按列储存可以更好地支持复杂查询。这种存储模型也叫做 Decomposition Storage Model (DSM)。

## HTAP 作用

既可以应用于事务型数据库场景，亦可以应用于分析型数据库场景。实现**实时业务决策**。

## 重点技术

**行列混存**：热数据行存，冷数据列存。

- 热数据查询和修改次数比较频繁，采用行存可以减少修改开销
- 冷数据查询和修改次数比较少，一般用于分析，采用列存便于分析
