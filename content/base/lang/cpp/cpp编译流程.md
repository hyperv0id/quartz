
```sh
g++ -E main.cpp -o main.i
g++ -S main.i -o main.s
g++ -C main.s -o main.o
```

## 预编译 .i

删除注释、头文件引入、宏展开

## 编译 .s

代码优化、汇总所有符号
## 汇编 .o

转成机器码，可`objdump`查看c
## 链接可执行

合并所有obj的段，调整偏移、段长度、合并符号表

符号重定位、注意强弱符号


符号处理：对obj文件的global符号处理，local(static修饰)符号不处理，符号解析后分配内存地址



