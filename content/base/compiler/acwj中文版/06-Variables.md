#  第 6 部分：变量

我刚刚完成向编译器添加全局变量，正如我怀疑的那样，这是一项艰巨的工作。此外，编译器中的几乎每个文件都在此过程中被修改。所以这一部分的旅程将会很长。

## 我们想从变量中得到什么？

我们希望能够：

- 声明变量
- 使用变量来获取存储的值
- 分配给变量

这是`input02`我们的测试程序：

```
int fred;
int jim;
fred= 5;
jim= 12;
print fred + jim;
```

最明显的变化是语法现在有了变量声明、赋值语句和表达式中的变量名称。然而，在我们开始之前，让我们看看如何实现变量。

## 符号表

每个编译器都需要一个 [符号表](https://en.wikipedia.org/wiki/Symbol_table)。稍后，我们将不仅仅保存全局变量。但现在，这是表中条目的结构（来自`defs.h`）：

```c
// Symbol table structure
struct symtable {
  char *name;                   // Name of a symbol
};
```

我们有一个符号数组`data.h`：

```c
#define NSYMBOLS        1024            // Number of symbol table entries
extern_ struct symtable Gsym[NSYMBOLS]; // Global symbol table
static int Globs = 0;                   // Position of next free global symbol slot
```

`Globs`实际上位于`sym.c`管理符号表的文件中。在这里我们有这些管理功能：

- `int findglob(char *s)`：判断符号s是否在全局符号表中。返回其插槽位置，如果未找到则返回 -1。
- `static int newglob(void)`：获取新的全局符号槽的位置，或者如果我们用完位置就死。
- `int addglob(char *name)`：将全局符号添加到符号表中。返回符号表中的槽号。

该代码相当简单，因此我不会在讨论中给出代码。通过这些函数，我们可以查找符号并将新符号添加到符号表中。

## 扫描和新令牌

如果您查看示例输入文件，我们需要一些新标记：

- 'int'，称为 T_INT
- '='，称为 T_EQUALS
- 标识符名称，称为 T_IDENT

'=' 的扫描很容易添加到`scan()`：

```c
  case '=':
    t->token = T_EQUALS; break;
```

我们可以将 'int' 关键字添加到`keyword()`：

```c
  case 'i':
    if (!strcmp(s, "int"))
      return (T_INT);
    break;
```

对于标识符，我们已经使用`scanident()`将单词存储到 `Text`变量中。如果一个单词不是关键字，我们可以返回一个 T_IDENT 标记，而不是死掉：

```c
   if (isalpha(c) || '_' == c) {
      // Read in a keyword or identifier
      scanident(c, Text, TEXTLEN);

      // If it's a recognised keyword, return that token
      if (tokentype = keyword(Text)) {
        t->token = tokentype;
        break;
      }
      // Not a recognised keyword, so it must be an identifier
      t->token = T_IDENT;
      break;
    }
```

## 新语法

我们即将准备好查看输入语言语法的变化。和以前一样，我将使用 BNF 表示法来定义它：

```
 statements: statement
      |      statement statements
      ;

 statement: 'print' expression ';'
      |     'int'   identifier ';'
      |     identifier '=' expression ';'
      ;

 identifier: T_IDENT
      ;
```

标识符作为 T_IDENT 标记返回，并且我们已经有了解析 print 语句的代码。但是，由于我们现在拥有三种类型的语句，因此编写一个函数来处理每种语句是有意义的。我们的顶级 `statements()`函数`stmt.c`现在看起来像：

```c
// Parse one or more statements
void statements(void) {

  while (1) {
    switch (Token.token) {
    case T_PRINT:
      print_statement();
      break;
    case T_INT:
      var_declaration();
      break;
    case T_IDENT:
      assignment_statement();
      break;
    case T_EOF:
      return;
    default:
      fatald("Syntax error, token", Token.token);
    }
  }
}
```

我已将旧的打印语句代码移入其中`print_statement()`，您可以自己浏览。

## 变量声明

让我们看一下变量声明。这是一个新文件 ，`decl.c`因为我们将来会有许多其他类型的声明。

```c
// Parse the declaration of a variable
void var_declaration(void) {

  // Ensure we have an 'int' token followed by an identifier
  // and a semicolon. Text now has the identifier's name.
  // Add it as a known identifier
  match(T_INT, "int");
  ident();
  addglob(Text);
  genglobsym(Text);
  semi();
}
```

和函数是`ident()`的`semi()`包装器`match()`：

```c
void semi(void)  { match(T_SEMI, ";"); }
void ident(void) { match(T_IDENT, "identifier"); }
```

回到`var_declaration()`，一旦我们将标识符扫描到`Text`缓冲区中，我们就可以使用 将该标识符添加到全局符号表中 `addglob(Text)`。那里的代码允许多次声明一个变量（目前）。

## 赋值语句

`assignment_statement()`这是in的代码`stmt.c`：

```c
void assignment_statement(void) {
  struct ASTnode *left, *right, *tree;
  int id;

  // Ensure we have an identifier
  ident();

  // Check it's been defined then make a leaf node for it
  if ((id = findglob(Text)) == -1) {
    fatals("Undeclared variable", Text);
  }
  right = mkastleaf(A_LVIDENT, id);

  // Ensure we have an equals sign
  match(T_EQUALS, "=");

  // Parse the following expression
  left = binexpr(0);

  // Make an assignment AST tree
  tree = mkastnode(A_ASSIGN, left, right, 0);

  // Generate the assembly code for the assignment
  genAST(tree, -1);
  genfreeregs();

  // Match the following semicolon
  semi();
}
```

我们有几个新的 AST 节点类型。A_ASSIGN 获取左侧子级中的表达式并将其分配给右侧子级。右侧子节点将是 A_LVIDENT 节点。

为什么我将此节点称为*A_LVIDENT*？因为它代表一个*左值* 标识符。那么什么是 [左值](https://en.wikipedia.org/wiki/Value_(computer_science)#lrvalue)？

左值是与特定位置相关的值。在这里，它是内存中保存变量值的地址。当我们这样做时：

```
   area = width * height;
```

我们将右侧的结果（即右*值）**分配*给左侧的变量（即左*值*）。右*值*不绑定到特定位置。这里，表达式结果可能在某个任意寄存器中。

另请注意，虽然赋值语句的语法为

```
  identifier '=' expression ';'
```

我们将使表达式成为 A_ASSIGN 节点的左子树，并将 A_LVIDENT 详细信息保存在右子树中。为什么？因为我们需要先计算表达式*，然后*再将其保存到变量中。

## AST 结构的变化

现在，我们需要将整数文字值存储在 A_INTLIT AST 节点中，或者存储 A_IDENT AST 节点的符号详细信息。我已向AST 结构添加了一个*联合*`defs.h`来执行此操作（在 中）：

```c
// Abstract Syntax Tree structure
struct ASTnode {
  int op;                       // "Operation" to be performed on this tree
  struct ASTnode *left;         // Left and right child trees
  struct ASTnode *right;
  union {
    int intvalue;               // For A_INTLIT, the integer value
    int id;                     // For A_IDENT, the symbol slot number
  } v;
};
```

## 生成分配代码

```
genAST()`现在让我们看看in中的更改`gen.c
int genAST(struct ASTnode *n, int reg) {
  int leftreg, rightreg;

  // Get the left and right sub-tree values
  if (n->left)
    leftreg = genAST(n->left, -1);
  if (n->right)
    rightreg = genAST(n->right, leftreg);

  switch (n->op) {
  ...
    case A_INTLIT:
    return (cgloadint(n->v.intvalue));
  case A_IDENT:
    return (cgloadglob(Gsym[n->v.id].name));
  case A_LVIDENT:
    return (cgstorglob(reg, Gsym[n->v.id].name));
  case A_ASSIGN:
    // The work has already been done, return the result
    return (rightreg);
  default:
    fatald("Unknown AST operator", n->op);
  }
```

请注意，我们首先评估左侧 AST 子树，然后返回保存左侧子树值的寄存器编号。我们现在将此寄存器号传递到右侧子树。我们需要对 A_LVIDENT 节点执行此操作，以便`cgstorglob()`in 中的函数`cg.c` 知道哪个寄存器保存赋值表达式的右值结果。

所以，考虑一下这个 AST 树：

```
           A_ASSIGN
          /        \
     A_INTLIT   A_LVIDENT
        (3)        (5)
```

我们调用`leftreg = genAST(n->left, -1);`来评估 A_INTLIT 操作。这将`return (cgloadint(n->v.intvalue));`（即）向寄存器加载值 3 并返回寄存器 id。

然后，我们调用`rightreg = genAST(n->right, leftreg);`评估 A_LVIDENT 操作。这会将 `return (cgstorglob(reg, Gsym[n->v.id].name));`寄存器存储到名称为 的变量中`Gsym[5]`。

然后我们切换到 A_ASSIGN 情况。好了，我们所有的工作都已经完成了。右值仍在寄存器中，因此我们将其留在那里并返回它。稍后，我们将能够执行如下表达式：

```
  a= b= c = 0;
```

其中赋值不仅是一个语句，也是一个表达式。

## 生成 x86-64 代码

您可能已经注意到我将旧函数的名称更改`cgload()` 为`cgloadint()`. 这是更具体的。我们现在有一个函数可以从全局变量中加载值（在 中`cg.c`）：

```c
int cgloadglob(char *identifier) {
  // Get a new register
  int r = alloc_register();

  // Print out the code to initialise it
  fprintf(Outfile, "\tmovq\t%s(\%%rip), %s\n", identifier, reglist[r]);
  return (r);
}
```

同样，我们需要一个函数将寄存器保存到变量中：

```c
// Store a register's value into a variable
int cgstorglob(int r, char *identifier) {
  fprintf(Outfile, "\tmovq\t%s, %s(\%%rip)\n", reglist[r], identifier);
  return (r);
}
```

我们还需要一个函数来创建一个新的全局整型变量：

```c
// Generate a global symbol
void cgglobsym(char *sym) {
  fprintf(Outfile, "\t.comm\t%s,8,8\n", sym);
}
```

当然，我们不能让解析器直接访问这段代码。相反，通用代码生成器中有一个函数`gen.c` 充当接口：

```c
void genglobsym(char *s) { cgglobsym(s); }
```

## 表达式中的变量

现在我们可以分配给变量了。但是我们如何将变量的值放入表达式中。好吧，我们已经有了一个`primary()`获取整数字面量的函数。让我们修改它以加载变量的值：

```c
// Parse a primary factor and return an
// AST node representing it.
static struct ASTnode *primary(void) {
  struct ASTnode *n;
  int id;

  switch (Token.token) {
  case T_INTLIT:
    // For an INTLIT token, make a leaf AST node for it.
    n = mkastleaf(A_INTLIT, Token.intvalue);
    break;

  case T_IDENT:
    // Check that this identifier exists
    id = findglob(Text);
    if (id == -1)
      fatals("Unknown variable", Text);

    // Make a leaf AST node for it
    n = mkastleaf(A_IDENT, id);
    break;

  default:
    fatald("Syntax error, token", Token.token);
  }

  // Scan in the next token and return the leaf node
  scan(&Token);
  return (n);
}
```

请注意 T_IDENT 情况下的语法检查，以确保在尝试使用变量之前已声明该变量。

*另请注意，检索*变量值的AST 叶节点是 A_IDENT 节点。保存到变量中的叶子是 A_LVIDENT 节点。*这就是右值*和*左值*之间的区别。

## 尝试一下

我认为变量声明就是这样，所以让我们用该`input02`文件尝试一下：

```
int fred;
int jim;
fred= 5;
jim= 12;
print fred + jim;
```

我们可以`make test`这样做：

```
$ make test
cc -o comp1 -g cg.c decl.c expr.c gen.c main.c misc.c scan.c
               stmt.c sym.c tree.c
...
./comp1 input02
cc -o out out.s
./out
17
```

正如您所看到的，我们计算出`fred + jim`的是 5 + 12 或 17。以下是 中的新装配线`out.s`：

```
        .comm   fred,8,8                # Declare fred
        .comm   jim,8,8                 # Declare jim
        ...
        movq    $5, %r8
        movq    %r8, fred(%rip)         # fred = 5
        movq    $12, %r8
        movq    %r8, jim(%rip)          # jim = 12
        movq    fred(%rip), %r8
        movq    jim(%rip), %r9
        addq    %r8, %r9                # fred + jim
```

## 其他变化

我可能还做了一些其他的改变。我记得的唯一一个主要功能是创建一些辅助函数，`misc.c`以便更容易报告致命错误：

```c
// Print out fatal messages
void fatal(char *s) {
  fprintf(stderr, "%s on line %d\n", s, Line); exit(1);
}

void fatals(char *s1, char *s2) {
  fprintf(stderr, "%s:%s on line %d\n", s1, s2, Line); exit(1);
}

void fatald(char *s, int d) {
  fprintf(stderr, "%s:%d on line %d\n", s, d, Line); exit(1);
}

void fatalc(char *s, int c) {
  fprintf(stderr, "%s:%c on line %d\n", s, c, Line); exit(1);
}
```

## 结论和下一步是什么

所以这是很多工作。我们必须编写符号表管理的开始。我们必须处理两种新的语句类型。我们必须添加一些新的代币和一些新的 AST 节点类型。最后，我们必须添加一些代码来生成正确的 x86-64 汇编输出。

尝试编写一些示例输入文件，看看编译器是否按预期工作，特别是如果它检测到语法错误和语义错误（没有声明的变量使用）。

在编译器编写之旅的下一部分中，我们将向我们的语言添加六个比较运算符。这将使我们能够在之后的部分中开始控制结构。[下一步](07-Comparisons.md)
