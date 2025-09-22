# **Debian / Ubuntu**

v2rayA 的功能依赖于 V2Ray 内核，因此需要安装内核。

## **安装 v2rayA**

### **方法一：通过软件源安装**

#### **添加公钥**
```bash
wget -qO - https://apt.v2raya.org/key/public-key.asc | sudo tee /etc/apt/keyrings/v2raya.asc
```
#### **添加 V2RayA 软件源**
```bash
echo "deb [signed-by=/etc/apt/keyrings/v2raya.asc] https://apt.v2raya.org/ v2raya main" | sudo tee /etc/apt/sources.list.d/v2raya.list

sudo apt update
```
#### **安装 V2RayA**
```bash
sudo apt install v2raya v2ray ## 也可以使用 xray 包
```
### **方法二：手动安装 deb 包**

[从 Release 下载 v2rayA 的 deb 包](https://github.com/v2rayA/v2rayA/releases)

后可以使用 Gdebi、QApt 等图形化工具来安装，也可以使用命令行：
```bash
sudo apt install /path/download/installer_debian_xxx_vxxx.deb ### 自行替换 deb 包所在的实际路径
```
V2Ray / Xray 的 deb 包可以在 [APT 软件源中](https://github.com/v2rayA/v2raya-apt/tree/master/pool/main/)找到。

## **启动 v2rayA / 设置 v2rayA 自动启动**

> 从 1.5 版开始将不再默认为用户启动 v2rayA 及设置开机自动。

- 启动 v2rayA
    
```bash
sudo systemctl start v2raya.service
```
设置开机自动启动
```bash
sudo systemctl enable v2raya.service
``` 

## **使用 nftables**

如果你的系统上已经有了 `nftables` 防火墙，那么 v2rayA 将优先使用 `nft` 命令来创建防火墙规则。可以使用 `--nftables-support` 参数或者 `V2RAYA_NFTABLES_SUPPORT` 来进行控制是否开启 nftables 支持。