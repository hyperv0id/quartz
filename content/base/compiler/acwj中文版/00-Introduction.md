# 第0部分：介绍

我决定踏上编写编译器的旅程。在过去，我曾经编写过一些[汇编器](https://github.com/DoctorWkt/pdp7-unix/blob/master/tools/as7)，并为一种无类型语言编写过一个[简单的编译器](https://github.com/DoctorWkt/h-compiler)。但我从未编写过一个能够编译自身的编译器。这就是我在这个旅程中的目标。

作为这个过程的一部分，我打算记录我的工作，以便其他人也能够跟随。这也将帮助我澄清我的思路和想法。希望你和我都会找到这个过程有用！

## 旅程的目标

以下是我对这个旅程的目标和非目标：

 + 编写一个自举编译器。我认为如果编译器能够编译自己，那么它就可以称为一个*真正的*编译器。
 + 针对至少一个真实的硬件平台。我见过一些为假设机器生成代码的编译器。我希望我的编译器可以在真实的硬件上运行。而且，如果可能的话，我希望编写编译器以支持为不同硬件平台生成代码的多个后端。
 + 实践优先于研究。在编译器领域有很多研究。我想从零开始这个旅程，所以我会采用实用的方法而不是理论重的方法。也就是说，在某些情况下，我将需要引入（并实现）一些基于理论的东西。
 + 遵循KISS原则：保持简单，愚蠢！我肯定会在这里使用肯·汤普森（Ken Thompson）的原则：“当犹豫不决时，使用蛮力。”
 + 采取许多小步骤来达到最终目标。我将把旅程分解为许多简单的步骤，而不是大跃进。这将使每次对编译器的新添加都成为一件容易消化的事情。

## 目标语言

选择目标语言是困难的。如果我选择像Python、Go等高级语言，那么我将不得不实现一堆库和类，因为它们是语言内置的。

我可以为Lisp之类的语言编写编译器，但这可以[轻松完成](ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-039.pdf)。

相反，我退回到了老旧的选择，并且我将为C的一个子集编写编译器，足以让编译器能够编译自身。

C仅仅是汇编语言的一步升级（对于C的某个子集来说，而不是[C18](https://en.wikipedia.org/wiki/C18_(C_standard_revision))），这将有助于将C代码编译成汇编语言的任务变得更加容易。噢，我也喜欢C。

## 编译器工作的基础知识

编译器的工作是将一种语言（通常是高级语言）的输入转换为不同输出语言（通常是低于输入语言的语言）。主要步骤包括：

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/25/parsing_steps-1ce16d.png)

 + 进行[词法分析](https://en.wikipedia.org/wiki/Lexical_analysis)以识别词法元素。在几种语言中，`=`与`==`不同，因此你不能只读取单个`=`。我们将这些词法元素称为*token*。
 + [解析](https://en.wikipedia.org/wiki/Parsing)输入，即识别输入的语法和结构元素，并确保它们符合语言的*语法*。例如，你的语言可能有这个决策结构：

```
      if (x < 23) {
        print("x is smaller than 23\n");
      }
```

> 但在另一种语言中，你可能会写成：

```
      if (x < 23):
        print("x is smaller than 23\n")
```

> 这也是编译器可以检测到语法错误的地方，比如第一个*print*语句末尾缺少分号。

 + 进行[语义分析](https://en.wikipedia.org/wiki/Semantic_analysis_(compilers))
   输入，即理解输入的含义。这实际上不同于识别语法和结构。例如，在英语中，句子可能具有形式`<主语> <动词> <形容词> <宾语>`。以下两个句子具有相同的结构，但含义完全不同：

```
          David ate lovely bananas.
          Jennifer hates green tomatoes.
```

 + [翻译](https://en.wikipedia.org/wiki/Code_generation_(compiler))
   输入的含义为另一种语言。在这里，我们将输入，逐部分，转换为较低级别的语言。

## 资源

互联网上有很多编译器资源。以下是我将要查看的资源。

### 学习资源

如果你想从一些关于编译器、解释器和运行时的书籍、论文和工具开始，我

强烈推荐这个列表：

  + [编译器、解释器和运行时的资源清单](https://github.com/aalhour/awesome-compilers) by Ahmad Alhour

### 现有的编译器

尽管我将编写自己的编译器，但我计划查看其他编译器以获取想法，并可能借用其中的一些代码。以下是我正在查看的编译器：

  + [SubC](http://www.t3x.org/subc/) by Nils M Holm
  + [Swieros C Compiler](https://github.com/rswier/swieros/blob/master/root/bin/c.c) by Robert Swierczek
  + [fbcc](https://github.com/DoctorWkt/fbcc) by Fabrice Bellard
  + [tcc](https://bellard.org/tcc/)，同样是由Fabrice Bellard和其他人编写的
  + [catc](https://github.com/yui0/catc) by Yuichiro Nakada
  + [amacc](https://github.com/jserv/amacc) by Jim Huang
  + [Small C](https://en.wikipedia.org/wiki/Small-C) by Ron Cain,
    James E. Hendrix, derivatives by others

特别是，我将使用来自SubC编译器的许多思想，以及其中的一些代码。

## 设置开发环境

假设你想参与这个旅程，以下是你需要的东西。我将使用Linux开发环境，因此下载并设置你最喜欢的Linux系统：我使用的是Lubuntu 18.04。

我将以两个硬件平台为目标：Intel x86-64和32位ARM。我将使用运行Lubuntu 18.04的PC作为Intel目标，以及运行Raspbian的树莓派作为ARM目标。

在Intel平台上，我们将需要一个现有的C编译器。
因此，请安装这个软件包（我提供Ubuntu/Debian命令）：

```
  $ sudo apt-get install build-essential
```

如果对于一个普通的Linux系统还需要更多的工具，请告诉我。

最后，克隆这个Github存储库的副本。

## 下一步

在我们编写编译器的旅程的下一部分中，我们将从扫描输入文件并找到语言的*token*的代码开始。[下一步](01-Scanner.md)