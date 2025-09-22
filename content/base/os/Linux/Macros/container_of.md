`container_of(ptr, type, member)`宏根据指向结构体`type`中成员`member`的指针`ptr`获取该结构体的首地址，也就是获取成员`member`所在的结构体`container`。
```c
/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:	the pointer to the member.
 * @type:	the type of the container struct this is embedded in.
 * @member:	the name of the member within the struct.
 *
 * WARNING: any const qualifier of @ptr is lost.
 */
#define container_of(ptr, type, member) ({				\
	void *__mptr = (void *)(ptr);					\
	static_assert(__same_type(*(ptr), ((type *)0)->member) ||	\
		      __same_type(*(ptr), void),			\
		      "pointer type mismatch in container_of()");	\
	((type *)(__mptr - offsetof(type, member))); })
```

`container_of`宏实现的基本思想是从指向`member`的指针`ptr`中减去`member`在该结构体中的偏移量。`Line 9`通过`typeof`运算符`member`的实际类型，并将指向`member`的指针`ptr`转成具有该实际类型的`__mptr`指针。`Line 10`首先通过`offsetof(type, member)`宏获取`member`在结构体类型`type`中的偏移量。由于该偏移量表示的是字节数，因此在做指针减法运算之前，需要先将`__mptr`经过强制类型转换，转成`char *`类型。`(char *)__mptr - offsetof(type,member)`得到了结构体的首地址，但是是`char *`类型，最后还需要经过`(type *)`强制类型转换。