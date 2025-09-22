---
tags:
  - "#data-structure"
aliases:
  - 红黑树
  - rbtree
---
## 简介

> “在各种平衡查找树当中，AVL 树和2-3树已经成为了过去，而红黑树（red-black trees）看似变得越来越受人青睐。这种令人特别感兴趣的数据结构，亦称伸展树（splay tree）。它可以自我管理，且会使用轮换来移除任何访问过根节点的键。” —— Skiena


红黑树提供了在**最坏情况**下插入操作、删除操作和查找操作的时间保证。这些时间值的保障不仅对时间敏感型应用有用，例如实时应用，还对在其他数据结构中块的构建非常有用，而这些数据结构都提供了最坏情况下的保障；例如，许多用于计算几何学的数据结构都可以基于红黑树，而目前 Linux 内核所采用的完全公平调度器（the Completely Fair Scheduler）也使用到了该种树。在 Java 8中，Collection HashMap也从原本用Linked List实现，储存特定元素的哈希码，改为用红黑树实现。

## 定义

- 每个节点不是黑的就是红的
- 根节点黑色
- 红节点的子节点为黑色
- 从根节点到叶子节点的路径经过的黑色节点相同

![来自OIwiki](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/12/05/rbtree-example-4af891.svg)







红黑树可以保证插入和删除节点之后，恢复平衡的复杂度最差情况下都是 $O(\log{n})$ ，但是其中有很多情况需要考虑。

![Binary Tree Rotations (oi-wiki.org)](https://oi-wiki.org/ds/images/rbtree-rotations.svg)

## 资料

- OI-wiki：[红黑树 - OI Wiki](https://oi-wiki.org/ds/rbtree/)
- 可视化：[Red/Black Tree Visualization (usfca.edu)](https://www.cs.usfca.edu/~galles/visualization/RedBlack.html)
- 



