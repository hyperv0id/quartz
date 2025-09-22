什么是IO：什么与计算机最先接触到的设备

对于计算机系统来说，如果没有Io设备，它仅仅是一个有处理器和内存的简化系统。处理器从内存中读取数据进行处理，然后再将处理后的数据放回内存。但是，如果系统无法与外部世界进行交互，它对我们的日常生活影响就有限了。

- **基本原理**：是通过一些寄存器与外部设备进行交互。
	- eg：键盘，当按下键时，就会触发一个开关，产生一个信号，通过寄存器将按键信息传递给CPU。类似地，鼠标通过类似摄像头的方式捕捉移动信息，将这些信息转化成控制数据，再传输给CPU。
- 这些设备能与计算机系统交互的基础是**数据交换**。寄存器通过**记录**设备的状态、指令命令、信号和数据等信息，使得CPU能够与Io设备进行有效的交流和控制。比如对打印机发送打印指令、传输需要打印的数据等。


![IO方式1](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105161622-3d2061.png)


设备处理器接口：

- 设备：一组寄存器，每次交换一定量的数据
- 每个寄存器有特地的含义，格式
- 主要功能：读取、写入、配置

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105161855-778662.png)



IO通过总线连接到CPU，不同的引脚对应不同的外设端口。eg：抽象的USB端口

![IO方式2](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105162056-4ea6fb.png)

## 实际处理接口

我们实际上设备和我们的CPU的这样的一个处理的接口,基本上就这两大类：
- 一类是基于这样的一个**总线**的形式,实际上也是基于我们之前的这样类似于是CPU的端口,
- 第二种就是基于Memory的形式，也就是**mmap**，我们向IO写东西，相当于向特别的内存地址写东西。

### 优化？

如果我们向这个特别的内存一直写入0,编译器优化会删除循环，和我们的预期不符

![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105163105-08e1c3.png)

### x86-qemu URAT
putch实现：
![putch实现](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105163219-70dea3.png)


程序通过QEMU来建模了一个串口，绑定了3个寄存器，用来控制struct和data，来输入输出。

### 现代IO设备

打印机与PostScript
打印机：将字节流描述的文本/图形打印到纸上
- 简单：ASCII文本序列，打字机like
- 复杂：高清4k图片

一个复杂的pdf文件可能会传输更精简的postscript命令，和其他设备相同，其自身也是一个复杂的计算机系统。

### 两种特殊的IO设备

- 总线
	- 系统有很多IO设备
	- 总线的功能：设备查找、映射、数据（命令）转发
	- CPU可以只连接到其他总线
	- 总线可以连接到其他总线
	- PCI是一个多级总线管理的一种协议，我有一个总的PCI总线，我下面还有不同的比如USB总线来管理其他设备资源，比如说GPU、SSD
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105164250-03a058.png)


可以使用lspci命令查看设备PCI连接的设备
```sh
$ lspci
00:00.0 Host bridge: Intel Corporation Device a703 (rev 01)
00:1c.2 PCI bridge: Intel Corporation Device 7aba (rev 11)
00:1f.0 ISA bridge: Intel Corporation Device 7a86 (rev 11)
00:1f.3 Audio device: Intel Corporation Alder Lake-S HD Audio Controller (rev 11)
00:1f.4 SMBus: Intel Corporation Alder Lake-S PCH SMBus Controller (rev 11)
00:1f.5 Serial bus controller: Intel Corporation Alder Lake-S PCH SPI Controller (rev 11)
01:00.0 VGA compatible controller: NVIDIA Corporation GA104 [GeForce RTX 3060 Ti GDDR6X] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GA104 High Definition Audio Controller (rev a1)
02:00.0 Non-Volatile memory controller: ...
03:00.0 Non-Volatile memory controller: ...
04:00.0 USB controller: ...
05:00.0 Ethernet controller: ...
```

- 中断控制器
	- 管理多个产生中断的设备
	- 汇总成一个中断信号给CPU
	- 支持中断屏蔽、优先级管理
	- OS以前被看作调度中断的中间件
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105164701-7890b2.png)

## 2D图形绘制硬件

### 过去的游戏

6502芯片没有什么功能、内存、指令

游戏交互的要求：速度不能太慢、有交互

**双重循环**：

一个个素材拼接起来
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105170037-85652b.png)

后面PPT显示错误了TAT

## 3D图形绘制硬件

更好看、更美丽、更细节的计算

三维空间的多边形在视平面上也是多边形，任何n边型都可以切成多个三角形。

2000年左右，只能对二维进行拉伸
![2000年代的3D](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105170436-ba7a45.png)

之后，转换视角、渲染管道、


#### CUDA编程

GPU绘画
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105170723-82496d.png)

0809年的效果：船帆、水波、云、、、
![image.png](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/01/05/20240105170741-0c57f2.png)


