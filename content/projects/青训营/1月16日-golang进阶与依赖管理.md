## 资料



## 1 - 语言进阶

### 1.1 - 并发与并行

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-b1b753.png)

并发：在一个CPU核心上通过分时处理达到一起运行的效果

并行：CPU有多个核心，不需要划分

### 1.2 - Goroutine

协程：用户态，轻量级的线程，栈MB级别

线程：内核态，线程可以跑满多个协程，栈KB级别

开启协程语法：

```Go
go funcName() // 将函数交给 gorouine 处理
```

### 1.3 - CSP（Communication Sequential Process）

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-cc93b7.png)

通道（channel）：让 goroutine 发送特定值到另一个 goroutine 的机制，保证数据的先入先出。

后者，共享内存实现通信的话需要加锁，产生数据竞争的问题

### 1.4 - Channel

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-e9a68f.png)

创建方式：

```Go
// 创建语法
make(chan 元素类型, [缓冲大小])
// 无缓冲，也被称为同步通道
make(chan int)
// 有缓冲。典型生产消费模型
make(chan int, 2)
```

#### Example

```Go
package main

import "fmt"

func CalSquare() {
   src := make(chan int)
   // B 和 M 之间的执行效率差距，M可能很复杂，需要缓冲区
   dest := make(chan int, 3)
   go func() {
      // A, 生产数字并将数字发送到 src channel
      defer close(src)
      for i := 0; i < 10; i++ {
         src <- i
      }
   }()
   go func() {
      // B, 从 A 协程读取，计算 i*i 并传到 dest channel
      defer close(dest)
      for i := range src {
         dest <- i * i
      }
   }()
   for i := range dest {
      // M
      // 复杂的逻辑
      fmt.Println(i)

   }
}

func main() {

   CalSquare()
}
```

### 1.5 - 并发安全Lock

TODO

## 2 - 依赖管理

工程项目不能完全基于标准库从零搭建，需要管理依赖库

### 2.1 - Go 依赖管理演进

GOPATH -> Go Vendor -> Go Module

#### GOPATH

所有项目放到  `GOPATH` 下面的 `src` 目录下面

缺点：不利于多版本管理

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-57fc12.png)

#### Go Vendor

-  为了解决多版本依赖，在项目文件下新增 vendor 文件夹，将项目依赖以副本形式放到里面
- 每个项目引入一个依赖的副本

但是vendor还是依赖源码，还是会出现问题

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-d70b7c.png)

#### Go Mod

- 通过 go.mod 文件管理依赖包版本
- 通过 `go get XXX` 指令管理依赖包
- 可以定义版本规则和依赖关系

### 2.2 - 依赖管理三要素

1. 配置文件：go.mod
2. 中心仓库：Proxy
3. 本地工具：go get

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-3b0b73.png)

Indirect 关键字：不是直接依赖的库，是依赖的依赖

incompatible关键字：没有gomod文件且主版本2+的依赖

go在处理不同依赖时会选择最低兼容版本

依赖分发：

goproxy会缓存源站点的内容，goproxy保证不会修改软件源代码，降低了github等平台的压力。

配置 GOPROXY：

```Go
$ go env -w GO111MODULE=on
$ go env -w GOPROXY=https://goproxy.cn,direct
```

Gomod命令行

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-9ab210.png)

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-8e82d1.png)

## 3 - 测试

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-bf20a4.png)

回归测试：比如人工刷一下抖音，看看相关功能

集成测试：通过暴露的某个接口做自动化的测试

单元测试：开发者对单独函数模块做验证

### 3.1 - 单元测试

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-f1c7bc.png)

通过一个个的单元测试，保证代码的可用性

#### 单元测试规则

- 测试文件以`***_test.go` 命名
- 测试函数以 `func TestXXX(*testing.T)` 命名
- 测试初始化放到 `TestMain` 中
- 使用 `go test [flags] [packages]` 运行测试

#### 单元测试覆盖率

覆盖率越完备，那么代表程序越稳定

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-4cf114.png)

上面的代码只执行了3个语句中的两行，所以代码覆盖率为 66.7%，如果新增测试使得返回 false，覆盖率就是100%了

#### Tips

- 一般覆盖率：50%-60%，较高覆盖率80%
- 测试分支相互独立、全面覆盖
- 单元测试粒度足够小，意味着函数的职责单一

#### 单元测试依赖

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-c7dcee.png)

如果单元测试需要mysql、redis或者文件等的输入，因此单元测试的结果和mysql、redis相关了。

我们需要测试在任何时间，任何地点都可靠。这时需要使用mock机制，使用预先设置的假数据

常用的mock库是monkey：https://github.com/vektra/mockery

example：![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/16/Peek%202020-06-28%2000-08-707f01.gif)

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/02/01/asynccode-74c510.png)

patch可以篡改原来的函数，实现mock功能

### 3.2 - 基准测试

语法：

- `BenchmarkXXX(*testing.B)` 函数
- `b.ResetTimer()` 重置时间，表示忽略函数之前的动作耗时
- `b.RunParallel(func)` 并行运算

## 4 - 项目实战

一个社区话题页面

### 4.1 - 需求描述