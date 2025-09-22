# 第10部分：FOR循环

在我们编写编译器的旅程的这一部分，我将添加FOR循环。在实现方面有一个需要解决的问题，我想在讨论如何解决它之前先解释一下。

## FOR循环的语法

我假设你熟悉FOR循环的语法。一个例子是：

```c
  for (i=0; i < MAX; i++)
    printf("%d\n", i);
```

我将使用这个BNF语法来表示我们的语言：

```
 for_statement: 'for' '(' preop_statement ';'
                          true_false_expression ';'
                          postop_statement ')' compound_statement  ;

 preop_statement:  statement  ;        (目前为止)
 postop_statement: statement  ;        (目前为止)
```

`preop_statement`是在循环开始之前运行的。稍后，我们将不得不限制这里可以执行的操作类型（例如，不能有IF语句）。然后评估`true_false_expression`。如果为真，则执行`compound_statement`。完成后，执行`postop_statement`，代码循环回到重新做`true_false_expression`。

## 问题所在

问题是`postop_statement`在`compound_statement`之前被解析，但我们必须在`compound_statement`的代码之后生成`postop_statement`的代码。

有几种解决这个问题的方法。在我写上一个编译器时，我选择将`compound_statement`的汇编代码放在一个临时缓冲区中，并在生成了`postop_statement`的代码之后“回放”缓冲区。在SubC编译器中，Nils巧妙地使用标签和跳转到标签来“串联”代码的执行以确保正确的顺序。

但我们在这里构建了一个AST树。让我们用它来保证生成汇编代码的正确顺序。

## 什么样的AST树？

你可能已经注意到，FOR循环有四个结构组成部分：

 1. `preop_statement`
 2. `true_false_expression`
 3. `postop_statement`
 4. `compound_statement`

我不太想再次改变AST节点结构来拥有四个子节点。但我们可以将FOR循环视为增强的WHILE循环：

```
   preop_statement;
   while ( true_false_expression ) {
     compound_statement;
     postop_statement;
   }
```

我们可以用我们现有的节点类型构建一个AST树来反映这种结构吗？可以：

```
         A_GLUE
        /     \
   preop     A_WHILE
             /    \
        decision  A_GLUE
                  /    \
            compound  postop
```

手动从上到下从左到右遍历这棵树，并说服自己我们将以正确的顺序生成汇编代码。
我们必须将`compound_statement`和`postop_statement`粘合在一起，这样，当WHILE循环退出时，它将跳过`compound_statement`和`postop_statement`。

这也意味着我们需要一个新的T_FOR令牌，但我们不需要一个新的AST节点类型。因此，唯一的编译器更改将是扫描和解析。

## 令牌和扫描

这里有一个新的关键字'for'和一个关联的令牌，T_FOR。这里没有大的变化。

## 解析语句

我们确实需要对解析器进行结构性改变。对于FOR语法，我只想要`preop_statement`和`postop_statement`作为单个语句。目前，我们有一个`compound_statement()`函数，它只是循环直到遇到右花括号'}'。我们需要将其分离出来，以便`compound_statement()`调用`single_statement()`来获取一个语句。

但还有一个问题。看看现有的在`assignment_statement()`中解析赋值语

句。解析器必须在语句的末尾找到一个分号。

这对于复合语句是好的，但对于FOR循环则不行。我必须写类似这样的东西：

```c
  for (i=1 ; i < 10 ; i= i + 1; )
```

因为每个赋值语句*必须*以分号结束。

我们需要的是单个语句解析器*不*扫描分号，而是留给复合语句解析器来处理。并且我们扫描一些语句的分号（例如，在赋值语句之间）而不是其他语句（例如，不在连续的IF语句之间）。

解释了所有这些之后，让我们来看看新的单个和复合语句解析代码：

```c
// 解析一个单个语句
// 并返回其AST
static struct ASTnode *single_statement(void) {
  switch (Token.token) {
    case T_PRINT:
      return (print_statement());
    case T_INT:
      var_declaration();
      return (NULL);		// 这里没有生成AST
    case T_IDENT:
      return (assignment_statement());
    case T_IF:
      return (if_statement());
    case T_WHILE:
      return (while_statement());
    case T_FOR:
      return (for_statement());
    default:
      fatald("语法错误, token", Token.token);
  }
}

// 解析一个复合语句
// 并返回其AST
struct ASTnode *compound_statement(void) {
  struct ASTnode *left = NULL;
  struct ASTnode *tree;

  // 需要一个左花括号
  lbrace();

  while (1) {
    // 解析一个单个语句
    tree = single_statement();

    // 一些语句后面必须跟一个分号
    if (tree != NULL &&
	(tree->op == A_PRINT || tree->op == A_ASSIGN))
      semi();

    // 对于每个新树，如果left为空，则将其保存在left中，
    // 否则将left和新树粘合在一起
    if (tree != NULL) {
      if (left == NULL)
	left = tree;
      else
	left = mkastnode(A_GLUE, left, NULL, tree, 0);
    }
    // 当我们碰到右花括号时，
    // 跳过它并返回AST
    if (Token.token == T_RBRACE) {
      rbrace();
      return (left);
    }
  }
}
```

我还删除了`print_statement()`和`assignment_statement()`中的`semi()`调用。

## 解析 FOR 循环

根据上述 FOR 循环的BNF语法，这一过程很直接。鉴于我们想要的AST树的结构，构建这棵树的代码也很直接。以下是代码：

```c
// 解析一个FOR语句
// 并返回它的AST
static struct ASTnode *for_statement(void) {
  struct ASTnode *condAST, *bodyAST;
  struct ASTnode *preopAST, *postopAST;
  struct ASTnode *tree;

  // 确保我们有 'for' '('
  match(T_FOR, "for");
  lparen();

  // 获取前操作语句和分号 ';'
  preopAST = single_statement();
  semi();

  // 获取条件和分号 ';'
  condAST = binexpr(0);
  if (condAST->op < A_EQ || condAST->op > A_GE)
    fatal("比较操作符错误");
  semi();

  // 获取后操作语句和右括号 ')'
  postopAST = single_statement();
  rparen();

  // 获取复合语句，即循环体
  bodyAST = compound_statement();

  // 目前，所有四个子树都必须非空。
  // 稍后，我们将改变语义以适应某些子树缺失的情况

  // 将复合语句和后操作树粘合
  tree = mkastnode(A_GLUE, bodyAST, NULL, postopAST, 0);

  // 使用条件和这个新体构造一个WHILE循环
  tree = mkastnode(A_WHILE, condAST, NULL, tree, 0);

  // 并将前操作树粘到A_WHILE树上
  return(mkastnode(A_GLUE, preopAST, NULL, tree, 0));
}
```

## 生成汇编代码

好吧，我们所做的就是合成了一棵包含WHILE循环的树，其中有一些子树被粘合在一起，因此编译器生成部分没有变化。

## 尝试一下

`tests/input07` 文件中包含了这个程序：

```c
{
  int i;
  for (i = 1; i <= 10; i = i + 1) {
    print i;
  }
}
```

当我们执行 `make test7` 时，我们得到以下输出：

```
cc -o comp1 -g cg.c decl.c expr.c gen.c main.c misc.c scan.c
    stmt.c sym.c tree.c
./comp1 tests/input07
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

以下是相关的汇编输出：

```
	.comm	i,8,8
	movq	$1, %r8
	movq	%r8, i(%rip)		# i = 1
L1:
	movq	i(%rip), %r8
	movq	$10, %r9
	cmpq	%r9, %r8		# i < 10?
	jg	L2			# i >= 10, 跳到 L2
	movq	i(%rip), %r8
	movq	%r8, %rdi
	call	printint		# 打印 i
	movq	i(%rip), %r8
	movq	$1, %r9
	addq	%r8, %r9		# i = i + 1
	movq	%r9, i(%rip)
	jmp	L1			# 跳到循环顶部
L2:
```

## 结论和下一步

我们的语言现在有了相当数量的控制结构：IF语句、WHILE循环和FOR循环。问题是，接下来做什么？有很多事情我们可以考虑：

 + 类型
 + 局部与全局事物
 + 函数
 + 数组和指针
 + 结构体和联合
 + 自动变量、静态变量及其朋友们

我决定先研究函数。所以，在我们编写编译器的旅程中

的下一部分，我们将开始添加函数到我们语言的几个阶段中的第一个。[下一步](11-Functions_pt1.md)
