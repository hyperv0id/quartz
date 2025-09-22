---
aliases:
  - 后缀树
tags:
  - data-structure/tree
---
## 问题描述

1. 如何查找PDF文件中的某段文字，可以预处理
2. DNA序列中有很多重复，如何找到最长重复字串？

## 从前缀树开始

引理：一个子串一定是字符串的后缀的前缀。
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024%2F01%2F16%2F20240116180116-f0e70e.png)

我们找到所有可能的后缀，然后构造一个前缀树：

![nonsense的后缀树](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024%2F01%2F16%2F20240116180358-ee8056.png)

