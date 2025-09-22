#  第 7 部分：比较运算符

接下来我打算添加 IF 语句，但后来我意识到我最好先添加一些比较运算符。事实证明这非常简单，因为它们与现有的运算符一样都是二元运算符。

那么让我们快速看看添加六个比较运算符有什么变化：`==`、`!=`、`<`、`>`和`<=`。`>=`

## 添加新令牌

我们将有六个新令牌，所以让我们将它们添加到`defs.h`：

```c
// Token types
enum {
  T_EOF,
  T_PLUS, T_MINUS,
  T_STAR, T_SLASH,
  T_EQ, T_NE,
  T_LT, T_GT, T_LE, T_GE,
  T_INTLIT, T_SEMI, T_ASSIGN, T_IDENT,
  // Keywords
  T_PRINT, T_INT
};
```

我重新排列了标记，以便具有优先级的标记按照从低到高的优先级顺序出现在没有任何优先级的标记之前。

## 扫描令牌

现在我们必须扫描它们。请注意，我们必须区分 `=`and `==`、`<`and `<=`、`>`and `>=`。因此，我们需要从输入中读取额外的字符，如果不需要，则将其放回原处。`scan()`这是from中的新代码`scan.c`：

```c
  case '=':
    if ((c = next()) == '=') {
      t->token = T_EQ;
    } else {
      putback(c);
      t->token = T_ASSIGN;
    }
    break;
  case '!':
    if ((c = next()) == '=') {
      t->token = T_NE;
    } else {
      fatalc("Unrecognised character", c);
    }
    break;
  case '<':
    if ((c = next()) == '=') {
      t->token = T_LE;
    } else {
      putback(c);
      t->token = T_LT;
    }
    break;
  case '>':
    if ((c = next()) == '=') {
      t->token = T_GE;
    } else {
      putback(c);
      t->token = T_GT;
    }
    break;
```

我还将令牌的名称更改`=`为 T_ASSIGN，以确保我不会混淆它和新的 T_EQ 令牌。

## 新的表达代码

我们现在可以扫描六个新令牌。因此，现在我们必须在它们出现在表达式中时解析它们，并强制执行它们的运算符优先级。

现在你应该已经明白了：

- 我正在构建一个自编译编译器
- 在C语言中
- 使用 SubC 编译器作为参考。

这意味着我正在为足够的 C 子集（就像 SubC）编写一个编译器，以便它能够自行编译。因此，我应该使用正常的 [C 运算符优先顺序](https://en.cppreference.com/w/c/language/operator_precedence)。这意味着比较运算符的优先级低于乘法和除法。

我还意识到，我用来将令牌映射到 AST 节点类型的 switch 语句只会变得更大。因此，我决定重新排列 AST 节点类型，以便所有二元运算符（在 中）的它们之间存在 1:1 映射`defs.h`：

```c
// AST node types. The first few line up
// with the related tokens
enum {
  A_ADD=1, A_SUBTRACT, A_MULTIPLY, A_DIVIDE,
  A_EQ, A_NE, A_LT, A_GT, A_LE, A_GE,
  A_INTLIT,
  A_IDENT, A_LVIDENT, A_ASSIGN
};
```

现在`expr.c`，我可以简化令牌到 AST 节点的转换，并添加新令牌的优先级：

```c
// Convert a binary operator token into an AST operation.
// We rely on a 1:1 mapping from token to AST operation
static int arithop(int tokentype) {
  if (tokentype > T_EOF && tokentype < T_INTLIT)
    return(tokentype);
  fatald("Syntax error, token", tokentype);
}

// Operator precedence for each token. Must
// match up with the order of tokens in defs.h
static int OpPrec[] = {
  0, 10, 10,                    // T_EOF, T_PLUS, T_MINUS
  20, 20,                       // T_STAR, T_SLASH
  30, 30,                       // T_EQ, T_NE
  40, 40, 40, 40                // T_LT, T_GT, T_LE, T_GE
};
```

这就是解析和运算符优先级！

## 代码生成

由于这六个新运算符是二元运算符，因此很容易修改通用代码生成器来`gen.c`处理它们：

```c
  case A_EQ:
    return (cgequal(leftreg, rightreg));
  case A_NE:
    return (cgnotequal(leftreg, rightreg));
  case A_LT:
    return (cglessthan(leftreg, rightreg));
  case A_GT:
    return (cggreaterthan(leftreg, rightreg));
  case A_LE:
    return (cglessequal(leftreg, rightreg));
  case A_GE:
    return (cggreaterequal(leftreg, rightreg));
```

## x86-64 代码生成

现在有点棘手了。在 C 语言中，比较运算符返回一个值。如果它们评估为真，则结果为 1。如果评估为假，则结果为 0。我们需要编写 x86-64 汇编代码来反映这一点。

幸运的是，有一些 x86-64 指令可以执行此操作。不幸的是，在此过程中还有一些问题需要处理。考虑这个 x86-64 指令：

```
    cmpq %r8,%r9
```

上述`cmpq`指令执行 %r9 - %r8 并设置多个状态标志，包括负标志和零标志。因此，我们可以查看标志组合来查看比较结果：

| 比较       | 手术      | 如果为真则标记 |
| ---------- | --------- | -------------- |
| %r8==%r9   | %r9 - %r8 | 零             |
| %r8!=%r9   | %r9 - %r8 | 不为零         |
| %r8 > %r9  | %r9 - %r8 | 不为零，负数   |
| %r8 < %r9  | %r9 - %r8 | 不为零，不负   |
| %r8 >= %r9 | %r9 - %r8 | 零或负数       |
| %r8 <= %r9 | %r9 - %r8 | 零或非负数     |

有 6 个 x86-64 指令，它们根据两个标志值将寄存器设置为 1 或 0：`sete`、`setne`、`setg`、`setl`、`setge` 和`setle`按照上表行的顺序。

问题是，这些指令仅设置寄存器的最低字节。如果寄存器已在最低字节之外设置了位，则它们将保持设置状态。因此，我们可以将变量设置为 1，但如果它的值已经是 1000（十进制），那么它现在将是 1001，这不是我们想要的。

解决办法是`andq`在指令后的`setX`寄存器中去掉不需要的位。有`cg.c`一个通用的比较函数可以做到这一点：

```c
// Compare two registers.
static int cgcompare(int r1, int r2, char *how) {
  fprintf(Outfile, "\tcmpq\t%s, %s\n", reglist[r2], reglist[r1]);
  fprintf(Outfile, "\t%s\t%s\n", how, breglist[r2]);
  fprintf(Outfile, "\tandq\t$255,%s\n", reglist[r2]);
  free_register(r1);
  return (r2);
}
```

`how`其中一项说明在哪里`setX`。请注意，我们执行

```
   cmpq reglist[r2], reglist[r1]
```

因为这实际上`reglist[r1] - reglist[r2]`就是我们真正想要的。

## x86-64 寄存器

这里我们需要稍微岔开一下话题来讨论 x86-64 架构中的寄存器。x86-64 有几个 64 位通用寄存器，但我们也可以使用不同的寄存器名称来访问和处理这些寄存器的子部分。

![img](../../../../../../Pictures/typora/07-Comparisons/N0KnG.png)

上图来自*[stack.imgur.com](http://stack.imgur.com)*，显示对于64位*r8*寄存器，我们可以使用“ *r8d* ”寄存器访问该寄存器的低32位。同样，“ *r8w* ”寄存器是r8寄存器的低16位，“ *r8b ”寄存器是**r8*寄存器的低8位。

在该`cgcompare()`函数中，代码使用`reglist[]`数组来比较两个 64 位寄存器，然后使用数组中的名称在第二个寄存器的 8 位版本中设置标志`breglist[]`。x86-64架构只允许`setX`指令对8位寄存器名称进行操作，因此需要数组`breglist[]`。

## 创建多个比较指令

现在我们有了这个通用函数，我们可以编写六个实际的比较函数：

```c
int cgequal(int r1, int r2) { return(cgcompare(r1, r2, "sete")); }
int cgnotequal(int r1, int r2) { return(cgcompare(r1, r2, "setne")); }
int cglessthan(int r1, int r2) { return(cgcompare(r1, r2, "setl")); }
int cggreaterthan(int r1, int r2) { return(cgcompare(r1, r2, "setg")); }
int cglessequal(int r1, int r2) { return(cgcompare(r1, r2, "setle")); }
int cggreaterequal(int r1, int r2) { return(cgcompare(r1, r2, "setge")); }
```

与其他二元运算符函数一样，一个寄存器被释放，另一个寄存器返回结果。

# 付诸行动

看一下`input04`输入文件：

```c
int x;
x= 7 < 9;  print x;
x= 7 <= 9; print x;
x= 7 != 9; print x;
x= 7 == 7; print x;
x= 7 >= 7; print x;
x= 7 <= 7; print x;
x= 9 > 7;  print x;
x= 9 >= 7; print x;
x= 9 != 7; print x;
```

所有这些比较都是正确的，所以我们应该打印出 9 个 1。做一个`make test`来确认这一点。

我们看一下第一次比较输出的汇编代码：

```
        movq    $7, %r8
        movq    $9, %r9
        cmpq    %r9, %r8        # Perform %r8 - %r9, i.e. 7 - 9
        setl    %r9b            # Set %r9b to 1 if 7 is less than 9
        andq    $255,%r9        # Remove all other bits in %r9
        movq    %r9, x(%rip)    # Save the result in x
        movq    x(%rip), %r8
        movq    %r8, %rdi
        call    printint        # Print x out
```

是的，上面有一些低效的汇编代码。我们甚至还没有开始担心优化代码。引用唐纳德·高德纳 (Donald Knuth) 的话：

> **过早的优化是编程中万恶（或至少是大部分）的根源。**

## 结论和下一步是什么

这是对编译器的一个很好且简单的补充。旅程的下一部分将会更加复杂。

在编译器编写之旅的下一部分中，我们将向编译器添加 IF 语句并使用刚刚添加的比较运算符。[下一步](08-If_Statements.md)
