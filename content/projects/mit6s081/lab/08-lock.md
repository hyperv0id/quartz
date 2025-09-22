# 实验：锁

在本实验中，您将获得重新设计代码以提高并行性的经验。多核机器上并行性差的一个常见症状是高锁争用。提高并行性通常涉及更改数据结构和锁定策略以减少争用。您将为 xv6 内存分配器和块缓存执行此操作。

在编写代码之前，请务必阅读[xv6 书中](https://pdos.csail.mit.edu/6.828/2023/xv6/book-riscv-rev3.pdf)的以下部分：

- 第6章：“锁定”以及相应的代码。
- 第 3.5 节：“代码：物理内存分配器”
- 第 8.1 节到 8.3 节：“概述”、“缓冲区高速缓存层”和“代码：缓冲区高速缓存”

```
  $ git fetch
  $ git checkout lock
  $ make clean
```

## 内存分配器（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

程序 user/kalloctest 强调 xv6 的内存分配器：三个进程增大和缩小其地址空间，导致对`kalloc`和 的多次调用`kfree`。 `kalloc`并`kfree` 获得`kmem.lock`. kalloctest 打印（作为“#test-and-set”）`acquire`由于尝试获取另一个核心已持有的锁（该 `kmem`锁和其他一些锁）而导致的循环迭代次数。循环迭代次数`acquire` 是锁争用的粗略衡量标准。`kalloctest`在开始实验之前，输出看起来与此类似：

```
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #test-and-set 83375 #acquire() 433015
lock: bcache: #test-and-set 0 #acquire() 1260
--- top 5 contended locks:
lock: kmem: #test-and-set 83375 #acquire() 433015
lock: proc: #test-and-set 23737 #acquire() 130718
lock: virtio_disk: #test-and-set 11159 #acquire() 114
lock: proc: #test-and-set 5937 #acquire() 130786
lock: proc: #test-and-set 4080 #acquire() 130786
tot= 83375
test1 FAIL
start test2
total free number of pages: 32497 (out of 32768)
.....
test2 OK
start test3
child done 1
child done 100000
test3 OK
start test2
total free number of pages: 32497 (out of 32768)
.....
test2 OK
start test3
child done 1
child done 100000
test3 OK
```

您可能会看到与此处显示的计数不同的计数，并且前 5 个竞争锁的顺序也不同。

`acquire`对于每个锁，维护对该锁的调用计数，以及尝试但未能设置锁的`acquire`循环次数。 `acquire`kalloctest 调用一个系统调用，使内核打印 kmem 和 bcache 锁（这是本实验的重点）以及 5 个竞争最激烈的锁的计数。如果存在锁争用，则`acquire`循环迭代的次数将会很大。该系统调用返回 kmem 和 bcache 锁的循环迭代次数之和。

对于本实验，您必须使用具有多个内核的专用卸载计算机。如果您使用正在执行其他操作的机器，则 kalloctest 打印的计数将是无意义的。您可以使用专用的 Athena 工作站或您自己的笔记本电脑，但不要使用拨号计算机。

kalloctest 中锁争用的根本原因是`kalloc()`具有单个空闲列表，并受单个锁的保护。要消除锁争用，您必须重新设计内存分配器以避免单个锁和列表。基本思想是为每个 CPU 维护一个空闲列表，每个列表都有自己的锁。不同CPU上的分配和释放可以并行运行，因为每个CPU将在不同的列表上操作。主要挑战是处理一个CPU的空闲列表为空，但另一个CPU的列表有空闲内存的情况；在这种情况下，一个 CPU 必须“窃取”另一个 CPU 的空闲列表的一部分。窃取可能会导致锁争用，但希望这种情况不会频繁发生。

您的工作是实现每个 CPU 的空闲列表，并在 CPU 的空闲列表为空时进行窃取。您必须给出所有以“kmem”开头的锁名称。也就是说，您应该调用`initlock`每个锁，并传递一个以“kmem”开头的名称。运行 kalloctest 来查看您的实现是否减少了锁争用。要检查它是否仍然可以分配所有内存，请运行`usertests sbrkmuch`。您的输出将类似于下图所示，尽管具体数字会有所不同，但 kmem 锁的争用总数大大减少。确保所有测试都`usertests -q`通过。 `make grade`应该说 kalloctests 通过了。

```
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #test-and-set 0 #acquire() 42843
lock: kmem: #test-and-set 0 #acquire() 198674
lock: kmem: #test-and-set 0 #acquire() 191534
lock: bcache: #test-and-set 0 #acquire() 1242
--- top 5 contended locks:
lock: proc: #test-and-set 43861 #acquire() 117281
lock: virtio_disk: #test-and-set 5347 #acquire() 114
lock: proc: #test-and-set 4856 #acquire() 117312
lock: proc: #test-and-set 4168 #acquire() 117316
lock: proc: #test-and-set 2797 #acquire() 117266
tot= 0
test1 OK
start test2
total free number of pages: 32499 (out of 32768)
.....
test2 OK
start test3
child done 1
child done 100000
test3 OK
$ usertests sbrkmuch
usertests starting
test sbrkmuch: OK
ALL TESTS PASSED
$ usertests -q
...
ALL TESTS PASSED
$
```

一些提示：

- `NCPU`您可以使用`kernel/param.h` 中的常量

- 让`freerange`所有空闲内存都给正在运行的CPU `freerange`。

- 该函数`cpuid`返回当前的核心编号，但只有在中断关闭时调用它并使用其结果才是安全的。您应该使用 `push_off()`和`pop_off()`来关闭和打开中断。

- 查看`snprintf`kernel/sprintf.c 中的函数以了解字符串格式化的想法。不过，将所有锁命名为“kmem”就可以了。

- （可选）使用 xv6 的竞争检测器运行您的解决方案：

  ```
  	$ make clean
  	$ make KCSAN=1 qemu
  	$ kalloctest
  	  ..
        
  ```

`kalloctest` 可能会失败，但你不应该看到任何竞态。如果 xv6 的竞争检测器观察到竞争，它将打印两个堆栈跟踪，按照以下几行描述竞争：

  ```
  	 == race detected ==
  	 backtrace for racing load
  	 0x000000008000ab8a
  	 0x000000008000ac8a
  	 0x000000008000ae7e
  	 0x0000000080000216
  	 0x00000000800002e0
  	 0x0000000080000f54
  	 0x0000000080001d56
  	 0x0000000080003704
  	 0x0000000080003522
  	 0x0000000080002fdc
  	 backtrace for watchpoint:
  	 0x000000008000ad28
  	 0x000000008000af22
  	 0x000000008000023c
  	 0x0000000080000292
  	 0x0000000080000316
  	 0x000000008000098c
  	 0x0000000080000ad2
  	 0x000000008000113a
  	 0x0000000080001df2
  	 0x000000008000364c
  	 0x0000000080003522
  	 0x0000000080002fdc
  	 ==========
        
  ```

   在您的操作系统上，您可以通过`addr2line`将回溯跟踪剪切并粘贴到其中，将其转换为带有行号的函数名称

  ```
 $ riscv64-linux-gnu-addr2line -e kernel/kernel
 0x000000008000ab8a
 0x000000008000ac8a
 0x000000008000ae7e
 0x0000000080000216
 0x00000000800002e0
 0x0000000080000f54
 0x0000000080001d56
 0x0000000080003704
 0x0000000080003522
 0x0000000080002fdc
ctrl-d
kernel/kcsan.c:157
	  kernel/kcsan.c:241
	  kernel/kalloc.c:174
	  kernel/kalloc.c:211
	  kernel/vm.c:255
	  kernel/proc.c:295
	  kernel/sysproc.c:54
	  kernel/syscall.c:251
        
  ```

您不需要运行竞争检测器，但您可能会发现它很有帮助。请注意，竞争检测器会显着减慢 xv6 的速度，因此您可能不想在运行时使用它

## 缓冲区高速缓存（[困难](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

这一半的作业与前半部分是独立的；无论您是否完成了前半部分，您都可以完成这半部分（并通过测试）。

如果多个进程密集使用文件系统，它们可能会争夺`bcache.lock`，这个锁保护 `kernel/bio.c` 中的磁盘块缓存。 `bcachetest`会创建多个重复读取不同文件的进程，以便在 上产生争用`bcache.lock`；在完成本实验之前，其输出如下所示：

```
$ bcachetest
start test0
test0 results:
--- lock kmem/bcache stats
lock: kmem: #test-and-set 0 #acquire() 33035
lock: bcache: #test-and-set 16142 #acquire() 65978
--- top 5 contended locks:
lock: virtio_disk: #test-and-set 162870 #acquire() 1188
lock: proc: #test-and-set 51936 #acquire() 73732
lock: bcache: #test-and-set 16142 #acquire() 65978
lock: uart: #test-and-set 7505 #acquire() 117
lock: proc: #test-and-set 6937 #acquire() 73420
tot= 16142
test0: FAIL
start test1
test1 OK
```

您可能会看到不同的输出，但锁的测试和设置次数`bcache`会很高。如果您查看`kernel/bio.c`中的代码，您将看到`bcache.lock`保护缓存块缓冲区列表、`b->refcnt`每个块缓冲区中的引用计数 ( ) 以及缓存块的标识 (`b->dev`和`b->blockno`)。

修改块缓存，使运行 `bcachetest` 时，`bcache` 中所有锁的获取循环迭代次数接近于零。理想情况下，块缓存中所有锁的计数总和应为零，但如果总和小于 500 也没关系。修改 `bget` 和 `brelse`，使对 `bcache` 中不同块的并发查找和释放不太可能发生锁冲突（例如，不必全部等待 `bcache.lock`）。您必须保持每个块最多缓存一份副本的不变性。完成后，输出结果应与下图类似（但不完全相同）。确保 `usertests -q`仍能通过。完成后，`make grade` 应能通过所有测试。


```
$ bcachetest
start test0
test0 results:
--- lock kmem/bcache stats
lock: kmem: #test-and-set 0 #acquire() 32954
lock: kmem: #test-and-set 0 #acquire() 75
lock: kmem: #test-and-set 0 #acquire() 73
lock: bcache: #test-and-set 0 #acquire() 85
lock: bcache.bucket: #test-and-set 0 #acquire() 4159
lock: bcache.bucket: #test-and-set 0 #acquire() 2118
lock: bcache.bucket: #test-and-set 0 #acquire() 4274
lock: bcache.bucket: #test-and-set 0 #acquire() 4326
lock: bcache.bucket: #test-and-set 0 #acquire() 6334
lock: bcache.bucket: #test-and-set 0 #acquire() 6321
lock: bcache.bucket: #test-and-set 0 #acquire() 6704
lock: bcache.bucket: #test-and-set 0 #acquire() 6696
lock: bcache.bucket: #test-and-set 0 #acquire() 7757
lock: bcache.bucket: #test-and-set 0 #acquire() 6199
lock: bcache.bucket: #test-and-set 0 #acquire() 4136
lock: bcache.bucket: #test-and-set 0 #acquire() 4136
lock: bcache.bucket: #test-and-set 0 #acquire() 2123
--- top 5 contended locks:
lock: virtio_disk: #test-and-set 158235 #acquire() 1193
lock: proc: #test-and-set 117563 #acquire() 3708493
lock: proc: #test-and-set 65921 #acquire() 3710254
lock: proc: #test-and-set 44090 #acquire() 3708607
lock: proc: #test-and-set 43252 #acquire() 3708521
tot= 128
test0: OK
start test1
test1 OK
$ usertests -q
  ...
ALL TESTS PASSED
$
```

请提供所有以“bcache”开头的锁名称。也就是说，您应该调用`initlock`每个锁，并传递一个以“bcache”开头的名称。

减少块缓存中的争用比 kalloc 更棘手，因为 bcache 缓冲区真正在进程（以及 CPU）之间共享。对于 kalloc，可以通过为每个 CPU 提供自己的分配器来消除大多数争用。这不适用于块缓存。我们建议您使用每个哈希桶都有一个锁的哈希表在缓存中查找块号。

在某些情况下，如果您的解决方案存在锁冲突也没关系：

- 当两个进程同时使用相同的块号时。`bcachetest` `test0`从来不这样做。
- 当两个进程同时在缓存中丢失，并且需要找到一个未使用的块来替换时。`bcachetest` `test0`从来不这样做。
- 当两个进程同时使用块时，无论您使用什么方案来分区块和锁，这些块都会发生冲突；例如，如果两个进程使用的块的块号散列到哈希表中的同一槽。`bcachetest` `test0`可能会这样做，具体取决于您的设计，但您应该尝试调整方案的详细信息以避免冲突（例如，更改哈希表的大小）。

`bcachetest`使用`test1`比缓冲区更多的不同块，并使用大量文件系统代码路径。

以下是一些提示：

- 阅读 xv6 书中对块缓存的描述（第 8.1-8.3 节）。
- 使用固定数量的桶并且不动态调整哈希表的大小是可以的。使用素数的桶（例如，13）来减少散列冲突的可能性。
- 在哈希表中搜索缓冲区并在未找到缓冲区时为该缓冲区分配条目必须是原子的。
- 删除所有缓冲区（等）的列表`bcache.head`，并且不实现 LRU。通过此更改`brelse`不需要获取 bcache 锁。在中`bget`，您可以选择任何有的块，`refcnt == 0`而不是最近最少使用的块。
- 您可能无法自动检查缓存的 buf 并（如果未缓存）找到未使用的 buf；如果缓冲区不在缓存中，您可能必须放弃所有锁定并从头开始。序列化查找未使用的 buf 是可以的（即，当缓存中查找未命中时，该`bget`部分选择要重用的缓冲区）。`bget`
- 在某些情况下，您的解决方案可能需要持有两个锁；例如，在驱逐期间，您可能需要持有 bcache 锁和每个存储桶的锁。确保避免僵局。
- 替换块时，您可能会将块`struct buf`从一个存储桶移动到另一个存储桶，因为新块会散列到不同的存储桶。您可能会遇到一个棘手的情况：新块可能会散列到与旧块相同的存储桶。在这种情况下请确保避免死锁。
- 一些调试技巧：实现桶锁，但将全局 bcache.lock acquire/release 保留在 bget 的开头/结尾以序列化代码。一旦确定它在没有竞争条件的情况下是正确的，请删除全局锁并处理并发问题。您也可以`make CPUS=1 qemu`使用一个核心来运行测试。
- 使用 xv6 的竞争检测器来查找潜在的竞争（请参阅上面如何使用竞争检测器）。

## 可选的挑战练习

- 维护 LRU 列表，以便逐出最近最少使用的缓冲区，而不是任何未使用的缓冲区。
- 使缓冲区高速缓存中的查找无锁。提示：使用 gcc 的`__sync_*`函数。您如何说服自己您的实施是正确的？