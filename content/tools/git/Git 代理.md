## Git 代理

### 设置代理

- `git config --global http.proxy ``http://127.0.0.1:1234`
- `git config --global https.proxy ``https://127.0.0.1:1234`
- 单独给GitHub设置代理：
  - 设置：`git config --global http.``https://github.com.proxy`` socks5://127.0.0.1:1080`
  - 取消：`git config --global --unset http.``https://github.com.proxy`
- 如果使用`ssh`连接需要单独设置
  - 编辑ssh配置文件（好像是`.ssh/conf`）
  - 
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