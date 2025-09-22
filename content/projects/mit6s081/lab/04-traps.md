## 实验：自陷（机翻）

本实验探讨如何使用自陷实现系统调用。您将首先使用堆栈进行热身练习，然后将实现用户级陷阱处理的示例。

在开始编码之前，请阅读[xv6 书](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf)的第 4 章以及相关源文件：

-   `kernel/trampoline.S`：涉及从用户空间到内核空间并返回的程序集
-   `kernel/trap.c`：处理所有中断的代码

要开始实验，请切换到 trap 分支：

```sh
$ git fetch
$ git checkout traps
$ make clean
```

## RISC-V 汇编 ([简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html))

了解一些 RISC-V 汇编非常重要，您在 6.1910 (6.004) 中接触过它。您的 xv6 存储库中有一个文件 `user/call.c`。 `make fs.img`编译它并在`user/call.asm`中生成程序的可读汇编版本 。

阅读 `call.asm` 中函数`g`、`f`和`main`的代码。 RISC-V 的说明手册位于[参考页](https://pdos.csail.mit.edu/6.828/2023/reference.html)上。在`answers-traps.txt`中回答以下问题：

哪些寄存器包含函数的参数？例如，在 main 对`printf`的调用中哪个寄存器保存 13 ？

main 的汇编代码中对函数`f` 的调用在哪里？对`g`的调用在哪里？ （提示：编译器可能内联函数。）

`printf` 函数位于什么地址？

`main`中`jalr`到`printf` 之后的 寄存器`ra`中的值是什么？

运行以下代码。

```
unsigned int i = 0x00646c72；
printf("H%x Wo%s", 57616, &i);
```

输出是什么？ [这是一个](https://www.asciitable.com/)将字节映射到字符的 ASCII 表。

输出取决于 RISC-V 是小尾数这一事实。如果 RISC-V 是大端字节序，您会设置什么才能`i`产生相同的输出？您需要更改 `57616`为不同的值吗？

[这是对小端和大端的描述](http://www.webopedia.com/TERM/b/big_endian.html) 以及 [一个更异想天开的描述](https://www.rfc-editor.org/ien/ien137.txt)。

在下面的代码中，之后会打印什么 `'y='`？ （注：答案不是具体值。）为什么会出现这种情况？

```c
printf("x=%dy=%d", 3);
```

## 回溯（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

在调试过程中，回溯通常是非常有用的：回溯可以查看发生错误时堆栈上的函数调用列表。为了帮助进行回溯，编译器生成的机器代码会在堆栈上维护一个与当前调用链中每个函数相对应的堆栈帧。每个堆栈帧由返回地址和指向调用者堆栈帧的 "帧指针 "组成。寄存器 s0 包含指向当前栈帧的指针（实际上指向栈中保存的返回地址加 8 的地址）。反向跟踪应使用帧指针在堆栈中向上走动，并在每个堆栈帧中打印保存的返回地址。

在`kernel/printf.c`中 实现`backtrace()`函数。在`sys_sleep`中插入对此函数的调用，然后运行，这会调用`sys_sleep`。您的输出应该是具有此形式的返回地址列表（但数字可能会有所不同）： `bttest`

```
backtrace：
0x0000000080002cda
0x0000000080002bb6
0x0000000080002898
```

`bttest`退出 qemu 后。在终端窗口中：运行`addr2line -e kernel/kernel`（或`riscv64-unknown-elf-addr2line -e kernel/kernel`）并剪切并粘贴回溯中的地址，如下所示：

```
$addr2line -e kernel/kernel
0x0000000080002de2
0x0000000080002f4a
0x0000000080002bfc
Ctrl-D
```

你应该看到这样的东西：

```
kernel/sysproc.c:74
kernel/syscall.c:224
kernel/trap.c:85
```

一些提示：

- `将backtrace()`的原型添加到`kernel/defs.h`中，以便您可以在`sys_sleep`中调用`backtrace`。

- GCC编译器将当前正在执行的函数的帧指针存储在寄存器`s0`中。将以下函数添加到`kernel/riscv.h`中：

  ```c
  static inline uint64
  r_fp()
  {
    uint64 x;
    asm volatile("mv %0, s0" : "=r" (x) );
    return x;
  }
  ```

  并在`backtrace` 中调用该函数来读取当前帧指针。 `r_fp()`使用内联汇编来读取s0。

- 这些 [讲义](https://pdos.csail.mit.edu/6.1810/2023/lec/l-riscv.txt)有堆栈帧布局的图片。请注意，返回地址位于距堆栈帧的帧指针的固定偏移量 (-8) 处，并且保存的帧指针位于距帧指针的固定偏移量 (-16) 处。

- 你的`backtrace()`需要一种方法来识别它已经看到最后一个堆栈帧，并且应该停止。一个有用的事实是，为每个内核堆栈分配的内存由单个页面对齐的页面组成，因此给定堆栈的所有堆栈帧都位于同一页面上。您可以使用 `PGROUNDDOWN(fp)` （请参阅`kernel/riscv.h`）来识别帧指针引用的页面。

一旦你的回溯开始工作，就可以在`kernel/printf.c` 中`panic`中调用它，这样你就可以在发生`panic`时看到内核的回溯。

## 报警（[困难](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

在本练习中，您将为 xv6 添加一项功能，在进程占用 CPU 时间时定期发出警报。这可能对希望限制占用 CPU 时间的计算绑定进程，或希望计算但也希望采取某些定期行动的进程很有用。更一般地说，您将实现一种原始形式的用户级中断/故障处理程序；例如，您可以使用类似的方法来处理应用程序中的页面故障。如果您的解决方案通过了 alarmtest 和 "usertests -q"，那么它就是正确的。

您应该添加一个新的 `sigalarm(interval, handler) `系统调用。如果应用程序调用 `sigalarm(n,fn)`，那么程序每消耗 n 个 CPU 时间后，内核就会调用应用程序函数`fn`。当 `fn` 返回时，应用程序应继续运行。在 xv6 中，"tick "是一个相当随意的时间单位，由硬件定时器产生中断的频率决定。如果应用程序调用 `sigalarm(0,0)`，内核就应停止产生周期性警报调用。

您将在 xv6 资源库中找到 `user/alarmtest.c` 文件。将其添加到 `Makefile` 中。在添加 `sigalarm` 和 `sigreturn` 系统调用（见下文）之前，该文件无法正确编译。

`alarmtest` 在 `test0` 中调用 `sigalarm(2,periodic)`，要求内核每隔 2 个刻度强制调用 `periodic()`，然后旋转一段时间。您可以在 `user/alarmtest.asm` 中查看 `alarmtest` 的汇编代码，这可能有助于调试。如果 `alarmtest` 能产生这样的输出，并且 `usertests -q` 也能正确运行，那么您的解决方案就是正确的：

```
$ alarmtest
test0 start
........alarm!
test0 passed
test1 start
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
test1 passed
test2 start
................alarm!
test2 passed
test3 start
test3 passed
$ usertest -q
...
ALL TESTS PASSED
$
```

完成后，您的解决方案将只有几行代码，但要正确执行可能会很困难。我们将使用原始存储库中的`alarmtest.c` 版本来测试您的代码。您可以修改`alarmtest.c`来帮助您调试，但要确保原始`alarmtest`表明所有测试都通过。

### test0：调用处理程序

首先修改内核以跳转到用户空间中的警报处理程序，这将导致 `test0` 打印“alarm!”。不用担心“警报”之后会发生什么！输出;如果您的程序在打印“alarm!”后崩溃，现在没关系。以下是一些提示：

-   您需要修改 Makefile 以使`alarmtest.c` 编译为 xv6 用户程序。
-   放入`user/user.h`中的正确声明是：
    
    ```c
    int sigalarm(int ticks, void (*handler)());
    int sigreturn(void);
    ```
    
-   更新 `user/usys.pl`（生成 `user/usys.S`）、`kernel/syscall.h` 和 `kernel/syscall.c` 以允许`alarmtest`调用 `sigalarm` 和 `sigreturn` 系统调用。
-   目前，您的`sys\_sigreturn`应该只返回零。
-   您的`sys\_sigalarm()`应该将警报间隔和指向处理函数的指针存储在proc 结构中的新字段中（在`kernel/proc.h`中）。
-   您需要跟踪自上次调用进程的警报处理程序以来已经经过了多少个滴答声（或直到下一次调用为止）；为此，您还需要在`struct proc`中添加一个新字段。您可以在`proc.c`的`allocproc()` 中初始化proc字段。
-   每个时钟周期，硬件时钟都会强制产生一个中断，该中断在`kernel/trap.c`中的`usertrap()`中处理。
-   如果存在计时器中断，您只想操纵进程的警报滴答声；你可能需要类似的东西
    
    ```c
     if(which_dev == 2) ...
    ```
    
-   仅当进程有未完成的计时器时才调用警报函数。请注意，用户警报函数的地址可能为0（例如，在`user/alarmtest.asm` 中，periodic位于地址0）。
-   您需要修改 `usertrap()`，以便当进程的警报间隔到期时，用户进程执行处理程序函数。当 RISC-V 上的陷阱返回到用户空间时，什么决定了用户空间代码恢复执行的指令地址？
-   如果您告诉 qemu 仅使用一个 CPU，那么使用 gdb 查看陷阱会更容易，您可以通过运行
    
    ```sh
    make CPUS=1 qemu-gdb
    ```
    
-   如果`alarmtest`打印出“alarm!”，那么你就成功了。

### test1/test2()/test3()：恢复中断的代码

有可能alarmtest 在打印“alarm!”后在test0 或test1 中崩溃，或者alarmtest（最终）打印“test1 失败”，或者alarmtest 退出而不打印“test1 通过”。要解决此问题，必须确保当警报处理程序完成时，控制返回到用户程序最初被定时器中断中断的指令。您必须确保寄存器内容恢复到中断时所保存的值，以便用户程序在报警后可以不受干扰地继续运行。最后，您应该在每次警报计数器响起后“重新启动”警报计数器，以便定期调用处理程序。

作为起点，我们为您做出了一个设计决策：用户警报处理程序需要 在完成后调用`sigreturn`系统调用。看一下 `alarmtest.c`中的`periodic`示例。这意味着您可以将代码添加到`usertrap`和 `sys\_sigreturn`中，以配合使用户进程在处理警报后正确恢复。

一些提示：

-   您的解决方案将要求您保存和恢复寄存器——您需要保存和恢复哪些寄存器才能正确恢复中断的代码？ （提示：会很多）。
-   当计时器关闭时，让`usertrap`在`struct proc`中保存足够的状态 ，以便`sigreturn`可以正确返回到中断的用户代码。
-   防止对处理程序的重入调用——如果处理程序尚未返回，内核不应再次调用它。`test2 `对此进行测试。
-   确保恢复`a0`。 `sigreturn`是一个系统调用，它的返回值存储在`a0`中。

通过`test0、test1、test2和test3`后， 运行`usertests -q`以确保没有破坏内核的任何其他部分。

## 可选的挑战练习

-   在`backtrace()`中打印函数名称和行号，而不是数字地址（[hard](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）。