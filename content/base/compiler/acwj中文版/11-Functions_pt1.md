# 第11部分：函数，第1部分

我想开始在我们的语言中实现函数的工作，但我知道这将涉及很多步骤。沿途我们将不得不处理的一些事情包括：

 + 数据类型：`char`、`int`、`long` 等。
 + 每个函数的返回类型
 + 每个函数的参数数量
 + 函数内部局部变量与全局变量

在我们的编程旅程的这一部分完成这些内容显然是太多了。所以我现在要做的是能够*声明*不同的函数。只有我们最终可执行文件中的 `main()` 函数会运行，但我们将能够为多个函数生成代码。

希望不久，我们编译器所识别的语言将成为C语言的足够子集，以至于我们的输入能被“真正”的C编译器识别。但就目前而言还做不到。

## 简单的函数语法

这肯定是一个临时的解决方案，让我们可以解析像函数一样的东西。一旦完成这个，我们就可以添加其他重要的东西：类型、返回类型、参数等。

所以，现在，我将添加一个看起来像这样的函数语法BNF：

```
 function_declaration: 'void' identifier '(' ')' compound_statement   ;
```

所有函数都将被声明为 `void` 并且没有参数。我们也不会引入调用函数的能力，因此只有 `main()` 函数会执行。

我们需要一个新的关键字 `void` 和一个新的标记 T_VOID，这两个都很容易添加。

## 解析简单的函数语法

新的函数语法非常简单，我们可以编写一个简洁的函数来解析它（在 `decl.c` 中）：

```c
// 解析简单函数的声明
struct ASTnode *function_declaration(void) {
  struct ASTnode *tree;
  int nameslot;

  // 找到 'void'，标识符，以及 '(' ')'。
  // 现在先不对它们做任何处理
  match(T_VOID, "void");
  ident();
  nameslot = addglob(Text);
  lparen();
  rparen();

  // 获取复合语句的AST树
  tree = compound_statement();

  // 返回一个 A_FUNCTION 节点，它具有函数的nameslot
  // 和复合语句子树
  return(mkastunary(A_FUNCTION, tree, nameslot));
}
```

这将进行语法检查和AST构建，但几乎没有语义错误检查。如果一个函数被重新声明了怎么办？好吧，我们还没有注意到这一点。

## 对 `main()` 的修改

有了上述函数，我们现在可以重写 `main()` 中的一些代码，以便一个接一个地解析多个函数：

```c
  scan(&Token);                 // 从输入中获取第一个令牌
  genpreamble();                // 输出序言
  while (1) {                   // 解析一个函数并且
    tree = function_declaration();
    genAST(tree, NOREG, 0);     // 为它生成汇编代码
    if (Token.token == T_EOF)   // 当我们到达EOF时停止
      break;
  }
```

注意我已经移除了 `genpostamble()` 函数调用。那是因为它的输出在技术上是针对 `main()` 生成的汇编的尾声。我们现在需要一些代码生成函数来生成一个函数的开始和结束。

## 针对函数的通用代码生成

现在我们有了一个 A_FUNCTION AST 节点，我们最好在通用代码生成器 `gen.c` 中添加一些代码来处理它。如上所述，这是一个具有单个子节点的*一元* AST 节点：

```c
  // 返回一个具有函数 nameslot 的 A_FUNCTION 节点
  // 以及复合语句子树
  return(mkastunary(A_FUNCTION, tree, nameslot));
```

子节点包含一个保存函数体的复合语句的子树。我们需要在生成复合语句的代码*之前*生成函数的开始部分。因此，这是 `genAST()` 中用于此目的的代码：

```c
    case A_FUNCTION:
      // 在生成代码之前生成函数的序言
      cgfuncpreamble(Gsym[n->v.id].name);
      genAST(n->left, NOREG, n->op);
      cgfuncpostamble();
      return (NOREG);
```

## x86-64 代码生成

现在我们到了必须为每个函数生成设置堆栈和帧指针的代码的地步，同时在函数结束时撤销这些设置并返回给函数的调用者。

我们已经在 `cgpreamble()` 和 `cgpostamble()` 中有这些代码，但 `cgpreamble()` 还包含了 `printint()` 函数的汇编代码。因此，这就是将这些汇编代码片段分离出来到 `cg.c` 中新函数的过程：

```c
// 输出汇编语言的序言
void cgpreamble() {
  freeall_registers();
  // 只输出 printint() 函数的代码
}

// 输出函数的序言
void cgfuncpreamble(char *name) {
  fprintf(Outfile,
          "\t.text\n"
          "\t.globl\t%s\n"
          "\t.type\t%s, @function\n"
          "%s:\n" "\tpushq\t%%rbp\n"
          "\tmovq\t%%rsp, %%rbp\n", name, name, name);
}

// 输出函数的尾声
void cgfuncpostamble() {
  fputs("\tmovl $0, %eax\n" "\tpopq     %rbp\n" "\tret\n", Outfile);
}
```

通过这些步骤，编译器现在能够处理简单的函数声明，并为它们生成相应的x86-64汇编代码。这是在构建完整编译器的过程中的一个重要步骤，接下来的步骤将包括处理函数参数、返回类型以及本地和全局变量之间的交互。

## 测试函数生成功能

我们有了一个新的测试程序 `tests/input08`，它开始看起来像一个C程序（除了 `print` 语句）：

```c
void main()
{
  int i;
  for (i = 1; i <= 10; i = i + 1) {
    print i;
  }
}
```

为了测试这个，执行 `make test8`，其过程如下：

```
cc -o comp1 -g cg.c decl.c expr.c gen.c main.c misc.c scan.c
    stmt.c sym.c tree.c
./comp1 tests/input08
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

我不打算查看汇编输出，因为它与上一部分中FOR循环测试生成的代码相同。

然而，我已经在所有之前的测试输入文件中添加了 `void main()`，因为我们的语言在复合语句代码之前要求有一个函数声明。

测试程序 `tests/input09` 中声明了两个函数。编译器愉快地为每个函数生成了工作中的汇编代码，但目前我们无法运行第二个函数的代码。

## 结论和下一步

我们已经在向我们的语言中添加函数方面取得了良好的开端。目前，这只是一个相当简单的函数声明。

在我们编写编译器的旅程的下一部分，我们将开始向我们的编译器添加类型的过程。[下一步](12-Types_pt1.md)