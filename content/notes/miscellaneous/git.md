---
title: Git Fundamentals
type: docs
---

## 远程仓库（remote）

### 基本信息（-v）

载入远程仓库后，可使用 `git remote -v` 然后查看信息。可以看出，`origin` 就是远程仓库位置：

```shell
$ git clone https://github.com/tianqixin/runoob-git-test  
$ cd runoob-git-test  
$ git remote -v  
origin  https://github.com/tianqixin/runoob-git-test (fetch)  
origin  https://github.com/tianqixin/runoob-git-test (push)  
```

可以使用 `git remote show [remote]` 查看当前远程仓库信息：

```shell
$ git remote show
origin
$ git remote show origin
* remote origin
  Fetch URL: git@git.tsinghua.edu.cn:guyc19/git-test.git
  Push  URL: git@git.tsinghua.edu.cn:guyc19/git-test.git
  HEAD branch: master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

### 添加（add）、删除（rm）、重命名（rename）

添加远程版本库，`shortname` 为版本库名称：

```shell
git remote add [shortname] [url]
```

其他命令：

```shell
git remote rm name  # 删除远程仓库
git remote rename old_name new_name  # 修改仓库名
```

## 分支（branch）

创建分支的操作只是创建一个新的指针，这一过程并不会对仓库进行任何修改。

### 基本操作

执行下面的命令来创建一个分支：

```bash
git branch crazy-experiment
```

此时的仓库提交历史没有任何变化。你得到的仅仅是指向当前 commit 的一个新指针。

此时你只是**创建**了这个分支。如需开始对新分支进行提交，要先选择这个新的分支，使用 `git checkout` 命令，然后再使用标准流程 `git add` 、`git commit` 等命令。

### 列出、创建、切换与删除分支

创建分支 `git branch testing`：

```shell
$ git branch
* master
$ git branch testing
$ git branch
* master
  testing
```

切换到分支 `git checkout testing`（当前工作区会发生变化）：

```shell
$ git checkout testing
Switched to branch 'testing'
```

也可以使用 `git checkout -b (branchname)` 命令来创建新分支并立即切换到该分支下。

删除分支：

```shell
git branch -d (branchname)
```

### 合并分支

使用以下命令将任何分支（或全部分支）合并到当前分支中去：

```shell
git merge (branchname)
```

合并完后就可以删除分支。

### 合并冲突

当出现合并冲突时，会出现以下提示：

```shell
$ git merge change_site
Auto-merging runoob.php
CONFLICT (content): Merge conflict in runoob.php
Automatic merge failed; fix conflicts and then commit the result.
```

会自动将出现冲突的提示信息添加到文件中，便于修改。需要手动修改文件内容，实现合并操作，最后 `git commit` 提交更改即可。

## 版本提交与远程推送

### 查看版本变化（diff）

`git diff` 命令显示已写入暂存区和已经被修改但尚未写入暂存区文件的区别。

显示暂存区和工作区的差异:

```shell
$ git diff [file]
```

显示两次提交之间的差异:

```shell
$ git diff [first-branch]...[second-branch]
```

### 修改状态（status）

`git status` 命令用于查看在你上次提交之后是否有对文件进行再次修改。通常我们使用 `-s` 参数来获得简短的输出结果：

```shell
$ git status -s
AM README
A  hello.php
```

`AM` 状态的意思是这个文件在我们将它添加到缓存之后又有改动（added、modified）。

### 添加到暂存区（add）

`git add` 命令可将该文件添加到暂存区。

添加一个或多个文件到暂存区：

```shell
git add [file1] [file2] ...
```

添加指定目录到暂存区，包括子目录：

```shell
git add [dir]
```

添加当前目录下的所有文件到暂存区：

```shell
git add .
```

### 提交到本地仓库（commit）

`git commit` 命令将暂存区内容添加到本地仓库中。

提交暂存区到本地仓库中（`[message]` 可以是一些备注信息）:

```shell
git commit -m [message]
```

提交暂存区的指定文件到仓库区：

```shell
git commit [file1] [file2] ... -m [message]
```

可以使用以下命令一次执行添加和提交操作（相当于 `git add .`）：

```shell
git commit -am [message]
```

### 远程推送（push）

`git push` 命令用于从将本地的分支版本上传到远程并合并。命令格式如下：

```shell
git push <远程主机名> <本地分支名>:<远程分支名>
```

如果本地分支名与远程分支名相同，则可以省略冒号：

```shell
git push <远程主机名> <本地分支名>
```

使用 `git push -u ...` 可以指定推送的仓库位置，以后使用 `git push` 即可。

## 撤销修改

### 回退工作区代码

> 适用：在上一次提交后的修改还未添加到暂存区，不想保留。

`git checkout [FILENAME]` 即可撤销修改。

### 回退暂存区代码

> 适用：已经使用 `add` 进行暂存，但希望撤销。

包括以下两步：

1.  将暂存区的代码撤销到工作区：使用 `git reset HEAD` 命令来实现（具体见下面）。
2.  将工作区的代码撤销（见上一节）。

### 回退本地仓库代码

> 适用：已经使用 `commit` 提交，希望回退。

利用 `git reset --hard <VERSION>` 命令来实现版本回退（具体见下面）。

### 重置（reset）

这里的 `git reset` 命令可用以下的参数：

```shell
git reset [--soft | --mixed | --hard] [HEAD]
```

- `--mixed` 为**默认**，可以不用带该参数，用于重置暂存区的文件与上一次的提交(commit)保持一致，就是**把暂存区放入工作区**；
- `--soft` 会保留工作目录和暂存区中的内容，并把重置 HEAD 所带来的新的差异放进暂存区。
- `--hard` 参数撤销工作区中所有未提交的修改内容，**将暂存区与工作区都回到上一次版本**，并**删除之前的所有信息提交**（不会删除代码的提交，**尽量不用**）：

```shell
$ git reset --hard HEAD~3  # 回退上上上一个版本
$ git reset --hard bae128  # 回退到某个版本回退点之前的所有信息。
$ git reset --hard origin/master    # 将本地的状态回退到和远程的一样
```

使用 `HEAD` 表示版本号：

- `HEAD` 表示当前版本
- `HEAD^` 上一个版本
- `HEAD^^` 上上一个版本
- `HEAD^^^` 上上上一个版本
- 以此类推...

还可以使用 `~数字 ` 表示：

- `HEAD~0` 表示当前版本
- `HEAD~1` 上一个版本
- `HEAD~2` 上上一个版本
- `HEAD~3` 上上上一个版本
- 以此类推...

版本号也可以通过 `git log` 查看所有的历史版本，进行定位。

### 重置后的恢复

可以使用 `git reset <VERSION>` 恢复到任意一个版本号。可通过 `git reflog` 查看之前用过的 git 命令，定位版本号。

## 参考资料

- https://zhuanlan.zhihu.com/p/461090867
- https://www.runoob.com/git/
- https://blog.csdn.net/weixin_39691748/article/details/113451662
- https://zhuanlan.zhihu.com/p/79686518?ivk_sa=1024320u
