---
tags: ["#data-structure/bloom-filter"]
---

空间布隆过滤器，可以在不经过地图服务提供商的情况下，告知用户其是否在某位置附近。

如果你不知道什么是布隆过滤器，请看看我写的这篇文章：[Bloom Filter](Bloom%20Filter.md)

## 构建

给定集合：$S=\left\{\Delta_{1}, \Delta_{2}, \ldots, \Delta_{s}\right\}$，其中每一项都是一个感兴趣的目标地点
在添加元素时，将 $\Delta_{1}$ 中的所有元素设为1，将 $\Delta_{2}$ 中的所有元素设为2，直到集合S被添加完
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/15/20221115154352-93acd9.png)
在判断时，找到对应最小的值

## 代码实现

### 原作者的代码仓库

- [Python Implementation](https://github.com/spatialbloomfilter/libSBF-python)
- [Cpp Implementation](https://github.com/spatialbloomfilter/libSBF-cpp)
- [testdatasets](https://github.com/spatialbloomfilter/libSBF-testdatasets)

### 我的Java实现

SBF类需要的参数：
- byteSet：用于存储地区信息
- cellSize：一个单位占多少字节。在这里考虑区域数可能超过255，在超过255时取2字节作为一个单位
- m：数组长度，表示包含多少个单位
- k：哈希函数个数
- 还有一些统计信息，如插入次数，冲突数，等等

### 构造
