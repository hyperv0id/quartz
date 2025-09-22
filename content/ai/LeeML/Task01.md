# 框架
![在这里插入图片描述](https://img-blog.csdnimg.cn/0193080a7ffb42b29f374dd613beb6ee.png#pic_center)


机器学习可以简化成三个步骤：

1. 生成一系列函数
2. 判断函数好坏
3. 找出表现最好的函数

# Learning Map

![image-20220613094726509](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/24/0525b8bcb84ed3172c0c1219cbbc10b5-cdf129.png)

# 监督学习

在监督学习中我们需要预先定义输入数据的标准输出作为训练集和验证集。

**回归**：函数的输出是一个数值

如：输入历史比特币价格预测未来比特币价格

**分类**：输出为类别，根据类总数不同有二分类和多分类

在分类中需要选择模型，最简单的有线性模型，其次有DL、SVM等非线性模型

**结构化学习**：输出是有结构的，如AI翻译后的译文就是有结构的。





# 半监督学习

与监督学习不同，半监督学习不需要人为定义所有标准输出，如下图未定义的猫狗图片，其可能对于学习也有帮助（TODO：查资料）


![在这里插入图片描述](https://img-blog.csdnimg.cn/9ad5d36c8f01418cb5d736388489f8a0.png#pic_center)





# 迁移学习

来自知乎：https://www.zhihu.com/question/41979241

> 为了节省人工标注样本的时间，让模型可以通过已有的标记数据（source domain data）向未标记数据（target domain data）迁移。从而训练出适用于target domain的模型。



# 无监督学习

没有给定事先标记过的训练范例，自动对输入的资料进行分类或分群。无监督学习的主要运用包含：**聚类分析**（cluster analysis）、关联规则（association rule）、维度缩减（dimensionality reduce）。它是监督式学习和强化学习等策略之外的一种选择。

常用于**聚类**中。



# 强化学习

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2022/11/24/55cbc4118fc58becde221dd90312a1e2-8c2c73.png)

基于机器人与环境之间交互的学习，能够解决很多有监督学习方法无法解决的问题。