# git 显示中文 

在git中，如果创建了中文文件，那么会被显示为一大堆数字，对于查看改动十分不方便：

![image-20231205085534464](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023%2F12%2F05%2Fimage-20231205085534464-1cfbca.png)

## 解决办法

修改配置`core.quotepath`项为`false`。

```sh
git config --global core.quotepath false
```

这样就可以显示中文了

![image-20231205085437288](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2023%2F12%2F05%2Fimage-20231205085437288-dd8364.png)

## 修改后中文乱码

这一般为**编码格式**不统一导致的，可能是终端使用`gbk`而git配置为了`utf-8`，抑或正好相反。

可以在git配置中修改字符编码：

```toml
[gui]
    encoding = utf-8
    # 代码库统一使用utf-8
[i18n]  
    commitencoding = utf-8
    # log编码
[svn]
    pathnameencoding = utf-8
    # 支持中文路径
[core]
	# 引用路径，就是上一条代码修改的
	quotepath = false
```

并执行：（因为 `git log` 默认使用 `less` 分页，所以需要 `bash` 对 `less` 命令进行 `utf-8` 编码

```sh
export LESSCHARSET=utf-8
```

