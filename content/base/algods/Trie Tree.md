---
aliases:
  - 前缀树
  - 字典树
tags:
  - data-structure/tree
---
**前缀树**，又称为**字典树**或**Trie树**，是一种专门处理字符串匹配的数据结构。它的应用领域广泛，经常被搜索引擎系统用于文本词频统计。前缀树的核心思想是利用字符串的公共前缀来减少查询时间，最大限度地减少无谓的字符串比较。

## 背景

在浏览器中输入某些字符，输入框会立刻提示你可能会输入的单词，这就是前缀树。
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024%2F01%2F16%2F20240116161717-e664c0.png)

在这个场景中，我需要查询所有字符串集合，找出前缀和输入相同的字符串，呈现给用户。

## 暴力解法

就是遍历所有的字符串集合，一个个匹配。
这个方法中我们花了太多时间用于**匹配**公共的前缀，有更高效的方法吗？

## Trie

我们可以将字符串的公共前缀以**树**的形式保存，在查询时遍历这个树就行


![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024%2F01%2F16%2F20240116162631-5777d8.png)

代码实现（来自[@one-seventh](https://www.zhihu.com/people/one-seventh)）：
```cpp
const int MAXN = 500005;
int next[MAXN][26], cnt; // 用类似链式前向星的方式存图，next[i][c]表示i号点所连、存储字符为c+'a'的点的编号
void init() // 初始化
{
    memset(next, 0, sizeof(next)); // 全部重置为0，表示当前点没有存储字符
    cnt = 1;
}
void insert(const string &s) // 插入字符串
{
    int cur = 1;
    for (auto c : s)
    {
        // 尽可能重用之前的路径，如果做不到则新建节点
        if (!next[cur][c - 'a']) 
            next[cur][c - 'a'] = ++cnt; 
        cur = next[cur][c - 'a']; // 继续向下
    }
}


bool find_prefix(const string &s) // 查找某个前缀是否出现过
{
    int cur = 1;
    for (auto c : s)
    {
        // 沿着前缀所决定的路径往下走，如果中途发现某个节点不存在，说明前缀不存在
        if (!next[cur][c - 'a'])
            return false;
        cur = next[cur][c - 'a'];
    }
    return true;
}
```

## 参考

- [CS166: Advanced Data Structures](https://web.stanford.edu/class/cs166/)
- [算法学习笔记(43): 字典树 - 知乎](https://zhuanlan.zhihu.com/p/173981140)
- 例题：
	- https://www.luogu.com.cn/problem/P2580