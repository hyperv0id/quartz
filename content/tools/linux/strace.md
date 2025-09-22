
strace命令是一个集诊断、调试、统计与一体的工具，可以用来监控**用户进程和内核**的交互，比如对应用程序的**系统调用**、**信号**传递与**进程状态变更**等进行跟踪与分析。

## 命令解析

### 常见命令
- \[-o file\]：将strace的输出写入指定的文件。
    
- \[-s strsize\]：将打印字符串的长度限制为strsize个字符（默认值为32）。
    
- -d：输出strace关于标准错误的调试信息。
    
- -r：打印出相对时间关于每一个系统调用。
    
- -f：跟踪由fork()调用所产生的子进程。
    
- -t：在输出中的每一行前加上时间信息。
    
- -tt：在输出中的每一行前加上时间信息（微秒级）。
    
- -T：打印在每个系统调用中花费的时间。
    
- -r：打印相对时间戳。
    
- -x：以十六进制打印非ascii字符串。
    
- -xx：以十六进制打印所有字符串。
    
- -y：打印与文件描述符参数关联的路径。
    
- -yy：打印与套接字文件描述符关联的协议特定信息。

### 命令统计

- -c：统计每一系统调用的所执行的时间，次数和出错的次数等。
    
- -C：像-c一样，也可以打印常规输出。
    
- -w：汇总系统调用延迟（默认为系统时间）
    
- \[-O overhead\]：设置将系统调用跟踪到OVERUSE的开销
    
- \[-S sortby\]：按以下顺序对系统调用计数：时间，调用，名称等
    

### 命令过滤

- \[-e expr\]…：指定一个表达式，用来控制如何跟踪。格式如下： `[qualifier=][!]value1[,value2]...` 。

> 描述说明： `qualifier` 只能是 `trace、abbrev、verbose、raw、signal、read、write` 其中之一，value是用来限定的符号或数字，默认的 qualifier是trace。感叹号是否定符号，例如: `-e open` 等价于 `-e trace=open` ，表示只跟踪 `open` 调用；而 `-etrace!=open` 表示跟踪除了open以外的其他调用，还有两个特殊的符号 `all` 和 `none` ，它们分别代表所有选项与没有选项。

- `-e trace=set` 只跟踪指定的系统调用，例如: `-e trace=open,close,rean,write` 表示只跟踪这四个系统调用，默认的为 `set=all` 。
    
- `-e trace=file` 只跟踪有关文件操作的系统调用。
    
- `-e trace=process` 只跟踪有关进程控制的系统调用。
    
- `-e trace=network` 跟踪与网络有关的所有系统调用。
    
- `-e strace=signal` 跟踪所有与信号有关的系统调用。
    
- `-e trace=ipc` 跟踪所有与进程通讯有关的系统调用。
    
- `-e raw=set` 将指定的系统调用的参数以十六进制显示。
    
- `-e signal=se` t 指定跟踪的系统信号。默认为all（所有信号）。如 `signal=!SIGIO` 或者 `signal=!io` ，表示不跟踪 `SIGIO` 信号。
    
- `-e read=set` 输出从指定文件中读出的数据。
    
- `-e write=set` 输出写入到指定文件中的数据。

## eg

### 打印Hello World

```c
#include <stdio.h>
int main() {
    printf(".");
    return 0;
}
```

```sh
strace ./a.out
execve("./a.out", ["./a.out"], 0x7ffc0b5355e0 /* 72 vars */) = 0
brk(NULL)                               = 0x5865db828000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (没有那个文件或目录)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=199691, ...}) = 0
mmap(NULL, 199691, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7da562d32000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0`^\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
fstat(3, {st_mode=S_IFREG|0755, st_size=1989944, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7da562d30000
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 2014064, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7da562b44000
mmap(0x7da562b68000, 1490944, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x24000) = 0x7da562b68000
mmap(0x7da562cd4000, 319488, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x190000) = 0x7da562cd4000
mmap(0x7da562d22000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1dd000) = 0x7da562d22000
mmap(0x7da562d28000, 31600, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7da562d28000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7da562b41000
arch_prctl(ARCH_SET_FS, 0x7da562b41740) = 0
set_tid_address(0x7da562b41a10)         = 60198
set_robust_list(0x7da562b41a20, 24)     = 0
rseq(0x7da562b42060, 0x20, 0, 0x53053053) = 0
mprotect(0x7da562d22000, 16384, PROT_READ) = 0
mprotect(0x5865da155000, 4096, PROT_READ) = 0
mprotect(0x7da562d95000, 8192, PROT_READ) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
munmap(0x7da562d32000, 199691)          = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}) = 0
getrandom("\x1a\x1f\x54\xd6\x62\x46\xb1\x4c", 8, GRND_NONBLOCK) = 8
brk(NULL)                               = 0x5865db828000
brk(0x5865db849000)                     = 0x5865db849000
write(1, ".", 1.)                        = 1
exit_group(0)                           = ?
+++ exited with 0 +++
```

可见，一个简单的打印程序就需要调用大量的系统调用，编写高效的系统内核显得尤其重要。