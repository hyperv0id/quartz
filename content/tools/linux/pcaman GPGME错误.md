## 问题描述

在更新arch系统时出现错误：

```sh
# sudo pacman -Syu  
错误：GPGME 错误：无数据  
错误：GPGME 错误：无数据  
错误：未能同步所有数据库（无效或已损坏的数据库 (PGP 签名)）
```

## 解决方案

```sh
sudo rm -r /var/lib/pacman/sync
# sudo rm /var/lib/pacman/sync/*.sig
```

## 原理

pacman 更新系统时会尝试下载数据库签名`$*.db.sig`，但大多数数据库没有签名，因此不会影响。

但是在**校园网**等需要登陆的环境中，会被重定向到登陆页面，因此pcaman获得的是错误的签名，就会报这个错误。



## Reference

> pacman更新时遇到「GPGME 错误：无数据」
>
> https://zhul.in/2022/01/01/pacman-gpgme-error-no-data/
>
> 作者：竹林里有冰
>
> 发布于：2022年1月1日
>
> 更新于：2022年8月22日
