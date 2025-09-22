#  第五部分：声明

是时候在我们的语言语法中添加一些“正确的”语句了。我希望能够编写这样的代码行：

```
   print 2 + 3 * 5;
   print 18 - 6/3 + 4*2;
```

当然，由于我们忽略空格，因此一条语句的所有标记不必位于同一行。每个语句都以关键字开头`print`，并以分号结束。所以这些将成为我们语言中的新标记。

## BNF 语法描述

我们已经了解了表达式的 BNF 表示法。现在让我们为上述类型的语句定义 BNF 语法：

```
statements: statement
     | statement statements
     ;

statement: 'print' expression ';'
     ;
```

输入文件由多个语句组成。它们要么是一个语句，要么是一个语句后面跟着多个语句。每条语句都以关键字 开头`print`，然后是一个表达式，最后是一个分号。

## 词法扫描器的更改

在我们获得解析上述语法的代码之前，我们需要在现有代码中添加一些零碎的东西。让我们从词法扫描器开始。

添加分号标记将很容易。现在，`print`关键字。稍后，我们将在该语言中拥有许多关键字，以及变量的标识符，因此我们需要添加一些代码来帮助我们处理它们。

在 中`scan.c`，我添加了从 SubC 编译器借来的代码。它将字母数字字符读入缓冲区，直到遇到非字母数字字符。

```c
// Scan an identifier from the input file and
// store it in buf[]. Return the identifier's length
static int scanident(int c, char *buf, int lim) {
  int i = 0;

  // Allow digits, alpha and underscores
  while (isalpha(c) || isdigit(c) || '_' == c) {
    // Error if we hit the identifier length limit,
    // else append to buf[] and get next character
    if (lim - 1 == i) {
      printf("identifier too long on line %d\n", Line);
      exit(1);
    } else if (i < lim - 1) {
      buf[i++] = c;
    }
    c = next();
  }
  // We hit a non-valid character, put it back.
  // NUL-terminate the buf[] and return the length
  putback(c);
  buf[i] = '\0';
  return (i);
}
```

我们还需要一个函数来识别语言中的关键字。一种方法是拥有一个关键字列表，然后针对 的`strcmp()` 缓冲区遍历该列表和每个关键字`scanident()`。SubC 的代码有一个优化：在执行`strcmp()`. 这加快了与数十个关键字的比较。现在我们不需要这种优化，但我已将其放入稍后：

```c
// Given a word from the input, return the matching
// keyword token number or 0 if it's not a keyword.
// Switch on the first letter so that we don't have
// to waste time strcmp()ing against all the keywords.
static int keyword(char *s) {
  switch (*s) {
    case 'p':
      if (!strcmp(s, "print"))
        return (T_PRINT);
      break;
  }
  return (0);
}
```

现在，在 switch 语句的底部`scan()`，我们添加以下代码来识别分号和关键字：

```c
    case ';':
      t->token = T_SEMI;
      break;
    default:

      // If it's a digit, scan the
      // literal integer value in
      if (isdigit(c)) {
        t->intvalue = scanint(c);
        t->token = T_INTLIT;
        break;
      } else if (isalpha(c) || '_' == c) {
        // Read in a keyword or identifier
        scanident(c, Text, TEXTLEN);

        // If it's a recognised keyword, return that token
        if (tokentype = keyword(Text)) {
          t->token = tokentype;
          break;
        }
        // Not a recognised keyword, so an error for now
        printf("Unrecognised symbol %s on line %d\n", Text, Line);
        exit(1);
      }
      // The character isn't part of any recognised token, error
      printf("Unrecognised character %c on line %d\n", c, Line);
      exit(1);
```

我还添加了一个全局`Text`缓冲区来存储关键字和标识符：

```c
#define TEXTLEN         512             // Length of symbols in input
extern_ char Text[TEXTLEN + 1];         // Last identifier scanned
```

## 对表达式解析器的更改

到目前为止，我们的输入文件只包含一个表达式；`binexpr()`因此，在(in )中的 Pratt 解析器代码中`expr.c`，我们使用以下代码退出解析器：

```c
  // If no tokens left, return just the left node
  tokentype = Token.token;
  if (tokentype == T_EOF)
    return (left);
```

在我们的新语法中，每个表达式都以分号结尾。因此，我们需要更改表达式解析器中的代码以发现标记`T_SEMI` 并退出表达式解析：

```c
// Return an AST tree whose root is a binary operator.
// Parameter ptp is the previous token's precedence.
struct ASTnode *binexpr(int ptp) {
  struct ASTnode *left, *right;
  int tokentype;

  // Get the integer literal on the left.
  // Fetch the next token at the same time.
  left = primary();

  // If we hit a semicolon, return just the left node
  tokentype = Token.token;
  if (tokentype == T_SEMI)
    return (left);

    while (op_precedence(tokentype) > ptp) {
      ...

          // Update the details of the current token.
    // If we hit a semicolon, return just the left node
    tokentype = Token.token;
    if (tokentype == T_SEMI)
      return (left);
    }
}
```

## 代码生成器的更改

我想将通用代码生成器`gen.c` 与`cg.c`. 这也意味着编译器的其余部分应该只调用 中的函数 `gen.c`，并且只`gen.c`应该调用 中的代码`cg.c`。

为此，我在中定义了一些新的“前端”函数`gen.c`：

```c
void genpreamble()        { cgpreamble(); }
void genpostamble()       { cgpostamble(); }
void genfreeregs()        { freeall_registers(); }
void genprintint(int reg) { cgprintint(reg); }
```

## 添加语句解析器

我们有一个新文件`stmt.c`。这将保存我们语言中所有主要语句的解析代码。现在，我们需要解析我上面放弃的语句的 BNF 语法。这是通过这个单一函数完成的。我已将递归定义转换为循环：

```c
// Parse one or more statements
void statements(void) {
  struct ASTnode *tree;
  int reg;

  while (1) {
    // Match a 'print' as the first token
    match(T_PRINT, "print");

    // Parse the following expression and
    // generate the assembly code
    tree = binexpr(0);
    reg = genAST(tree);
    genprintint(reg);
    genfreeregs();

    // Match the following semicolon
    // and stop if we are at EOF
    semi();
    if (Token.token == T_EOF)
      return;
  }
}
```

在每个循环中，代码都会找到一个 T_PRINT 标记。然后它调用`binexpr()`解析表达式。最后，它找到了 T_SEMI 令牌。如果后面有 T_EOF 标记，我们就会跳出循环。

在每个表达式树之后，调用 in 中的代码`gen.c`将树转换为汇编代码并调用汇编`printint()`函数打印出最终值。

## 一些辅助函数

上面的代码中有几个新的辅助函数，我已将其放入一个新文件中`misc.c`：

```c
// Ensure that the current token is t,
// and fetch the next token. Otherwise
// throw an error 
void match(int t, char *what) {
  if (Token.token == t) {
    scan(&Token);
  } else {
    printf("%s expected on line %d\n", what, Line);
    exit(1);
  }
}

// Match a semicon and fetch the next token
void semi(void) {
  match(T_SEMI, ";");
}
```

这些构成解析器中语法检查的一部分。稍后，我将添加更多短函数来调用，`match()`以使我们的语法检查更容易。

## 更改为`main()`

`main()`用于`binexpr()`直接调用来解析旧输入文件中的单个表达式。现在它这样做：

```c
  scan(&Token);                 // Get the first token from the input
  genpreamble();                // Output the preamble
  statements();                 // Parse the statements in the input
  genpostamble();               // Output the postamble
  fclose(Outfile);              // Close the output file and exit
  exit(0);
```

## 尝试一下

新的和更改的代码就是这样。让我们尝试一下新代码。这是新的输入文件`input01`：

```
print 12 * 3;
print 
   18 - 2
      * 4; print
1 + 2 +
  9 - 5/2 + 3*5;
```

是的，我决定检查一下我们是否有分布在多条线上的代币。要编译并运行输入文件，请执行以下操作`make test`：

```make
$ make test
cc -o comp1 -g cg.c expr.c gen.c main.c misc.c scan.c stmt.c tree.c
./comp1 input01
cc -o out out.s
./out
36
10
25
```

它有效！

## 结论和下一步是什么

我们已经在我们的语言中添加了第一个“真正的”语句语法。我已经用 BNF 表示法定义了它，但使用循环而不是递归来实现它更容易。别担心，我们很快就会回去进行递归解析。

在此过程中，我们必须修改扫描器，添加对关键字和标识符的支持，并更清晰地分离通用代码生成器和特定于 CPU 的生成器。

在编译器编写之旅的下一部分中，我们将向语言添加变量。这将需要大量的工作。[下一步](06-Variables.md)
