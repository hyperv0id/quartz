## 安装

### windows下wsl2安装

Use Redis on Windows for development

Redis is not officially supported on Windows. However, you can install Redis on Windows for development by following the instructions below.

To install Redis on Windows, you'll first need to enable [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install) (Windows Subsystem for Linux). WSL2 lets you run Linux binaries natively on Windows. For this method to work, you'll need to be running Windows 10 version 2004 and higher or Windows 11.

### Install or enable WSL2

Microsoft provides [detailed instructions for installing WSL](https://docs.microsoft.com/en-us/windows/wsl/install). Follow these instructions, and take note of the default Linux distribution it installs. This guide assumes Ubuntu.

#### Install Redis

Once you're running Ubuntu on Windows, you can follow the steps detailed at [Install on Ubuntu/Debian](https://redis.io/docs/getting-started/installation/install-redis-on-linux#install-on-ubuntu-debian) to install recent stable versions of Redis from the official `packages.redis.io` APT repository. Add the repository to the `apt` index, update it, and then install:

```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
```

Lastly, start the Redis server like so:

```bash
sudo service redis-server start
```

#### Connect to Redis

You can test that your Redis server is running by connecting with the Redis CLI:

```bash
redis-cli 
127.0.0.1:6379> ping
PONG
```

## 配置

官方配置地址：[Redis configuration | Redis](https://redis.io/docs/management/config/)

### ubuntu配置

配置文件地址：`/etc/redis/redis.conf`



### 配置外部访问

将 `bind 127.0.0.1 -::1` 注释掉 

```
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# COMMENT OUT THE FOLLOWING LINE.
#
# You will also need to set a password unless you explicitly disable protected
# mode.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# bind 127.0.0.1 -::1
```

### 设置密码

将下面的密码改成你想要的

```
# requirepass foobared
```

比如说：

```
requirepass 123456
```

此时cli能访问但不能使用：

```bash
root@DESKTOP-OTLK4SJ:/etc/redis# redis-cli
127.0.0.1:6379> set name cjj
(error) NOAUTH Authentication required.
127.0.0.1:6379> get name
(error) NOAUTH Authentication required.
```

登录后能使用：

```bash
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> set name cjj
OK
127.0.0.1:6379> get name
"cjj"
```

### 关闭保护模式

如果客户端连接不上，可能是没有关闭保护模式

```
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured.
protected-mode yes
```

改成

```
protected-mode no
```

### 删除/禁用命令

删除危险命令

```
rename-command flushall ""
```

## 客户端

### redis-plus

[RedisPlus: RedisPlus是为Redis可视化管理开发的一款开源免费的桌面客户端软件(gitee.com)](https://gitee.com/MaxBill/RedisPlus)

下载解压双击运行

配置并连接：

![image-20230101173300041](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/01/image-20230101173300041-2a83a5.png)



![image-20230101173324363](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023/01/01/image-20230101173324363-5d1c09.png)



### Java客户端

