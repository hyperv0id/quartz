1. 编译器：将“高级”语言转换成“低级”语言的程序，不过转换成另一种语言也行。
2. 高级：适合程序员
3. 低级：适合机器


## 语言类应用程序

- 配置文件：yaml、properties、xml、toml
- 数据分析：csv/excel
- web领域：json文件
- 数据库领域：[SQL](../../../../system/information-system/02-SQL.md)引擎
- 分布式：TLA+（用于分析分布式协议的语言）eg：[PaxosStore-tla/specification/TPaxos.tla at master · Starydark/PaxosStore-tla · GitHub](https://github.com/Starydark/PaxosStore-tla/blob/master/specification/TPaxos.tla)
- Java字节码解释器：javac
- C/CPP编译器：gcc、llvm
- 排版绘图语言：Latex、Dot
- 演奏音乐的语言：[alda.io :: Alda](https://alda.io/)


## 编译器的构成

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226231859-2a4a2a.png)

- 前端（分析阶段）：分析语言程序，收集必要信息
- 后端（综合阶段）：利用收集到的信息，生产目标语言程序
- IR（Intermediate Representation）：中间表示

添加优化阶段，这和系统无关

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226232147-4e49a4.png)


### 前端（分析阶段）
前后端构成
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226232257-4f1505.png)

工业上前端的词法分析、语法分析、语义分析技术已经非常成熟，如华为方舟编译器的这几个部分全都没有。因此最后的中间代码**优化**才是最重要的。

```
position = initial + rate * 60
```

词法：标识符、数字、运算符
语法：包含算术运算 的  赋值语句
语义：position、initial、rate属于数值类型
物理定律：当前位置 = 初始位置 + 速度 \* 时间

#### 词法分析器（Lexer/Scanner）
词法分析器（Lexer/Scanner）：将**字符流**转换成**词法单元(Token)** 流

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226233116-2e1b4b.png)

#### 语法分析器（Parser）

语法分析器：构建**词法单元**之间的语法结构，生成**语法树**

![语法分析器y](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226233247-4cca1e.png)

#### 语义分析

语义分析器：语义检查，如**类型检查**，“先声明后使用”等**约束检查**

![语义分析](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226233323-6a1bea.png)


#### 中间代码优化器

中间代码优化器：编译时计算、消除冗余临时变量

最简单的就是自动删除没用的变量，如这里的`t1`
![代码优化器](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226233559-e659b1.png)

#### 代码生成器
代码生成器：生成目标代码，包括**指令选择**、**寄存器分配**
![代码生成器](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226233801-8e9ba5.png)

#### 符号表

符号表：收集&管理**变量名**、**函数名**相关信息

![符号表](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/26/20231226233929-5a08d8.png)

为了方便表达作用域、嵌套结构，符号表可能**嵌套**
### 后端（综合阶段）


