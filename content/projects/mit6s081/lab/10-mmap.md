# 实验室：mmap（[困难](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）

和系统调用允许 UNIX`mmap`程序`munmap`对其地址空间进行详细控制。它们可用于在进程之间共享内存，将文件映射到进程地址空间，并作为用户级页面错误方案的一部分，例如讲座中讨论的垃圾收集算法。在本实验中，您将向 xv6 添加`mmap`和`munmap`，重点关注内存映射文件。

获取实验室的 xv6 源代码并查看分支`mmap`：

```
  $ git fetch
  $ git checkout mmap
  $ make clean
```

手册页（运行 man 2 mmap）显示了以下声明`mmap`：

```
void *mmap(void *addr, size_t len, int prot, int flags,
           int fd, off_t offset);
```

`mmap`可以通过多种方式调用，但本实验仅需要其与内存映射文件相关的功能的子集。您可以假设它`addr`始终为零，这意味着内核应该决定映射文件的虚拟地址。`mmap`返回该地址，如果失败则返回 0xffffffffffffffff。`len`是要映射的字节数；它可能与文件的长度不同。`prot`指示内存是否应映射为可读、可写和/或可执行；您可以假设`prot`是`PROT_READ`或`PROT_WRITE`或两者兼而有之。`flags`将为`MAP_SHARED`，表示对映射内存的修改应写回文件，或`MAP_PRIVATE`，表示不应写回文件。您不必在`flags`.`fd`是要映射的文件的打开文件描述符。您可以假设`offset`为零（它是文件中要映射的起点）。

`MAP_SHARED`如果映射同一文件的进程**不**共享物理页也没关系。

手册页（运行 man 2 munmap）显示了以下声明`munmap`：

```
int munmap(void *addr, size_t len);
```

`munmap`应删除指定地址范围内的 mmap 映射。如果进程修改了内存并将其映射`MAP_SHARED`，则应首先将修改写入文件。调用`munmap`可能只覆盖 mmap 区域的一部分，但您可以假设它将在开始、结束或整个区域取消映射（但不会在区域中间打孔）。

您应该实现足够的`mmap`功能`munmap`以使`mmaptest`测试程序正常工作。如果`mmaptest`不使用某个`mmap`功能，则无需实现该功能。

完成后，您应该看到以下输出：

```
$ mmaptest
mmap_test starting
test mmap f
test mmap f: OK
test mmap private
test mmap private: OK
test mmap read-only
test mmap read-only: OK
test mmap read/write
test mmap read/write: OK
test mmap dirty
test mmap dirty: OK
test not-mapped unmap
test not-mapped unmap: OK
test mmap two files
test mmap two files: OK
mmap_test: ALL OK
fork_test starting
fork_test OK
mmaptest: all tests succeeded
$ usertests -q
usertests starting
...
ALL TESTS PASSED
$ 
```

以下是一些提示：

- 首先添加 `_mmaptest` 到makefile中的 `UPROGS`变量下，然后添加`mmap` 和 `munmap` 系统调用为了使得 `user/mmaptest.c` 成功编译。现在`mmap` 和 `munmap`只需要返回错误。我们在 `kernel/fcntl.h` 中定义了 `PROT_READ` 等宏。运行 `mmaptest`，看看能否执行 `mmap`。     
- 延迟填充页表，以响应页错误。也就是说，`mmap`不应分配物理内存或读取文件。相反，请在 中（或由其调用）的页面错误处理代码中执行此操作`usertrap`，如写时复制实验室中所示。之所以偷懒，是为了保证`mmap`大文件的速度快，`mmap`大于物理内存的文件是可以的。
- 跟踪`mmap`每个进程的映射内容。定义一个与“应用程序的虚拟内存”讲座中描述的VMA（虚拟内存区域）相对应的结构。这应该记录由创建的虚拟内存范围的地址、长度、权限、文件等`mmap`。由于 xv6 内核在内核中没有内存分配器，因此可以声明固定大小的 VMA 数组并根据需要从该数组进行分配。 16 的大小应该足够了。
- 实现`mmap`：在进程的地址空间中找到一个未使用的区域来映射文件，并将VMA添加到进程的映射区域表中。 VMA 应该包含一个指向`struct file`被映射文件的**指针**；`mmap`应该增加文件的**引用计数**，以便在文件关闭时该结构不会消失（提示：请参阅参考资料`filedup`）。 Run `mmaptest`：第一个`mmap`应该成功，但是第一次访问 mmap-ed 内存将导致页面错误并终止`mmaptest`。
- 添加代码以导致 mmap 区域中的页错误分配物理内存页，将相关文件的 4096 字节读入该页，并将其映射到用户地址空间。使用 读取文件`readi`，它采用一个偏移量参数来读取文件（但您必须锁定/解锁传递给 的 inode `readi`）。不要忘记在页面上正确设置权限。运行`mmaptest`;它应该到达第一个`munmap`。
- 实现`munmap`：找到地址范围的VMA并取消映射指定的页面（提示：使用`uvmunmap`）。如果`munmap`删除前一个页面的所有页面`mmap`，则应减少相应页面的引用计数`struct file`。如果未映射的页面已被修改并且文件已映射`MAP_SHARED`，则将该页面写回文件。看看`filewrite`寻找灵感。
- 理想情况下，您的实现只会写回 `MAP_SHARED`程序实际修改的页面。 RISC-V PTE 中的脏位 ( `D`) 指示页面是否已被写入。但是，`mmaptest`不检查非脏页是否未写回；因此，您可以在不查看位的情况下将页面写回`D`。
- 修改`exit`以取消映射进程的映射区域，就像`munmap`已被调用一样。跑步`mmaptest`;`mmap_test`应该会通过，但可能不会`fork_test`。
- 进行修改`fork`以确保子级具有与父级相同的映射区域。不要忘记增加 VMA 的引用计数`struct    file`。在子进程的页面错误处理程序中，可以分配一个新的物理页面，而不是与父进程共享页面。后者会更酷，但需要更多的实施工作。跑步`mmaptest`;它应该同时通过`mmap_test`和`fork_test`。

运行`usertests -q`以确保一切仍然正常。

## 可选挑战

- 如果两个进程具有相同的 mmap 文件（如`fork_test`），则共享它们的物理页。您将需要物理页上的引用计数。
- 您的解决方案可能会为从 mmap 文件读取的每个页面分配一个新的物理页面，即使数据也在缓冲区高速缓存的内核内存中。修改您的实现以使用该物理内存，而不是分配新页面。这要求文件块的大小与页的大小相同（设置`BSIZE`为 4096）。您需要将 mmap 块固定到缓冲区高速缓存中。您需要担心引用计数。
- 消除延迟分配的实现和 mmap 文件的实现之间的冗余。 （提示：为惰性分配区域创建一个 VMA。）
- 修改`exec`为对二进制文件的不同部分使用 VMA，以便您获得按需分页的可执行文件。这将使程序启动速度更快，因为`exec`不必从文件系统读取任何数据。
- 实现页出和页入：当物理内存不足时，让内核将进程的某些部分移至磁盘。然后，当进程引用调出的内存时，对其进行分页。