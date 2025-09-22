# 第二部分：解析简介

在我们的编译器之旅的这一部分，我将介绍解析器的基础知识。正如我在第一部分中提到的，解析器的工作是识别输入的语法和结构元素，并确保它们符合语言的*语法*。

我们已经有了几个可以扫描的语言元素，即我们的标记：

+ 四个基本的数学运算符：`*`、`/`、`+` 和 `-`
+ 十进制整数，其中包含1个或多个数字 `0` 到 `9`

现在让我们为我们的解析器定义一种语言的语法。

## BNF：巴科斯-诺尔范式

在处理计算机语言时，您可能会在某个时候遇到[BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form)的使用。我将在这里介绍足够表达我们想要识别的语法的BNF语法的基础知识。

我们希望有一个用于表示带有整数的数学表达式的语法。以下是语法的BNF描述：

```
expression: number
          | expression '*' expression
          | expression '/' expression
          | expression '+' expression
          | expression '-' expression
          ;

number:  T_INTLIT
         ;
```

竖线分隔语法中的选项，因此上面的语法表示：

+ 表达式可以只是一个数字，或者
+ 表达式是两个由 '\*' 标记分隔的表达式，或者
+ 表达式是两个由 '/' 标记分隔的表达式，或者
+ 表达式是两个由 '+' 标记分隔的表达式，或者
+ 表达式是两个由 '-' 标记分隔的表达式
+ 数字总是一个 T_INTLIT 标记

很明显，语法的BNF定义是*递归的*：一个表达式通过引用其他表达式来定义。但是有一种*底层递归*的方法：当表达式最终是一个数字时，这总是一个 T_INTLIT 标记，因此不是递归的。

在BNF中，我们说"expression"和"number"是*非终结符*符号，因为它们是由语法规则产生的。然而，T_INTLIT是*终结符*符号，因为它不由任何规则定义。相似地，四个数学运算符标记也是终结符号。

## 递归下降解析

考虑到我们语言的语法是递归的，我们可以尝试使用递归地解析它。我们需要做的是读入一个标记，然后*预读*下一个标记。根据下一个标记是什么，我们可以决定解析输入需要采取哪个路径。这可能需要我们递归调用已经调用过的函数。

在我们的情况下，任何表达式中的第一个标记将是一个数字，后面可能会有数学运算符。之后可能只有一个数字，或者可能是一个全新表达式的开始。我们如何递归解析这个呢？

我们可以编写伪代码，看起来像这样：

```
function expression() {
  扫描并检查第一个标记是否是数字。如果不是，则报错
  获取下一个标记
  如果我们已经到达输入的末尾，则返回，即基本情况

  否则，调用 expression()
}
```

让我们在输入 `2 + 3 - 5 T_EOF` 上运行这个函数，其中 `T_EOF` 是一个反映输入结束的标记。我将为 `expression()` 的每次调用编号。

```
expression0:
  扫描 2，它是一个数字
  获取下一个标记，+，它不是 T_EOF
  调用 expression()

    expression1:
      扫描 3，它是一个数字
      获取下一个标记，-，它不是 T_EOF
      调用 expression()

        expression2:
          扫描 5，它是一个数字
          获取下一个标记，T_EOF，因此从 expression2 返回

      从 expression1 返回
  从 expression0 返回
```

是的，这个函数能够递归解析输入 `2 + 3 - 5 T_EOF`。

当然，我们还没有对输入做任何处理，但这不是解析器的工作。解析器的工作是*识别*输入，并警告任何语法错误。有人将进行输入的*语义分析*，即理解并执行此输入的含义。

> 稍后，你会发现这并不是真的。将语法分析和语义分析交织在一起通常是有道理的。

## 构建抽象语法树

要进行语义分析，我们需要代码来解释已识别的输入，或将其翻译成另一种格式，例如汇编代码。在这一部分的旅程中，我们将为输入构建一个解释器。但为了达到这个目标，我们首先要将输入转换为[抽象语法树](https://en.wikipedia.org/wiki/Abstract_syntax_tree)，也称为AST。

我强烈建议你阅读这篇关于AST的简短解释：

+ [通过AST提升解析游戏](https://medium.com/basecs/leveling-up-ones-parsing-game-with-asts-d7a6fc2400ff) by Vaidehi Joshi

这篇文章写得很好，真的有助于解释AST的目的和结构。别担心，你阅读完后我还会在这里。

我们将要构建的AST中的每个节点的结构在 `defs.h` 中描述：

```c
// AST节点类型
enum {
  A_ADD, A_SUBTRACT, A_MULTIPLY, A_DIVIDE, A_INTLIT
};

// 抽象语法树结构
struct ASTnode {
  int op;                               // 在此树上执行的 "操作"
  struct ASTnode *left;                 // 左右子树
  struct ASTnode *right;
  int intvalue;                         // 对于 A_INTLIT，整数值
};
```

一些AST节点，例如 `op` 值为 `A_ADD` 和 `A_SUBTRACT` 的节点，具有两个由 `left` 和 `right` 指向的子AST。稍后，我们将对子树的值进行相加或相减。

另外，`op` 值为 `A_INTLIT` 的AST节点表示整数值。它没有子树，只有 `intvalue` 字段中的值。

## 构建AST节点和树

`tree.c` 中的代码包含构建AST的函数。最通用的函数是 `mkastnode()`，它接受AST节点中所有四个字段的值。它分配节点，填充字段值，并返回指向节点的指针：

```c
// 构建并返回通用的AST节点
struct ASTnode *mkastnode(int op, struct ASTnode *left,
                          struct ASTnode *right, int intvalue) {
  struct ASTnode *n;

  // 分配一个新的AST节点
  n = (struct ASTnode *) malloc(sizeof(struct ASTnode));
  if (n == NULL) {
    fprintf(stderr, "Unable to malloc in mkastnode()\n");
    exit(1);
  }
  // 复制字段值并返回
  n->op = op;
  n->left = left;
  n->right = right;
  n->intvalue = intvalue;
  return (n);
}
```

有了这个函数，我们可以编写更具体的函数，制作一个叶子AST节点（即没有子节点的节点），以及制作一个具有单个子节点的AST节点：

```c
// 制作一个AST叶子节点
struct ASTnode *mkastleaf(int op, int intvalue) {
  return (mkastnode(op, NULL, NULL, intvalue));
}

// 制作一个一元AST节点：只有一个子节点
struct ASTnode *mkastunary(int op, struct ASTnode *left, int intvalue) {
  return (mkastnode(op, left, NULL, intvalue));
}
```
## AST的目的

我们将使用AST来存储我们识别的每个表达式，以便以后可以递归遍历它以计算表达式的最终值。我们确实希望处理数学运算符的优先级。这里有一个例子。

考虑表达式 `2 * 3 + 4 * 5`。现在，乘法比加法有更高的优先级。因此，我们希望在执行加法之前*绑定*乘法操作数并执行这些操作。

如果我们生成的AST树看起来像这样：

```
          +
         / \
        /   \
       /     \
      *       *
     / \     / \
    2   3   4   5
```

然后，当遍历树时，我们将首先执行 `2*3`，然后执行 `4*5`。一旦我们有了这些结果，我们就可以将它们传递到树的根部执行加法。

## 一个简单的表达式解析器

现在，我们可以重新使用来自我们扫描器的标记值作为AST节点操作值，但我喜欢将标记和AST节点的概念分开。因此，首先，我将有一个函数将标记值映射到AST节点操作值。这个函数以及解析器的其余部分都在 `expr.c` 中：

```c
// 将标记转换为AST操作。
int arithop(int tok) {
  switch (tok) {
    case T_PLUS:
      return (A_ADD);
    case T_MINUS:
      return (A_SUBTRACT);
    case T_STAR:
      return (A_MULTIPLY);
    case T_SLASH:
      return (A_DIVIDE);
    default:
      fprintf(stderr, "unknown token in arithop() on line %d\n", Line);
      exit(1);
  }
}
```

在switch语句中，当我们无法将给定的标记转换为AST节点类型时，默认语句将触发。这将成为我们解析器中的语法检查的一部分。

我们需要一个函数来检查下一个标记是否是整数文字，并构建一个AST节点来保存文字值。以下是它的实现：

```c
// 解析主要因子并返回表示它的AST节点。
static struct ASTnode *primary(void) {
  struct ASTnode *n;

  // 对于INTLIT标记，为其制作叶子AST节点
  // 并扫描下一个标记。否则，对于任何其他标记类型，都是语法错误。
  switch (Token.token) {
    case T_INTLIT:
      n = mkastleaf(A_INTLIT, Token.intvalue);
      scan(&Token);
      return (n);
    default:
      fprintf(stderr, "syntax error on line %d\n", Line);
      exit(1);
  }
}
```

这假定有一个全局变量 `Token`，并且它已经从输入中扫描到了最新的标记。在 `data.h` 中：

```c
extern_ struct token    Token;
```

在 `main()` 中：

```c
  scan(&Token);                 // 从输入中获取第一个标记
  n = binexpr();                // 解析文件中的表达式
```

现在我们可以编写解析器的代码：

```c
// 返回根为二进制运算符的AST树
struct ASTnode *binexpr(void) {
  struct ASTnode *n, *left, *right;
  int nodetype;

  // 获取左边的整数文字。
  // 同时获取下一个标记。
  left = primary();

  // 如果没有剩余的标记，只返回左节点
  if (Token.token == T_EOF)
    return (left);

  // 将标记转换为节点类型
  nodetype = arithop(Token.token);

  // 获取下一个标记
  scan(&Token);

  // 递归获取右侧树
  right = binexpr();

  // 现在使用两个子树构建树
  n = mkastnode(nodetype, left, right, 0);
  return (n);
}
```

请注意，在这个简单的解析器代码中，没有任何处理不同运算符优先级的内容。就目前而言，该代码将所有运算符视为具有相等优先级。如果你跟随代码解析表达式 `2 * 3 + 4 * 5`，你会发现它构建了以下AST：

```
     *
    / \
   2   +
      / \
     3   *
        / \
       4   5
```

这显然是不正确的。它将执行 `4*5` 以得到 20，然后执行 `3+20` 以得到 23，而不是执行 `2*3` 以得到 6。

那么为什么我这样做呢？我想向你展示编写一个简单的解析器很容易，但让它也执行语义分析更难。

## 解释树

既然我们有了（不正确的）AST树，让我们编写一些代码来解释它。同样，我们将编写递归代码来遍历树。这里是伪代码：

```
interpretTree:
  首先，解释左子树并获取其值
  然后，解释右子树并获取其值
  在树的根部的节点上执行操作
  对两个子树值执行此操作，并返回该值
```

回到正确的AST树：

```
          +
         / \
        /   \
       /     \
      *       *
     / \     / \
    2   3   4   5
```

调用结构看起来像：

```
interpretTree0(带+的树):
  调用interpretTree1(带*的左树):
     调用interpretTree2(带2的树):
       没有数学运算，只返回2
     调用interpretTree3(带3的树):
       没有数学运算，只返回3
     执行2 * 3，返回6

  调用interpretTree1(带*的右树):
     调用interpretTree2(带4的树):
       没有数学运算，只返回4
     调用interpretTree3(带5的树):
       没有数学运算，只返回5
     执行4 * 5，返回20

  执行6 + 20，返回26
```

## 解释树的代码

这部分代码在 `interp.c` 中，遵循上面的伪代码：

```c
// 给定一个AST，解释其中的运算符并返回最终值。
int interpretAST(struct ASTnode *n) {
  int leftval, rightval;

  // 获取左子树和右子树的值
  if (n->left)
    leftval = interpretAST(n->left);
  if (n->right)
    rightval = interpretAST(n->right);

  switch (n->op) {
    case A_ADD:
      return (leftval + rightval);
    case A_SUBTRACT:
      return (leftval - rightval);
    case A_MULTIPLY:
      return (leftval * rightval);
    case A_DIVIDE:
      return (leftval / rightval);
    case A_INTLIT:
      return (n->intvalue);
    default:
      fprintf(stderr, "Unknown AST operator %d\n", n->op);
      exit(1);
  }
}
```

同样，在switch语句中，当我们无法解释AST节点类型时，默认语句将触发。这将成为我们解析器中的语义检查的一部分。

## 构建解析器

这里还有一些其他的代码，比如在 `main()` 中调用解释器：

```c
  scan(&Token);                 // Get the first token from the input
  n = binexpr();                // Parse the expression in the file
  printf("%d\n", interpretAST(n));      // Calculate the final result
  exit(0);
```

现在，你可以通过执行以下命令构建解析器：

```
$ make
cc -o parser -g expr.c interp.c main.c scan.c tree.c
```

我为你提供了一些测试解析器的输入文件，当然你也可以创建自己的文件。请记住，计算结果是不正确的，但是解析器应该能够检测到输入错误，比如连续的数字、连续的运算符以及在输入末尾缺少数字。我还在解释器中添加了一些调试代码，以便你可以看到AST树节点的计算顺序：

```
$ cat input01
2 + 3 * 5 - 8 / 3

$ ./parser input01
int 2
int 3
int 5
int 8
int 3
8 / 3
5 - 2
3 * 3
2 + 9
11

$ cat input02
13 -6+  4*
5
       +
08 / 3

$ ./parser input02
int 13
int 6
int 4
int 5
int 8
int 3
8 / 3
5 + 2
4 * 7
6 + 28
13 - 34
-21

$ cat input03
12 34 + -56 * / - - 8 + * 2

$ ./parser input03
unknown token in arithop() on line 1

$ cat input04
23 +
18 -
45.6 * 2
/ 18

$ ./parser input04
Unrecognised character . on line 3

$ cat input05
23 * 456abcdefg

$ ./parser input05
Unrecognised character a on line 1
```

## 结论和接下来的步骤

解析器识别语言的语法并检查编译器的输入是否符合这个语法。如果不符合，解析器应该打印出错误消息。由于我们的表达式语法是递归的，我们选择编写递归下降解析器来识别我们的表达式。

目前，解析器能够工作，如上面的输出所示，但它未能正确地处理输入的语义。换句话说，它不能计算表达式的正确值。

在我们编写编译器的旅程的下一部分中，我们将修改解析器，以便它还能够对表达式进行语义分析，以获得正确的数学结果。 [下一步](03-Precedence.md)