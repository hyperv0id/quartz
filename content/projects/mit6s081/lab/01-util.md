# 实验：Xv6和Unix实用工具

这个实验将使您熟悉 xv6 及其系统调用。

## 启动 xv6（[简易](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

您可以在 Athena 机器上或自己的计算机上进行这些实验。如果您使用自己的计算机，请查看 [实验工具页面](https://pdos.csail.mit.edu/6.828/2023/tools.html) 获取设置提示。

如果您使用 Athena，则必须使用 x86 机器；即，`uname -a` 应该显示 `i386 GNU/Linux` 或 `i686 GNU/Linux` 或 `x86_64 GNU/Linux`。您可以使用 `ssh -X athena.dialup.mit.edu` 登录到公共的 Athena 主机。我们已经在 Athena 上为您设置了适当的编译器和模拟器。要使用它们，请运行 `add -f 6.828`。您必须在每次登录时运行此命令（或将其添加到 `~/.environment` 文件中）。如果在编译或运行 `qemu` 时出现晦涩的错误，请检查是否添加了课程 locker。

获取实验的 xv6 源代码的 git 存储库：

```bash
$ git clone git://g.csail.mit.edu/xv6-labs-2023
Cloning into 'xv6-labs-2023'...
...
$ cd xv6-labs-2023
```

您将在这个实验和随后的实验分配中需要的文件是使用 [Git](http://www.git-scm.com/) 版本控制系统分发的。对于每个实验，您将签出为该实验定制的 xv6 版本。要了解有关 Git 的更多信息，请查看 [Git 用户手册](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html)，或者您可能会发现这个 [面向计算机科学家的 Git 概述](http://eagain.net/articles/git-for-computer-scientists/) 有用。Git 允许您跟踪对代码的更改。例如，如果您完成了一个练习，并希望保存您的进度，可以通过运行以下命令 *提交* 您的更改：

```bash
$ git commit -am 'my solution for util lab exercise 1'
Created commit 60d2135: my solution for util lab exercise 1
 1 files changed, 1 insertions(+), 0 deletions(-)
$
```

您可以使用 git diff 命令跟踪您的更改。运行 git diff 将显示自上次提交以来对代码的更改，而 git diff origin/util 将显示相对于初始 `util` 代码的更改。这里，`origin/util` 是此实验的 git 分支的名称。

构建和运行 xv6：

```bash
$ make qemu
riscv64-unknown-elf-gcc    -c -o kernel/entry.o kernel/entry.S
riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -DSOL_UTIL -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o kernel/start.o kernel/start.c
...
riscv64-unknown-elf-ld -z max-page-size=4096 -N -e main -Ttext 0 -o user/_zombie user/zombie.o user/ulib.o user/usys.o user/printf.o user/umalloc.o
riscv64-unknown-elf-objdump -S user/_zombie > user/zombie.asm
riscv64-unknown-elf-objdump -t user/_zombie | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$/d' > user/zombie.sym
mkfs/mkfs fs.img README  user/xargstest.sh user/_cat user/_echo user/_forktest user/_grep user/_init user/_kill user/_ln user/_ls user/_mkdir user/_rm user/_sh user/_stressfs user/_usertests user/_grind user/_wc user/_zombie
nmeta 46 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 1) blocks 954 total 1000
balloc: first 591 blocks have been allocated
balloc: write bitmap block at sector 45
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$
```

如果您在提示符处键入 `ls`，则应该看到类似以下的输出：

```bash
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2227
xargstest.sh   2 3 93
cat            2 4 32864
echo           2 5 31720
forktest       2 6 15856
grep           2 7 36240
init           2 8 32216
kill           2 9 31680
ln             2 10 31504
ls             2 11 34808
mkdir          2 12 31736
rm             2 13 31720
sh             2 14 54168
stressfs       2 15 32608
usertests      2 16 178800
grind          2 17 47528
wc             2 18 33816
zombie         2 19 31080
console        3 20 0
```

这些是 `mkfs` 在初始文件系统中包含的文件；其中大多数是您可以运行的程序。您刚才运行了其中之一：`ls`。

xv6 没有 `ps` 命令，但是，如果您键入 Ctrl-p，内核将打印关于每个进程的信息。如果您现在尝试，您将看到

两行：一个是 `init`，另一个是 `sh`。

要退出 qemu，请键入：Ctrl-a x（同时按下 Ctrl 和 a，然后按 x）。

## sleep ([简易](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))

为 xv6 实现一个用户级别的 `sleep` 程序，类似于 UNIX 的 `sleep` 命令。您的 `sleep` 应该暂停用户指定的时钟数。时钟滴答是由 xv6 内核定义的时间概念，即来自定时器芯片的两个中断之间的时间。您的解决方案应该在文件 `user/sleep.c` 中。

一些建议：

- 在开始编码之前，请阅读 [xv6 书](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf) 的第1章。
- 将您的代码放在 `user/sleep.c` 中。查看 `user/` 中的其他一些程序（例如 `user/echo.c`、`user/grep.c` 和 `user/rm.c`）以了解如何将命令行参数传递给程序。
- 在 Makefile 的 `UPROGS` 中添加您的 `sleep` 程序；一旦完成，`make qemu` 将编译您的程序，并可以从 xv6 shell 中运行它。
- 如果用户忘记传递参数，`sleep` 应该打印错误消息。
- 命令行参数以字符串形式传递；您可以使用 `atoi` 将其转换为整数（参见 `user/ulib.c`）。
- 使用系统调用 `sleep`。
- 查看 `kernel/sysproc.c` 获取实现 `sleep` 系统调用的 xv6 内核代码（查找 `sys_sleep`），查看 `user/user.h` 获取从用户程序调用的 `sleep` 的 C 定义，查看 `user/usys.S` 获取从用户代码跳转到内核进行 `sleep` 的汇编代码。
- `main` 函数应在完成时调用 `exit(0)`。
- 查看 Kernighan 和 Ritchie 的书 *The C Programming Language (second edition)*（K&R）以了解有关 C 语言的信息。

从 xv6 shell 运行程序：

```bash
$ make qemu
...
init: starting sh
$ sleep 10
(等待片刻，然后继续执行)
$
```

如果您的程序按上述方式暂停，则您的解决方案是正确的。运行 `make grade` 查看是否通过 `sleep` 测试。请注意，`make grade` 运行所有测试，包括下面的任务。如果要运行一个任务的评分测试，请键入：

```bash
$ ./grade-lab-util sleep
```

这将运行与 "sleep" 匹配的评分测试。或者，您可以键入：

```bash
$ make GRADEFLAGS=sleep grade
```

这也是一样的。

## pingpong ([简易](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))

编写一个使用 xv6 系统调用进行“乒乓”的用户级程序，在两个进程之间通过一对管道传递一个字节，每个方向一个管道。父进程应将一个字节发送到子进程；子进程应打印“<pid>：received ping”（其中 `<pid>` 是其进程 ID），将字节写入管道以传递给父进程，并退出；父进程应从子进程读取字节，打印“<pid>：received pong”并退出。您的解决方案应该在文件 `user/pingpong.c` 中。

一些建议：

- 在 Makefile 的 `UPROGS` 中添加程序。
- 使用 `pipe` 创建管道。
- 使用 `fork` 创建子进程。
- 使用 `read` 从管道读取，使用 `write` 向管道写入。
- 使用 `getpid` 查找调用进程的进程 ID。
- 在 xv6 中，用户程序有一组有限的库函数可用。您可以在 `user/user.h` 中查看列表；源代码（除了系统调用之外）在 `user/ulib.c`、`user/printf.c` 和 `user/umalloc.c` 中。

从 xv6 shell 运行程序，它应该产生以下输出：

```bash
$ make qemu
...
init: starting sh
$ pingpong
4: received ping
3: received pong
$
```

如果您的程序在两个进程之间交换一个字节并产生如上所示的输出，则您的解决方案是正确的。

## primes ([适中](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))/([困难](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))

使用管道和[此页面](http://swtch.com/~rsc/thread/)中下半部分所示的设计，为 xv6 编写一个并发的质数筛程序。这个想法归功于 Unix 管道的发明者 Doug McIlroy。您的解决方案应该在文件 `user/primes.c` 中。

您的目标是使用 `pipe` 和 `fork` 设置管道。第一个进程将数字 2 到 35 馈送到管道中。对于每个质数，您将安排创建一个从其左邻居通过一个管道读取并通过另一个管道写入其右邻居的进程。由于 xv6 的文件描述符和进程数量有限，第一个进程可以在 35 时停止。

一些建议：

- 谨慎关闭进程不需要的文件描述符，因为否则您的程序将在第一个进程达到 35 之前耗尽 xv6 资源。
- 一旦第一个进程达到 35，它应该等待整个管道终止，包括所有子进程、孙进程等。因此，主 primes 进程应仅在所有输出都已打印，并且所有其他 primes 进程都已退出之后退出。
- 提示：当管道的写侧关闭时，`read` 返回零。
- 直接将 

32 位（4 字节）的 `int` 写入管道最简单，而不是使用格式化的 ASCII I/O。
- 您应该只在需要时才创建管道中的进程。
- 在 Makefile 的 `UPROGS` 中添加程序。

如果您的解决方案实现了基于管道的筛法并产生了如下输出，则解决方案是正确的：

```bash
$ make qemu
...
init: starting sh
$ primes
prime 2
prime 3
prime 5
prime 7
prime 11
prime 13
prime 17
prime 19
prime 23
prime 29
prime 31
$
```

## find ([适中](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))

为 xv6 编写一个简单版本的 UNIX find 程序：查找目录树中所有具有特定名称的文件。您的解决方案应该在文件 `user/find.c` 中。

一些建议：

- 查看 `user/ls.c` 以了解如何读取目录。
- 使用递归使 `find` 能够进入子目录。
- 不要递归进入 "." 和 ".."。
- 在运行 qemu 之前，更改的文件系统会保留；要获得干净的文件系统，请运行 `make clean`，然后再运行 `make qemu`。
- 您需要使用 C 字符串。查看 K&R（C 语言书）的第 5.5 节，例如。
- 请注意，== 在 C 中不像在 Python 中那样比较字符串。请使用 `strcmp()` 代替。
- 在 Makefile 的 `UPROGS` 中添加程序。

如果解决方案产生以下输出（当文件系统包含文件 `b`、`a/b` 和 `a/aa/b` 时）则解决方案是正确的：

```bash
$ make qemu
...
init: starting sh
$ echo > b
$ mkdir a
$ echo > a/b
$ mkdir a/aa
$ echo > a/aa/b
$ find . b
./b
./a/b
./a/aa/b
$
```

## xargs ([适中](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))

为 xv6 编写一个简单版本的 UNIX xargs 程序：其参数描述要运行的命令，它从标准输入读取行，并为每一行运行该命令，将该行附加到命令的参数中。您的解决方案应该在文件 `user/xargs.c` 中。

以下示例说明了 xargs 的行为：

```bash
$ echo hello too | xargs echo bye
bye hello too
$
```

请注意，这里的命令是 "echo bye"，而附加的参数是 "hello too"，使命令变为 "echo bye hello too"，输出 "bye hello too"。

请注意，UNIX 上的 xargs 会进行一种优化，即一次性向命令传递多个参数。我们不期望您进行此优化。为了使 UNIX 上的 xargs 在本实验中表现出我们希望的行为，请使用 -n 选项设置为 1 运行它。例如：

```bash
$ (echo 1 ; echo 2) | xargs -n 1 echo
1
2
$
```

一些建议：

- 使用 `fork` 和 `exec` 来在每个输入行上调用命令。在父进程中使用 `wait` 等待子进程完成命令。
- 要读取输入的各行，请每次读取一个字符，直到出现换行符（'\n'）。
- `kernel/param.h` 声明了 `MAXARG`，如果需要声明 `argv` 数组，则可能会有用。
- 在 Makefile 的 `UPROGS` 中添加程序。
- 在运行 qemu 之前，更改的文件系统会保留；要获得干净的文件系统，请运行 `make clean`，然后再运行 `make qemu`。

xargs、find 和 grep 很好地结合在一起：

```bash
$ find . b | xargs grep hello
```

将在目录下的每个名为 b 的文件上运行 "grep hello"。

要测试 xargs 的解决方案，请运行 shell 脚本 `xargstest.sh`。如果解决方案产生以下输出，则解决方案是正确的：

```bash
$ make qemu
...
init: starting sh
$ sh < xargstest.sh
$ $ $ $ $ $ hello
hello
hello
$ $
```

您可能需要回去修复 find 程序中的错误。输出中有很多 `$` 是因为 xv6 shell 没有意识到它正在从文件而不是从控制台处理命令，因此对于文件中的每个命令都打印一个 `$`。

## 提交实验

### 花费时间

创建一个新文件，命名为 `time.txt`，并输入一个整数，表示你在实验上花费的小时数。使用 git add 和 git commit 提交这个文件。

### 答案

如果这个实验包含问题，将你的答案写在 `answers-*.txt` 中。使用 git add 和 git commit 提交这些文件。

### 提交

实验的提交由 Gradescope 处理。你需要一个 MIT gradescope 账户。查看 Piazza 获取加入课程的入口代码。如果需要更多帮助加入，请使用 [此链接](https://help.gradescope.com/article/gi7gm49peg-student-add-course#joining_a_course_using_a_course_code)。

当你准备好提交时，运行 `make zipball` 命令，它将生成 `lab.zip`。将此 zip 文件上传到相应的 Gradescope 作业中。

如果运行 `make zipball` 时存在未提交的更改或未跟踪的文件，你将看到类似以下输出：

```
 M hello.c
?? bar.c
?? foo.pyc
Untracked files will not be handed in.  Continue? [y/N]
```

检查上述行，并确保你的实验解决方案所需的所有文件都已被跟踪，即未列在以 `??` 开头的行中。你可以使用 `git add {filename}` 命令使 git 跟踪你创建的新文件。

- 请运行 `make grade` 确保你的代码通过所有测试。Gradescope 自动评分器将使用相同的评分程序为你的提交分配一个分数。
- 在运行 `make zipball` 之前提交任何修改过的源代码。
- 你可以在 Gradescope 上检查提交的状态并下载已提交的代码。Gradescope 实验成绩将是你的最终实验成绩。

## 可选挑战练习

- 编写一个打印以 ticks 为单位的运行时间的程序，使用 `uptime` 系统调用。([简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))
- 为 `find` 支持正则表达式的名称匹配。`grep.c` 对正则表达式有一些基本支持。([简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))
- xv6 shell (`user/sh.c`) 只是另一个用户程序，你可以改进它。它是一个简化的 shell，缺少真正的 shell 中常见的许多功能。例如，修改 shell 以在处理来自文件的 shell 命令时不打印 `$`（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)），修改 shell 以支持等待（[简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)），修改 shell 以支持由 ";" 分隔的命令列表（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)），修改 shell 以支持通过实现 "(" 和 ")" 进行子 shell（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)），修改 shell 以支持制表符补全（[简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)），修改 shell 以保存已传递的 shell 命令的历史记录（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)），或者任何你希望你的 shell 完成的其他任务。（如果你非常雄心勃勃，可能需要修改内核以支持你需要的内核功能；xv6 支持有限。）