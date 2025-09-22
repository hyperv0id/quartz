## 基本概念

### 本地函数调用

eg：

```go
func main()
	var a =2
	var b =3
	result calculate(a,b)
	fmt.Println(result)
	return
}
func calculate(x,y int)
	Z :x*y
	return Z
}
```

执行步骤：

1. 将值 `a,b` 压栈
2. 找到 `calculate` 函数，进入函数栈中给 `x,y` 赋值
3. 计算 `x*y`，结果保存在`z` 中
4. 从栈中取回`z`返回值，赋值给`result`,打印



### 远程函数调用（RPC - Remote Procedure Call）

<img src="https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/24/image-20230124164842966-d729e4.png" alt="image-20230124164842966" style="zoom:50%;" />

RPC和本地函数调用不同，RPC需要解决一下问题：

1. 函数映射
2. 数据流转换成字节流
3. 网络传输

#### 网络映射

本地函数调用直接找函数指针就行了，但是远程不行，地址空间都是不一样的，所以我们给每个函数给一个ID，在传输时附带id，执行时寻找。

#### 数据流转换成字节流

本地：压栈，出栈

远程：客户端转换成字节流给服务端，服务端再转换成自己能读取的形式

#### 网络传输

高效稳定地传输数据


## RPC概念模型

![image-20230124165538874](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/24/image-20230124165538874-59a844.png)

Nelson提出rpc的过程由5个模型组成：User、User-Stub、RPC-Runtime、Server-Stub、Server

## 一次RPC的完整模型

**IDL**：Interface description Language，IDL使用中立的方式来描述接口，使得不同平台上的对象和不同语言编写的程序可以互相通信。

**生成代码**：指编译器工具将IDL转换成语言对应的静态库

**编解码**：也叫序列化和反序列化，从内存中表示到字节序列称为编码，反之为解码。

**通信协议**：规范了数据再网络中的传输内容和格式。除必须的请求/响应数据外，还包括额外的元数据

**网络传输**：基于成熟的网络库走 `TCP/UDP` 协议



![image-20230124170148887](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/24/image-20230124170148887-fe2c70.png)


## RPC优缺点
### 好处

1. 单一职责，有利于分工协作和运维开发
2. 可扩展性强，资源使用率更优
3. 故障隔离，服务整体可靠性更高

<img src="https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/25/image-20230125134331494-e2ce14.png" alt="image-20230125134331494" style="zoom:67%;" />



### 缺点

引出了新问题：
1. 服务宕机如何处理
2. 网络异常，如何保障消息的可达性
3. 请求量突然增大如何及时处理？

![image-20230125134456951](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/25/image-20230125134456951-40ed01.png)

## 小结

1. 本地函数调用和RPC调用的区别：函数映射、数据流转换成比特流、网络传输
2. RPC概念模型：User、User-Stub、RPC-Runtime、Server-Stub、Server
3. RPC的完整过程
4. 优缺点