## 回溯（[中等](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）
### 添加函数定义
`将backtrace()`的原型添加到`kernel/defs.h`中，以便您可以在`sys_sleep`中调用`backtrace`。

```c
// in defs.h
// printf.c
void            backtrace(void);
```


```c
// in sys_proc.c
uint64
sys_sleep(void)
{
  backtrace();// 
}
```


### 实现`回溯`方法
前情提要， 在 RISC-V 架构中，函数调用时的栈帧布局如下： 
- **栈帧指针 (fp)** ：指向当前栈帧的底部（高地址）。
- **返回地址 (ra)** ：存储在栈帧中，表示函数返回时的地址。
- **调用者的栈帧指针** ：存储在栈帧中，用于链接到上一个栈帧。
```
+-------------------+
| 返回地址 (ra)     | <- fp - 8
+-------------------+
| 上一个 fp         | <- fp - 16
+-------------------+
| 局部变量          |
+-------------------+
| ...               |
```

调用时汇编大概是这样：
```asm
addi sp, sp, -16
sd ra, 8(sp)
sd s0, 0(sp)
addi s0, sp, 16
```

因此，根据RISC-V的栈帧结构
- `fp - 8` 存储的是返回地址（`ra`）。
- `fp - 16` 存储的是调用者的栈帧指针（`fp`）。

通过这种方式，可以通过当前栈帧指针逐步回溯到调用者的栈帧。


代码如下：
```c
void backtrace(void)
{
  printf("backtrace:\n");
  struct proc *p = myproc();
  if (p == 0)
    return;
  
  uint64 fp = r_fp(); // 帧指针 fp 包含当前栈帧的底部地址
  // 将fp看作指针，指向栈帧的底部地址，现在需要将其转换为实际的栈帧地址
  // fp = (fp - 16) & ~0xf; // 栈帧底部地址减去16字节，然后对齐到16字节边界

  // 反向追踪栈帧，打印返回地址
  while (PGROUNDUP(fp) != fp)
  {
    uint64* ret_addr = (uint64 *)(fp - 8);
    printf("%p\n", *ret_addr);
    uint64* caller_fp = (uint64*)(fp - 16);
    fp = *caller_fp;
  }
}
```

终止条件`PGROUNDUP(fp) != fp`是检查`fp`是否位于页面顶端。栈向下增长，当`fp`到达页面对齐地址时，认为已遍历到栈顶，可以退出。

## 报警（[困难](https://pdos.csail.mit.edu/6.828/2023/labs/guidance.html)）
