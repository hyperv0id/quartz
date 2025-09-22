[QEMU](https://www.qemu.org/)：通用开源模拟器和虚拟器

## 工作方式

### 全系统模拟

模拟出完整的系统，包括处理器和外围设备。

命令：`qemu-system-{arch}`, eg: `qemu-system-aarch64`



### 用户模式

只负责翻译elf可执行文件，通过翻译系统调用的方式使用host内核&资源。



命令：`qemu-x86-64`



## 启动kernel


## 添加块设备



## 配置网络


## 共享宿主机目录

#todo 