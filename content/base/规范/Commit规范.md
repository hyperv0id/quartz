# Commit 规范



代码开发时，经常需要提交代码，提交代码时需要填写 `Commit Message`（提交说明），否则就不允许提交。

而在实际开发中，我发现每个研发人员提交 `Commit Message` 的格式可以说是五花八门，有用中文的、有用英文的，甚至有的直接填写“11111”。这样的 Commit Message，时间久了可能连提交者自己都看不懂所表述的修改内容，更别说给别人看了。



并且一个好的Commit带来的好处是显而易见的：

1. **提高团队协作效率**：统一的提交信息格式有助于团队成员之间的沟通和理解。
2. **便于代码审查**：规范的提交信息使得代码审查者能更快地理解变更的内容和目的。
3. 可以基于这些 Commit Message 进行**过滤查找**，比如只查找某个版本新增的功能：`git log --oneline --grep "^feat|^fix|^perf"`
4. 可以基于规范化的 Commit Message 生成 **Change Log**。
5. **触发构建或者发布流程**：比如当 type 类型为 feat、fix 时我们才触发 CI 流程。
6. **确定语义化版本的版本号**：比如 `fix` 类型可以映射为 `PATCH` 版本，`feat` 类型可以映射为 `MINOR` 版本。带有 `BREAKING CHANGE` 的 ，可以映射为 `MAJOR` 版本。

## Angular规范

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/07/01/1699f5c1933cfe72803dfb038152fc48-f07750.png)



Angular 规范是一种语义化的提交规范（Semantic Commit Messages），所谓语义化的提交规范包含以下内容：

- 语义化的：Commit Message 都会被归为一个有意义的类型，用来说明本次 commit 的类型。
- 规范化的：Commit Message 遵循预先定义好的规范，比如 Commit Message 格式固定、都属于某个类型，这些规范不仅可被开发者识别也可以被工具识别。

### 基本结构

Angular commit 规范的基本结构如下：（`[]`表示可选）

```
<type>[（scope）]: <subject>
<空行>
<body>
<空行>
[footer]
```

### 主要组成部分

1. **类型 (type)**：必须是以下之一：
   - feat: 新功能
   - fix: 修复 bug
   - docs: 文档更改
   - style: 不影响代码含义的更改（空白、格式化、缺少分号等）
   - refactor: 既不修复 bug 也不添加新功能的代码更改
   - perf: 提高性能的代码更改
   - test: 添加缺失的测试或更正现有的测试
   - build: 影响构建系统或外部依赖项的更改（例如：gulp, broccoli, npm）
   - ci: 对 CI 配置文件和脚本的更改
   - chore: 其他不修改 src 或 test 文件的更改

2. **作用域 (scope)**：可选，用括号包围，指定 commit 影响的范围

3. **简短描述 (subject)**：
   - 使用祈使语气，现在时态
   - 不要大写第一个字母
   - 结尾不要加句号

4. **正文 (body)**：可选，应该包含更改的动机，并与以前的行为进行对比

5. **脚注 (footer)**：可选，用于说明 Breaking Changes 或关闭 Issue

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/07/01/89c618a7415c0c38b09d86d7f882a427-20bf91.png)



![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/07/01/3509bd169ce285f59fbcfa6ebea75aa7-a7cc33.png)



### 示例

1. 简单的 feature 提交：
   ```
   feat(user): add email verification
   ```

2. 带有作用域和正文的 bug 修复：
   ```
   fix(auth): correct JWT token validation
   
   The previous implementation did not properly check the token expiration.
   This commit adds a proper check to ensure that expired tokens are rejected.
   ```

3. 带有 Breaking Change 的提交：
   ```
   feat(api): rename user endpoint
   
   BREAKING CHANGE: The user endpoint has been renamed from `/user` to `/users`.
   All API calls to the old endpoint will now return a 404 error.
   ```

4. 关闭 Issue 的提交：
   ```
   fix(dashboard): correct data rendering
   
   Closes #123
   ```

### 特殊规则

1. Breaking Changes 必须在 footer 中注明，以 "BREAKING CHANGE:" 开头
2. 如果 type 是 feat 或 fix，必须在 changelog 中出现
3. 如果存在 scope，必须用括号包围：例如 `feat(parser):`



## Conventional Commits 规范

### 基本结构

一个符合 Conventional Commits 规范的提交消息应该是这样的：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### 主要组成部分

1. **类型 (type)**：描述本次提交的类型，必须是以下类型之一：
   - feat: 新功能
   - fix: 修复 bug
   - docs: 文档更新
   - style: 代码格式修改，不影响代码逻辑
   - refactor: 代码重构，既不修复 bug 也不添加新功能
   - perf: 性能优化
   - test: 添加或修改测试用例
   - build: 构建系统或外部依赖项的更改
   - ci: 持续集成的配置文件和脚本的更改
   - chore: 其他不修改源代码与测试文件的更改

2. **作用域 (scope)**：可选，用于说明 commit 影响的范围，比如数据层、控制层、视图层等等

3. **描述 (description)**：简短描述，不超过 50 个字符

4. **正文 (body)**：可选，对本次 commit 的详细描述，可以分成多行

5. **脚注 (footer)**：可选，用于关闭 Issue 或破坏性变更的描述

### 示例

1. 简单提交：
   ```
   feat: 添加用户登录功能
   ```

2. 带作用域的提交：
   ```
   fix(auth): 修复登录验证的 bug
   ```

3. 带正文和脚注的提交：
   ```
   feat: 允许用户配置登录选项
   
   现在用户可以选择记住登录状态，也可以选择使用双因素认证。
   
   BREAKING CHANGE: 登录 API 的接口已更改，不再兼容旧版本。
   ```





## 协作规范

### 提交频率

如果是个人项目，随意 commit 可能影响不大，但如果是多人开发的项目，随意 commit 不仅会让 Commit Message 变得难以理解，还会让其他研发同事觉得你不专业。

因此，我们要规定 commit 的提交频率。那到底什么时候进行 commit 最好呢？

1. 进行了修改，一通过测试就立即 commit：比如修复完一个 bug、开发完一个小功能，或者开发完一个完整的功能，测试通过后就提交
2. 我们规定一个时间，定期提交：这里我建议代码下班前固定提交一次，并且要确保本地未提交的代码，延期不超过 1 天。这样，如果本地代码丢失，可以尽可能减少丢失的代码量。

###  commit 太多怎么办

如果想等开发完一个完整的功能之后，放在一个 commit 中一起提交。这时候，我们可以在最后合并代码或者提交 `Pull Request` 前，执行 `git rebase -i` 合并之前的所有 commit。



### 合并提交

就是将多个 commit 合并为一个 commit 提交。

合并到主干时，使用 `git rebase` 命令来合并，只保留 2~3 个 commit 记录。

#### git rebase 命令介绍

git rebase 的最大作用是它可以重写历史。

我们通常会通过 `git rebase -i` 使用 `git rebase` 命令，`-i` 参数表示交互（`interactive`），该命令会进入到一个交互界面中，其实就是编辑器。在该界面中，我们可以对里面的 commit 做一些操作，交互界面如图所示：

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/07/01/c63a8682c03862802e5eacf1641b86ac-fc6cbb.png)

这个交互界面会首先列出给定`Commit ID`之前（不包括，越下面越新）的所有 commit，每个 commit 前面有一个操作命令，默认是 pick。

可以选择不同的 commit，并修改 commit 前面的命令，来对该 commit 执行不同的变更操作。

![img](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/2024/07/01/5f5a79a5d2bde029d4de9d98026ef3f2-eeed54.png)