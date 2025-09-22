内存泄露检测工具，[官网地址](https://valgrind.org/)

## 问题描述

即使使用调试器，我们也可能**无法捕获所有错误**。 有些错误就是我们所说的“bohrbug”，这意味着它们在明确定义但可能未知的条件下可靠地表现出来。 其他错误就是我们所说的“heisenbug”，它们不是决定性的，而是在人们试图研究它们时消失或改变其行为。 我们可以使用调试器检测第一种类型，但第二种类型可能会被我们忽视，因为它们（至少在 C 语言中）通常是由于**内存管理不当**造成的。 请记住，与其他编程语言不同，C 要求您（程序员）**手动管理内存**。

## 未初始化
```c
#include<stdio.h>
#include<stdlib.h>

int main(){
	char *p; // 未初始化空间
	char c = *p; //访问野指针
	printf("\n [%c]\n", c);
	return 0;
}
```

输出：
```sh
Use of uninitialised value of size 4
	at 0x10420:main (in /tmp/demo/a.out)
```

## 内存泄漏

```c
#include<stdio.h>
#include<stdlib.h>

int foo(){
	char *p = malloc(1); 
	*p = 'a'; // 赋值两次
	char c = *p; //访问野指针
	printf("\n [%c]\n", c);
	return 0;
}
```
输出
```sh
	1 bytes in 1 blocks are definitely lost in loss record 1 of 1
		at 0x483FE64: malloc (vg replace malloc.c:381)
		by 0x10457: main (in /tmp/demo/a.out)
```

## 释放后读写

```c
#include<stdio.h>
#include<stdlib.h>

int main(){
	char *p, c; 
	*p = 'a';
	printf("\n [%c]\n", c);
	free(p);
	c = *p; // 释放后读写
	return 0;
}
```
输出：
```sh
	Invalid read of size 1
		at 0x104CC:main (in /root/a.out)
		Address 0x494c028 is 0 bytes inside a block of size 1 free'd
		at 0x4843220:free (vg_replace_malloc.c:872)
		by 0x104C7:main (in /tmp/demo/a.out)
```


## 尾部读写
```c
#include<stdio.h>
#include<stdlib.h>

int main(){
	char *p; 
	*p = 'a';
	char c = *(p+1); // 访问“越界”
	printf("\n [%c]\n", c);
	
	return 0;
}
```
输出
```sh
Invalid read of size 1
	at 0x104A4:main (in /root/a.out)
	Address 0x494c029 is 0 bytes after a block of size 1 alloc'd
	at 0x483FE64:malloc (vg replace malloc.c:381)
	by 	0x1048B:main (in /root/a.out)
```