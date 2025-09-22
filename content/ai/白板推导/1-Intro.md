---
aliases: 
tags:
  - ai
  - ML
  - 概率论
---

# 频率派VS贝叶斯派


假设我们以$X$表示数据，$\theta$代表参数。

现在有一个概率模型：$x~P(x|\theta)$

## 频率派

频率派认为

- $\theta$是一个未知的常量
- 数据是变量。

常用的方法是最大似然估计（MLE）：

$$
\theta_{MLE}=\mathop{argmax}\limits _{\theta}\log p(X|\theta)\mathop{=}\limits _{iid}\mathop{argmax}\limits _{\theta}\sum\limits _{i=1}^{N}\log p(x_{i}|\theta)
$$

## 贝叶斯派

贝叶斯派认为：

- $\theta$是变量，服从某个概率分布
- 通过贝叶斯公式将先验概率和后验概率联系起来
- MAP：最大后验证概率
- 找一个使得$\theta$最大的点（其实就是众数）


根据贝叶斯定理依赖观测集参数的后验可以写成：
$$
p(\theta|X)=\frac{p(X|\theta)\cdot p(\theta)}{p(X)}=\frac{p(X|\theta)\cdot p(\theta)}{\int\limits _{\theta}p(X|\theta)\cdot p(\theta)d\theta}
$$

MAP计算公式：

$$
\theta_{MAP}=\mathop{argmax}\limits _{\theta}p(\theta|X)=\mathop{argmax}\limits _{\theta}p(X|\theta)\cdot p(\theta)
$$

其中第二个等号是由于分母和 $\theta$ 没有关系。求解这个 $\theta$ 值后计算$\frac{p(X|\theta)\cdot p(\theta)}{\int\limits _{\theta}p(X|\theta)\cdot p(\theta)d\theta}$ ，就得到了参数的后验概率。其中 $p(X|\theta)$ 叫似然，是我们的模型分布。



求出了分布后，我们使用$\theta$作为新旧数据的桥梁，将这个分布用于预测贝叶斯预测：
$$
p(x_{new}|X)=\int\limits _{\theta}p(x_{new}|\theta)\cdot p(\theta|X)d\theta
$$
 这个积分很困难，又时解析解甚至求不出来。因此发展出了概率图模型，使用蒙特卡罗方法求数值解。