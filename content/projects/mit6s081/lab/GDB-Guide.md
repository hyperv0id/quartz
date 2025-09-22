# 在Athena上运行GDB的步骤

在第一个窗口：

1. `ssh <你的Kerberos账号>@athena.dialup.mit.edu`
2. `add -f 6.828`
3. `git clone git://g.csail.mit.edu/xv6-labs-2023`（如果已经克隆了实验仓库，则忽略此步骤）
4. `cd xv6-labs-2023`
5. `make qemu-gdb`
6. 注意前一条命令的输出的最后一行。它应该显示类似于 `tcp::<端口号>` 的内容。例如，可能会看到 `tcp::26000`

在第二个窗口：

1. `ssh <你的Kerberos账号>@athena.dialup.mit.edu`
2. `add -f 6.828`
3. `cd xv6-labs-2023`
4. `riscv64-unknown-elf-gdb`
5. 在gdb提示符内：`target remote localhost:<上述步骤6中的端口号>`

假设你想在内核进入 `kernel/syscall.c` 中的 `syscall` 函数时每次都中断：

1. 在gdb提示符内：`file kernel/kernel`（这是包含所有内核代码的二进制文件）
2. 在gdb提示符内：`b syscall`
3. 按下 `c` 键。此时你将开始触发上面的断点
4. 不断按 `c` 键，查看内核何时调用 `syscall` 函数。你将看到第一个窗口中的输出是如何进行的。

现在，假设你想在 `user/ls.c` 中的 `ls` 函数中中断。那么，在步骤6中，你需要运行 `file user/_ls`，因为这是包含该函数的二进制文件的名称。在步骤7中，你还需要运行 `b ls`。

# 运行GDB时的常见问题

1. 从错误的目录运行GDB。你需要在 `xv6-labs-2023` 目录内运行 `make qemu-gdb` 和 `riscv64-unknown-elf-gdb`（或替代的 `gdb-multiarch`）。不要在 `~` 目录、`xv6-labs-2023/kernel` 或 `xv6-labs-2023/user` 目录中运行。
2. 忘记添加Athena工具的Athena锁。参见步骤2（`add -f 6.828`）。这仅在你在Athena上时相关。
3. 运行错误类型的gdb。如果只运行 `gdb`，它将无法理解二进制文件所在的机器代码（你的计算机/Athena 正在使用 x86，而二进制文件 `kernel/kernel` 正在使用 RISCV）。你需要运行 `riscv64-unknown-elf-gdb` 或替代的 `gdb-multiarch`。如果在Athena上，请运行 `riscv64-unknown-elf-gdb`。
4. gdb无法找到你要调试的内容。你可能需要运行 `target remote localhost:<上述步骤6中的端口号>`，即使在Athena上运行也是如此。
5. 没有添加要调试的文件。如果出现关于运行 `file` 命令或未知符号的错误，请运行 `file kernel/kernel`，以便gdb知道在何处查找要调试的代码。
6. 如果退出GDB，然后重新运行 `gdb-multiarch`（或 `riscv64-unknown-elf-gdb`）而没有重新运行 `make qemu-gdb`，可能会发生奇怪的情况。大多数情况下它可以工作，但有时候不行，然后你只需重新启动两者并重新开始。这可能表现为GDB对你的请求不响应的行为。
7. 如果使用Athena，请确保两个窗口在同一台Athena机器上。你可以通过查看每个提示行的开头来查找你在哪个机器上，它将显示 `<你的Kerberos账号>@<Athena机器名称>`。如果在将两个窗口放在同一台机器上遇到问题，可以查看 [tmux](https://www.ocf.berkeley.edu/~ckuehl/tmux/)。
8. 不使用GDB来帮助你调试！我知道一开始使用GDB很烦人，但在后面的实验中它非常非常有用，我们希望你们都了解基本的命令，以减轻未来调试的痛苦。

## 调试提示

以下是调试解决方案的一些提示：

- 确保你理解C语言和指针。Kernighan和Ritchie的书《C程序设计语言（第二版）》对C语言进行了简明扼要的描述。查看这个例子 [code](https://pdos.csail.mit.edu/6.828/2019/lec/pointers.c)，确保你理解它为什么产生这样的结果。

  有一些常见的指针习惯用法特别值得记住：

  - 如果 `int *p = (int*)100`，那么 `(int)p + 1` 和 `(int)(p + 1)` 是不同的数字：第一个是 `101`，但第二个是 `104`。在将整数添加到指针时，如第二种情况，整数隐式乘以指针指向的对象的大小。
  - `p[i]` 被定义为与 `*(p+i)` 相同，指的是指针p指向的内存中的第i个对象。上述加法规则在对象大于一个字节时很有帮助。
  - `&p[i]` 与 `(p+i)` 相同，得到指针p指向的内存中第i个对象的地址。

  虽然大多数C程序不需要在指针和整数之间进行转换，但操作系统经常需要。每当你看到涉及内存地址的加法时，请自己问一下它是整数加法还是指针加法，并确保要添加的值是否适当地乘以或不乘以对象的大小。

- 如果你的练习部分工作，通过提交你的代码进行进度检查。如果以后出现问题，你可以回滚到检查点并以更小的步骤前进。要了解更多关于Git的信息，查看 [Git用户手册](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html)，或者你可能会发现这个 [面向计算机科学家的Git概述](http://eagain.net/articles/git-for-computer-scientists/) 有用。

- 如果测试失败，请确保你理解为什么你的代码未通过测试。插入打印语句直到你理解发生了什么。

- 你可能会发现你的打印语句产生了大量输出，你希望通过搜索进行查找；一种方法是在 `script` 内运行 `make qemu`（在你的机器上运行 `man script`），它将所有控制台输出记录到文件中，然后你可以搜索该文件。不要忘记退出 `script`。

- 打印语句通常是强大的调试工具，但有时能够单步执行一些汇编代码或检查堆栈上的变量是有帮助的。要使用gdb调试xv6，请在一个窗口中运行 `make qemu-gdb`，在另一个窗口中运行gdb-multiarch（或riscv64-linux-gnu-gdb），设置断点，然后 'c'（继续），xv6将运行直到达到断点。参见 [使用GNU调试器](https://pdos.csail.mit.edu/6.828/2019/lec/gdb_slides.pdf) 获取有关GDB的有用提示。（如果你启动gdb并看到类似 'warning: File ".../.gdbinit" auto-loading has been declined' 的警告，请编辑 ~/.gdbinit 添加 "add-auto-load-safe-path..."，如警告所建议的。）

- 如果你想查看编译器为xv6内核生成的汇编代码，或找出特定内核地址处的指令是什么，请查看文件 `kernel/kernel.asm`，这是Makefile在编译内核时生成的。 （Makefile还为所有用户程序生成 `.asm`。）

- 如果内核导致意外错误（例如使用无效内存地址），它将打印包含崩溃点处程序计数器（"sepc"）的错误消息；你可以搜索 `kernel.asm` 找到包含该程序计数器的函数，或者你可以运行 `addr2line -e kernel/kernel *pc-value*` （运行 `man addr2line` 了解详情）。如果要获取回溯信息，请使用gdb重新启动：在一个窗口中运行 'make qemu-gdb'，在另一个窗口中运行gdb（或riscv64-linux-gnu-gdb），在panic（'b panic'）中设置断点，然后 'c' 继续。当内核达到断点时，键入 'bt' 获取回溯。

- 如果你的内核挂起，可能是由于死锁，你可以使用gdb找出它挂起的位置。在一个窗口中运行 'make qemu-gdb'，在另一个窗口中运行gdb（riscv64-linux-gnu-gdb），然后 'c' 继续。当内核似乎挂起时，在qemu-gdb窗口中按Ctrl-C并键入 'bt' 以获取回溯。

- `qemu` 有一个"监视器"，可以查询模拟机的状态。你可以通过键入 `control-a c`（"c"代表控制台）来访问它。一个特别有用的监视器命令是 `info mem` 用于打印页表。你可能需要使用 `cpu` 命令选择 `info mem` 查看的核心，或者你可以使用 `make CPUS=1 qemu` 启动qemu 以使只有一个核心。