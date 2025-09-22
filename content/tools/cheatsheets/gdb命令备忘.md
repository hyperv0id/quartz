---
tags:
  - cheatsheet
  - tool/gdb
created: 2023-03-16T00:43:34 (UTC +08:00)
---

> [!abstract]
> gdb 相关的一些指令备忘、插件安装方式等


## gdb 命令


### 配置、杂
- `gdb`/`gdb <file>`
	- 文件需要可调试：`gcc -g`
- 开始会首先执行`.gdbinit`，可以将重复的指令放到这个文件
- `(gdb) help [cmd]`：打印手册
- `(gdb) attach <pid>` 连接程序
- `(gdb) detach` 从当前程序断连
- `(gdb) target remote localhost:1234` 连接 qemu
- `(gdb) tui en` 打开两个窗口，一个源码，一个命令行
- `(gdb) tui dis` 关闭tui

### 运行、结束

- `(gdb) run` 直接运行 (r)
	- `(gdb) run args`：带参数运行
- `(gdb) kill`：终止程序
- `(gdb) quit`：离开gdb `q`,`Ctrl+d`
	- `(gdb) continue` 继续运行 (c)
### 执行
- `(gdb) next` 单步运行，跳过函数 (n)
- `(gdb) step` 运行到下一条源码 (s)
- `(gdb) stepi` 运行到下一条指令 (si)
	- `step i 4`：执行4个指令
- `(gdb) until 3`：运行，知道碰见断点3
- `(gdb) finish` 运行完当前函数 (fin)
- `(gdb) calc foo`：执行某个函数，并打印结果

### 断点

- `(gdb) break [main]` 断在main符号处 `(b)`
- `(gdb) break *0x....` 断在地址
- `(gdb) info breakpoints` 查看断点及状态 `(i b)`
- `(gdb) delete / clear` 清除所有断点 `(d/cl)`
- `(gdb) delete <breakpoint#>` 删除某一断点（从 `i b` 得来断点号）
- `(gdb) clear ...` 清除某一符号、地址处的断点
- `(gdb) disable <breakpoint#>` 禁用某一断点
- `(gdb) enable <breakpoint#>` 启用某一断点
- `(gdb) watch ...` 在某处增加观察点，delete、enable、disable 与断点共用
- `(gdb) break/watch <where> if <condition>` 如果条件满足则断
- `(gdb) condition <breakpoint#> <condition>` 更改条件

### 调用栈

- `(gdb) backtrace` 查看调用栈 `(bt)`
- `(gdb) frame` 查看当前帧栈
- `(gdb) up/down` 移动当前帧栈（向 main / 远离 main）
- `(gdb) info locals` 查看当前帧栈变量
- `(gdb) info args` 查看函数参数

### 查看寄存器 / 变量 / 内存

- `(gdb) print/[format] <expr>`
	- eg：`print/t expr`二进制打印当前值
    - `format`
        - a: pointer
        - c: int -> char
        - d: signed decimal
        - f: floating point number
        - o: int -> octal
        - s: treat as string
        - t: int -> bin
        - u: unsigned decimal
        - x: int -> hex
    - `<expr>`
        - 可以是类 C 表达式
        - 可以是 file_name::variable_name
        - 可以是 function::variable_name
        - 可以是 {type}address
        - 可以是 $register
- `(gdb) display/[format] <expr>`
- `(gdb) undisplay <display#>`
- `(gdb) enable display <display#>`
- `(gdb) disable display <display#>`
- `(gdb) x/nfu <address>` 显示内存
    - n 表示查找、打印几个单位
    - f 表示 format，在前面写了
    - u 表示单位：b 一字节、h 两字节、w 四字节、g 八字节
- `(gdb) info register` 查看所有寄存器 (i r)
- `(gdb) info register <register>` 查看某一寄存器

## gdb 插件

### gdb-peda

每条指令带寄存器、汇编、内存数据回显
```sh
$ git clone https://github.com/longld/peda.git ~/peda
$ echo "source ~/peda/peda.py" >> ~/.gdbinit
```



