#  第 8 部分：If 语句

现在我们可以比较值了，是时候将 IF 语句添加到我们的语言中了。因此，首先让我们看一下 IF 语句的一般语法以及它们如何转换为汇编语言。

## IF 语法

IF 语句的语法为：

```
  if (condition is true) 
    perform this first block of code
  else
    perform this other block of code
```

现在，通常如何将其转换为汇编语言？事实证明，如果相反的比较为真，我们会进行相反的比较并跳转/分支：

```
       perform the opposite comparison
       jump to L1 if true
       perform the first block of code
       jump to L2
L1:
       perform the other block of code
L2:
   
```

其中`L1`和`L2`是汇编语言标签。

## 在我们的编译器中生成程序集

现在，我们输出代码来根据比较设置寄存器，例如

```
   int x; x= 7 < 9;         From input04
```

变成

```
        movq    $7, %r8
        movq    $9, %r9
        cmpq    %r9, %r8
        setl    %r9b        Set if less than 
        andq    $255,%r9
```

但对于 IF 语句，我们需要进行相反的比较：

```
   if (7 < 9) 
```

应该变成：

```
        movq    $7, %r8
        movq    $9, %r9
        cmpq    %r9, %r8
        jge     L1         Jump if greater then or equal to
        ....
L1:
```

因此，我在这部分旅程中实现了 IF 语句。由于这是一个工作项目，我确实必须撤消一些事情并重构它们作为旅程的一部分。我将尽力介绍沿途的更改和添加。

## 新代币和悬空的其他代币

我们的语言将需要一堆新的标记。我（现在）也想避免[悬空的 else 问题](https://en.wikipedia.org/wiki/Dangling_else)。为此，我更改了语法，以便所有语句组都用“{”...“}”大括号括起来；我将这样的分组称为“复合语句”。我们还需要“(”...“)”括号来保存 IF 表达式，以及关键字“if”和“else”。因此，新的代币是（在 中`defs.h`）：

```c
  T_LBRACE, T_RBRACE, T_LPAREN, T_RPAREN,
  // Keywords
  ..., T_IF, T_ELSE
```

## 扫描令牌

单字符标记应该是显而易见的，我不会给出扫描它们的代码。关键字也应该非常明显，但我将给出来自 `keyword()`以下的扫描代码`scan.c`：

```c
  switch (*s) {
    case 'e':
      if (!strcmp(s, "else"))
        return (T_ELSE);
      break;
    case 'i':
      if (!strcmp(s, "if"))
        return (T_IF);
      if (!strcmp(s, "int"))
        return (T_INT);
      break;
    case 'p':
      if (!strcmp(s, "print"))
        return (T_PRINT);
      break;
  }
```

## 新 BNF 语法

我们的语法开始变得庞大，所以我稍微重写了它：

```
 compound_statement: '{' '}'          // empty, i.e. no statement
      |      '{' statement '}'
      |      '{' statement statements '}'
      ;

 statement: print_statement
      |     declaration
      |     assignment_statement
      |     if_statement
      ;

 print_statement: 'print' expression ';'  ;

 declaration: 'int' identifier ';'  ;

 assignment_statement: identifier '=' expression ';'   ;

 if_statement: if_head
      |        if_head 'else' compound_statement
      ;

 if_head: 'if' '(' true_false_expression ')' compound_statement  ;

 identifier: T_IDENT ;
```

我省略了 的定义`true_false_expression`，但在某个时候，当我们添加更多运算符时，我会将其添加进去。

请注意 IF 语句的语法：它要么是一个`if_head`（没有“else”子句），要么是一个`if_head`后跟“else”和一个`compound_statement`。

我已经将所有不同的语句类型分开，以拥有自己的非终端名称。另外，之前的`statements`非终结符现在`compound_statement` 是非终结符，这需要在语句周围使用 '{' ... '}'。

这意味着`compound_statement`头部的 被 '{' ... '}' 包围，`compound_statement`'else' 关键字之后的任何内容也是如此。因此，如果我们有嵌套的 IF 语句，它们必须如下所示：

```
  if (condition1 is true) {
    if (condition2 is true) {
      statements;
    } else {
      statements;
    }
  } else {
    statements;
  }
```

并且每个“其他”属于哪个“如果”是没有歧义的。这解决了悬空的 else 问题。稍后，我会将 '{' ... '}' 设为可选。

## 解析复合语句

旧`void statements()`函数现在`compound_statement()`如下所示：

```c
// Parse a compound statement
// and return its AST
struct ASTnode *compound_statement(void) {
  struct ASTnode *left = NULL;
  struct ASTnode *tree;

  // Require a left curly bracket
  lbrace();

  while (1) {
    switch (Token.token) {
      case T_PRINT:
        tree = print_statement();
        break;
      case T_INT:
        var_declaration();
        tree = NULL;            // No AST generated here
        break;
      case T_IDENT:
        tree = assignment_statement();
        break;
      case T_IF:
        tree = if_statement();
        break;
    case T_RBRACE:
        // When we hit a right curly bracket,
        // skip past it and return the AST
        rbrace();
        return (left);
      default:
        fatald("Syntax error, token", Token.token);
    }

    // For each new tree, either save it in left
    // if left is empty, or glue the left and the
    // new tree together
    if (tree) {
      if (left == NULL)
        left = tree;
      else
        left = mkastnode(A_GLUE, left, NULL, tree, 0);
    }
  }
```

首先，请注意，代码强制解析器将复合语句开头的“{”与 相匹配`lbrace()`，并且只有当我们将结尾的“}”与 相匹配时才能退出`rbrace()`。

其次，请注意`print_statement()`和`assignment_statement()`都 `if_statement()`返回一个 AST 树，就像 一样`compound_statement()`。在我们的旧代码中，`print_statement()`调用自身`genAST()`来计算表达式，然后调用`genprintint()`. 同样， `assignment_statement()` 也叫做`genAST()`作业。

嗯，这意味着我们这里有 AST 树，那里有其他树。`genAST()`仅生成一棵 AST 树并调用一次为其生成汇编代码是有意义的。

这不是强制性的。例如，SubC 仅为表达式生成 AST。对于语言的结构部分（例如语句），SubC 对代码生成器进行特定调用，就像我在以前版本的编译器中所做的那样。

我现在决定使用解析器为整个输入生成一个 AST 树。一旦输入被解析，就可以从一棵 AST 树生成汇编输出。

稍后，我可能会为每个函数生成一个 AST 树。之后。

## 解析 IF 语法

因为我们是递归下降解析器，所以解析 IF 语句还不错：

```c
// Parse an IF statement including
// any optional ELSE clause
// and return its AST
struct ASTnode *if_statement(void) {
  struct ASTnode *condAST, *trueAST, *falseAST = NULL;

  // Ensure we have 'if' '('
  match(T_IF, "if");
  lparen();

  // Parse the following expression
  // and the ')' following. Ensure
  // the tree's operation is a comparison.
  condAST = binexpr(0);

  if (condAST->op < A_EQ || condAST->op > A_GE)
    fatal("Bad comparison operator");
  rparen();

  // Get the AST for the compound statement
  trueAST = compound_statement();

  // If we have an 'else', skip it
  // and get the AST for the compound statement
  if (Token.token == T_ELSE) {
    scan(&Token);
    falseAST = compound_statement();
  }
  // Build and return the AST for this statement
  return (mkastnode(A_IF, condAST, trueAST, falseAST, 0));
}
```

现在，我不想处理像 那样的输入`if (x-2)`，所以我限制了二进制表达式 from 的`binexpr()`根，它是六个比较运算符 A_EQ、A_NE、A_LT、A_GT、A_LE 或 A_GE 之一。

## 第三个孩子

我差点儿在没有正确解释的情况下就从你身边走私了一些东西。在最后一行，`if_statement()`我使用以下命令构建了 AST 节点：

```c
   mkastnode(A_IF, condAST, trueAST, falseAST, 0);
```

那是*三个*AST 子树！这里发生了什么？正如您所看到的，IF 语句将有三个子语句：

- 评估条件的子树
- 紧随其后的复合语句
- “else”关键字后面的可选复合语句

所以我们现在需要一个具有三个子节点的 AST 节点结构（在 中`defs.h`）：

```c
// AST node types.
enum {
  ...
  A_GLUE, A_IF
};

// Abstract Syntax Tree structure
struct ASTnode {
  int op;                       // "Operation" to be performed on this tree
  struct ASTnode *left;         // Left, middle and right child trees
  struct ASTnode *mid;
  struct ASTnode *right;
  union {
    int intvalue;               // For A_INTLIT, the integer value
    int id;                     // For A_IDENT, the symbol slot number
  } v;
};
```

因此，A_IF 树如下所示：

```
                      IF
                    / |  \
                   /  |   \
                  /   |    \
                 /    |     \
                /     |      \
               /      |       \
      condition   statements   statements
```

## 粘合 AST 节点

还有一个新的 A_GLUE AST 节点类型。这是做什么用的？我们现在构建了一个包含大量语句的 AST 树，因此我们需要一种方法将它们粘合在一起。

查看循环结束代码`compound_statement()`：

```c
      if (left != NULL)
        left = mkastnode(A_GLUE, left, NULL, tree, 0);
```

每次我们获得一个新的子树时，我们都会将其粘合到现有的树上。因此，对于这一系列语句：

```
    stmt1;
    stmt2;
    stmt3;
    stmt4;
```

我们最终得到：

```
             A_GLUE
              /  \
          A_GLUE stmt4
            /  \
        A_GLUE stmt3
          /  \
      stmt1  stmt2
```

而且，当我们从左到右深度优先遍历树时，这仍然会按正确的顺序生成汇编代码。

## 通用代码生成器

现在我们的 AST 节点有多个子节点，我们的通用代码生成器将变得更加复杂。另外，对于比较运算符，我们需要知道我们是否将比较作为 IF 语句的一部分（跳转到相反的比较）或普通表达式（在普通比较中将寄存器设置为 1 或 0）。

为此，我进行了修改，`getAST()`以便我们可以传入父 AST 节点操作：

```c
// Given an AST, the register (if any) that holds
// the previous rvalue, and the AST op of the parent,
// generate assembly code recursively.
// Return the register id with the tree's final value
int genAST(struct ASTnode *n, int reg, int parentASTop) {
   ...
}
```

### 处理特定 AST 节点

现在的代码`genAST()`必须处理特定的 AST 节点：

```c
  // We now have specific AST node handling at the top
  switch (n->op) {
    case A_IF:
      return (genIFAST(n));
    case A_GLUE:
      // Do each child statement, and free the
      // registers after each child
      genAST(n->left, NOREG, n->op);
      genfreeregs();
      genAST(n->right, NOREG, n->op);
      genfreeregs();
      return (NOREG);
  }
```

如果我们不返回，我们将继续执行正常的二元运算符 AST 节点，但有一个例外：比较节点：

```c
    case A_EQ:
    case A_NE:
    case A_LT:
    case A_GT:
    case A_LE:
    case A_GE:
      // If the parent AST node is an A_IF, generate a compare
      // followed by a jump. Otherwise, compare registers and
      // set one to 1 or 0 based on the comparison.
      if (parentASTop == A_IF)
        return (cgcompare_and_jump(n->op, leftreg, rightreg, reg));
      else
        return (cgcompare_and_set(n->op, leftreg, rightreg));
```

我将介绍新功能`cgcompare_and_jump()`及 `cgcompare_and_set()`以下内容。

### 生成 IF 汇编代码

我们使用特定函数处理 A_IF AST 节点，以及生成新标签编号的函数：

```c
// Generate and return a new label number
static int label(void) {
  static int id = 1;
  return (id++);
}

// Generate the code for an IF statement
// and an optional ELSE clause
static int genIFAST(struct ASTnode *n) {
  int Lfalse, Lend;

  // Generate two labels: one for the
  // false compound statement, and one
  // for the end of the overall IF statement.
  // When there is no ELSE clause, Lfalse _is_
  // the ending label!
  Lfalse = label();
  if (n->right)
    Lend = label();

  // Generate the condition code followed
  // by a zero jump to the false label.
  // We cheat by sending the Lfalse label as a register.
  genAST(n->left, Lfalse, n->op);
  genfreeregs();

  // Generate the true compound statement
  genAST(n->mid, NOREG, n->op);
  genfreeregs();

  // If there is an optional ELSE clause,
  // generate the jump to skip to the end
  if (n->right)
    cgjump(Lend);

  // Now the false label
  cglabel(Lfalse);

  // Optional ELSE clause: generate the
  // false compound statement and the
  // end label
  if (n->right) {
    genAST(n->right, NOREG, n->op);
    genfreeregs();
    cglabel(Lend);
  }

  return (NOREG);
}
```

实际上，代码正在执行以下操作：

```c
  genAST(n->left, Lfalse, n->op);       // Condition and jump to Lfalse
  genAST(n->mid, NOREG, n->op);         // Statements after 'if'
  cgjump(Lend);                         // Jump to Lend
  cglabel(Lfalse);                      // Lfalse: label
  genAST(n->right, NOREG, n->op);       // Statements after 'else'
  cglabel(Lend);                        // Lend: label
```

## x86-64 代码生成函数

现在我们有了一些新的 x86-64 代码生成函数。其中一些取代了`cgXXX()`我们在旅程的最后部分创建的六个比较函数。

对于普通的比较函数，我们现在传入 AST 操作来选择相关的 x86-64`set`指令：

```c
// List of comparison instructions,
// in AST order: A_EQ, A_NE, A_LT, A_GT, A_LE, A_GE
static char *cmplist[] =
  { "sete", "setne", "setl", "setg", "setle", "setge" };

// Compare two registers and set if true.
int cgcompare_and_set(int ASTop, int r1, int r2) {

  // Check the range of the AST operation
  if (ASTop < A_EQ || ASTop > A_GE)
    fatal("Bad ASTop in cgcompare_and_set()");

  fprintf(Outfile, "\tcmpq\t%s, %s\n", reglist[r2], reglist[r1]);
  fprintf(Outfile, "\t%s\t%s\n", cmplist[ASTop - A_EQ], breglist[r2]);
  fprintf(Outfile, "\tmovzbq\t%s, %s\n", breglist[r2], reglist[r2]);
  free_register(r1);
  return (r2);
}
```

我还发现了一条 x86-64 指令`movzbq`，该指令从一个寄存器中移动最低字节并将其扩展以适合 64 位寄存器。我现在使用它而不是`and $255`旧代码中的。

我们需要一个函数来生成标签并跳转到它：

```c
// Generate a label
void cglabel(int l) {
  fprintf(Outfile, "L%d:\n", l);
}

// Generate a jump to a label
void cgjump(int l) {
  fprintf(Outfile, "\tjmp\tL%d\n", l);
}
```

最后，我们需要一个函数来进行比较并根据相反的比较进行跳转。因此，使用 AST 比较节点类型，我们进行相反的比较：

```c
// List of inverted jump instructions,
// in AST order: A_EQ, A_NE, A_LT, A_GT, A_LE, A_GE
static char *invcmplist[] = { "jne", "je", "jge", "jle", "jg", "jl" };

// Compare two registers and jump if false.
int cgcompare_and_jump(int ASTop, int r1, int r2, int label) {

  // Check the range of the AST operation
  if (ASTop < A_EQ || ASTop > A_GE)
    fatal("Bad ASTop in cgcompare_and_set()");

  fprintf(Outfile, "\tcmpq\t%s, %s\n", reglist[r2], reglist[r1]);
  fprintf(Outfile, "\t%s\tL%d\n", invcmplist[ASTop - A_EQ], label);
  freeall_registers();
  return (NOREG);
}
```

## 测试 IF 语句

执行`make test`编译`input05`文件的操作：

```c
{
  int i; int j;
  i=6; j=12;
  if (i < j) {
    print i;
  } else {
    print j;
  }
}
```

这是最终的汇编输出：

```
        movq    $6, %r8
        movq    %r8, i(%rip)    # i=6;
        movq    $12, %r8
        movq    %r8, j(%rip)    # j=12;
        movq    i(%rip), %r8
        movq    j(%rip), %r9
        cmpq    %r9, %r8        # Compare %r8-%r9, i.e. i-j
        jge     L1              # Jump to L1 if i >= j
        movq    i(%rip), %r8
        movq    %r8, %rdi       # print i;
        call    printint
        jmp     L2              # Skip the else code
L1:
        movq    j(%rip), %r8
        movq    %r8, %rdi       # print j;
        call    printint
L2:
```

当然，还`make test`显示：

```
cc -o comp1 -g cg.c decl.c expr.c gen.c main.c misc.c
      scan.c stmt.c sym.c tree.c
./comp1 input05
cc -o out out.s
./out
6                   # As 6 is less than 12
```

## 结论和下一步是什么

我们已经使用 IF 语句将第一个控制结构添加到我们的语言中。一路上我不得不重写一些现有的东西，并且考虑到我脑子里没有完整的架构计划，我将来可能不得不重写更多的东西。

这部分旅程的问题在于，我们必须对 IF 决策执行与正常比较运算符相反的比较。我的解决方案是告知每个 AST 节点其父节点的节点类型；比较节点现在可以查看父节点是否是 A_IF 节点。

我知道 Nils Holm 在实现 SubC 时选择了不同的方法，因此您应该查看他的代码，以了解同一问题的不同解决方案。

在编译器编写之旅的下一部分中，我们将添加另一个控制结构：WHILE 循环。[下一步](09-While_Loops.md)
