## git使用

以这个仓库为例，可以看到现在已经写了一部分了

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-20-caada8.png)

### 命令行

- `git status`: 查看还有哪些没有提交
  - ![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-1-a8df2a.png)
- `git add --all`: 保持所有文件的改动
- `git commit -m "对改动描述"`: 提交改动
- `git remote XXX`：添加远程仓库
- `git push AAA BBB`: 把改动提交到远程仓库(GitHub)
  - AAA表示是哪一个远程仓库，一般是 origin
  - BBB 表示本地的分支，一般是当前的分支(main)
- `git log`: git 提交记录
  - ![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-2-fed690.png)
- `git branch XXX`: 创建分支
  - 在两个分支上做不同的事情
  - 比如说在 `main` 分支上维护稳定的，在 `test` 分支执行测试
- `git merge XXX`: 将别的分支的改动搬回到当前分支，不认识`test`测试完毕，改回`main`

### 其他命令补充

#### 临时保存文件

当你保存了某个文件，需要切换分支，但是不想提交

```Bash
git stash save "xxx"
```

#### 日志单行流程图

```Bash
git log --oneline --graph
```

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-3-5dc7b4.png)

#### 取消追踪某个文件

不小心commit了一个不需要的文件

```Bash
git rm --cached filename
```

#### 提交但是记录到上一次提交

```Bash
git commit --amend
```

#### 提交特定一次修改

```Bash
git cherry-pick commit-id
```

#### 文件快照打包

```Bash
git archive -o filename.zip master
```

### 图形化

#### Visual studio

改动操作

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-4-01854d.png)

分支操作

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-5-727e32.png)

#### Vscode

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-6-818682.png)

更多操作可以在这里看

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-7-bc5daa.png)

## GitHub使用

### 准备

现有一仓库：bustub

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-8-906d5d.png)

### fork

另一用户点击 `fork` 会将仓库克隆到自己名下

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-9-2321f9.png)

现在做一些改动

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144761-10-30558c.png)

### 提PR

现在要告诉作者

1. 点击contribute
   1. ![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-11-ae9761.png)
2. 点击Open pull request（PR）
3. 提交，最好描述一下你干了什么
   1. ![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-12-21f7ed.png)

这个页面可以看到PR信息

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-13-3aa519.png)

### 处理PR

回到作者的仓库，作者会看到一个PR

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-14-4bb549.png)

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-15-1ca942.png)

作者可以选择接受或者拒绝PR

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-16-ea2e0b.png)

作者的仓库也改动了

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-17-7d9b85.png)

### GitHub团队协作

私有的仓库里可以在设置里添加成员

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-18-451c57.png)

Accept

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/14/1678726144762-19-439828.png)

> You now have push access to the hyperv0id/group-project repository.       

成员的更改会立刻同步到主仓库！！！

要取消需要单独设置

最佳方案是使用fork，让仓库管理员负责（背锅）

## 其他

### [Git 代理](Git%20代理.md)

### [git 中文支持](git%20中文支持.md)

