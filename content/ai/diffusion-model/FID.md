---
aliases:
  - Frechet Inception Distance
---
[Frechet Inception Distance (FID)](https://arxiv.org/abs/1706.08500)是一种用于评估生成模型质量的指标,常用于评估生成对抗网络(GAN)生成的图像质量。它通过比较真实图像和生成图像在 Inception v3 网络某一中间层特征空间的统计距离来衡量二者的相似程度。

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/04/09/20240409133657-4f0828.png)
图中红色表示真实图片，蓝色表示生成的图片。

FID假设两组Representation是高斯分布，计算两组分布之间的距离。



FID的计算过程如下:

1. 使用预训练的 Inception v3 网络提取真实图像和生成图像在某一中间层(如pool3)的特征。 

2. 假设真实图像特征服从**多元高斯分布**,计算其均值向量μ₁和协方差矩阵Σ₁。同理,计算生成图像特征的μ₂和Σ₂。

3. 计算两个高斯分布之间的Frechet距离,作为FID得分:

   $d^2 = |\mu_1 - \mu_2|^2 + \mathrm{Tr}(\Sigma_1 + \Sigma_2 - 2(\Sigma_1 \Sigma_2)^{1/2})$

其中$\mathrm{Tr}$表示矩阵的迹。

**FID值越小,说明生成图像与真实图像在特征空间越接近,生成质量越好。** 

优点：FID能更全面地反映生成图像的质量和多样性。
缺点：对数据集敏感,需要很多的样本。
