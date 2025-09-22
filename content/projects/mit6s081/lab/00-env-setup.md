[TOC]



## 实验环境搭建

### 系统准备

官方教程：[6.S081 / Fall 2021 (mit.edu)](https://pdos.csail.mit.edu/6.828/2021/tools.html)

我这里使用的是 `wsl2`下的`ubuntu22`

### 克隆源码

```shell
git clone git://g.csail.mit.edu/xv6-labs-2021
```

克隆后只有一个`.git`文件夹，检查版本管理分支，这里每一个分支都代表一个作业

```shell
git branch --remote
```

![image-20230129133331462](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/29/image-20230129133331462-af9acd.png)



切换到`util`分支

```
git checkout util
```

```
Branch 'util' set up to track remote branch 'util' from 'origin'.
Switched to a new branch 'util'
```

### 环境依赖

不同环境的包管理器不同，ubuntu使用的是arch

```bash
sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu 
```

### 测试安装

#### 测试能否启动`xv6`：

```shell
make qemu
# 一大段输出并进入qemu模拟的xv6的shell，<C-a> x退出
# 在shell中你可以运行 ls，cat，echo等命令
```

#### 测试工具链

```
$ riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-gcc (GCC) 9.2.0
$ qemu-system-riscv64 --version
QEMU emulator version 4.1.0
```

#### 错误处理：自己编译工具链

如果报错，即使能运行xv6，也**无法调试**，那么可能得[自己编译](https://stackoverflow.com/questions/68611071/how-to-install-riscv64-gdbhttps://stackoverflow.com/questions/68611071/how-to-install-riscv64-gdb)了：

下载源代码([百度网盘]())：

```
$ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
```

配置编译后路径，[mit网站]()上是`/usr/local`，而[github仓库](https://github.com/riscv-collab/riscv-gnu-toolchainhttps://github.com/riscv-collab/riscv-gnu-toolchain)是`/opt/riscv`，我这里折中取`/usr/local/riscv`

```shell
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev ninja-build # 编译依赖项
./configure --prefix=/usr/local/riscv
make # 可能需要sudo
# make -j8 # 多线程提速，8指8个线程
# make install # 可能的步骤
```

配置环境变量

```bash
vim ~/.bashrc # 如果是zsh等shell改成相应rc文件
export PATH="$PATH:/usr/local/riscv/bin" # 在文件最后写入
source ~/.bashrc
```



## 如何调试







## 参考

官方：

- xv6文档：[xv6: a simple, Unix-like teaching operating system (mit.edu)](https://pdos.csail.mit.edu/6.828/2021/xv6/book-riscv-rev2.pdf)
- 环境搭建：[6.S081 / Fall 2021 (mit.edu)](https://pdos.csail.mit.edu/6.828/2021/tools.html)
- [Lab guidance (mit.edu)](https://pdos.csail.mit.edu/6.828/2021/labs/guidance.html)



- 课程中文翻译：[简介 - MIT6.S081 (gitbook.io)](https://mit-public-courses-cn-translatio.gitbook.io/mit6-s081/)
- [课程介绍 · 6.S081 All-In-One (dgs.zone)](http://xv6.dgs.zone/)
