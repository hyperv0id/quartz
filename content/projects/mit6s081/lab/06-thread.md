# 实验：多线程

本实验将使您熟悉多线程。您将在用户级线程包中实现线程之间的切换，使用多个线程加速程序，并实现一个屏障。

在编写代码之前，您应确保已阅读《第7章：调度》的[xv6书籍](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf)并研究相应的代码。

要开始实验，请切换到线程分支：

```sh
git fetch
git checkout thread
make clean
```

## Uthread：线程之间的切换（中等难度）

在这个练习中，您将设计用户级线程系统的上下文切换机制，然后实现它。为了帮助您入门，您的xv6中有两个文件user/uthread.c和user/uthread_switch.S，以及Makefile中的一个规则来构建uthread程序。uthread.c包含了大部分用户级线程包的代码，以及三个简单测试线程的代码。线程包缺少一些创建线程和在线程之间切换的代码。

您的任务是制定一个计划来创建线程，并保存/恢复寄存器以在线程之间切换，并实现该计划。完成后，`make grade`应该显示您的解决方案通过了`uthread`测试。

完成后，在xv6上运行`uthread`时，您应该看到以下输出（三个线程可能以不同的顺序启动）：

```
$ make qemu
...
$ uthread
thread_a started
thread_b started
thread_c started
thread_c 0
thread_a 0
thread_b 0
thread_c 1
thread_a 1
thread_b 1
...
thread_c 99
thread_a 99
thread_b 99
thread_c: exit after 100
thread_a: exit after 100
thread_b: exit after 100
thread_schedule: no runnable threads
$
```

这个输出来自三个测试线程，每个线程都有一个循环，打印一行，然后将CPU让给其他线程。

然而，在这一点上，由于没有上下文切换的代码，您将看不到任何输出。

您需要在`user/uthread.c`的`thread_create()`和`thread_schedule()`以及`user/uthread_switch.S`的`thread_switch`中添加代码。一个目标是确保当`thread_schedule()`第一次运行给定线程时，该线程在自己的堆栈上执行传递给`thread_create()`的函数。另一个目标是确保`thread_switch`保存要切换的线程的寄存器，恢复要切换到的线程的寄存器，并返回到后者线程的指令中上次离开的位置。您需要决定在哪里保存/恢复寄存器；修改`struct thread`以保存寄存器是一个好的计划。您需要在`thread_schedule`中添加对`thread_switch`的调用；您可以传递任何您需要的参数给`thread_switch`，但意图是从线程`t`切换到`next_thread`。

一些建议：

- `thread_switch`只需要保存/恢复被调用者保存的寄存器。为什么？

- 您可以在user/uthread.asm中看到uthread的汇编代码，这对于调试可能很有用。

- 为了测试您的代码，逐步执行您的`thread_switch`可能会有所帮助，可以使用`riscv64-linux-gnu-gdb`。您可以通过以下方式开始：

```
(gdb) file user/_uthread
Reading symbols from user/_uthread...
(gdb) b uthread.c:60
```

这将在`uthread.c`的第60行设置一个断点。断点可能在您运行`uthread`之前就被触发了。这是如何发生的呢？

一旦您的xv6 shell运行起来，键入"uthread"，gdb将在第60行断下来。如果您从另一个进程中触发了断点，请继续执行，直到您在`uthread`进程中触发断点。现在，您可以键入以下命令来检查`uthread`的状态：

```
  (gdb) p/x *next_thread
```

使用"x"命令，您可以检查内存位置的内容：

```
  (gdb) x/x next_thread->stack
```

您可以通过以下方式跳转到`thread_switch`的开头：

```
   (gdb) b thread_switch
   (gdb) c
```

您可以使用以下命令逐步执行汇编指令：

```
   (gdb) si
```

有关gdb的在线文档可以在[这里](https://sourceware.org/gdb/current/onlinedocs/gdb/)找到。

## 使用线程（中等难度）

在这个任务中，您将使用线程和锁来探索并行编程，使用哈希表。您应该在一台真实的Linux或MacOS计算机上完成此任务（不是xv6，不是qemu），该计算机具有多个核心。大多数最新的笔记本电脑都配备了多核处理器。

此任务使用UNIX的`pthread`线程库。您可以从手册页中找到有关它的信息，使用`man pthreads`命令，您还可以在网上查找，例如[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_lock.html)、[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_init.html)和[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_create.html)。

文件`notxv6/ph.c`包含一个简单的哈希表，如果从单个线程中使用，它是正确的，但在从多个线程中使用时是错误的。在您的主要xv6目录（可能是`~/xv6-labs-2021`）中，输入以下命令：

```
$ make ph
$ ./ph 1
```

请注意，为了构建`ph`，Makefile使用的是您的操作系统的gcc，而不是6.1810工具。`ph`的参数指定在哈希表上执行put和get操作的线程数量。运行一段时间后，`ph 1`将产生类似于以下输出：

```
100000 puts, 3.991 seconds, 25056 puts/second
0: 0 keys missing
100000 gets, 3.981 seconds, 25118 gets/second
```

您看到的数字可能与此示例输出有所不同，差异可能超过两倍，这取决于您的计算机速度有多快，是否具有多个核心以及是否正在执行其他任务。

`ph`运行两个基准测试。首先，它通过调用`put()`向哈希表添加大量键，并打印每秒实现的插入速率。然后，它使用`get()`从哈希表中获取键。它打印应该作为puts结果而缺失的键的数量（在本例中为零），并打印它实现的每秒获取数。

您可以通过给`ph`传递大于1的参数，告诉它同时使用多个线程访问其哈希表。尝试`ph 2`：

```
$ ./ph 2
100000 puts, 1.885 seconds, 53044 puts/second
1: 16579 keys missing
0: 16579 keys missing
200000 gets, 4.322 seconds, 46274 gets/second
```

`ph 2`输出的第一行指示当两个线程同时向哈希表中添加条目时，它们的总插入速率达到53,044个插入/秒。这大约是运行`ph 1`时单个线程的两倍速率。这是一个很好的“并行加速”，大约为2倍，这是我们可以期望的最好情况（即两倍的核心产生两倍的工作量）。

然而，两行中的`16579 keys missing`表示应该在哈希表中的大量键却不在那里。也就是说，puts操作应该将这些键添加到哈希表中，但出现了问题。请查看`notxv6/ph.c`文件，特别是`put()`和`insert()`函数。

为什么使用2个线程会导致丢失键，而使用1个线程不会？在`answers-thread.txt`中找出两个线程可能导致键丢失的事件序列，并附上简短的解释。

为了避免这个事件序列，您需要在`notxv6/ph.c`文件的`put`和`get`函数中插入锁定和解锁语句，以便在使用两个线程时，丢失的键总数始终为0。相关的pthread调用如下：

```
pthread_mutex_t lock;            // declare a lock
pthread_mutex_init(&lock, NULL); // initialize the lock
pthread_mutex_lock(&lock);       // acquire lock
pthread_mutex_unlock(&lock);     // release lock
```

当`make grade`命令显示您的代码通过`ph_safe`测试（要求使用两个线程时没有丢失的键）时，您就完成了任务。此时`ph_fast`测试失败是可以接受的。

不要忘记调用`pthread_mutex_init()`。首先使用1个线程测试您的代码，然后再使用2个线程测试。它是否正确（即您是否消除了丢失的键）？相对于单线程版本，使用两个线程的版本是否实现了并行加速（即单位时间内的总工作量更多）？

在某些情况下，并发的`put()`操作在哈希表中读取或写入的内存没有重叠，因此它们不需要锁来相互保护。您能否更改`ph.c`以利用这种情况来获得某些`put()`操作的并行加速？提示：每个哈希桶使用一个锁如何？

修改您的代码，使得某些`put`操作在保持正确性的同时并行运行。当`make grade`命令显示您的代码通过`ph_safe`和`ph_fast`测试时，您就完成了任务。`ph_fast`测试要求两个线程的插入速率至少比一个线程的插入速率高1.25倍。

## 屏障（中等难度）

在这个任务中，您将实现一个[屏障](http://en.wikipedia.org/wiki/Barrier_(computer_science))：在应用程序中的一个点，所有参与的线程必须等待，直到所有其他参与的线程也到达该点。您将使用pthread条件变量，这是一种类似于xv6的sleep和wakeup的序列协调技术。

您应该在一台真实的计算机上完成此任务（不是xv6，不是qemu）。

文件`notxv6/barrier.c`包含一个有问题的屏障实现。

```
$ make barrier
$ ./barrier 2
barrier: notxv6/barrier.c:42: thread: Assertion `i == t' failed.
```

您需要实现期望的屏障行为。每个线程在循环中执行。在每次循环迭代中，线程调用`barrier()`，然后休眠一段随机的微秒数。断言触发是因为一个线程在另一个线程到达屏障之前离开了屏障。期望的行为是每个线程在调用`barrier()`后都会在`barrier()`中阻塞，直到所有`nthreads`个线程都调用了`barrier()`。

您的目标是实现期望的屏障行为。除了`ph`任务中见过的锁原语之外，您还需要以下新的pthread原语；请参考[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_wait.html)和[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_broadcast.html)获取详细信息。

```
pthread_cond_wait(&cond, &mutex);  // 在cond上进入睡眠状态，释放锁mutex，并在唤醒时重新获取锁
pthread_cond_broadcast(&cond);     // 唤醒所有在cond上睡眠的线程
```

确保您的解决方案通过`make grade`的`barrier`测试。

`pthread_cond_wait`在调用时会释放`mutex`，并在返回之前重新获取`mutex`。

我们已经为您提供了`barrier_init()`函数。您的任务是实现`barrier()`函数，以避免发生panic错误。我们已经为您定义了`struct barrier`结构体，其中的字段可供您使用。

有两个问题使您的任务变得复杂：

- 您需要处理一系列的屏障调用，我们将每个调用称为一轮（round）。`bstate.round`记录当前轮数。每当所有线程都到达屏障时，您应该增加`bstate.round`的值。
- 您需要处理一种情况，即一个线程在其他线程退出屏障之前就已经绕过循环了。特别地，您需要在每一轮中重新使用`bstate.nthread`变量。确保在上一轮仍在使用`bstate.nthread`时，离开屏障并绕过循环的线程不会增加`bstate.nthread`的值。

使用一个、两个和多于两个线程对您的代码进行测试。

## uthread的可选挑战

在几个方面，用户级线程包与操作系统的交互存在问题。例如，如果一个用户级线程在系统调用中阻塞，另一个用户级线程将无法运行，因为用户级线程调度器不知道其中一个线程已被xv6调度器取消调度。另一个例子是，两个用户级线程不会在不同的核心上并发运行，因为xv6调度器不知道存在可以并行运行的多个线程。请注意，如果两个用户级线程真正并行运行，由于存在多个竞争条件（例如，两个位于不同处理器上的线程可以同时调用`thread_schedule`，选择相同的可运行线程，并在不同处理器上同时运行它），这种实现将无法正常工作。

解决这些问题有几种方法。一种是使用[调度器激活](http://en.wikipedia.org/wiki/Scheduler_activations)，另一种是为每个用户级线程使用一个内核线程（就像Linux内核一样）。在xv6中实现其中一种方法。这并不容易实现正确；例如，当更新多线程用户进程的页表时，您需要实现TLB shootdown。

在您的线程包中添加锁、条件变量、屏障等功能。