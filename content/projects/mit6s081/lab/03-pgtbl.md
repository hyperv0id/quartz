# 实验：页表(机翻)

在本实验中，您将探索页表并修改它们以加速某些系统调用并检测哪些页面已被访问。

在开始编码之前，请阅读[xv6 书](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf)的第 3 章以及相关文件：

- `kernel/memlayout.h`，捕获内存布局。
- `kernel/vm.c`，其中包含大部分虚拟内存 (VM) 代码。
- `kernel/kalloc.c`，其中包含分配和释放物理内存的代码。

查阅[RISC-V 特权架构手册](https://github.com/riscv/riscv-isa-manual/releases/download/Ratified-IMFDQC-and-Priv-v1.11/riscv-privileged-20190608.pdf)也可能有所帮助。

要启动实验，请切换到 pgtbl 分支：

```sh
git fetch
git checkout pgtbl
make clean
```

## 加快系统调用速度（[简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

一些操作系统（例如Linux）通过在用户空间和内核之间的只读区域中共享数据来加速某些系统调用。这消除了执行这些系统调用时状态切换的需要。为了帮助您学习如何将映射插入页表，您的第一个任务是为xv6 中的`getpid()`系统调用实现此优化。

创建每个进程时，在 `USYSCALL` （ `memlayout.h` 中定义的虚拟地址）映射一个只读页面。在此页面的开头，存储一个`struct usyscall` （也在`memlayout.h`中定义），并初始化它以存储当前进程的 PID。对于本实验，`ugetpid()`已在用户空间端提供，并将自动使用 USYSCALL 映射。如果运行`pgtbltest`时`ugetpid`测试用例通过，您将获得本实验部分的全部学分。

一些提示：

- 选择允许用户空间仅读取页面的权限位。(2023)
- 在新页面的生命周期中需要完成一些事情。为了获得灵感，请了解`kernel/proc.c`中的 `trapframe` 处理。(2023)

使用此共享页面可以使其他哪些 xv6 系统调用更快？解释一下怎么做。

> 2023相比2021缺少了很多提示
>
> - 您可以在 `kernel/proc.c` 中的 `proc_pagetable()` 中执行映射。
> - 选择只允许用户空间读取页面的权限位。
> - 你可能会发现 `mappages()` 是一个非常有用的工具。
> - 不要忘记在 `allocproc()` 中分配和初始化页面。
> - 确保在 `freeproc()` 中释放页面。
>

## 打印页表（[简单](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

为了帮助您可视化 RISC-V 页表，并且可能有助于将来的调试，您的第二个任务是编写一个打印页表内容的函数。

`定义一个名为vmprint()` 的函数。它应该采用`pagetable_t`参数，并以下面描述的格式打印该页表。在 `exec.c` 中`return argc`之前插入`if(p->pid==1) vmprint(p->pagetable)`，以打印第一个进程的页表。如果您通过了`make Grade`的`PTE 打印输出`测试，您将获得本部分实验的满分。

`现在，当您启动 xv6 时，它应该` 打印如下输出，描述第一个进程刚刚完成`exec() init`时的页表：

```
page table 0x0000000087f6b000
 ..0: pte 0x0000000021fd9c01 pa 0x0000000087f67000
 .. ..0: pte 0x0000000021fd9801 pa 0x0000000087f66000
 .. .. ..0: pte 0x0000000021fda01b pa 0x0000000087f68000
 .. .. ..1: pte 0x0000000021fd9417 pa 0x0000000087f65000
 .. .. ..2: pte 0x0000000021fd9007 pa 0x0000000087f64000
 .. .. ..3: pte 0x0000000021fd8c17 pa 0x0000000087f63000
 ..255: pte 0x0000000021fda801 pa 0x0000000087f6a000
 .. ..511: pte 0x0000000021fda401 pa 0x0000000087f69000
 .. .. ..509: pte 0x0000000021fdcc13 pa 0x0000000087f73000
 .. .. ..510: pte 0x0000000021fdd007 pa 0x0000000087f74000
 .. .. ..511: pte 0x0000000020001c0b pa 0x0000000080007000
init: starting sh
```

`第一行显示vmprint` 的参数。之后，每个 PTE 都有一行，包括引用树中更深层次的页表页面的 PTE。每条 PTE 行都缩进了一些`“..”`，表示其在树中的深度。每条 PTE 行显示其页表页中的 PTE 索引、pte 位以及从 PTE 中提取的物理地址。不要打印无效的 PTE。在上面的示例中，顶级页表页具有条目 0 和 255 的映射。条目 0 的下一级仅映射索引 0，而该索引 0 的底层具有条目 0、1 和 255。 2 映射。

您的代码可能会发出与上面显示的不同的物理地址。条目数和虚拟地址数应该相同。

一些提示：

- 您可以将`vmprint()`放在`kernel/vm.c`中。
- 使用文件 `kernel/riscv.h` 末尾的宏。
- `freework`函数可能很有用。
- `在 kernel/defs.h 中定义vmprint`的原型，以便可以从 `exec.c` 调用它。
- 在 `printf` 调用中使用`%p`打印出完整的 64 位十六进制 `PTE` 和地址，如示例中所示。

对于`vmprint`输出中的每个叶页，解释其逻辑上包含的内容及其权限位。 xv6 书中的图 3.4 可能会有所帮助，但请注意，该图的页面集可能与 此处检查的 `init进程略有不同。`

## 检测哪些页面已被访问（[困难](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

一些垃圾收集器（自动内存管理的一种形式）可以从有关哪些页面已被访问（读取或写入）的信息中受益。在实验的这一部分中，您将向 xv6 添加一项新功能，通过检查 RISC-V 页表中的访问位来检测此信息并将其报告给用户空间。每当 RISC-V 硬件页遍历器解决 TLB 未命中问题时，它就会在 PTE 中标记这些位。

您的工作是实现`pgaccess()`，这是一个报告哪些页面已被访问的系统调用。系统调用需要三个参数。首先，它需要检查第一个用户页面的起始虚拟地址。其次，需要检查页数。最后，它将用户地址发送到缓冲区，以将结果存储到位掩码（每页使用一位的数据结构，其中第一页对应于最低有效位）。如果运行`pgtbltest`时`pgaccess`测试用例通过，您将获得这部分实验的全部学分。

一些提示：

- 阅读`user/pgtbltest.c`中的`pgaccess_test()`来了解`pgaccess`是如何使用的。
- 首先在`kernel/sysproc.c`中实现`sys_pgaccess()`。
- 您需要使用`argaddr()`和`argint()`解析参数。
- 对于输出位掩码，更容易在内核中存储临时缓冲区，并在用正确的位填充后将 其复制给用户（通过`copyout() `
- 可以设置可扫描页数的上限。
- `kernel/vm.c`中的`walk()`对于查找正确的 PTE 非常有用。
- 您需要在`kernel/riscv.h`中定义`PTE_A`（访问位 Access）。请查阅[RISC-V 特权架构手册](https://github.com/riscv/riscv-isa-manual/releases/download/Ratified-IMFDQC-and-Priv-v1.11/riscv-privileged-20190608.pdf)以确定其值。
- 确保在检查`PTE_A`是否设置后将其清除。否则，将无法确定自上次调用`pgaccess()`以来该页面是否被访问过（即，该位将永远被设置）。
- `vmprint()`可能会在调试页表时派上用场。

## 可选的挑战练习

- 使用超级页面来减少页表中 PTE 的数量。
- 取消映射用户进程的第一页，以便取消引用空指针将导致错误。您必须更改`user.ld`以从 4096 而不是 0 开始用户文本段。
- `添加使用PTE_D`报告脏页（修改页）的系统调用。