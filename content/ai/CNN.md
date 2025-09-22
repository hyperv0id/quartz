---
aliases:
  - Convolutional neural network
  - 卷积神经网络
tags:
  - ai/神经网络/cnn
---

## 概述

1. 1980年提出神经认知模型
2. 1998年LeNet神经网络
3. 2012年AlexNet神经网络
4. ZFNet、VGGNet、GoogleNet、Re sNe t

![CNN结构](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/05/05/image-20240505145243964-b66d7f.png)

## 卷积层

最开始算力不行，所有神经元都是连起来的（全连接神经网络），参数太多难以训练。

使用卷积核后，可以高效地提取图片中的特征。如果使用3*3卷积核，那么只需要训练9各参数。

此外，卷积层数越多，对复杂图像的理解能力也就越强。

![image-20240505145703754](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/05/05/image-20240505145703754-23a7eb.png)



## 池化层

将局部多个神经元的输入组合成下层单个神经元，起到降维、防止过拟合的作用。

![image-20240505145903446](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/05/05/image-20240505145903446-b5aa83.png)


## 全连接层

使用卷积、池化层提取到的特征进行处理，可以提高计算效率。

## 改进方向

可以使用Gabor滤波器代替卷积核、非线性激活函数选择

### 更多神经网络

- 循环神经网络RNN
- 长短期记忆网络LSTM
- 门控循环单元GRU