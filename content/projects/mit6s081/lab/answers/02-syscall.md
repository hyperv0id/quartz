


## Using GDB




## Syscall Tracing

应该是`Linux`系统中的`strace`命令

### 系统调用执行过程

1. 定义一个用户态接口`int trace(int);`(`user.h`)中
2. 使用`usys.pl`定义一个entry，将用户态的`trace()`转成`SYS_trace`
3. 在内核态执行根据传入的数字，选择对应内核函数执行（`syscall.c`、`syscall.h`）

`usys.pl`中`entry`的作用就是生成下列汇编：
```asm
.global trace
trace:
	li a7, SYS_trace # SYS_trace是系统调用号
	ecall # riscv指令ecall，硬件调用系统调用
ret
```

### Lab过程

1. 照着lab提示在proc结构体里面创建新变量`trace_mask`
2. 在proc初始化时将其一并初始化
3. 在调用trace时覆盖mask
4. 还有一个测试用例要求在`fork`时也要一并传递mask,在fork中添加复制代码

编辑器格式化了好多代码TAT，还是看commit吧：[lab-syscall-trace · GitHub](https://github.com/hyperv0id/mit6.828/commit/c40e1a877b3c53338a62a876b345d3265f6a13f8)


## Sysinfo

照着`trace`an一样添加代码，在`def.h`添加相关的入口，在对应文件实现相关函数。

可以注释一部分测试代码分多次排查，不知道怎么给test设断点TAT

```diff
diff --git a/Makefile b/Makefile
index ccf335b..b206b7a 100644
--- a/Makefile
+++ b/Makefile
@@ -189,6 +189,7 @@ UPROGS=\
 	$U/_wc\
 	$U/_zombie\
 	$U/_trace\
+	$U/_sysinfotest



diff --git a/answers-syscall.txt b/answers-syscall.txt
index 30404ce..dbb01dd 100644
--- a/answers-syscall.txt
+++ b/answers-syscall.txt
@@ -1 +1,5 @@
+TODO
+TODO
+TODO
+TODO
 TODO
\ No newline at end of file
diff --git a/kernel/defs.h b/kernel/defs.h
index a3c962b..0ef4535 100644
--- a/kernel/defs.h
+++ b/kernel/defs.h
@@ -63,6 +63,7 @@ void            ramdiskrw(struct buf*);
 void*           kalloc(void);
 void            kfree(void *);
 void            kinit(void);
+uint64          kmem_free(void);

 // log.c
 void            initlog(int, struct superblock*);
@@ -106,6 +107,7 @@ void            yield(void);
 int             either_copyout(int user_dst, uint64 dst, void *src, uint64 len);
 int             either_copyin(void *dst, int user_src, uint64 src, uint64 len);
 void            procdump(void);
+uint64          proccnt(void);

 // swtch.S
 void            swtch(struct context*, struct context*);
diff --git a/kernel/kalloc.c b/kernel/kalloc.c
index 0699e7e..e6f8803 100644
--- a/kernel/kalloc.c
+++ b/kernel/kalloc.c
@@ -80,3 +80,18 @@ kalloc(void)
     memset((char*)r, 5, PGSIZE); // fill with junk
   return (void*)r;
 }
+
+// Calculate Free Memory in Bytes
+uint64
+kmem_free()
+{
+  uint64 free_bytes = 0;
+  struct run *r;
+
+  acquire(&kmem.lock);
+  for(r = kmem.freelist; r; r = r->next)
+    free_bytes += PGSIZE;
+  release(&kmem.lock);
+
+  return free_bytes;
+}
\ No newline at end of file
diff --git a/kernel/proc.c b/kernel/proc.c
index 423e585..fd41961 100644
--- a/kernel/proc.c
+++ b/kernel/proc.c
@@ -704,3 +704,18 @@ void procdump(void)
     printf("\n");
   }
 }
+
+// count the number of processes in the process table
+uint64
+proccnt()
+{
+  int cnt = 0;
+  for (int i = 0; i < NPROC; i++)
+  {
+    if (proc[i].state != UNUSED)
+    {
+      cnt++;
+    }
+  }
+  return cnt;
+}
\ No newline at end of file
diff --git a/kernel/syscall.c b/kernel/syscall.c
index 8efdd6e..46bdabe 100644
--- a/kernel/syscall.c
+++ b/kernel/syscall.c
@@ -98,6 +98,7 @@ extern uint64 sys_link(void);
 extern uint64 sys_mkdir(void);
 extern uint64 sys_close(void);
 extern uint64 sys_trace(void);
+extern uint64 sys_sysinfo(void);

 // An array mapping syscall numbers from syscall.h
 // to the function that handles the system call.
@@ -124,6 +125,7 @@ static uint64 (*syscalls[])(void) = {
     [SYS_mkdir] sys_mkdir,
     [SYS_close] sys_close,
     [SYS_trace] sys_trace,
+    [SYS_sysinfo] sys_sysinfo,
 };

 static char *syscall_names[] = {
@@ -149,6 +151,7 @@ static char *syscall_names[] = {
     [SYS_mkdir] "mkdir",
     [SYS_close] "close",
     [SYS_trace] "trace",
+    [SYS_sysinfo] "sysinfo",
 };

 void syscall(void)
diff --git a/kernel/syscall.h b/kernel/syscall.h
index c8decab..db6727f 100644
--- a/kernel/syscall.h
+++ b/kernel/syscall.h
@@ -22,3 +22,4 @@
 #define SYS_close 21
 // trace的系统调用号
 #define SYS_trace 22
+#define SYS_sysinfo 23
diff --git a/kernel/sysinfo.h b/kernel/sysinfo.h
index fb878e6..407af1c 100644
--- a/kernel/sysinfo.h
+++ b/kernel/sysinfo.h
@@ -1,4 +1,5 @@
 struct sysinfo {
   uint64 freemem;   // amount of free memory (bytes)
   uint64 nproc;     // number of process
 };
diff --git a/kernel/sysproc.c b/kernel/sysproc.c
index e39f412..12cae3c 100644
--- a/kernel/sysproc.c
+++ b/kernel/sysproc.c
@@ -5,17 +5,34 @@
 #include "proc.h"
+#include "sysinfo.h"

+uint64
+sys_sysinfo(void)
+{
+  uint64 addr; // user pointer to struct stat
+  argaddr(0, &addr);
+  struct sysinfo info;
+  // calc sysinfo
+  info.freemem = kmem_free();
+  info.nproc = proccnt();
+  // copy system info to user-space
+  // copyout(p->pagetable, dst, src, len)
+  int check = copyout(myproc()->pagetable, addr, (char *)&info, sizeof(info));
+  if(check < 0)
+      return -1;
+  return 0;
+}
+
 uint64
 sys_exit(void)
 {
diff --git a/time.txt b/time.txt
new file mode 100644
index 0000000..9a03714
--- /dev/null
+++ b/time.txt
@@ -0,0 +1 @@
+10
diff --git a/user/user.h b/user/user.h
index ed00c2b..0a692fb 100644
--- a/user/user.h
+++ b/user/user.h
@@ -23,6 +23,8 @@ char *sbrk(int);
 int sleep(int);
 int uptime(void);
 int trace(int); // 用户trace系统调用入口
+struct sysinfo;
+int sysinfo(struct sysinfo *);

 // ulib.c
 int stat(const char *, struct stat *);
diff --git a/user/usys.pl b/user/usys.pl
index 2efecff..87929ba 100755
--- a/user/usys.pl
+++ b/user/usys.pl
@@ -42,4 +42,5 @@ entry("uptime");
 #  li a7, SYS_trace # SYS_trace是系统调用号
 #  ecall # riscv指令ecall，调用系统调用
 #  ret
-entry("trace");
+entry("trace");
+entry("sysinfo");
```




## Result Log

[commit: lab-syscall-sysinfo](https://github.com/hyperv0id/mit6.828/commit/e12da8f36aa82a30641a4ccf7289f14bffb6d12e)

```sh
$ make grade
== Test answers-syscall.txt ==
answers-syscall.txt: OK
== Test trace 32 grep ==
$ make qemu-gdb
trace 32 grep: OK (5.5s)
== Test trace all grep ==
$ make qemu-gdb
trace all grep: OK (1.2s)
== Test trace nothing ==
$ make qemu-gdb
trace nothing: OK (1.1s)
== Test trace children ==
$ make qemu-gdb
trace children: OK (22.3s)
== Test sysinfotest ==
$ make qemu-gdb
sysinfotest: OK (4.7s)
== Test time ==
time: OK
Score: 40/40
```

