# 第三部分：运算符优先级

在我们编写编译器的旅程的上一部分中，我们看到解析器并不一定强制执行语言的语义。它只强制执行语法和语法规则。

由于代码创建了一个看起来像这样的AST（抽象语法树）：

```plaintext
     *
    / \
   2   +
      / \
     3   *
        / \
       4   5
```

而不是：

```plaintext
          +
         / \
        /   \
       /     \
      *       *
     / \     / \
    2   3   4   5
```

我们得到了计算类似 `2 * 3 + 4 * 5` 表达式的错误值的代码。为了解决这个问题，我们必须在解析器中添加代码以执行运算符优先级。有至少两种方法可以做到这一点：

+ 在语言的语法中明确运算符优先级
+ 使用运算符优先级表影响现有解析器

## 使运算符优先级明确

这是我们在旅程的最后一部分中的语法：

```plaintext
expression: number
          | expression '*' expression
          | expression '/' expression
          | expression '+' expression
          | expression '-' expression
          ;

number:  T_INTLIT
         ;
```

请注意，四个数学运算符之间没有区分。让我们调整语法以进行区分：

```plaintext
expression: additive_expression
    ;

additive_expression:
      multiplicative_expression
    | additive_expression '+' multiplicative_expression
    | additive_expression '-' multiplicative_expression
    ;

multiplicative_expression:
      number
    | number '*' multiplicative_expression
    | number '/' multiplicative_expression
    ;

number:  T_INTLIT
         ;
```

现在我们有两种类型的表达式：*加法* 表达式和 *乘法* 表达式。请注意，语法现在强制要求数字只能是乘法表达式的一部分。这迫使 '*' 和 '/' 运算符紧密绑定到两侧的数字，从而具有更高的优先级。

任何加法表达式实际上都是一个乘法表达式本身，或者是一个加法（即乘法）表达式，后面跟着一个 '+' 或 '-' 运算符，然后是另一个乘法表达式。加法表达式现在比乘法表达式的优先级要低得多。

## 在递归下降解析器中实现上述内容

我们如何将上述版本的语法实现到我们的递归下降解析器中呢？我在文件 `expr2.c` 中完成了这个工作，并将在下面介绍代码。

答案是使用 `multiplicative_expr()` 函数处理 '*' 和 '/' 运算符，以及 `additive_expr()` 函数处理优先级较低的 '+' 和 '-' 运算符。

这两个函数将读取一些输入和一个运算符。然后，当存在相同优先级的后续运算符时，每个函数都将解析更多的输入，并使用第一个运算符将左半部分和右半部分结合起来。

但是，`additive_expr()` 必须推迟到优先级更高的 `multiplicative_expr()` 函数。以下是实现方法。

## `additive_expr()`

```c
// 返回根为'+'或'-'二元运算符的AST树
struct ASTnode *additive_expr(void) {
  struct ASTnode *left, *right;
  int tokentype;

  // 获取比我们更高优先级的左子树
  left = multiplicative_expr();

  // 如果没有令牌了，只返回左节点
  tokentype = Token.token;
  if (tokentype == T_EOF)
    return (left);

  // 在我们的优先级上工作的令牌循环
  while (1) {
    // 获取下一个整数文字
    scan(&Token);

    // 获取比我们更高优先级的右子树
    right = multiplicative_expr();

    // 使用我们的低优先级运算符连接两个子树
    left = mkastnode(arithop(tokentype), left, right, 0);

    // 并获取我们的优先级的下一个令牌
    tokentype = Token.token;
    if (tokentype == T_EOF)
      break;
  }

  // 返回我们创建的任何树
  return (left);
}
```

在开始时，我们立即调用 `multiplicative_expr()`，以防第一个运算符是高优先级的 '*' 或 '/'。该函数只在遇到低优先级的 '+' 或 '-' 运算符时才返回。

因此，当我们进入 `while` 循环时，我们知道我们有一个 '+' 或 '-' 运算符。我们循环直到输入中没有令牌，即当我们遇到 T_EOF 令牌时。

在循环内部，我们再次调用 `multiplicative_expr()`，以防未来的运算符比我们更高优先级。同样，这将在它们不再更高优先级时返回。

一旦我们有了左右子树，我们可以使用上一次循环时获得的运算符将它们结合起来。这将重复，因此如果我们有表达式 `2 + 4 + 6`，我们最终得到的AST树是：

```plaintext
       +
      / \
     +   6
    / \
   2   4
```

但是如果 `multiplicative_expr()` 有自己的更高优先级运算符，我们将结合具有多个节点的子树。
## multiplicative_expr()

```c
// 返回根为 '*' 或 '/' 二元运算符的AST树
struct ASTnode *multiplicative_expr(void) {
  struct ASTnode *left, *right;
  int tokentype;

  // 获取左侧的整数文字。
  // 同时获取下一个令牌。
  left = primary();

  // 如果没有令牌了，只返回左节点
  tokentype = Token.token;
  if (tokentype == T_EOF)
    return (left);

  // 当令牌是 '*' 或 '/' 时循环
  while ((tokentype == T_STAR) || (tokentype == T_SLASH)) {
    // 获取下一个整数文字
    scan(&Token);
    right = primary();

    // 使用左整数文字连接右整数文字
    left = mkastnode(arithop(tokentype), left, right, 0);

    // 更新当前令牌的详细信息。
    // 如果没有令牌了，只返回左节点
    tokentype = Token.token;
    if (tokentype == T_EOF)
      break;
  }

  // 返回我们创建的任何树
  return (left);
}
```

该代码与 `additive_expr()` 类似，只是我们可以调用 `primary()` 来获取真正的整数文字！我们仅在具有高优先级的运算符（即 '*' 和 '/' 运算符）时循环。一旦遇到低优先级运算符，我们只需返回到目前为止构建的子树。这返回到 `additive_expr()` 处理低优先级运算符。

## 以上方法的缺点

使用显式运算符优先级构建递归下降解析器的上述方法可能效率较低，因为需要所有的函数调用才能达到正确的优先级水平。还必须有函数来处理每个运算符优先级级别，因此我们最终得到很多代码行。

## 另一种选择：Pratt解析

减少代码量的一种方法是使用[Pratt解析器](https://en.wikipedia.org/wiki/Pratt_parser)，该解析器具有与每个令牌相关联的优先级值表，而不是具有在语法中复制显式优先级的函数。

此时，我强烈建议阅读Bob Nystrom的[Pratt Parsers: Expression Parsing Made Easy](https://journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/)。Pratt解析器仍然让我头疼，因此请尽量阅读尽可能多的内容，并熟悉基本概念。

## `expr.c`: Pratt解析

我在`expr.c`中实现了Pratt解析，这是`expr2.c`的替代品。让我们开始这次导览。

首先，我们需要一些代码来确定每个令牌的优先级级别：

```c
// 每个令牌的运算符优先级
static int OpPrec[] = { 0, 10, 10, 20, 20,    0 };
//                     EOF  +   -   *   /  INTLIT

// 检查我们是否有一个二元运算符并
// 返回其优先级。
static int op_precedence(int tokentype) {
  int prec = OpPrec[tokentype];
  if (prec == 0) {
    fprintf(stderr, "在第 %d 行，令牌 %d 处发生语法错误\n", Line, tokentype);
    exit(1);
  }
  return (prec);
}
```

较高的数字（例如20）意味着比较低的数字（例如10）具有更高的优先级。

现在，你可能会问：为什么有一个查找表叫做 `OpPrec[]`，但还要一个函数？答案是：为了发现语法错误。

考虑一个类似 `234 101 + 12` 的输入。我们可以扫描前两个令牌。但如果我们简单地使用 `OpPrec[]` 获取第二个 `101` 令牌的优先级，我们就不会注意到它不是运算符。因此，`op_precedence()` 函数强制执行正确的语法结构。

现在，不再有每个优先级级别一个函数，而是有一个使用运算符优先级表的单个表达式函数：

```c
// 返回根为二元运算符的AST树。
// 参数ptp是前一个令牌的优先级。
struct ASTnode *binexpr(int ptp) {
  struct ASTnode *left, *right;
  int tokentype;

  // 获取左侧的整数文字。
  // 同时获取下一个令牌。
  left = primary();

  // 如果没有令牌了，只返回左节点
  tokentype = Token.token;
  if (tokentype == T_EOF)
    return (left);

  // 当此令牌的优先级大于前一个令牌的优先级时
  while (op_precedence(tokentype) > ptp) {
    // 获取下一个整数文字
    scan(&Token);

    // 以我们令牌的优先级递归调用binexpr()构建子树
    right = binexpr(OpPrec[tokentype]);

    // 将该子树与我们的子树连接起来。同时将令牌
    // 转换为AST操作。
    left = mkastnode(arithop(tokentype), left, right, 0);

    // 更新当前令牌的详细信息。
    // 如果没有令牌了，只返回左节点
    tokentype = Token.token;
    if (tokentype == T_EOF)
      return (left);
  }

  // 在优先级相同或更低时返回树
  return (left);
}
```

首先，请注意这仍然像之前的解析器函数一样是递归的。这一次，我们在调用之前得到的令牌的优先级级别。`main()` 将以最低的优先级0调用我们，但我们将以更高的值调用自己。

你还应该注意到，该代码与 `multiplicative_expr()` 函数非常相似：读入整数文字，获取运算符的令牌类型，然后循环构建树。

不同之处在于循环条件和主体：

```c
multiplicative_expr():
  while ((tokentype == T_STAR) || (tokentype == T_SLASH)) {
    scan(&Token); right = primary();

    left = mkastnode(arithop(tokentype), left, right, 0);

    tokentype = Token.token;
    if (tokentype == T_EOF) return (left);
  }

binexpr():
  while (op_precedence(tokentype) > ptp) {
    scan(&Token); right = binexpr(OpPrec[tokentype]);

    left = mkastnode(arithop(tokentype), left, right, 0);

    tokentype = Token.token;
    if (tokentype == T_EOF) return (left);
  }
```

使用Pratt解析器，当下一个运算符的优先级高于我们当前令牌时，我们不仅仅使用 `primary()` 获取下一个整数文字，而是使用 `binexpr(OpPrec[tokentype])` 调用自己来提高运算符优先级。

一旦我们遇到与我们的优先级级别相同或更低的令牌，我们将简单地：

```c
  return (left);
```

这可能是一个具有许多节点和在调用我们的运算符的优先级更高的运算符的子树，或者它可能是一个具有与我们相同优先级的运算符的单个整数文字。

现在我们有一个单一的函数来进行表达式解析。它使用一个小的辅助函数来强制执行运算符优先级，从而实现我们语言的语义。

## 让两个解析器同时运行

你可以制作两个程序，一个使用每个解析器：

```
$ make parser                                        # Pratt解析器
cc -o parser -g expr.c interp.c main.c scan.c tree.c

$ make parser2                                       # 优先级上升
cc -o parser2 -g expr2.c interp.c main.c scan.c tree.c
```

你还可以使用我们旅程的上一部分相同的输入文件测试两个解析器：

```
$ make test
(./parser input01; \
 ./parser input02; \
 ./parser input03; \
 ./parser input04; \
 ./parser input05)
15                                       # input01 结果
29                                       # input02 结果
syntax error on line 1, token 5          # input03 结果
Unrecognised character . on line

 3       # input04 结果
Unrecognised character a on line 1       # input05 结果

$ make test2
(./parser2 input01; \
 ./parser2 input02; \
 ./parser2 input03; \
 ./parser2 input04; \
 ./parser2 input05)
15                                       # input01 结果
29                                       # input02 结果
syntax error on line 1, token 5          # input03 结果
Unrecognised character . on line 3       # input04 结果
Unrecognised character a on line 1       # input05 结果

```

## 结论和接下来的步骤

现在是时候稍微退后一步，看看我们到达了哪里。我们现在有：

 + 一个识别和返回我们语言中令牌的扫描器
 + 一个识别我们语法、报告语法错误并构建抽象语法树的解析器
 + 解析器的优先级表，实现我们语言的语义
 + 一个解释器，深度优先遍历抽象语法树并计算输入表达式的结果

我们尚未拥有的是一个编译器。但我们离制作我们的第一个编译器非常近了！

在我们编写编译器的旅程的下一部分中，我们将替换解释器。我们将编写一个翻译器，为每个具有数学运算符的AST节点生成x86-64汇编代码。我们还将生成一些汇编序言和序尾来支持生成器输出的汇编代码。 [下一步](04-Assembly.md)