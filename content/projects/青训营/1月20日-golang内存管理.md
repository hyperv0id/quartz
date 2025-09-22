# Go语言内存管理详解

[TOC]

- 课件：[‌⁤⁢⁤‍﻿⁢⁤⁡⁢⁤‍⁤⁢‌⁢﻿⁤‌⁡﻿﻿⁢⁡⁣‌⁣⁣⁡⁤⁢⁢﻿⁤⁣⁣﻿⁣﻿‍⁢‍‍‍高性能 Go 语言发行版优化与落地实践 .pptx - 飞书云文档 (feishu.cn)](https://bytedance.feishu.cn/file/boxcngF8NWGNFuXUkdyQViZq6vd)
- 🎶手册：
[【后端专场 学习资料二】第五届字节跳动青训营 - 掘金](https://juejin.cn/post/7189525739836801085/)
- 📚 观看链接： 
	- [Go 语言内存管理详解（1)](https://juejin.cn/course/bytetech/7140987981803814919/section/7142749780140097550)
	- [Go 语言内存管理详解（2)](https://juejin.cn/course/bytetech/7140987981803814919/section/7142746789945278500)

## 0 - 概述

性能优化是什么：提高软件系统算力，减少不必要的消耗

目的：提升用户体验，降本增效。减少IO延迟

![image-20230119113221478](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/19/image-20230119113221478-99e1a7.png)

### 性能优化与软件质量

- 质量：软件质量至关重要
- 稳定：在保证接口稳定的前提下改进具体实现
- 测试用例：覆盖尽可能多的场景，方便回归
- 文档：做了什么，没做什么，能达到怎样的效果
- 隔离：通过选项控制是否开启优化
- 可观测：必要的日志输出

![image-20230119143356609](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/19/image-20230119143356609-7acbf1.png)

## 1 - 自动内存管理

### 1.1 - 概念

#### 1.1.1 - 什么是自动内存管理

动态内存：程序在运行时根据需求分配的内存：`malloc()`

自动内存管理：

- 避免手动内存管理，专注于实现业务逻辑。
- 保证内存使用的正确性和安全性：double-free问题，use-after-free问题

三个任务：

- 为新对象分配空间
- 找到存活对象
- 回收死亡对象的内存空间

#### 1.1.2 - 自动内存管理相关概念

- `Mutator`：业务线程，分配新对象，修改对象的指向关系
- `Collector`：`GC`线程，找到存活对象，回收死亡对象的内存空间
- `Serial GC`：只有一个 `collector`，会有暂停
- `Parallel GC`：支持多个`collector`同时回收GC算法，也会暂停，不过效率比`Serial GC`高
- `Concurrent GC`：`mutator(s)` 和 `collector(s)` 可以同时执行，不会暂停。
  - Collectors必须感知到对象指向关系的改变

![image-20230119144954965](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/19/image-20230119144954965-395e7e.png)



ConturrentGC必须感知到内存指向的改变

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/19/20230119145355-82c2d9.png)

#### 1.1.3 - 评价GC算法

- 安全性：基本要求，不能回收存活对象
- 吞吐率：花在GC上的时间，$$1-\frac{{GC}_{time}}{{Total}_{time}}$$ 
- 暂停时间：业务是否感知 stop the world
- 内存开销：GC元数据开销

### 1.2 - Tracing Garbage Collection（GC）

对象被回收的条件：指针指向关系不可达的对象

步骤：

1. 标记根对象：常量，静态变量，线程栈，全局变量
2. 找到可达对象：从根对象出发找到所有可达对象
3. 清理：清除不可达对象
   - 将存活的对象拷贝到另外的内存空间（Copying GC）![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120122442-035853.png)
   - 将死亡对象标记为可分配（Mark-sweep GC）![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120122454-e521de.png)
   - 移动并整理存活对象（Mark-compact GC）![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120122515-1a7e75.png)
4. **因地制宜**：根据对象的生命周期，使用不同的标记和清理策略

<img src="https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/20/image-20230120121746554-2f8b2b.png" alt="image-20230120121746554" style="zoom:67%;" />









### 1.3 - Generational GC

分代假说：most objects die young

对象的年龄：经历过的GC次数

很多对象在分配出来之后很快就会不再使用，对于年轻的对象和年老的对象，制定不同的GC策略



年轻的对象和老年的对象处于heap的不同区域

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120123329-fdcef2.png)



- 年轻代
  - 常规的对象分配
  - 存活对象很少，采用`copy GC`
  - GC的吞吐率很高
- 老年代
  - 对象趋于一直活着，反复复制的带价很大
  - 采用`mark-sweep GC`







### 1.4 - Reference Counting

每个对象都有一个引用的数目，如果引用数为正数对象存活，反之会被清理掉。

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120124554-bf6e87.png)

- 优点：

  - 内存管理被平摊到了程序的运行中

  - 内存管理不需要了解runtime的细节，如cpp中的智能指针

- 缺点

  - 维护引用计数开销较大，需要用原子操作保证**原子性**和**可见性**
  - 无法回收环形数据结构（weak reference解决了）![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120125226-dbb3ae.png)
  - 每个对象引入额外的内存存储引用数量
  - 内存回收可能引发暂停

![image-20230120124951107](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/20/image-20230120124951107-f90140.png)



## 2 - Go内存管理及优化

### 2.1 - Go内存分配

目标：在heap上分配一块内存出来

#### 2.1.1 - 分块

Go会提前将内存分块：

- 调用系统`mmap()` 申请一块大内存，如4MB
- `mapan`：将内存分成几块，如8KB
- 将大块分成**特定大小**的小块，用于对象分配
- `noscan mspan`：分配不包含指针的对象，GC不扫描
- `scan mspan`：分配包含指针的对象，GC需要扫描

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120130427-2ca61a.png)

在对象分配时，根据对象大小，选择最合适的块返回



#### 2.1.2 - 缓存

Golang的缓存机制借鉴了 `TCMalloc` 技术。

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120130636-355315.png)

- 每个`p`包含一个`mcache`用于快速分配内存，用于为绑定在`p`上的 `g`分配对象
- `mcache` 管理一组`mspan`，每个`mspan`中的空间不一样，每次会选择最接近的分配出去
- 如果`mspan`都是满的，就向`mcentral`申请未分配块的`mspan`
- 当`mspan`未分配对象时，`mspa`会被缓存到`mcentral`中而不是直接还给`OS`





### 2. 2 - Go内存管理优化

问题：

- 对象分配十分高频，每秒GB
- 小对象占用较高![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120131423-10be44.png)
- Go内存分配路径较长：`g -> m -> p -> mcache -> mspan -> memory block -> return`

#### 2.2.1 Balanced GC

字节跳动的解决方案：

- 每个`g`绑定一块大内存(1KB)，称作`goroutine allocation buffer(GAB)`
- GAB用于`noscan`的小内存分配
- 使用三个指针维护GAB：base，end，top<img src="https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120131854-e16dbc.png" style="zoom: 50%;" /> 
  - 如果要分配`8B`的内存，之间将top指针向后移动就行：<img src="https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120132122-4be2c7.png" style="zoom: 33%;" />



小细节：

- GAB对于内存管理来说是大对象，但是里面有一个小对象存活就不会被清理
- 方案
  - 当`GAB`总大小超过阈值时，将`GAB`复制到新`GAB`中
  - 原先`GAB`释放
  - 本质：`copying GC`

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120133039-c8bb0a.png)



效果：CPU降低4.6%，核心接口时延下降4.5%~7.7%

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120133224-7c1ff3.png)

## 3 - 编译器和静态分析

### 3.1 - 基本介绍

功能：

- 识别符合语法的和非法的程序
- 生成正确且高效的代码

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/20/20230120133552-fed2e4.png)

分析部分（front end）：

- ​	词法分析，生成词素（lexeme）
- 语法分析，生成语法树
- 语义分析，收集类型数据，进行语义检查
- 中间代码生成，生成与语言无关的IR(Intermediate Representation)

综合部分（back end）：

- 代码优化，生成优化后的IR
- 代码生成，生成目标代码







### 3.2 - 静态分析：数据流和控制流

静态分析：不执行代码，判断程序的行为，分析程序的性质

控制流：程序运行的流程

数据流：数据在控制流上的传递



通过分析，我们可以知道更多关于程序的性质，比如说下图中的程序只会返回4。

![image-20230120172854402](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/20/image-20230120172854402-d68212.png)







### 3.3 - 过程内和过程间分析

过程内分析(Intro-procedural analysis)：仅在函数内部分析

过程间分析(Inter-procedural analysis)：考虑函数调用时参数传递和返回值的数据流和控制流



过程间分析难点：
- 需要数据流分析才能知道变量的类型
- 根据类型的不同，产生了不同的数据流和控制流
- 联合求解比较复杂
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/26/20230226122444-979fc4.png)








## 4 - Go编译器优化

- WHY：
	- 用户无感知，直接重新编译就可以提升效率
	- 通用性
- 思路
	- **编译时间换取更高效的机器码**
- BeastMode
	- **函数内联**
	- **逃逸分析**
	- 默认栈大小调整
	- 边界检查消除
	- 循环展开
	- ...

### 4.1-函数内联

- 内联：被调用函数的函数体(callee)的副本替换到调用位置(caller)上，同时重写代码以反映参数的绑定
- 优点：
	- 消除函数调用的开销：传递参数，保持寄存器
	- 将过程间分析转化为过程内分析，帮助优化
- 缺点
	- 函数体变大，instruction 擦车不友好
	- 编译的go镜像变大了（10%左右）
	- 编译时间变大

benchmark

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/26/20230226123611-7afa87.png)

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/26/20230226123627-f13f31.png)

### 4.2-beast mode
- golang函数内联限制较多
	- interface、defer等等
	- 本身自带的内联策略比较保守
- beast mode：调整内联策略，使得更多函数被内联
	- 降低函数开销
	- 增加其他优化机会：**逃逸分析**


### 4.2-逃逸分析

分析指针的动态作用域，指针在何处何以被访问

大致思路：

- 从对象分配出发，沿着数据流观察控制流

- 若发现指针p在当前作用域S:

  - 作为参数传递给其他函数
  - 传递给全局变量

  - 传递给其他的goroutine

  - 传递给已逃逸的指针指向的对像

- 则指针p指向的对象逃逸出S,反之则没有逃逸出s



优化：

- 未逃逸出去的在栈上分配



#### 性能收益

![image-20230226125038677](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/26/image-20230226125038677-0e365a.png)