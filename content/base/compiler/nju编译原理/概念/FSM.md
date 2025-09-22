---
aliases:
  - 有限状态机
  - Finite State Machine
---
有限状态机（Finite State Machine，FSM）是一种抽象的计算模型，它表示一个系统的行为，这个系统可以处于有限个状态中的一个。FSM由以下几个基本组成部分构成：

1. **状态（States）：** 描述系统可能处于的不同状态。状态可以包括系统内部的某些属性或条件，以反映系统在某一时刻的特定情况。
2. **转移（Transitions）：** 描述系统在不同状态之间如何转换的规则。转移通常与输入事件相关联，即当系统接收到特定的输入时，它会根据转移规则从一个状态转移到另一个状态。
3. **初始状态（Initial State）：** 系统在开始时所处的状态。
4. **终止状态（Final States or Accepting States）：** 表示系统执行特定序列的输入后可能达到的结束状态。在某些情况下，FSM可能没有终止状态，而是一直循环执行。

有限状态机可以分为两种主要类型：

- **[确定性有限状态机](DFA.md)（[Deterministic Finite State Machine](DFA.md)，[DFSM](DFA.md)）：** 对于给定的输入和当前状态，系统具有唯一确定的下一个状态。每个状态只有一个出边与每个输入相关联。
- **[非确定性有限状态机](NFA.md)（[Nondeterministic Finite State Machine](NFA.md)，[NFSM](NFA.md)）：** 对于给定的输入和当前状态，系统可以有多个可能的下一个状态。一个输入可能导致系统在多个状态之间非确定性地选择。