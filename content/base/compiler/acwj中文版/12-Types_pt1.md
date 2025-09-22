# 第12部分：类型，第1部分

我刚开始向我们的编译器中添加类型。现在，我应该提醒你，这对我来说是新的，因为在我之前的[编译器](https://github.com/DoctorWkt/h-compiler)中我只使用了`int`类型。我抵制住了去看 SubC 源代码寻找灵感的冲动。因此，我正在自己尝试，而且很可能我将不得不重新处理一些代码，以应对涉及类型的更大问题。

## 目前的类型是什么？

我将从全局变量的 `char` 和 `int` 开始。我们已经为函数添加了 `void` 关键字。在下一步中，我将添加函数返回值。所以，目前 `void` 存在，但我还没有完全处理它。

显然，`char` 的值范围比 `int` 小得多。像 SubC 一样，我将使用 0 .. 255 的范围作为 `char`，以及 `int` 的一系列有符号值。

这意味着我们可以将 `char` 值扩展为 `int`，但如果开发者试图将 `int` 值缩小到 `char` 范围内，我们必须警告他们。

## 新的关键字和令牌

这里只有新的 'char' 关键字和 T_CHAR 令牌。这里没什么激动人心的。

## 表达式类型

从现在开始，每个表达式都有一个类型。这包括：

 + 整数字面量，例如 56 是一个 `int`
 + 数学表达式，例如 45 - 12 是一个 `int`
 + 变量，例如如果我们声明 `x` 为一个 `char`，那么它的*右值*是一个 `char`

我们将不得不跟踪我们评估的每个表达式的类型，以确保我们可以根据需要扩展它，或者在必要时拒绝缩小它。

在 SubC 编译器中，Nils 创建了一个单一的 *左值* 结构。一个指向这个单一结构的指针在递归解析器中传递，以跟踪任何表达式在解析过程中的类型。

我采取了不同的方法。我修改了我们的抽象语法树节点，以包含一个 `type` 字段，该字段保存了该点的树的类型。在 `defs.h` 中，这是我到目前为止创建的类型：

```c
// 原始类型
enum {
  P_NONE, P_VOID, P_CHAR, P_INT
};
```

我像 Nils 在 SubC 中那样称它们为 *原始* 类型，因为我想不出更好的名字。数据类型，也许？P_NONE 值表示 AST 节点*不*代表表达式且没有类型。一个例子是 A_GLUE 节点类型，它将语句粘合在一起：一旦生成了左手边的语句，就没有类型可言。

如果你看 `tree.c`，你会看到用于构建 AST 节点的函数已被修改，以便也为新 AST 节点结构中的 `type` 字段（在 `defs.h` 中）赋值：

```c
struct ASTnode {
  int op;                       // 要在这棵树上执行的“操作”
  int type;                     // 这棵树生成的任何表达式的类型
  ...
};
```
## 变量声明及其类型

我们现在至少有两种方法来声明全局变量：

```c
  int x; char y;
```

我们需要解析这个，没错。但首先，我们如何记录每个变量的类型呢？我们需要修改 `symtable` 结构。我还添加了关于将来会使用的符号的“结构类型”的细节（在 `defs.h` 中）：

```c
// 结构类型
enum {
  S_VARIABLE, S_FUNCTION
};

// 符号表结构
struct symtable {
  char *name;                   // 符号的名称
  int type;                     // 符号的原始类型
  int stype;                    // 符号的结构类型
};
```

在 `sym.c` 中的 `newglob()` 中有新代码来初始化这些新字段：

```c
int addglob(char *name, int type, int stype) {
  ...
  Gsym[y].type = type;
  Gsym[y].stype = stype;
  return (y);
}
```

## 解析变量声明

是时候将变量本身的解析从类型的解析中分离出来了。因此，在 `decl.c` 中我们现在有：

```c
// 解析当前令牌并
// 返回一个原始类型枚举值
int parse_type(int t) {
  if (t == T_CHAR) return (P_CHAR);
  if (t == T_INT)  return (P_INT);
  if (t == T_VOID) return (P_VOID);
  fatald("非法类型, 令牌", t);
}

// 解析一个变量的声明
void var_declaration(void) {
  int id, type;

  // 获取变量的类型，然后是标识符
  type = parse_type(Token.token);
  scan(&Token);
  ident();
  id = addglob(Text, type, S_VARIABLE);
  genglobsym(id);
  semi();
}
```

## 处理表达式类型

以上所有都是简单的部分完成了！我们现在有：

  + 一组三种类型：`char`、`int` 和 `void`，
  + 解析变量声明以找到它们的类型，
  + 在符号表中捕获每个变量的类型，
  + 在每个 AST 节点中存储表达式的类型

现在我们需要实际填充我们构建的 AST 节点中的类型。然后我们必须决定何时扩展类型和/或拒绝类型冲突。让我们继续工作吧！

## 解析主要术语

我们将从解析整数字面值和变量标识符开始。一个问题是我们希望能够执行：

```c
  char j; j = 2;
```

但是如果我们将 `2` 标记为 P_INT，那么当我们尝试将其存储在 P_CHAR 类型的 `j` 变量中时，我们将无法缩小这个值。目前，我添加了一些语义代码以保持小的整数字面值为 P_CHAR：

```c
// 解析一个主要因子并返回一个
// 表示它的 AST 节点。
static struct ASTnode *primary(void) {
  struct ASTnode *n;
  int id;

  switch (Token.token) {
    case T_INTLIT:
      // 对于 INTLIT 令牌，为其创建一个叶子 AST 节点。
      // 如果它在 P_CHAR 范围内，将其设为 P_CHAR
      if ((Token.intvalue) >= 0 && (Token.intvalue < 256))
        n = mkastleaf(A_INTLIT, P_CHAR, Token.intvalue);
      else
        n = mkastleaf(A_INTLIT, P_INT, Token.intvalue);
      break;

    case T_IDENT:
      // 检查这个标识符是否存在
      id = findglob(Text);
      if (id == -1)
        fatals("未知变量", Text);

      // 为其创建一个叶子 AST 节点
      n = mkastleaf(A_IDENT, Gsym[id].type, id);
      break;

    default:
      fatald("语法错误,

 令牌", Token.token);
  }

  // 扫描下一个令牌并返回叶子节点
  scan(&Token);
  return (n);
}
```

另外请注意，对于标识符，我们可以很容易地从全局符号表中获取它们的类型详细信息。
## 构建二元表达式：比较类型

当我们使用二元数学运算符构建数学表达式时，我们会有来自左子节点的类型和来自右子节点的类型。在这里，我们将不得不扩展类型、保持不变，或者如果两种类型不兼容则拒绝表达式。

目前，我在一个新文件 `types.c` 中有一个函数来比较两侧的类型。这是代码：

```c
// 给定两个原始类型，如果它们兼容则返回 true，否则返回 false。
// 同时返回零或者一个 A_WIDEN 操作，如果其中一个需要扩展以匹配另一个。
// 如果 onlyright 为 true，则只从左到右扩展。
int type_compatible(int *left, int *right, int onlyright) {

  // Void 与任何类型都不兼容
  if ((*left == P_VOID) || (*right == P_VOID)) return (0);

  // 相同类型，它们是兼容的
  if (*left == *right) { *left = *right = 0; return (1); }

  // 必要时将 P_CHAR 扩展为 P_INT
  if ((*left == P_CHAR) && (*right == P_INT)) {
    *left = A_WIDEN; *right = 0; return (1);
  }
  if ((*left == P_INT) && (*right == P_CHAR)) {
    if (onlyright) return (0);
    *left = 0; *right = A_WIDEN; return (1);
  }
  // 其他任何剩余的组合都是兼容的
  *left = *right = 0;
  return (1);
}
```

这里有很多事情发生。首先，如果两种类型相同，我们可以简单地返回 True。任何带有 P_VOID 的类型不能与其他类型混合。

如果一侧是 P_CHAR 而另一侧是 P_INT，我们可以将结果扩展为 P_INT。我这样做的方式是修改传入的类型信息，并用零（什么都不做）或一个新的 AST 节点类型 A_WIDEN 替换它。这意味着：将更窄的子节点的值扩展到与较宽的子节点的值一样宽。我们很快就会看到这一点。

`onlyright` 是一个额外的参数。当我们处理 A_ASSIGN AST 节点时，我们正在将左子节点的表达式分配给右侧的 *左值* 变量。如果设置了这个参数，不要让 P_INT 表达式转移到 P_CHAR 变量。

最后，目前先让任何其他类型对通过。

我可以保证一旦我们引入数组和指针，这将需要更改。我也希望我能找到一种方法使代码更简单、更优雅。但目前这样就可以了。

## 在表达式中使用 `type_compatible()`

我在这个版本的编译器中的三个不同地方使用了 `type_compatible()`。我们从合并带有二元运算符的表达式开始。我修改了 `expr.c` 中的 `binexpr()` 代码来做这件事：

```c
    // 确保两种类型是兼容的。
    lefttype = left->type;
    righttype = right->type;
    if (!type_compatible(&lefttype, &righttype, 0))
      fatal("类型不兼容");

    // 必要时扩展任意一侧。type 变量现在是 A_WIDEN
    if (lefttype)
      left = mkastunary(lefttype, right->type, left, 0);
    if (righttype)
      right = mkastunary(righttype, left->type, right, 0);

    // 将该子树与我们的树结合。同时将令牌
    // 转换为 AST 操作。
    left = mkastnode(arithop(tokentype), left->type, left, NULL, right, 0);
```

我们拒绝不

兼容的类型。但是，如果 `type_compatible()` 返回了非零的 `lefttype` 或 `righttype` 值，这些实际上是 A_WIDEN 值。我们可以使用这个值来构建一个以窄子节点为子节点的一元 AST 节点。当我们进入代码生成阶段时，它将知道这个子节点的值必须被扩展。

那么，我们还需要在哪些地方扩展表达式值呢？

## 使用 `type_compatible()` 打印表达式

当我们使用 `print` 关键字时，我们需要有一个 `int` 表达式来打印。所以我们需要更改 `stmt.c` 中的 `print_statement()`：

```c
static struct ASTnode *print_statement(void) {
  struct ASTnode *tree;
  int lefttype, righttype;
  int reg;

  ...
  // 解析接下来的表达式
  tree = binexpr(0);

  // 确保两种类型是兼容的。
  lefttype = P_INT; righttype = tree->type;
  if (!type_compatible(&lefttype, &righttype, 0))
    fatal("类型不兼容");

  // 必要时扩展树。 
  if (righttype) tree = mkastunary(righttype, P_INT, tree, 0);
```

## 使用 `type_compatible()` 为变量赋值

这是我们需要检查类型的最后一个地方。当我们为变量赋值时，我们需要确保可以扩展右侧表达式。我们必须拒绝任何尝试将宽类型存储到窄变量中的尝试。这是 `stmt.c` 中 `assignment_statement()` 中的新代码：

```c
static struct ASTnode *assignment_statement(void) {
  struct ASTnode *left, *right, *tree;
  int lefttype, righttype;
  int id;

  ...
  // 为变量创建一个左值节点
  right = mkastleaf(A_LVIDENT, Gsym[id].type, id);

  // 解析接下来的表达式
  left = binexpr(0);

  // 确保两种类型是兼容的。
  lefttype = left->type;
  righttype = right->type;
  if (!type_compatible(&lefttype, &righttype, 1))  // 注意这里的 1
    fatal("类型不兼容");

  // 必要时扩展左侧。
  if (lefttype)
    left = mkastunary(lefttype, right->type, left, 0);
```

注意对 `type_compatible()` 调用的末尾有一个 1。这强制了我们不能将宽值保存到窄变量的语义。

鉴于以上所有内容，我们现在可以解析一些类型并强制执行一些合理的语言语义：在可能的情况下扩展值，防止类型缩小和防止不适合的类型冲突。现在我们转向代码生成的方面。
## 对 x86-64 代码生成的更改

我们的汇编输出基于寄存器，本质上它们的大小是固定的。我们可以影响的是：

 + 用于存储变量的内存位置的大小，以及
 + 用于保存数据的寄存器的使用量，例如，一个字节用于字符，八个字节用于 64 位整数。

我将从 `cg.c` 中的 x86-64 特定代码开始，然后我将展示这是如何在 `gen.c` 中的通用代码生成器中使用的。

让我们开始生成变量的存储。

```c
// 生成一个全局符号
void cgglobsym(int id) {
  // 选择 P_INT 或 P_CHAR
  if (Gsym[id].type == P_INT)
    fprintf(Outfile, "\t.comm\t%s,8,8\n", Gsym[id].name);
  else
    fprintf(Outfile, "\t.comm\t%s,1,1\n", Gsym[id].name);
}
```

我们从符号表中的变量槽提取类型，并根据此类型选择分配 1 字节或 8 字节。现在我们需要将值加载到寄存器中：

```c
// 将变量的值从内存加载到寄存器。
// 返回寄存器的编号
int cgloadglob(int id) {
  // 获取一个新寄存器
  int r = alloc_register();

  // 输出初始化它的代码：P_CHAR 或 P_INT
  if (Gsym[id].type == P_INT)
    fprintf(Outfile, "\tmovq\t%s(\%%rip), %s\n", Gsym[id].name, reglist[r]);
  else
    fprintf(Outfile, "\tmovzbq\t%s(\%%rip), %s\n", Gsym[id].name, reglist[r]);
  return (r);
```

`movq` 指令将八个字节移动到 8 字节寄存器中。`movzbq` 指令将 8 字节寄存器归零，然后将单个字节移动进去。这也隐含地将一个字节值扩展到八个字节。我们的存储函数也类似：

```c
// 将寄存器的值存储到变量中
int cgstorglob(int r, int id) {
  // 选择 P_INT 或 P_CHAR
  if (Gsym[id].type == P_INT)
    fprintf(Outfile, "\tmovq\t%s, %s(\%%rip)\n", reglist[r], Gsym[id].name);
  else
    fprintf(Outfile, "\tmovb\t%s, %s(\%%rip)\n", breglist[r], Gsym[id].name);
  return (r);
}
```

这次我们必须使用寄存器的“字节”名称和 `movb` 指令来移动单个字节。

幸运的是，`cgloadglob()` 函数已经完成了 P_CHAR 变量的扩展。所以这是我们新的 `cgwiden()` 函数的代码：

```c
// 将寄存器中的值从旧类型
// 扩展到新类型，并返回一个包含
// 这个新值的寄存器
int cgwiden(int r, int oldtype, int newtype) {
  // 无需操作
  return (r);
}
```
## 对通用代码生成的更改

有了以上的基础，`gen.c` 中的通用代码生成器只需少量更改：

  + 对 `cgloadglob()` 和 `cgstorglob()` 的调用现在接收符号的槽号，而不是符号的名称。
  + 同样，`genglobsym()` 现在接收符号的槽号，并将其传递给 `cgglobsym()`。

唯一的主要更改是处理新的 A_WIDEN AST 节点类型的代码。
我们不需要这个节点（因为 `cgwiden()` 什么也不做），但它是为其他硬件平台准备的：

```c
    case A_WIDEN:
     

 // 将子节点的类型扩展到父节点的类型
      return (cgwiden(leftreg, n->left->type, n->type));
```

## 测试新类型更改

这是我的测试输入文件 `tests/input10`：

```c
void main()
{
  int i; char j;

  j= 20; print j;
  i= 10; print i;

  for (i= 1;   i <= 5; i= i + 1) { print i; }
  for (j= 253; j != 2; j= j + 1) { print j; }
}
```

我验证我们可以对 `char` 和 `int` 类型进行赋值和打印。
我还验证了，对于 `char` 变量，我们将在值序列中溢出：253, 254, 255, 0, 1, 2 等。

```
$ make test
cc -o comp1 -g cg.c decl.c expr.c gen.c main.c misc.c scan.c
   stmt.c sym.c tree.c types.c
./comp1 tests/input10
cc -o out out.s
./out
20
10
1
2
3
4
5
253
254
255
0
1
```

让我们看一下生成的一些汇编代码：

```
        .comm   i,8,8                   # 八字节 i 存储
        .comm   j,1,1                   # 一个字节 j 存储
        ...
        movq    $20, %r8
        movb    %r8b, j(%rip)           # j= 20
        movzbq  j(%rip), %r8
        movq    %r8, %rdi               # 打印 j
        call    printint

        movq    $253, %r8
        movb    %r8b, j(%rip)           # j= 253
L3:
        movzbq  j(%rip), %r8
        movq    $2, %r9
        cmpq    %r9, %r8                # 当 j != 2
        je      L4
        movzbq  j(%rip), %r8
        movq    %r8, %rdi               # 打印 j
        call    printint
        movzbq  j(%rip), %r8
        movq    $1, %r9                 # j= j + 1
        addq    %r8, %r9
        movb    %r9b, j(%rip)
        jmp     L3
```

虽然这不是最优雅的汇编代码，但它确实有效。此外，`$ make test` 确认所有之前的代码示例仍然有效。

## 结论和下一步

在我们的编译器编写旅程的下一部分，我们将添加带有一个参数的函数调用，并从函数中返回一个值。[下一步](13-Functions_pt2.md)