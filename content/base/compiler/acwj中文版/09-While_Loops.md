# 第9部分：While循环

在这部分的学习旅程中，我们将向我们的语言中添加WHILE循环。在某种意义上，WHILE循环非常类似于没有'else'子句的IF语句，只是我们总是跳回到循环的顶部。

所以，这样的代码：

```
  while (条件为真) {
    语句;
  }
```

应该被翻译为：

```
Lstart: 计算条件
	如果条件为假则跳转到Lend
	语句
	跳转到Lstart
Lend:
```

这意味着我们可以借用我们在IF语句中使用的扫描、解析和代码生成结构，并对其进行一些小的修改，以便也能处理WHILE语句。

让我们看看如何实现这一点。

## 新的令牌

我们需要一个新的令牌，T_WHILE，用于新的'while'关键字。对`defs.h`和`scan.c`的修改是显而易见的，所以在这里我将省略它们。

## 解析While语法

WHILE循环的BNF语法是：

```
// while_statement: 'while' '(' true_false_expression ')' compound_statement  ;
```

我们需要在`stmt.c`中添加一个函数来解析这个。这是该函数；注意这与解析IF语句相比的简单性：

```c
// 解析一个WHILE语句
// 并返回其AST
struct ASTnode *while_statement(void) {
  struct ASTnode *condAST, *bodyAST;

  // 确保我们有'while' '('
  match(T_WHILE, "while");
  lparen();

  // 解析接下来的表达式
  // 并且解析随后的')'。确保
  // 树的操作是比较。
  condAST = binexpr(0);
  if (condAST->op < A_EQ || condAST->op > A_GE)
    fatal("比较操作符错误");
  rparen();

  // 获取复合语句的AST
  bodyAST = compound_statement();

  // 构建并返回这个语句的AST
  return (mkastnode(A_WHILE, condAST, NULL, bodyAST, 0));
}
```

我们需要一个新的AST节点类型，A_WHILE，它已经被添加到`defs.h`中。这个节点有一个左子树来评估条件，和一个右子树来作为WHILE循环的复合语句的主体。

## 通用代码生成

我们需要创建一个开始和结束标签，计算条件，并插入适当的跳转以退出循环并返回到循环的顶部。再次说明，这比生成IF语句的代码要简单得多。在`gen.c`中：

```c
// 为WHILE语句生成代码
// 和一个可选的ELSE子句
static int genWHILE(struct ASTnode *n) {
  int Lstart, Lend;

  // 生成开始和结束标签
  // 并输出开始标签
  Lstart = label();
  Lend = label();
  cglabel(Lstart);

  // 生成条件代码，紧随其后
  // 是跳转到结束标签。
  // 我们通过将Lfalse标签作为寄存器发送来作弊。
  genAST(n->left, Lend, n->op);
  genfreeregs();

  // 生成主体的复合语句
  genAST(n->right, NOREG, n->op);
  genfreeregs();

  // 最后输出回到条件的跳转，
  // 和结束标签
  cgjump(Lstart);
  cglabel(Lend);
  return (NOREG);
}
```

我必须做的一件事是认识到比较运算符的父AST节点现在可以是A_WHILE，所以在`genAST()`中，比较运算符的代码看起来像：

```c
    case A_EQ:
    case A_NE:
    case A

_LT:
    case A_GT:
    case A_LE:
    case A_GE:
      // 如果父AST节点是A_IF或A_WHILE，生成
      // 比较后跟随一个跳转。否则，比较寄存器
      // 并根据比较设置一个为1或0。
      if (parentASTop == A_IF || parentASTop == A_WHILE)
        return (cgcompare_and_jump(n->op, leftreg, rightreg, reg));
      else
        return (cgcompare_and_set(n->op, leftreg, rightreg));
```

这总的来说，就是实现WHILE循环所需要的全部！

## 测试新的语言添加

我已经将所有的输入文件移动到了一个`test/`目录中。如果你现在执行`make test`，它将进入这个目录，编译每个输入文件，并将输出与已知的良好输出进行比较：

```
cc -o comp1 -g cg.c decl.c expr.c gen.c main.c misc.c scan.c stmt.c
      sym.c tree.c
(cd tests; chmod +x runtests; ./runtests)
input01: OK
input02: OK
input03: OK
input04: OK
input05: OK
input06: OK
```

你还可以执行`make test6`。这将编译`tests/input06`文件：

```c
{ int i;
  i=1;
  while (i <= 10) {
    print i;
    i= i + 1;
  }
}
```

这将打印出从1到10的数字：

```sh
cc -o comp1 -g cg.c decl.c expr.c gen.c main.c misc.c scan.c
      stmt.c sym.c tree.c
./comp1 tests/input06
cc -o out out.s
./out
1
2
3
4
5
6
7
8
9
10
```

这里是编译产生的汇编输出：

```
	.comm	i,8,8
	movq	$1, %r8
	movq	%r8, i(%rip)		# i= 1
L1:
	movq	i(%rip), %r8
	movq	$10, %r9
	cmpq	%r9, %r8		# i是否小于等于10？
	jg	L2			# 大于，跳转到L2
	movq	i(%rip), %r8
	movq	%r8, %rdi		# 打印出i
	call	printint
	movq	i(%rip), %r8
	movq	$1, %r9
	addq	%r8, %r9		# i加1
	movq	%r9, i(%rip)
	jmp	L1			# 然后回到循环
L2:
```


## 结论和接下来的内容

一旦我们已经完成了IF语句，添加WHILE循环就变得容易了，因为它们有很多相似之处。

我认为我们现在也拥有了一个[图灵完备](https://zh.wikipedia.org/wiki/图灵完备)的语言：

  + 无限量的存储，即无限数量的变量
  + 基于存储值做出决策的能力，即IF语句
  + 改变方向的能力，即WHILE循环

所以我们现在可以停止了，我们的任务完成了！不，当然不是。我们仍在努力使编译器能够编译自身。

在我们的编译器编写旅程的下一部分，我们将向语言中添加FOR循环。[下一步](10-FOR%20Loops.md)