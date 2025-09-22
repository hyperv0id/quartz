---
aliases:
  - 非确定性有限状态机
  - Nondeterministic Finite State Machine
  - Nondeterministic FSM
  - NFSM
---

# NFA

## 简介


Nondeterministic Finite State Machine（Nondeterministic FSM）是一种[有限状态机](FSM.md)（[Finite State Machine](FSM.md)，FSM）的变体，它在状态转换时允许多个可能的选择，而不是[确定性有限状态机](DFA.md)（Deterministic FSM）那样只允许一种确定的状态转换。

在[确定性有限状态机](DFA.md)中，任何给定的输入符号和当前状态都会唯一确定下一个状态。但是，对于[非确定性有限状态机](NFA.md)，一个输入和当前状态可能导致多个可能的下一个状态，系统可以在这些状态中进行非确定性的选择。

## 定义
NFA：非确定的有穷状态机，组成是一个五元组 $A = (\Sigma, S, s_0, \delta, F)$
1. 字母表：$\Sigma$
2. **有穷**状态集合：$S$
3. **唯一**的初始状态： $s_0 \in S$
4. 状态转移函数：$\delta: S \times(\Sigma \cup\{\epsilon\}) \rightarrow 2^{S}$
5. 接受状态集合：$F \in S$
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229184840-b1043f.png)

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229185305-8643e8.png)

NFA论文：[Finite Automata and Their Decision Problems | IBM Journals & Magazine | IEEE Xplore](https://ieeexplore.ieee.org/abstract/document/5392601)

- 接受：从初始状态读第一个字符，查状态转移函数，直到消耗完所有字符，停在一个接受状态。
- $L(A)$：机器能接受的所有字符串的集合

## 例子

规则是 以`abb`结尾 的NFA例子
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229185859-6da849.png)

一个或者任意多个a或者一个或多个b
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229190209-f9f211.png)


偶数个0或偶数个1
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229190239-00caf4.png)
