---
aliases:
  - Sigmoid
  - Logistic函数
  - Sigmoid 函数
tags:
  - ai/激活函数
  - math
---
将一个实数域上的值映射到`[0,1]`空间，十分适合线性分类问题。
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/11/20240111143301-8ad4d0.png)


Sigmoid函数的特性与优缺点：

- 导数可以由自身表示：$$\mathrm S'(\mathrm x)=\frac{\mathrm e^{-\mathrm x}}{(1+\mathrm e^{-\mathrm x})^2}=\mathrm S(\mathrm x)(1-\mathrm S(\mathrm x))$$
- Sigmoid函数的输出范围是0到1。由于输出值限定在0到1，因此它对每个神经元的输出进行了归一化。
- 用于将预测概率作为输出的模型。由于概率的取值范围是0到1，因此Sigmoid函数非常合适
- 梯度平滑，避免跳跃的输出值
- 函数是可微的。这意味着可以找到任意两个点的Sigmoid曲线的斜率
- 明确的预测，即非常接近1或0。
- 函数输出不是以0为中心的，这会降低权重更新的效率
- Sigmoid函数执行指数运算，计算机运行得较慢。

