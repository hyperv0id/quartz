使用`gcc`对`.c`文件编译后生成的`.out`可执行文件后，系统是如何运行这个可执行文件的呢？

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229113549-cf6a18.png)

## elf文件是什么
[ELF](../os/Linux/ELF文件.md)（Executable and Linkable Format）文件，也就是在 Linux 中的目标文件，主要有以下三种类型

可重定位文件（Relocatable File），包含由编译器生成的代码以及数据。链接器会将它与其它目标文件链接起来从而创建可执行文件或者共享目标文件。在 Linux 系统中，这种文件的后缀一般为 .o 。
可执行文件（Executable File），就是我们通常在 Linux 中执行的程序。

共享目标文件（Shared Object File），包含代码和数据，这种文件是我们所称的库文件，一般以 .so 结尾。一般情况下，它有以下两种使用情景：

链接器（Link eDitor, ld）可能会处理它和其它可重定位文件以及共享目标文件，生成另外一个目标文件。
动态链接器（Dynamic Linker）将它与可执行文件以及其它共享目标组合在一起生成进程镜像。


文件格式：
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229114107-c354c1.png)


## 宏定义与展开

### 封装库函数

懒得复制粘贴需要的库的代码
```c
#include<bits/stdc++.h>
```

### 搞垮OJ

- 编译器无限展开

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229121242-3f504b.png)

### 半夜改你代码

甚至true和false都能重定义
```c
#define true(__LINE__ % 2 != 0)
```


![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229121139-0888a4.png)

### 简单的 bit-hack数组
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229121034-65ce7e.png)


![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229121043-3aba25.png)


### Xmacro

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229121509-c40c83.png)

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229121532-eeb3c8.png)


### 用户元编程

- gcc的预处理器处理汇编代码
- cpp的模板元编程
- rust中的macros

优点：
- 灵活的用法
- 接近自然语言
缺点：
- 破坏IOCCC
- 破坏程序分析、补全
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/12/29/20231229121731-946cc1.png)
