# lab1-util

## 启动xv6（简单）

下载代码并切换分支：

```bash
$ git clone git://g.csail.mit.edu/xv6-labs-2021
Cloning into 'xv6-labs-2021'...
...
$ cd xv6-labs-2021
$ git checkout util
Branch 'util' set up to track remote branch 'util' from 'origin'.
Switched to a new branch 'util'
```

编译运行，make编译，qemu模拟

```bash
$ make qemu # <C-a> + x退出
```

![image-20230130140010733](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/30/image-20230130140010733-b9b57d.png)

## sleep（简单）

实现unix`sleep`程序，sleep程序可以按照用户指定的时间停止程序。

程序需要在 `user/sleep.c`中实现，文件未创建，我们将`user/echo.c`拷贝成我们的模板：

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h" # 三者引用关系不能交换顺序

int main(int argc, char *argv[])
{
  exit(0);
}
```

在`user.h`中已经定义了sleep函数，这里直接使用

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h" // 会使用 user.h 中的 atoi 函数

int main(int argc, char *argv[])
{
  // 如果用户未指定sleep时间，那么sleep 一秒
  if (argc < 2)
  {
    printf("Require sleep time, use: sleep 1");
    exit(1);
  }
  int sleep_time = atoi(argv[1]);
  printf("(nothing happens for a little while)");
  sleep(sleep_time);
  exit(0);
}

```

将`sleep`程序写入`MakeFile`中的`UPROGS`里，

```makefile
UPROGS=\
	...
	$U/_sleep\ # here!!!
```

测试程序：

```bash
$ ./grade-lab-util sleep
```

![image-20230130152357307](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/01/30/image-20230130152357307-cc297e.png)

## pingpong（简单）

使用UNIX系统调用在两个进程之间通过一对管道“乒乓”一个字节，每个方向一个管道。

1. 父进程应该向子进程发送一个字节
2. 子进程应该输出：`<pid>: received ping`"，其中`<pid>`是它的进程号，把这个字节写在管道上给父进程，然后退出
3. 父进程应该从子进程读取字节，打印`<pid>: received pong`，然后退出。

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[])
{
    // 0用于读，1用于写
    int p2c[2], c2p[2]; // 两个管道
    pipe(p2c);          // 父进程 -> 子进程 的管道
    pipe(c2p);          // 子进程 -> 父进程 的管道

    if (fork() != 0)
    {
        // I'm Parent
        write(p2c[1], "!", 1); // send to child
        char buf;
        read(c2p[0], &buf, 1); // read from child
        printf("%d: received pong\n", getpid());
        wait(0);
    }
    else
    {
        // I'm Child
        char buf;
        read(p2c[0], &buf, 1); // read from parent
        printf("%d: received ping\n", getpid());
        write(c2p[1], "!", 1); // send to child
    }
    exit(0);
}
```



## primes（中难）

使用管道实现求质数，看不懂要干啥的看这里：[Bell Labs and CSP Threads](https://swtch.com/~rsc/thread/)
![img](https://swtch.com/~rsc/thread/sieve.gif)


基本原理还是[埃式筛](https://oi-wiki.org/math/number-theory/sieve/#%E5%9F%83%E6%8B%89%E6%89%98%E6%96%AF%E7%89%B9%E5%B0%BC%E7%AD%9B%E6%B3%95)，直接原来是开一个数组存bool，现在将开一个数组存子进程

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

/**
 * p = get a number from left neighbor
 * print p
 * loop:
 *      n = get a number from left neighbor
 *      if (p does not divide n)
 *          send n to right neighbor
 */
void prime(int msg)
{
    int n;
    if (read(msg, &n, 4) == 0)
        exit(0);

    while ((read(msg, &n, 4) != 0))
    {
        if (msg % n != 0)
            printf("prime %d\n", n);
    }
}

int main(int argc, char *argv[])
{
    int pip[2]; // pipe
    pipe(pip);

    int pid = fork();
    if (pid != 0)
    {
        // root
        close(pip[0]); // do not read
        for (int i = 2; i <= 35; i++)
        {
            write(pip[1], &i, 4); // send message to childs
        }
        close(pip[1]); // write done
        wait(0);
    }
    else
    {
        // first child
        close(pip[1]); // i won't write
        prime(pip[0]); // read
        close(pip[0]); // read done
    }
    exit(0);
}
```

这里的`main`方法和`prime`几乎是等价的

可以先改变一下题目要求：

> main 函数生成 2-35 之间的数字，将其发送给子进程，让其输出

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

void prime(int msg)
{
    int n;
    while ((read(msg, &n, 4) != 0))
    {
        printf("%d is prime\n", n);
    }
}

int main(int argc, char* argv[]){
    int pip[2]; // pipe
    pipe(pip);

    int pid = fork();
    if(pid != 0){
        // root
        close(pip[0]); // do not read
        for (int i = 2; i <= 35; i++)
        {
            write(pip[1], &i, 4); // send message to childs
        }
        close(pip[1]);// write done
        wait(0);

    }else{
        // first child
        close(pip[1]); // i won't write
        prime(pip[0]); // read
        close(pip[0]); //read done

    }
    exit(0);
}

```


然后在改编一下，子进程会拿到第一个消息 n 后，只会输出与之不互质的

```c
void prime(int msg)
{
    int n;
    if (read(msg, &n, 4) == 0)
        exit(0);

    while ((read(msg, &n, 4) != 0))
    {
        if (msg % n != 0)
            printf("%d is prime\n", n);
    }
}
```

然后应该很容易想到如何送给子进程了


## find（中等）

```c
// 网上抄的，懒得写字符串匹配了
// https://clownote.github.io/2021/02/24/xv6/Xv6-Lab-Utilities/
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"
#include "kernel/fcntl.h"

char *fmtname(char *path)
{
    char *p;

    // Find first character after last slash.
    for (p = path + strlen(path); p >= path && *p != '/'; p--)
        ;
    p++;

    return p;
}

void find(char *path, char *targetname)
{
    char buf[512], *p;
    int fd;
    struct dirent de;
    struct stat st;

    if (!strcmp(fmtname(path), targetname))
    {
        printf("%s\n", path);
    }

    if ((fd = open(path, O_RDONLY)) < 0)
    {
        fprintf(2, "find: cannot open [%s], fd=%d\n", path, fd);
        return;
    }

    if (fstat(fd, &st) < 0)
    {
        fprintf(2, "find: cannot stat %s\n", path);
        close(fd);
        return;
    }

    if (st.type != T_DIR)
    {
        close(fd);
        return;
    }

    // st.type == T_DIR

    if (strlen(path) + 1 + DIRSIZ + 1 > sizeof buf)
    {
        printf("find: path too long\n");
        close(fd);
        return;
    }
    strcpy(buf, path);
    p = buf + strlen(buf);
    *p++ = '/';
    while (read(fd, &de, sizeof(de)) == sizeof(de))
    {
        if (de.inum == 0)
            continue;
        memmove(p, de.name, DIRSIZ);
        p[DIRSIZ] = 0;

        if (!strcmp(de.name, ".") || !strcmp(de.name, ".."))
            continue;

        find(buf, targetname);
    }
    close(fd);
}

int main(int argc, char *argv[])
{
    if (argc < 3)
    {
        fprintf(2, "usage: find path filename\n");
        exit(1);
    }

    find(argv[1], argv[2]);

    exit(0);
}

```







## xargs（中等）



```c
// https://clownote.github.io/2021/02/24/xv6/Xv6-Lab-Utilities/
#include "kernel/types.h"
#include "user/user.h"

#define is_blank(chr) (chr == ' ' || chr == '\t')

int main(int argc, char *argv[])
{
    char buf[2048], ch;
    char *p = buf;
    char *v[1024];
    int c;
    int blanks = 0;
    int offset = 0;

    if (argc <= 1)
    {
        fprintf(2, "usage: xargs <command> [argv...]\n");
        exit(1);
    }

    for (c = 1; c < argc; c++)
    {
        v[c - 1] = argv[c];
    }
    --c;

    while (read(0, &ch, 1) > 0)
    {
        if (is_blank(ch))
        {
            blanks++;
            continue;
        }

        if (blanks)
        { // 之前有过空格
            buf[offset++] = 0;

            v[c++] = p;
            p = buf + offset;

            blanks = 0;
        }

        if (ch != '\n')
        {
            buf[offset++] = ch;
        }
        else
        {
            v[c++] = p;
            p = buf + offset;

            if (!fork())
            {
                exit(exec(v[0], v));
            }
            wait(0);

            c = argc - 1;
        }
    }

    exit(0);
}

```





## 提交

