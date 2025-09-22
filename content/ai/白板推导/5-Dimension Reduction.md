---
aliases: 
tags:
  - ai
  - ML
---

# 降维

## 背景

在机器学习中我们更关心**泛化误差**而不是训练误差，过拟合会导致训练误差很小但是泛化误差很大，要抑制过拟合的一大方法就是降维。

维度过高会导致[维度灾难](../待分类/维度灾难.md)([The Curse of Dimensionality](../待分类/维度灾难.md))，我们每增加一个维度，需要的属性增加会是指数级别的，因此通过减少数据的特征数量来简化数据集来降维，可以帮助提高模型的性能、减少计算开销，并使数据更易于可视化理解。


降维的算法分为：

1.  直接降维/特征选择：有$P$个维度，选$q$个维度保留
2.  线性降维：PCA，MDS等
3.  非线性：流形学习，包括 Isomap，LLE 等

## 样本矩阵

首先将协方差矩阵（数据集）写成中心化的形式：
$$
\begin{align}S&=\frac{1}{N}\sum\limits_{i=1}^N(x_i-\overline{x})(x_i-\overline{x})^T\nonumber\\
&=\frac{1}{N}(x_1-\overline{x},x_2-\overline{x},\cdots,x_N-\overline{x})(x_1-\overline{x},x_2-\overline{x},\cdots,x_N-\overline{x})^T\nonumber\\
&=\frac{1}{N}(X^T-\frac{1}{N}X^T\mathbb{I}_{N1}\mathbb{I}_{N1}^T)(X^T-\frac{1}{N}X^T\mathbb{I}_{N1}\mathbb{I}_{N1}^T)^T\nonumber\\
&=\frac{1}{N}X^T(E_N-\frac{1}{N}\mathbb{I}_{N1}\mathbb{I}_{1N})(E_N-\frac{1}{N}\mathbb{I}_{N1}\mathbb{I}_{1N})^TX\nonumber\\
&=\frac{1}{N}X^TH_NH_N^TX\nonumber\\
&=\frac{1}{N}X^TH_NH_NX=\frac{1}{N}X^THX
\end{align}
$$
这个式子利用了中心矩阵 $H$的对称性，这也是一个投影矩阵。


![image-20240111204151249](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/11/image-20240111204151249-28ef2c.png)

## [PCA](../待分类/PCA.md)
![PCA](../待分类/PCA.md)