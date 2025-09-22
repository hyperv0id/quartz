# 在 wsl 中使用代理

## 原理浅析

下面这个脚本可以设置当前网络代理, 有的应用如果发现了这个环境变量就会自动使用代理

```bash
export HTTP_PROXY="XXX"
```

在 windows 宿主机上我们会设置 `HTTP_PROXY="http://127.0.0.1:1080"`, 但是wsl 基于 hyperv, 使用的是独立的网络, 因此不能将 wsl 的代理地址设置为 127.0.0.1, 需要设置为 `宿主机IP`.

宿主机(也叫本地DNS) 定义在 /etc/resolv.conf 文件下, 可使用 `cat` 命令查看

```bash
cat /etc/resolv.conf
```

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/31/20230331010244-c4b169.png)

## 代理设置

使用一下脚本提取 IP 部分, 将其作为参数保存, 便于设置代理

```bash
LOCAL_DNS=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
```

接下来设置代理: 

1. 将下面这段文字写入 `~/.bashrc` 或 `~/.profile` 等登录后立刻执行的文件中
2. 如果写在 `~/.profile` 中, 需要 `source ~/.profile` 

```bash
# ~/.profile
LOCAL_DNS=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export ALL_PROXY="socks5://$LOCAL_DNS:10808"
export HTTP_PROXY=$ALL_PROXY
```

注意端口号需要和宿主机的端口号一致:
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/31/20230331010758-7d2518.png)

## 检查设置

重新登录, 查看代理是否生效:

```
$ echo $ALL_PROXY
socks5://192.168.176.1:10808

$ echo $HTTP_PROXY
socks5://192.168.176.1:10808
```

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/31/20230331011439-5edb61.png)

# 设置常用应用的代理

## git

### 设置代理

#### 普通http代理
- `git config --global http.proxy ``http://127.0.0.1:1234`
- `git config --global https.proxy ``https://127.0.0.1:1234`
- 单独给GitHub设置代理：
  - 设置：`git config --global http.``https://github.com.proxy`` socks5://127.0.0.1:1080`
  - 取消：`git config --global --unset http.``https://github.com.proxy`

#### ssh代理

如果使用`ssh`连接需要单独设置

1. 编辑ssh配置文件: `.ssh/conf`
```
Host github.com
	# windows 系统
	ProxyCommand nc -v -x 192.168.10.120:7890 %h %p
	# Linux系统
	ProxyCommand nc --proxy-type socks5 --proxy 127.0.0.1:7891 %h %p
```


### 取消代理

- git config --global --unset http.proxy
- git config --global --unset https.proxy

## curl

### 使用参数, 临时代理

curl 使用代理的参数是 `-x`, 或 `--proxy`
```
curl -x $ALL_PROXY http://www.google.com # -x 参数等同于 --proxy
```

### 每次都代理

修改curl配置文件 `vim ~/.curlrc`, 写入以下字段:

```bash
socks5 = "127.0.0.1:1080"
```

