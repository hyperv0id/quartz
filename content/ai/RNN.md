---
aliases:
  - Recurrent neural network
  - 循环神经网络
tags:
  - ai/神经网络/rnn
---



![RNN](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/05/05/image-20240505150749288-b4a6c2.png)

原有的[CNN](CNN.md)没有考虑到时间先后的特征，在nlp领域效果不好，因此使用RNN来将时间序列的输入作为因素考虑。





梯度消失和梯度爆炸问题，这使得RNN在处理长序列时，难以捕捉序列中的长距离依赖关系。为了解决这些问题，研究者们提出了一些改进的RNN结构，如长短期记忆网络（LSTM）和门控循环单元（GRU）。