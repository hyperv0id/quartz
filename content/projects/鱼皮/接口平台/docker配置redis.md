```
docker run \
-p 6379:6379 --name myredis \
-v/Users/bochenghu/onlineinternship/redis/redis.conf:/etc/redis/redis.conf \
-v /Users/bochenghu/onlineinternship/redis/data:/data:rw \
--privileged=true -d redis redis-server /etc/redis/redis.conf \
--appendonly yes
```

参数解释说明：

- -p 6379:6379 端口映射：前表示主机部分，：后表示容器部分。
- --name myredis 指定该容器名称，查看和进行操作都比较方便。
- -v 挂载配置文件目录，规则与端口映射相同。
  为什么需要挂载目录：个人认为docker是个沙箱隔离级别的容器，这个是它的特点及安全机制，不能随便访问外部（主机）资源目录，所以需要这个挂载目录机制。
  例： -v /usr/local/docker/redis/redis.conf:/etc/redis/redis.conf 容器 /etc/redis/redis.conf 配置文件 映射宿主机 /usr/local/docker/redis/redis.conf。 会将宿主机的配置文件复制到docker中
  **重要： 配置文件映射，docker镜像redis 默认无配置文件。**
- -v 挂在数据卷
  /Users/bochenghu/onlineinternship/redis/data:/data:rw
  映射数据目录 rw 为读写
- -d redis 表示后台启动redis
- redis-server /etc/redis/redis.conf 以配置文件启动redis，加载容器内的conf文件，最终找到的是挂载的目录/usr/local/docker/redis/redis.conf
  **重要: docker 镜像redis 默认 无配置文件启动**
- --appendonly yes 开启redis 持久化