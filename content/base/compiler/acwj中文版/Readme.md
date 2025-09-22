# 编译器之旅

在这个 Github 存储库中，我正在记录我编写一个用于 C 语言子集的自编译编译器的过程。我也在详细描述我的做法，这样，如果你想跟着学习，就会有解释我做了什么，为什么这样做，并且有一些回溯到编译器理论的参考资料。

但不要太多理论，我希望这是一次实践之旅。

以下是我迄今为止所采取的步骤：

+ [第 0 部分](00-Introduction.md)：之旅介绍
+ [第 1 部分](01-Scanner.md)：词法扫描介绍
+ [第 2 部分](02-Parser.md)：解析介绍
+ [第 3 部分](03-Precedence.md)：运算符优先级
+ [第 4 部分](04-Assembly.md)：一个实际的编译器
+ [第 5 部分](05-Statements.md)：语句
+ [第 6 部分](06-Variables.md)：变量
+ [第 7 部分](07-Comparisons.md)：比较运算符
+ [第 8 部分](08-If_Statements.md)：if 语句
+ [第 9 部分](09-While_Loops.md)：while 循环
+ [第 10 部分](10-FOR%20Loops.md)：for 循环
+ [第 11 部分](11-Functions_pt1.md)：函数，第 1 部分
+ [第 12 部分](12-Types_pt1.md)：类型，第 1 部分
+ [第 13 部分](13-Functions_pt2.md)：函数，第 2 部分
+ [第 14 部分](14-ARM_Platform.md)：生成 ARM 汇编代码
+ [第 15 部分](15-Pointers_pt1.md)：指针，第 1 部分
+ [第 16 部分](16-Global_Vars.md)：正确声明全局变量
+ [第 17 部分](17-Scaling_Offsets.md)：更好的类型检查和指针偏移
+ [第 18 部分](18-Lvalues_Revisited.md)：Lvalues 和 Rvalues 重新审视
+ [第 19 部分](19-Arrays_pt1.md)：数组，第 1 部分
+ [第 20 部分](20-Char_Str_Literals.md)：字符和字符串字面量
+ [第 21 部分](21-More_Operators.md)：更多运算符
+ [第 22 部分](22-Design_Locals.md)：本地变量和函数调用的设计思路
+ [第 23 部分](23-Local_Variables.md)：本地变量
+ [第 24 部分](24_Function_Params.md)：函数参数
+ [第 25 部分](25_Function_Arguments.md)：函数调用和参数
+ [第 26 部分](26_Prototypes.md)：函数原型
+ [第 27 部分](27_Testing_Errors.md)：回归测试和一个令人愉快的惊喜
+ [第 28 部分](28_Runtime_Flags.md)：添加更多运行时标志
+ [第 29 部分](29_Refactoring.md)：稍微重构一下
+ [第 30 部分](30_Design_Composites.md)：设计结构体、联合体和枚举类型
+ [第 31 部分](31_Struct_Declarations.md)：实现结构体，第 1 部分
+ [第 32 部分](32_Struct_Access_pt1.md)：访问结构体成员
+ [第 33 部分](33_Unions.md)：实现联合体和成员访问
+ [第 34 部分](34_Enums_and_Typedefs.md)：枚举和类型定义
+ [第 35 部分](35_Preprocessor.md)：C 预处理器
+ [第 36 部分](36_Break_Continue.md)：`break` 和 `continue`
+ [第 37 部分](37_Switch.md)：switch 语句
+ [第 38 部分](38_Dangling_Else.md)：悬垂的 else 和更多内容
+ [第 39 部分](39_Var_Initialisation_pt1.md)：变量初始化，第 1 部分
+ [第 40 部分](40_Var_Initialisation_pt2.md)：全局变量初始化
+ [第 41 部分](41_Local_Var_Init.md)：本地变量初始化
+ [第 42 部分](42_Casting.md)：类型转换和 NULL
+ [第 43 部分](43_More_Operators.md)：错误修复和更多运算符
+ [第 44 部分](44_Fold_Optimisation.md)：常量折叠
+ [第 45 部分](45_Globals_Again.md)：全局变量声明，重新审视
+ [第 46 部分](46_Void_Functions.md)：void 函数参数和扫描更改
+ [第 47 部分](47_Sizeof.md)：`sizeof` 的一个子集
+ [第 48 部分](48_Static.md)：`static` 的一个子集
+ [第 49 部分](49_Ternary.md)：三元运算符
+ [第 50 部分](50_Mop_up_pt1.md)：清理工作，第 1 部分
+ [第 51 部分](51_Arrays_pt2.md)：数组，第 2 部分
+ [第 52 部分](52_Pointers_pt2.md)：指针，第 2 部分
+ [第 53 部分](53_Mop_up_pt2.md)：清理工作，第 2 部分
+ [第 54 部分](54_Reg_Spills.md)：寄存器溢出
+ [第 55 部分](55_Lazy_Evaluation.md)：延迟计算
+ [第 56 部分](56_Local_Arrays.md)：本地数组
+ [第 57 部分](57_Mop_up_pt3.md)：清理工作，第 3 部分
+ [第 58 部分](58-Ptr_Increments.md)：修复指针增量/减量
+ [第 59 部分](59-WDIW_pt1.md)：为什么不起作用，第 1 部分
+ [第 60 部分](60-TripleTest.md)：通过三重测试
+ [第 61 部分](61-What_Next.md)：下一步是什么？
+ [第 62 部分](62-Cleanup.md)：代码清理
+ [第 63 部分](63-QBE.md)：使用 QBE 的新后端

未来部分没有安排或时间表，所以请继续查看这里，看看我是否已经写了更多。

## 版权

我借用了一些代码和很多想法，来自 Nils M Holm 编写的 [SubC](http://www.t3x.org/subc/) 编译器。他的代码是公有领域的。我认为我的代码有着足够大的不同，可以对我的代码应用不同的许可证。

除非另有说明，

+ 所有源代码和脚本均为 Warren Toomey 在 GPL3 许可下的版权所有。
+ 所有非源代码文档（例如英文文档、图像文件）均为 Warren Toomey 在 Creative Commons BY-NC-SA 4.0 许可下的版权所有。
