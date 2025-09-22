图机器学习的重要性
## 图-计算机的基础

> 图：计算机世界的基石
> 是描述大自然的通用语言，蕴含巨大的商业价值、科研价值。
> 图在过去现在未来都在改变各行各业。图机器学习是长期通用技能。
>  图机器学习可以和人工智能各方向结合(大模型、多模态、可信计算、NLP)

### 描述大自然的通用语言

很多类型问题都可以用图来表示。
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105185028-e34325.png)
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105185044-20250f.png)
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105185119-701d3b.png)

## 传统机器学习

数据样本之间独立同分布

不论分类还是回归，都会取一个决策边界，或者回归曲线即可
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105185432-0b179b.png)

对于传统的，不带关联的，可以顺序、表格、像素矩阵处理，eg：图像、语音、文本
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105185607-04af78.png)

## 处理图数据
机器学习热点：
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105185809-d850df.png)

### 难点

1. 图的网络是任意大小的，有上限，没有顺序输入
2. 没有参考锚点
3. 多变、多模态
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105185915-5d6347.png)

## 图神经网络

输入：一个图
输出：节点类别、节点的新连接、新图、子图
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105190131-d73fe9.png)

神经网络中黑箱的功能：

1. 图潜入 / 端到端[表示学习](../../wiki/表示学习.md)：不需要人工
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105190227-a1d8c1.png)

## 课程概述

- 传统机器学习方法：Graphlets、Graph Kernels
- node embedding方法：DeepWalk、Node2Vec
- 图神经网络：GCN、GraphSAGE、GAT、Theory of GNNs
- 知识图谱、推理：TransE、BetaE
- 生成新图：GraphRNN
- 应用：医学、工业界、科学

前置知识：
- 机器学习
- 图相关算法
- 概率论
- pytorch

## 图机器学习的应用

- 寻路算法：A*
- 度中心性评价（重要性）：pagerank
- 社群检测：老赖村
- Link Prediction：可能认识的人
- 相似：节点相似度、天哪这根本就是我
- 图嵌入：Node2Vec、RandomProjections、GraphSAGE
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105192722-80a2c0.png)

- 节点层次：信用卡欺诈
	- 节点分类：由已知的节点推断未知节点
- 连接层面：推荐
	- Link Prediction：由已知链接推断未知链接
- 社群层面：用户聚类
- 整个图层面：分子性质 

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105193023-3ad175.png)



## 思考题

1. 打开你的手机，里面那些APP用到了图机器学习和图神经网络的技术？（内容个性化推荐、社交网络、银行金融）

2. A股、港股、美股市值最高的上市公司，哪些公司的核心资产是图？

3. 观看电影《社交网络》，图和图数据挖掘的商业价值体现在哪些方面？

4. 马化腾在2022年12月内部讲话提到，微信视频号是整个腾讯的希望，请从图的角度解释这句话。

5. 在你自己的研究领域，哪些数据可以用图或者网络来表示，如何进行图数据挖掘？

6. 近年来，图数据挖掘在哪些领域带来了革命性进展？

7. 图数据挖掘解决哪些基本任务？

8. 分别从图、连接、节点三个层面，举例解释图数据挖掘在生物医学方面的应用。

9. 图神经网络为什么是端到端的？为什么不需要人工做特征工程？

10. 图神经网络和其它神经网络有什么区别？

11. 简述AlphaFold的基本原理，它解决了哪些以前解决不了的问题？

12. 图机器学习和传统机器学习有什么区别和难点？

13. 图机器学习的编程工具有哪些？看看它们的官网吧（Graphgym、pyG、networkx、dgl、Pytorch、AntV、Echarts）

## 其它阅读材料

李笑来-惊喜与创造惊喜的方法论：<https://zhuanlan.zhihu.com/p/475615463>

乔布斯在斯坦福大学毕业典礼的演讲：<https://www.bilibili.com/video/BV1oW411h7Ea>

子豪兄1024脱口秀-乔布斯传奇：<https://www.bilibili.com/video/BV1Zf4y1g78Q>

哥尼斯堡七桥问题：<https://zhuanlan.zhihu.com/p/519123688>

2022 IDEA大会｜BIOS V2正式发布，数据驱动构建超级医学知识图谱：<https://mp.weixin.qq.com/s/vuHGUtWbiIH-pJ6MZaxl5Q>