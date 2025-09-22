![图灵奖得主](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/30/20231230113632-8c0f2d.png)


## 定义

基本思想：**按结构归纳**

在给定字母表$\Sigma$下，其[正则表达式](概念/正则表达式.md)可**由且仅由**以下规则定义：
1. $\epsilon$是作者表达式
2. $\forall a \in \Sigma$, $a$是[正则表达式](概念/正则表达式.md)
3. 如果$s$是[正则表达式](概念/正则表达式.md)，那么$(s)$是[正则表达式](概念/正则表达式.md)
4. 如果$s$与$t$是[正则表达式](概念/正则表达式.md)，那么$s|t$，$st$，$s^*$也是[正则表达式](概念/正则表达式.md)

## 例子

### $\epsilon$是一个[正则表达式](概念/正则表达式.md)

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/30/20231230114304-c8ea3a.png)

### $\forall a \in \Sigma$, $a$是[正则表达式](概念/正则表达式.md)
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/30/20231230114339-19549e.png)


### 如果$s$是[正则表达式](概念/正则表达式.md)，那么$(s)$是[正则表达式](概念/正则表达式.md)
仅加一个括号，当中的语言不变
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/30/20231230114408-6586a4.png)


### 如果$s$与$t$是[正则表达式](概念/正则表达式.md)，那么$s|t$是[正则表达式](概念/正则表达式.md)

如何表达**或者**？

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/30/20231230114455-5958bb.png)

问题：

> [!question]+
> 如果$N(s), N(t)$的开始状态和接受状态**唯一**？
> > [!note]- Answer
> > 在构造的过程中保证这一点，在所有的构造过程中，始终保证这一点。
> > 根据**归纳假设**，开始状态和接受状态均唯一

### 如果$s$与$t$是[正则表达式](概念/正则表达式.md)，那么$st$也是[正则表达式](概念/正则表达式.md)

将s的终止状态连上t的初始状态
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/30/20231230115535-2a5888.png)

同样可以在构造过程中保证状态的唯一性


### 如果$s$与$t$是[正则表达式](概念/正则表达式.md)，那么$s^*$也是[正则表达式](概念/正则表达式.md)

添加4挑$\epsilon$转移表达\*，即可以表示0次或多次
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/30/20231230115652-2d14cc.png)

## Thompson构造法分析

### 性质

1. $N(r)$的开始状态和接受状态**唯一**
2. 开始状态没有入边，结束状态没有出边
3. $N(r)$的**状态数**$|S| \leq 2\times |r|$，$|r|$表示$r$中运算符和运算分量的总和
4. 每个状态最多两个$\epsilon-$入边，和两个$\epsilon-$出边
5. $\forall a \in \Sigma$，每个状态最多有一个$a-$入边和一个$a-$出边

### 复杂度分析

$N(r)$的**状态数**$|S| \leq 2\times |r|$，$|r|$表示$r$中运算符和运算分量的总和

复杂度是线性关系


### 例子

> [!question]
> `r = (a|b)*abb`
> > [!note]-
> > ![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/30/20231230120335-b9e8b8.png)
> 




