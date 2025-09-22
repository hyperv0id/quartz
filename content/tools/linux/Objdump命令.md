---
tags:
  - linux/commands
  - manual
created: 2023-03-16T00:43:34 (UTC +08:00)
---
## 功能

用汇编语言的形式显示机器码

## 示例

首先是一个简单的程序(2017考研题)
```c
// add.c
int f1(unsigned n){
	int sum=1, power=1;
	for(unsigned i=0; i<=n-1; i++){
		power *= 2;
		sum+=power;
	}
	return sum;
}

```

使用gcc编译代码：`gcc -c add.c`

使用Objdump查看汇编代码：`objdump -d add.o`

```txt

add.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <f1>:
   0:	f3 0f 1e fa          	endbr64 
   4:	55                   	push   %rbp
   5:	48 89 e5             	mov    %rsp,%rbp
   8:	89 7d ec             	mov    %edi,-0x14(%rbp)
   b:	c7 45 f4 01 00 00 00 	movl   $0x1,-0xc(%rbp)
  12:	c7 45 f8 01 00 00 00 	movl   $0x1,-0x8(%rbp)
  19:	c7 45 fc 00 00 00 00 	movl   $0x0,-0x4(%rbp)
  20:	eb 0d                	jmp    2f <f1+0x2f>
  22:	d1 65 f8             	shll   -0x8(%rbp)
  25:	8b 45 f8             	mov    -0x8(%rbp),%eax
  28:	01 45 f4             	add    %eax,-0xc(%rbp)
  2b:	83 45 fc 01          	addl   $0x1,-0x4(%rbp)
  2f:	8b 45 ec             	mov    -0x14(%rbp),%eax
  32:	83 e8 01             	sub    $0x1,%eax
  35:	39 45 fc             	cmp    %eax,-0x4(%rbp)
  38:	76 e8                	jbe    22 <f1+0x22>
  3a:	8b 45 f4             	mov    -0xc(%rbp),%eax
  3d:	5d                   	pop    %rbp
  3e:	c3                   	ret    

```