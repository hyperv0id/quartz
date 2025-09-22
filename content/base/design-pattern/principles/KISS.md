---
tags:
  - design-pattern
Category: design-pattern
created: 2023-03-16T00:43:34 (UTC +08:00)
---


# KISS

KISS 原则是指在设计当中应当**注重简约**的原则。总结工程专业人员在设计过程中的经验，大多数系统的设计应保持简洁和单纯，而不掺入非必要的复杂性，这样的系统运作成效会取得最优；因此简单性应该是设计中的关键目标，尽量避免不必要的复杂性。

> 有点像奥卡姆剃刀原理，如无必要，勿增实体

## 使用动机

KISS原则是追求简单。KISS原则指出，当一个解决方案使用更少的继承、多态、类时，它会更好。

## 使用原则

- 避免继承、多态、动态绑定等复杂的OOP概念。请改用委派和简单的 if 构造。
- 避免算法的低级优化，尤其是在涉及汇编程序、位操作和指针时。较慢的实现将正常工作。
- 使用简单的暴力解决方案而不是复杂的算法。较慢的算法将首先起作用。
- 避免使用大量类和方法以及大型代码块，如20行的case，100行的方法
- 对于稍微不相关但相当小的功能，请使用私有方法而不是额外的类。
- 避免需要参数化的常规解决方案。一个特定的解决方案就足够了。

## 使用示例


## 资源

1. [KISS原则 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/KISS原则)
1. [Programming Principles | Java Design Patterns (中文) (java-design-patterns.com)](https://java-design-patterns.com/zh/principles/#kiss)
1. 