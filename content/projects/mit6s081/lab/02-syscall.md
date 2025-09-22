- [Link](https://pdos.csail.mit.edu/6.828/2023/labs/syscall.html)
- [Answer](./answers/02-syscall.md)
- See Also：
	- gdb常用命令：[gdb命令备忘](../../../../tools/cheatsheets/gdb命令备忘.md)
	- 如果不想/不会使用gdb的可以看这个：[从零开始使用Vscode调试XV6 | 知乎](https://zhuanlan.zhihu.com/p/501901665)

# 实验：系统调用

在上一个实验中，你使用系统调用编写了一些实用程序。在这个实验中，你将向 xv6 添加一些新的系统调用，这将帮助你理解它们的工作原理，并让你了解 xv6 内核的一些内部情况。你将在后续的实验中添加更多的系统调用。

在开始编码之前，请阅读 [xv6 书籍](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev1.pdf) 的第 2 章以及第 4 章的 4.3 和 4.4 节，以及相关的源文件：

- 将系统调用路由到内核的用户空间“stubs”位于 `user/usys.S` 中，由 `user/usys.pl` 在运行 `make` 时生成。声明在 `user/user.h` 中。
- 将系统调用路由到实现它的内核函数的内核空间代码位于 `kernel/syscall.c` 和 `kernel/syscall.h` 中。
- 与进程相关的代码在 `kernel/proc.h` 和 `kernel/proc.c` 中。

要开始实验，请切换到 syscall 分支：

```
$ git fetch
$ git checkout syscall
$ make clean
```

如果运行 `make grade`，你会发现评分脚本无法执行 `trace` 和 `sysinfotest`。你的任务是添加必要的系统调用和存根，使它们正常工作。

## 使用 gdb ([简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))

在许多情况下，使用打印语句就足以调试内核，但有时单步执行一些汇编代码或检查堆栈上的变量是有帮助的。

要了解有关如何运行 GDB 以及在使用 GDB 时可能出现的常见问题，请查看 [此页面](https://pdos.csail.mit.edu/6.828/2023/labs/gdb.html)。

为了帮助你熟悉 gdb，请运行 `make qemu-gdb`，然后在另一个窗口中启动 gdb（参见 [指南页面](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html) 上的 gdb 项目）。在打开两个窗口后，在 gdb 窗口中输入以下内容：

```
(gdb) b syscall
Breakpoint 1 at 0x80002142: file kernel/syscall.c, line 243.
(gdb) c
Continuing.
[Switching to Thread 1.2]

Thread 2 hit Breakpoint 1, syscall () at kernel/syscall.c:243
243     {
(gdb) layout src
(gdb) backtrace
```

`layout` 命令将窗口分为两部分，显示 gdb 在源代码中的位置。`backtrace` 打印出堆栈回溯。查看 [使用 GNU 调试器](https://pdos.csail.mit.edu/6.828/2019/lec/gdb_slides.pdf) 获取有用的 GDB 命令。

> [!question]
> 在查看回溯输出时，哪个函数调用了 `syscall`？
> > [!done]- Ans
> > `usertrap()` in `trap.c`




按下 n 几次，跳过 `struct proc *p = myproc();`。在跳过此语句后，输入 `p /x *p`，这会以十六进制打印当前进程的 `proc struct`（参见 `kernel/proc.h>`）。

`p->trapframe->a7` 的值是多少，该值代表什么？（提示：查看 `user/initcode.S`，xv6 启动的第一个用户程序。）

处理器正在内核模式下运行，我们可以打印特权寄存器，如 `sstatus`（参见 [RISC-V 特权指令](https://github.com/riscv/riscv-isa-manual/releases/download/Priv-v1.12/riscv-privileged-20211203.pdf) 进行说明）：

```
(gdb) p /x $sstatus
```

> [!question]
> CPU 先前处于哪种模式？
> > [!done]- Ans
> > ```
> > $1 = 0x200000022
> > ```
> > which means 超级用户模式
> > ```
> >  63  62  61  60  59  58  57  56  55  54  53  52  51  50  49  48
> > +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
> > | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | FS| XS| SD| WP| UPI|UPIE|SIE|UIE|
> > +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
 > > 47  46  45  44  43  42  41  40  39  38  37  36  35  34  33  32
> > +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
> > | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | XS| XS| XS| XS| XS| XS| XS| XS|
> > +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
> > ```
> > - `XS` (eXtended Supervisor mode) 位（12-13位）为 2，表示超级用户模式。



在此实验的后续部分（或在后续的实验中），可能会发生编程错误，导致 xv6 内核崩溃。例如，将语句 `num = p->trapframe->a7;` 替换为 `num = * (int *) 0;`，在 `syscall` 开头运行 `make qemu`，你将看到类似以下的内容：

```
xv6 kernel is booting

hart 2 starting
hart 1 starting
scause 0x000000000000000d
sepc=0x000000008000215a stval=0x0000000000000000
panic: kerneltrap
```

退出 `qemu`。

要追踪引起内核页面故障崩溃的源头，请在文件 `kernel/kernel.asm` 中搜索刚刚看到的崩溃的 `sepc` 值，该文件包含已编译内核的汇编代码。

> 写下内核崩溃的源头的汇编指令。哪个寄存器对应于变量 `num`？

要检查处理器和内核在故障指令处的状态，请启动 gdb，并在故障的 `epc` 处设置断点，如下所示：

```
(gdb) b *0x000000008000215a
Breakpoint 1 at 0x8000215a: file kernel/syscall.c, line 247.
(gdb) layout asm
(gdb) c
Continuing.
[Switching to Thread 1.3]

Thread 3 hit Breakpoint 1, syscall () at kernel/syscall.c:247
```

确认出现故障的汇编指令与上面找到的指令相同。

> [!question]
> 内核为什么崩溃？
> 提示：查看文本中的图 3-3；内核地址空间中是否映射了地址 0？`scause` 的值是否确认了这一点？（查看 [RISC-V 特权指令](https://pdos.csail.mit.edu/6.828/2023/labs/n//github.com/riscv/riscv-isa-manual/releases/download/Priv-v1.12/riscv-privileged-20211203.pdf)）


请注意，`scause` 是由上述内核慌乱打印出来的，但通常需要查看更多信息才能找出导致慌乱的问题。例如，要找出内核慌乱时运行的用户进程，可以打印出该进程的名称：

    (gdb) p p->name


> [!question]
> 内核崩溃时正在运行的二进制进程的名称是什么？进程 id (pid) 是多少？


使用 gdb 跟踪错误的简要介绍到此结束；在跟踪内核错误时，值得花时间重温一下《使用 GNU 调试器》（Using the GNU Debugger）。指导页面还有其他一些有用的调试技巧。

## System call tracing ([中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))

在本作业中，您将添加一项系统调用跟踪功能，这可能会在以后的实验调试中有所帮助。您将创建一个新的跟踪系统调用来控制跟踪。它应该接受一个参数，即一个整数 "掩码"，该掩码的位数指定了要跟踪的系统调用。例如，要跟踪 `fork` 系统调用，程序会调用 `trace(1 << SYS_fork)`，其中 `SYS_fork` 是 `kernel/syscall.h` 中的系统调用编号。你必须修改 xv6 内核，以便在每个系统调用即将返回时，如果掩码中设置了系统调用编号，就打印出一行。该行应包含进程 ID、系统调用名称和返回值；无需打印系统调用参数。跟踪系统调用应启用对调用该调用的进程及其随后分叉的子进程的跟踪，但不应影响其他进程。

我们提供了一个跟踪用户级程序，可在启用跟踪后运行另一个程序（参见 `user/trace.c`）。运行完成后，您将看到如下输出：

```sh
$ trace 32 grep hello README
3: syscall read -> 1023
3: syscall read -> 966
3: syscall read -> 70
3: syscall read -> 0
$
$ trace 2147483647 grep hello README
4: syscall trace -> 0
4: syscall exec -> 3
4: syscall open -> 3
4: syscall read -> 1023
4: syscall read -> 966
4: syscall read -> 70
4: syscall read -> 0
4: syscall close -> 0
$
$ grep hello README
$
$ trace 2 usertests forkforkfork
usertests starting
test forkforkfork: 407: syscall fork -> 408
408: syscall fork -> 409
409: syscall fork -> 410
410: syscall fork -> 411
409: syscall fork -> 412
410: syscall fork -> 413
409: syscall fork -> 414
411: syscall fork -> 415
...
$
```

在上面的第一个示例中，trace 调用 grep，只追踪了 read 系统调用。32 是 `1 << SYS_read`。在第二个示例中，trace 在追踪所有系统调用的同时运行 grep；2147483647 具有所有 31 个低位设置的值。在第三个示例中，程序没有被追踪，因此不会打印任何追踪输出。在第四个示例中，追踪了`usertests` 中 `forkforkfork` 测试的所有后代的 fork 系统调用。如果你的程序行为如上所示（尽管进程 ID 可能不同），则你的解决方案是正确的。

一些建议：

- 在 Makefile 的 UPROGS 中添加 `$U/_trace`。
- 运行 `make qemu`，你会看到编译器**无法编译** `user/trace.c`，因为尚不存在系统调用的用户空间存根：

	1. 在 `user/user.h` 中添加一个系统调用的原型，
	2. 在 `user/usys.pl` 中添加一个存根，
	3. 在 `kernel/syscall.h` 中添加一个系统调用号。
	4. Makefile 调用 perl 脚本 `user/usys.pl`，它生成 `user/usys.S`，实际的系统调用存根，这些存根使用 RISC-V 的 `ecall` 指令切换到内核。
	5. 一旦解决了编译问题，请运行 `trace 32 grep hello README`；
	6. 它会**失败**，因为你尚未在内核中实现该系统调用。
- 在 `kernel/sysproc.c` 中添加一个 `sys_trace()`函数，通过在 `proc` 结构（参见 `kernel/proc.h`）中的一个**新变量**中记住参数来实现新的系统调用。从用户空间获取系统调用参数的函数在 `kernel/syscall.c` 中，你可以在 `kernel/sysproc.c` 中看到使用这些函数的示例。
- 修改 `fork()`（参见 `kernel/proc.c`），将追踪掩码从父进程复制到子进程。
- 修改 `kernel/syscall.c` 中的 `syscall()` 函数以打印追踪输出。你需要添加一个系统调用名称数组进行索引。
- 如果在直接在 qemu 中运行测试用例时通过，但在使用 `make grade` 运行测试时超时，请尝试在 Athena 上测试你的实现。这个实验中的一些测试对于本地机器来说可能有点计算密集（特别是如果你使用 WSL）。



## Sysinfo（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

在这个任务中，你将添加一个系统调用 `sysinfo`，用于收集关于运行系统的信息。这个系统调用接受一个参数：一个指向 `struct sysinfo` 的指针（参见 `kernel/sysinfo.h`）。内核应该填充这个结构的字段：`freemem` 字段应该设置为空闲内存的字节数，而 `nproc` 字段应该设置为其 `state` 不是 `UNUSED` 的进程数量。我们提供了一个测试程序 `sysinfotest`；如果它打印出 "sysinfotest: OK"，则你通过了这个任务。

一些建议：

- 在 Makefile 的 UPROGS 中添加 `$U/_sysinfotest`。

- 运行 `make qemu`；`user/sysinfotest.c` 将无法编译。添加系统调用 sysinfo，按照前一个任务中的相同步骤进行。为了在 `user/user.h` 中声明 sysinfo() 的原型，你需要预先声明 `struct sysinfo` 的存在：

```c
struct sysinfo;
int sysinfo(struct sysinfo *);
```

  一旦解决了编译问题，运行

```sh
sysinfotest
```

  它将失败，因为你尚未在内核中实现该系统调用。

- sysinfo 需要将 `struct sysinfo` 复制回用户空间；参见 `sys_fstat()`（`kernel/sysfile.c`）和 `filestat()`（`kernel/file.c`）以了解如何使用 `copyout()` 完成这个操作的示例。

- 要收集空闲内存的数量，向 `kernel/kalloc.c` 添加一个函数。

- 要收集进程数量，向 `kernel/proc.c` 添加一个函数。

## 可选挑战练习

- 打印被跟踪系统调用的系统调用参数（[简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）。
- 计算负载平均值并通过 sysinfo 导出它（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）。