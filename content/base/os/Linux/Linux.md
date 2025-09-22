## 简介与基本概念

- 开源
- 一切皆文件
- 多任务、多用户
- 交互性非常好
  - 终端
  - GUI也支持
- 网络功能强大
- 多平台
- 支持多处理器

### shell是什么

- cli：command line interface，命令行
- terminal：终端，人与计算机交互的接口，早期终端是需要连接计算机上的外设
- console：控制台，系统管理员的终端
- shell：终端模拟器（现在都不用terminal和concole了）

![image-20230127230402032](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127230402032-6eea21.png)







## Linux用户管理

### 新建用户

```shell
sudo useradd [options] [username]
```

常用选项：

- `-d`：指定用户主目录
- `-g`：用户所在的用户组
- `-G`：用户附加组
- `-s`：用户登陆的shell

### 切换用户

```shell
su [username] # 不输入就是超级用户
```

不设密码不能`su`

### 设置密码

```
passwd [options] # 修改别人的需要超级用户
```

### 修改用户

如果用户组或者文件夹设置错了，可以改

```
sudo usermod [options] [username]
```







## Linux文件管理









## 常用Shell命令

### 查看帮助

```shell
man [命令名字]
```

![image-20230127233105626](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127233105626-1f7a09.png)

### 当前文件夹信息

```
ls
```

![image-20230127233502001](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127233502001-48dae9.png)



### 当前文件夹下所有文件信息

```
tree
```

![image-20230127233415530](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127233415530-b7ef0b.png)

### 切换目录

```
cd [PATH]
```

### 查找文件

```
find [options]
```

- `-path`：在哪个目录下找
- `-exec`：找到后怎么执行
- `-name`：文件名，-iname会忽略大小写
- `-amin [n]`：过去几分钟内使用过
- `-type`：文件类型，有dfl

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127234447620-ea5b35.png)

### 查找命令

```
whereis [command]
```

![image-20230127234950157](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127234950157-021f89.png)

### 创建文件

```
touch name # touch命令
> name # 使用管道
```

### 查看CPU占用

```
top # q退出
```

![image-20230127235229665](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127235229665-2293af.png)

### 查看进程

```shell
ps
```

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127235401049-c87bb0.png)

### 重定向

![image-20230127235623918](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/27/image-20230127235623918-17c7d3.png)

### 管道

将两个或者多个命令连接到一起，一个程序的输出可以作为下一个程序的输入

```
ls | grep 1 # 找文件名带1的
cat 1.txt | grep 'NO' # 找文件中带 'NO'的
cat P1000_1.in | python3 P1000.py # 将输入数据管道进程序中, oj用
echo {1..100} | tr ' '  '+' | bc# 计算 1-100 的和
```



## Shell脚本









## Shell环境变量









## SSH简介





