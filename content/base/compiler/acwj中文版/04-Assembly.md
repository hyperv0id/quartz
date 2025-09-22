# 第 4 部分：实际的编译器

现在是我兑现实际编写编译器的承诺的时候了。因此，在这部分旅程中，我们将用生成 x86-64 汇编代码的代码替换程序中的解释器。

## 修改解释器

在我们这样做之前，有必要重新审视一下 中的解释器代码`interp.c`：

```
int interpretAST(struct ASTnode *n) {
  int leftval, rightval;

  if (n->left) leftval = interpretAST(n->left);
  if (n->right) rightval = interpretAST(n->right);

  switch (n->op) {
    case A_ADD:      return (leftval + rightval);
    case A_SUBTRACT: return (leftval - rightval);
    case A_MULTIPLY: return (leftval * rightval);
    case A_DIVIDE:   return (leftval / rightval);
    case A_INTLIT:   return (n->intvalue);

    default:
      fprintf(stderr, "Unknown AST operator %d\n", n->op);
      exit(1);
  }
}
```

该`interpretAST()`函数深度优先遍历给定的 AST 树。它评估任何左子树，然后评估右子树。最后，它使用`op`当前树底部的值来对这些子节点进行操作。

如果该`op`值是四个数学运算符之一，则执行此数学运算。如果该`op`值指示该节点只是一个整数文字，则返回该文字值。

该函数返回该树的最终值。并且，由于它是递归的，因此它将一次计算整个树的最终值一个子子树。

## 更改为汇编代码生成

我们将编写一个通用的汇编代码生成器。反过来，这将调用一组特定于 CPU 的代码生成函数。

这是通用汇编代码生成器`gen.c`：

```
// Given an AST, generate
// assembly code recursively
static int genAST(struct ASTnode *n) {
  int leftreg, rightreg;

  // Get the left and right sub-tree values
  if (n->left) leftreg = genAST(n->left);
  if (n->right) rightreg = genAST(n->right);

  switch (n->op) {
    case A_ADD:      return (cgadd(leftreg,rightreg));
    case A_SUBTRACT: return (cgsub(leftreg,rightreg));
    case A_MULTIPLY: return (cgmul(leftreg,rightreg));
    case A_DIVIDE:   return (cgdiv(leftreg,rightreg));
    case A_INTLIT:   return (cgload(n->intvalue));

    default:
      fprintf(stderr, "Unknown AST operator %d\n", n->op);
      exit(1);
  }
}
```

看起来很熟悉吧？！我们正在进行相同的深度优先树遍历。这次：

- A_INTLIT：用文字值加载寄存器
- 其他运算符：对保存左子级和右子级值的两个寄存器执行数学函数

中的代码不是传递值，而是`genAST()`传递寄存器标识符。例如，`cgload()`将一个值加载到寄存器中，并返回具有加载值的寄存器的标识。

`genAST()`它本身返回保存此时树的最终值的寄存器的标识。这就是顶部代码获取寄存器标识的原因：

```
  if (n->left) leftreg = genAST(n->left);
  if (n->right) rightreg = genAST(n->right);
```

## 呼唤`genAST()`

`genAST()`只会计算给定的表达式的值。我们需要打印出最终的计算结果。我们还需要用一些前导代码（ *前导*码）和一些尾随代码（*后导*码）来包装我们生成的汇编代码。这是通过以下中的其他函数完成的`gen.c`：

```
void generatecode(struct ASTnode *n) {
  int reg;

  cgpreamble();
  reg= genAST(n);
  cgprintint(reg);      // Print the register with the result as an int
  cgpostamble();
}
```

## x86-64 代码生成器

这是通用代码生成器。现在我们需要看看一些真正的汇编代码的生成。目前，我的目标是 x86-64 CPU，因为这仍然是最常见的 Linux 平台之一。那么，打开`cg.c`并开始浏览吧。

### 分配寄存器

任何CPU的寄存器数量都是有限的。我们必须分配一个寄存器来保存整数文字值，以及我们对它们执行的任何计算。然而，一旦我们使用了一个值，我们通常可以丢弃该值，从而释放保存它的寄存器。然后我们可以重新使用该寄存器来获取另一个值。

共有三个处理寄存器分配的函数：

- `freeall_registers()`：将所有寄存器设置为可用
- `alloc_register()`：分配一个空闲寄存器
- `free_register()`：释放分配的寄存器

我不会详细介绍代码，因为它很简单，但会进行一些错误检查。现在，如果我用完寄存器，程序就会崩溃。稍后，我将处理可用寄存器用完的情况。

该代码适用于通用寄存器：r0、r1、r2 和 r3。有一个包含实际寄存器名称的字符串表：

```
static char *reglist[4]= { "%r8", "%r9", "%r10", "%r11" };
```

这使得这些功能相当独立于 CPU 架构。

### 加载寄存器

这是通过以下方式完成的`cgload()`：分配寄存器，然后`movq` 指令将文字值加载到分配的寄存器中。

```
// Load an integer literal value into a register.
// Return the number of the register
int cgload(int value) {

  // Get a new register
  int r= alloc_register();

  // Print out the code to initialise it
  fprintf(Outfile, "\tmovq\t$%d, %s\n", value, reglist[r]);
  return(r);
}
```

### 添加两个寄存器

`cgadd()`获取两个寄存器号并生成将它们加在一起的代码。结果保存在两个寄存器之一中，然后释放另一个以供将来使用：

```
// Add two registers together and return
// the number of the register with the result
int cgadd(int r1, int r2) {
  fprintf(Outfile, "\taddq\t%s, %s\n", reglist[r1], reglist[r2]);
  free_register(r1);
  return(r2);
}
```

请注意，加法是*可交换的*，因此我可以添加`r2`to`r1` 而不是`r1`to `r2`。返回具有最终值的寄存器的标识。

### 两个寄存器相乘

这与加法非常相似，并且该操作是 *可交换的*，因此可以返回任何寄存器：

```
// Multiply two registers together and return
// the number of the register with the result
int cgmul(int r1, int r2) {
  fprintf(Outfile, "\timulq\t%s, %s\n", reglist[r1], reglist[r2]);
  free_register(r1);
  return(r2);
}
```

### 两个寄存器相减

减法*不可*交换：我们必须确保顺序正确。第二个寄存器从第一个寄存器中减去，因此我们返回第一个并释放第二个：

```
// Subtract the second register from the first and
// return the number of the register with the result
int cgsub(int r1, int r2) {
  fprintf(Outfile, "\tsubq\t%s, %s\n", reglist[r2], reglist[r1]);
  free_register(r2);
  return(r1);
}
```

### 两个寄存器相除

除法也不可交换，因此前面的注释适用。在 x86-64 上，情况更加复杂。我们需要加载来自*的*`%rax` 股息。这需要使用 扩展至八个字节。然后，将除以中的除数，将*商*保留在 中，因此我们需要将其复制到 或中。然后我们就可以释放另一个寄存器了。`r1``cqo``idivq``%rax``r2``%rax``r1``r2`

```
// Divide the first register by the second and
// return the number of the register with the result
int cgdiv(int r1, int r2) {
  fprintf(Outfile, "\tmovq\t%s,%%rax\n", reglist[r1]);
  fprintf(Outfile, "\tcqo\n");
  fprintf(Outfile, "\tidivq\t%s\n", reglist[r2]);
  fprintf(Outfile, "\tmovq\t%%rax,%s\n", reglist[r1]);
  free_register(r2);
  return(r1);
}
```

### 打印寄存器

没有 x86-64 指令可以将寄存器打印为十进制数。为了解决这个问题，汇编前导码包含一个名为的函数`printint()`，该函数接受寄存器参数并调用`printf()` 以十进制打印出来。

我不会给出 中的代码`cgpreamble()`，但它也包含 的开始代码`main()`，以便我们可以组装输出文件以获得完整的程序。的代码`cgpostamble()`，这里也没有给出，只是简单地调用`exit(0)`来结束程序。

然而，这里是`cgprintint()`：

```
void cgprintint(int r) {
  fprintf(Outfile, "\tmovq\t%s, %%rdi\n", reglist[r]);
  fprintf(Outfile, "\tcall\tprintint\n");
  free_register(r);
}
```

Linux x86-64 期望函数的第一个参数位于寄存器中`%rdi` ，因此我们将寄存器移至`%rdi`之前`call printint`。

## 进行我们的第一次编译

x86-64 代码生成器的情况就是这样。有一些额外的代码可以作为我们的输出文件`main()`打开`out.s`。我还将解释器留在了程序中，以便我们可以确认我们的程序集为输入表达式计算出与解释器相同的答案。

让我们创建编译器并运行它`input01`：

```
$ make
cc -o comp1 -g cg.c expr.c gen.c interp.c main.c scan.c tree.c

$ make test
./comp1 input01
15
cc -o out out.s
./out
15
```

是的！前 15 是解释器的输出。第二个 15 是程序集的输出。

## 检查汇编输出

那么，装配输出到底是什么？好吧，这是输入文件：

```
2 + 3 * 5 - 8 / 3
```

这是`out.s`带有注释的输入：

```
        .text                           # Preamble code
.LC0:
        .string "%d\n"                  # "%d\n" for printf()
printint:
        pushq   %rbp
        movq    %rsp, %rbp              # Set the frame pointer
        subq    $16, %rsp
        movl    %edi, -4(%rbp)
        movl    -4(%rbp), %eax          # Get the printint() argument
        movl    %eax, %esi
        leaq    .LC0(%rip), %rdi        # Get the pointer to "%d\n"
        movl    $0, %eax
        call    printf@PLT              # Call printf()
        nop
        leave                           # and return
        ret

        .globl  main
        .type   main, @function
main:
        pushq   %rbp
        movq    %rsp, %rbp              # Set the frame pointer
                                        # End of preamble code

        movq    $2, %r8                 # %r8 = 2
        movq    $3, %r9                 # %r9 = 3
        movq    $5, %r10                # %r10 = 5
        imulq   %r9, %r10               # %r10 = 3 * 5 = 15
        addq    %r8, %r10               # %r10 = 2 + 15 = 17
                                        # %r8 and %r9 are now free again
        movq    $8, %r8                 # %r8 = 8
        movq    $3, %r9                 # %r9 = 3
        movq    %r8,%rax
        cqo                             # Load dividend %rax with 8
        idivq   %r9                     # Divide by 3
        movq    %rax,%r8                # Store quotient in %r8, i.e. 2
        subq    %r8, %r10               # %r10 = 17 - 2 = 15
        movq    %r10, %rdi              # Copy 15 into %rdi in preparation
        call    printint                # to call printint()

        movl    $0, %eax                # Postamble: call exit(0)
        popq    %rbp
        ret
```

出色的！我们现在有了一个合法的编译器：一个程序，它接受一种语言的输入并生成另一种语言的输入翻译。

我们仍然必须将输出组装为机器代码并将其与支持库链接，但目前我们可以手动执行此操作。稍后，我们将编写一些代码来自动执行此操作。

## 结论和下一步是什么

从解释器更改为通用代码生成器很简单，但随后我们必须编写一些代码来生成真正的汇编输出。为此，我们必须考虑如何分配寄存器：目前，我们有一个简单的解决方案。我们还必须处理一些 x86-64 奇怪的问题，比如`idivq` 指令。

我还没有触及的是：为什么要费心为表达式生成 AST？当然，当我们在 Pratt 解析器中点击“+”标记时，我们可以调用`cgadd()`，其他运算符也是如此。我将让您思考这个问题，但我会在一两步后回过头来讨论这个问题。

在编译器编写之旅的下一部分中，我们将为我们的语言添加一些语句，以便它开始类似于正确的计算机语言。[下一步](05-Statements.md)
