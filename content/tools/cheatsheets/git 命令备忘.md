---
tags:
  - cheatsheet
  - tool/git
created: 2023-03-16T00:43:34 (UTC +08:00)
---

> [!abstract]
> 一些常用 / 常忘的 git 命令



## 基础命令

### 分支

- `git checkout -b <branch_name>`：创建分支，并切换过去
- `git checkout master`：回到主分支
- `git push origin <branch_name>`：将分支推送到远程仓库
- `git pull`：将本地仓库更新
- `git diff <branch_name> master`：显示差别

### 克隆分支

- `git clone -b <branch_name> <repo_url>`：克隆单个分支
- `git branch -a`：查看所有分支
- `git checkout -b <branch_name> origin/<branch_name>`：关联分支
- `git clone --recursive`：克隆时一并克隆**子模组**

## 清除命令

- `git rm --cached <file>`：已 add 未 commit 的文件退回未 add 状态
- `git checkout -- <file>`：已修改的文件撤销修改
- `git reset --soft HEAD^`：撤销 commit（不更改文件）
- `git reset --hard HEAD^`：撤销 commit（文件回退到上一版本）
- `git update-ref -d HEAD`：撤销第一条 commit（不更改文件）
- `git push -f`：强制推送，覆盖 commit

## 查看

### git log

- `git log`: 查看commit信息
- `git log --oneline`：单行查看commit信息
- `git log --oneline`：查看commit信息，并以**图形化**方式显示分支和合并历史。

eg：

```
git log --oneline --graph
|\
| *   ed7e644 Merge pull request #1 from xigongyixue/main
| |\
| | * e466763 Create newfile
| |/
* | 1db2e2e change algo and datastructure to 'algods' folder
* | a10f49c delete useless algo problems
|/
...
```

### git diff

- `git diff --cached`：查看已经缓存的改动
- `git diff --stat`：现身改动的**摘要**信息
- `git diff -M[50]` 表示自动检查文件的**移动**，文件重复50%以上的会被看作修改，可以自行修改