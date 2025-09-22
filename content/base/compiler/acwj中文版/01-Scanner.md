# 第一部分：词法分析简介

我们以一个简单的词法分析器开始编译器编写之旅。
正如我在前一部分中提到的，词法分析器的工作是识别输入语言中的词法单元，或者称为 *令牌*。

我们将从一个只有五个词法单元的语言开始：

+ 四个基本的数学运算符：`*`、`/`、`+` 和 `-`
+ 十进制整数，具有 1 个或多个数字 `0` 到 `9`

我们扫描的每个令牌都将存储在这个结构中（来自 `defs.h`）：

```c
// 令牌结构
struct token {
  int token;
  int intvalue;
};
```

其中 `token` 字段可以是以下值之一（来自 `defs.h`）：

```c
// 令牌
enum {
  T_PLUS, T_MINUS, T_STAR, T_SLASH, T_INTLIT
};
```

当令牌是 `T_INTLIT`（即整数字面值）时，`intvalue` 字段将保存我们扫描到的整数的值。

## `scan.c` 中的函数

`scan.c` 文件包含我们词法分析器的函数。我们将逐个从输入文件中读取字符。然而，当我们在输入流中读取得太远时，有时需要“放回”一个字符。我们还希望跟踪我们当前所在的行，以便在调试消息中打印行号。所有这些都由 `next()` 函数完成：

```c
// 从输入文件获取下一个字符。
static int next(void) {
  int c;

  if (Putback) {                // 如果有字符被放回，使用它
    c = Putback;                // 返回该字符
    Putback = 0;
    return c;
  }

  c = fgetc(Infile);            // 从输入文件读取
  if ('\n' == c)
    Line++;                     // 增加行数
  return c;
}
```

`Putback` 和 `Line` 变量与我们的输入文件指针一起在 `data.h` 中定义：

```c
extern_ int     Line;
extern_ int     Putback;
extern_ FILE    *Infile;
```

所有 C 文件都将包含这个，其中 `extern_` 将被替换为 `extern`。但是，`main.c` 将去除 `extern_`，因此这些变量将“属于”`main.c`。

最后，我们如何将字符放回到输入流中呢？像这样：

```c
// 放回一个不需要的字符
static void putback(int c) {
  Putback = c;
}
```

## 忽略空白字符

我们需要一个函数，它可以读取并静默跳过空白字符，直到找到非空白字符为止，并返回它。因此：

```c
// 跳过我们不需要处理的输入，即空白、换行。返回第一个
// 需要处理的字符。
static int skip(void) {
  int c;

  c = next();
  while (' ' == c || '\t' == c || '\n' == c || '\r' == c || '\f' == c) {
    c = next();
  }
  return (c);
}
```

## 扫描令牌：`scan()`

现在我们可以在跳过空白字符的同时读取字符；如果我们读取了一个字符太远，我们也可以将其放回。我们现在可以编写我们的第一个词法分析器了：

```c
// 扫描并返回输入中找到的下一个令牌。
// 如果令牌有效，返回 1；如果没有令牌，返回 0。
int scan(struct token *t) {
  int c;

  // 跳过空白
  c = skip();

  // 根据输入字符确定令牌
  switch (c) {
  case EOF:
    return (0);
  case '+':
    t->token = T_PLUS;
    break;
  case '-':
    t->token = T_MINUS;
    break;
  case '*':
    t->token = T_STAR;
    break;
  case '/':
    t->token = T_SLASH;
    break;
  default:
    // 待续
  }

  // 我们找到了一个令牌
  return (1);
}
```

就是这样对于简单的单字符令牌：对于每个被识别的字符，将其转换为令牌。你可能会问：为什么不直接将被识别的字符放入 `struct token` 中？答案是，稍后我们将需要识别多字符令牌，如 `==` 和关键字如 `if` 和 `while`。因此，将令牌值列出为枚举值将使生活更加轻松。

## 整数字面值

实际上，我们已经不得不面对这种情况，因为我们还需要识别整数字面值，如 `3827` 和 `87731`。这是 `switch` 语句中缺失的 `default` 代码：

```c
  default:

    // 如果是数字，则扫描
    // 字面整数值
    if (isdigit(c)) {
      t->intvalue = scanint(c);
      t->token = T_INTLIT;
      break;
    }

    printf("Unrecognised character %c on line %d\n", c, Line);
    exit(1);
```

一旦我们遇到十进制数字字符，我们将使用这个第一个字符调用辅助函数 `scanint()`。它将返回扫描到的整数值。为了做到这一点，它必须逐个读取每个字符，检查它是否是合法数字，并构建最终数字。这是代码：

```c
// 从输入文件扫描并返回一个整数字面值
static int scanint(int c) {
  int k, val = 0;

  // 将

每个字符转换为整数值
  while ((k = chrpos("0123456789", c)) >= 0) {
    val = val * 10 + k;
    c = next();
  }

  // 我们遇到了非整数字符，将其放回。
  putback(c);
  return val;
}
```

我们从零 `val` 值开始。每次我们得到一个在集合 `0` 到 `9` 中的字符时，我们就用 `chrpos()` 将其转换为 `int` 值。我们将 `val` 变为 10 倍，然后将这个新的数字加到其中。

例如，如果我们有字符 `3`、`2`、`8`，我们做：

+ `val= 0 * 10 + 3`，即 3
+ `val= 3 * 10 + 2`，即 32
+ `val= 32 * 10 + 8`，即 328

在最后，你注意到了对 `putback(c)` 的调用吗？
在这一点上，我们找到了一个不是十进制数字的字符。
我们不能简单地丢弃它，但幸运的是我们可以将其放回
到输入流中以后消耗。

在这一点上，你可能会问：为什么不直接从 `c` 中减去 `'0'` 的 ASCII 值，将其转换为整数呢？答案是，稍后我们将能够执行 `chrpos("0123456789abcdef")` 以转换十六进制数字。

这是 `chrpos()` 的代码：

```c
// 返回字符 c 在字符串 s 中的位置，
// 如果找不到，则返回 -1
static int chrpos(char *s, int c) {
  char *p;

  p = strchr(s, c);
  return (p ? p - s : -1);
}
```

到目前为止，这就是 `scan.c` 中的词法分析器代码。

## 让词法分析器投入使用

`main.c` 中的代码将上述词法分析器投入使用。`main()` 函数打开一个文件，然后扫描它以查找令牌：

```c
void main(int argc, char *argv[]) {
  ...
  init();
  ...
  Infile = fopen(argv[1], "r");
  ...
  scanfile();
  exit(0);
}
```

而 `scanfile()` 在存在新令牌的情况下循环，并打印出令牌的详细信息：

```c
// 可打印令牌列表
char *tokstr[] = { "+", "-", "*", "/", "intlit" };

// 循环扫描输入文件中的所有令牌。
// 打印出每个找到的令牌的详细信息。
static void scanfile() {
  struct token T;

  while (scan(&T)) {
    printf("Token %s", tokstr[T.token]);
    if (T.token == T_INTLIT)
      printf(", value %d", T.intvalue);
    printf("\n");
  }
}
```

## 一些示例输入文件

我提供了一些示例输入文件，以便你可以看到扫描器在每个文件中找到了什么令牌，以及扫描器拒绝了哪些输入文件。

```
$ make
cc -o scanner -g main.c scan.c

$ cat input01
2 + 3 * 5 - 8 / 3

$ ./scanner input01
Token intlit, value 2
Token +
Token intlit, value 3
Token *
Token intlit, value 5
Token -
Token intlit, value 8
Token /
Token intlit, value 3

$ cat input04
23 +
18 -
45.6 * 2
/ 18

$ ./scanner input04
Token intlit, value 23
Token +
Token intlit, value 18
Token -
Token intlit, value 45
Unrecognised character . on line 3
```

## 结论和下一步

我们从小处开始，有一个简单的词法分析器，可以识别四个主要的数学运算符以及整数字面值。我们看到我们需要跳过空白并在读取太多输入时放回字符。

单字符令牌很容易扫描，但多字符令牌有点难。但最终，`scan()` 函数在一个 `struct token` 变量中返回来自输入文件的下一个令牌：

```c
struct token {
  int token;
  int intvalue;
};
```

在我们编写编译器的旅程的下一部分中，我们将构建一个递归下降解析器，以解释输入文件的语法，并计算并打印出每个文件的最终值。[下一步](02-Parser.md)