# 卷积

在概率论中，卷积表示成这样

连续型

![](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/25/equation_disccrete-35fc78.svg)

离散型

![](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/fix-dir/2022/06/25/equation-5a883e.svg)

> 并且也解释了，先对g函数进行翻转，相当于在数轴上把g函数从右边褶到左边去，也就是卷积的“**卷**”的由来。
>
> 然后再把g函数平移到n，在这个位置对两个函数的对应点相乘，然后相加，这个过程是卷积的“**积**”的过程。

下图很好地表示了卷积的过程

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/24/3c59ccecf89600bd3f2aa0c649f213da2d97dd39-380589.gif)

## 图像处理中的卷积

在图像处理中，同样需要卷积来减小运算量。

现在一个图片动则几MB，考虑三种颜色，参数可能至少上百万，单个图片就如此高，那么多个图片更不用想了。

于是我们通过卷积来减小运算量。



具体的，如下图，我们定义一些卷积核函数，这些函数能表示图像中的特征，且可以通过组合卷积核来拼凑出完整的图像。

![25种不同的卷积核](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/24/f0cfdfc2b0532d954d6bc6314f915cac85b292b4-f6740d.jpeg)

我们将卷积核逐个上面原始图片，将卷积后的值作为卷积后特征

![卷积层运算过程](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/24/05c0c84c210f5d25a8250ff1455d897f74aa372d-48af4c.gif)

## 卷积后的池化

可以看出，卷积后的结果没有减少太多资源，因为只有n-1行(列)被消除了

我们可以考虑将卷积后的结果降维，最简单的当然是分成几块各自运算没有重复。

![池化层过程](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/24/5f8cceb7e86049487b6e09aec97cf771f547dbfc_2_690x397-bd2900.gif)

## 全连接层

这个部分就是最后一步了，经过卷积层和池化层处理过的数据输入到全连接层，得到最终想要的结果。

经过卷积层和池化层降维过的数据，全连接层才能”跑得动”，不然数据量太大，计算成本高，效率低下，还会出现[过拟合](../watermelon/2-模型评估与选择/过拟合.md)。