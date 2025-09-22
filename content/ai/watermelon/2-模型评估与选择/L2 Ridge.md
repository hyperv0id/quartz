---
aliases:
  - L2正则
  - 岭回归
tags:
  - ai/正则化
---
# L2 Ridge

也叫做岭回归。

$$  
\begin{align} \hat{w}=\mathop{argmin}\limits_wL(w)+\lambda w^Tw&\longrightarrow\frac{\partial}{\partial w}L(w)+2\lambda w=0\nonumber\\ &\longrightarrow2X^TX\hat{w}-2X^TY+2\lambda \hat w=0\nonumber\\ &\longrightarrow \hat{w}=(X^TX+\lambda \mathbb{I})^{-1}X^TY \end{align}  
$$

  

可以看到，这个正则化参数和前面的 MAP 结果不谋而合。利用2范数进行正则化不仅可以是模型选择 w 较小的参数，同时也避免  X^TX不可逆的问题。