# NoteForGit


That is git!

<!--more-->

# 1.概述

**git** 是目前较为先进的分布式版本控制系统。

## 原理和流程：

![img](https://pic2.zhimg.com/80/v2-3bc9d5f2c49a713c776e69676d7d56c5_1440w.webp)

此处的Repository 一般指代本地仓库，Remote则为远程仓库。

## **SVN & Git** ：

SVN是集中式版本控制系统，版本库统一存放在中央服务器，工作时需要从中央服务器获取最新版本，工作后将工作成果返回中央服务器。基于此，SVN需要联网工作，互联网条件下易受带宽限制。

Git是分布式版本控制系统，无中央服务器，工作时可不必联网，但多人协作需要互相推送各自修改。

## **功能:**

一个良好的多端同步工具，可记录与返回历史版本，为多人协作实现便利。

## 缘起：

> Linux的发展过程中，开源文化与开源开发者扮演了重要角色，而版本控制的需求日益增长，手工合并的实现日渐不能满足需求，linus认为CVS和SVN既有带宽的弊端，也违背开源文化，于是乎Lin  us花了两周时间自己用C写了一个分布式版本控制系统





## Git on Windows

Windows 下在官网下载并安装后，可进行配置：

```shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

--global参数表示全局，即本机使用git时已经选定了特定仓库，当然也可以为特定的本地仓库指定远程仓库。

# 2.Repository

可以简单理解为一个目录，目录内所有文件均可被git管理。其中每个文件的修改、删除，均可被git追踪，一辩任何时刻均可追溯历史，或者在将来某个时刻进行还原。

## 2.1**创建Repository：**

Step_0:

```shell
$mkdir learngit
$cd learngit
$pwd
/halois/learngit
```

> win下保证目录树中无中文，以避免奇怪的问题发生

Step_1:

```shell
$git init
```

Step_2

```shell
$git add <filename> or <directories>
```

Step_3

```shell
$git commit -m "Explanatory statement"
```

**Warning：不可含有任何中文**

## 2.2快照功能

**git log**

查看最近到最远的提交日志，即 *"Explanatory statement"*

**git status**

```powershell
(base) PS E:\desktop\blogdemo> git status
On branch master
nothing to commit, working tree clean
```

**git  reset**

 回到上一个提交：

```powershell
git reset --hard HEAD^
```

如果没有上一个提交将会:

```powershell
PS E:\desktop\blogdemo> git reset --hard HEAD^
fatal: ambiguous argument 'HEAD^': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'
```

请注意，`git reset --hard`命令是一个危险的操作，它会丢弃所有未提交的更改，并将分支移动到指定的提交。请确保在执行此命令之前备份重要的更改

`HEAD`指向当前版本。



其后可接指令或commit的版本号前几位即可，回退到指定版本，即便又所删除，只要shell尚未关闭，一切仍然可以挽回。

**git reflog**

查看历史版本号。

## 2.3工作区和暂存区

Git区别于其他控制系统，如SVN含有暂存区。

![git-repo](https://s2.loli.net/2023/07/14/qV9xQKrFJICPsEz.jpg)

**工作区(Working Directory)**：可见的目录（文件夹）

**版本库(Repository)**: 

工作区内含有一个名为.git的隐藏文件夹,这里在工作区的目录之中却在工作区之外，称为版本库。

内含：stage, master(分支)、指向master的一个指针叫做`HEAD`等



**git add**:把文件添加进入暂存区

![git-stage](https://s2.loli.net/2023/07/14/32eOyPGQzLWJvMf.jpg)

**git commit**:把暂存区文件提交到当前分支

![git-stage-after-commit](https://s2.loli.net/2023/07/14/Mra6iKqmkCsYJNU.jpg)

## 2.4其他修改

**git checkout -- <filename>**

工作区丢弃最近一次修改，返回最近的commit后状态。

**git reset HEAD <filename>**

可以把暂存区的修改撤销掉，重新放回工作区

- 情况1：**文件只在工作区操作，未add**。撤销操作：**git restore <file>**。结果：**工作区文件回退**。
- 情况2：**文件已add，未commit**。撤销操作：**git restore --staged <file>**。结果：**暂存区文件回退，工作区文件未回退，如需继续回退，操按情况1操作。**
- 情况3：**文件已add，已commit**。撤销操作：**git reset --hard commit_id**。结果：**工作区文件、暂存区文件、本地仓库都回退**

## 2.5删除文件

**rm <filename>** 工作区内删除

**git rm <filename>**  版本库中删除

**git checkout -- <filename>** 版本库中覆盖工作区中指定文件

# 3.远程仓库(remote)

以github 为例。

## 3.1创建远程库

**Remote端**：

创建新仓库

**本地:**

在本地仓库目录下：`git remote add origin git@github.com:<repositoryname>`

> 此处的origin是git识别的远程仓库名字，可以修改，但是默认remote即命名为 origin

下一步就可以：本地上传远程

```powershell
git push -u origin master
```

`git push`是将本地内容推送到远程，上述命令把当前的分支`master`推送到远程。

`-u`参数 由于是第一次推送，使用`-u`推送后可以使得笨的的master和远程仓库的master关联起来，在以后的推送和拉去可以简化命令

```powershell
git push origin master
```

## 3.2删除远程库

查看远程库信息：

```shell
git remote -v
```

删除远程库：

```shell
git remote rm <Remotename>
```

删除的含义：解除本地与远程的链接。如需要彻底删除，则需要进入远程库的管理系统进行删除。

## 3.3远程传本地

```shell
git clone <ssh/https to your repository>
```

# 4.分支管理

![image-20230714153257350](https://s2.loli.net/2023/07/14/WflbuRtKwSNrvjz.png)

git 分支实际上是更改指向更改快照的指针。





## 4.1创建与合并分支



HEAD、Branch、Commit

$**p,*p,p$





**创建分支：**

```shell
git branch (branchname)
```

**切换分支：**

```shell
git checkout (branchname)
```

> 切换分支时git会用该分支的最近一次快照内容替换工作目录内容，多个分支不需要多个目录

**合并分支：**

`git merge`命令用于合并指定分支到当前分支

> **合并方式：**（可暂时不做了解）
>
> 1. Fast-Forward合并：
>    - 语法：`git merge <branch-name>`
>    - 描述：当目标分支（通常是主分支）没有新的提交时，可以使用Fast-Forward合并。这种合并方式会直接将目标分支指向源分支的最新提交，形成一个线性的提交历史。因为没有新的合并提交，所以合并后的提交历史非常简洁。
> 2. 普通合并（Regular Merge）：
>    - 语法：`git merge <branch-name>`
>    - 描述：普通合并是最常用的合并方式之一。它会创建一个新的合并提交，将两个分支的更改合并到一起。在执行合并操作时，Git会自动创建一个新的提交，包含两个分支的更改内容。这种合并方式会保留每个分支的提交历史，并在合并提交中保留合并的信息。
> 3. 变基（Rebase）：
>    - 语法：`git rebase <branch-name>`
>    - 描述：变基是另一种合并分支的方式。它将当前分支上的提交按照顺序逐个应用到目标分支上，使得目标分支的提交历史变得更加线性。变基操作可以将当前分支上的提交“移动”到目标分支的最新位置，这样就可以在合并时保持一个干净的提交历史。变基操作会改写提交的SHA标识，因此在共享分支时需要特别注意。
> 4. Squash合并（Squash Merge）：
>    - 语法：`git merge --squash <branch-name>`
>    - 描述：Squash合并是将多个提交压缩为一个提交的方式。它会将一个分支上的所有提交合并成一个新的提交，并将其应用到目标分支上。这种合并方式适用于需要保持干净、整洁的提交历史，将多个相关的提交合并为一个更有意义的提交。

分支的时间线理解：

![image-20230714155054624](https://s2.loli.net/2023/07/14/b1WNKuncEtDg9JT.png)







不直接修改时间线，仅仅修改指针。

```shell
git checkout -b dev
```

穿件并切换到；相当于

```shell
git branch dev
git checkout dev
```

然后使用`git branch`查看分支状态

切换到已有的分支：

```shell
git switch main
```

创建新的并切换到分支：

```shell
git switch -c dev
```

这一更新是为了使得 `git checkout `不那么容易混淆在版本库和工作区之间的操作。



## 4.2解决冲突

合并分支并非一帆风顺，代码相与亦非易如反掌。

**无法快速合并冲突**

![image-20230715095934670](https://s2.loli.net/2023/07/15/qdEVyOMwLWAPZJs.png)

准备分支`feature1`，`git switch -c feature1`

分别作不同修改并提交，此时feature1和master所指提交不同，无法快速合并

报错如：

```shell
$ git merge feature1
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
```

`git status`也可获得冲突信息

```shell
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
//对比远程和本地的提交次数
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

解决：

![image-20230715101420672](https://s2.loli.net/2023/07/15/bw8JOm4GshXMenz.png)



## 4.3分支协作

```shell
git merge --no-ff -m "Explanatory statement" <target branch> 
```

`--no-ff`参数表示禁用`Fast forward`模式，于是git在merge时会新生成一个提交，从分支历史上可清晰看见分支信息



![image-20230715102630246](https://s2.loli.net/2023/07/15/2sjWAXiehGENpqL.png)

基于此可以实现多人协作：

![git-br-policy](https://s2.loli.net/2023/07/15/x1BInPzjiAqMgE6.png)

## 4.4临时分支

当前工作区工作可以缓存

```shell
git stash
```

缓存后会自动清除工作区，可以通过

```shell
git stash list
```

查看缓存

**还原**

- 还原缓存至工作区，但是不删除缓存

  `git stash apply`

- 还原缓存至工作区，同时删除缓存

  `git stash pop`

- 删除缓存

  `git stash drop`

但是这不是简单的栈结构，这里可以多次缓存以特定编号指定恢复和删除。

```shell
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
$ git stash apply stash@{0}
```



**临时分支的修改需要作用与其他分支**

```
$git cherry-pick <code of commit>
```

## 4.5测试分支

**新的功能需求出现：建立测试分支**

```shell
$git switch -c feature-test
```

```shell
$git add test.py 
$git commit -m "add features test"
$git switch dev

$git branch -d feature-test
```

如中途功能取消：

删除未合并分支

```shell
$git branch -D feature-test
```

`-D`强行删除

## 4.6多人协作

查看远程仓库

```shell
$git remote 
origin
```

可以加入参数 `-v`以查看详细信息（origin的地址）

**推送分支**

把本地所有可推送内容推如指定分支。

```shell
$git push origin master
```

如果要推送其他分支如dev

```shell
$git push origin dev
```

**抓取分支**

一般的，默认master(main)分支和dev分支均及推送，而这两个分支也大多会被及时抓取

新来的伙伴需要构建他的dev

```shell
$git checkout -b dev origin/dev
```

建立了本地dev分支和远程的dev分支的关联。

值得注意的是需要先保持本地和远程一致，再提交自己的修改。

```shell
$git pull 
```

在pull之前也首先需要建立一个链接

```shell
$git branch --set-upstream-to=origin/dev dev
```

**多人协作一般方式：**

- `git push <branch name>`试图推送自己的修改
- 推送失败，`git pull`更新本地
- pull有冲突解决冲突
- 再次push

# 5.标签管理

标签(tag)类似branch,但是不可移动，定向指向一个commit

创建标签

1. 切换到需要打标签的分支上
2. `git tag <name>`
3. `git tag`查看标签

对历史提交打标签

```shell
$git tag <tagname> <code of commit>
```

tag状态的排序按照字母顺序。

查看详细信息：

```shell
$git show <tagname>
```

同样的可以加入一些说明性文字

```shell
$git tag -a v1.0 -m "version 0.1 released" 1024aa
```

`-a`指定了标签名，`-m`加入说明性文字。均可以在git show中查看

**标签是跨分支的，tag创建后在任何分支均可查看**

# 6.忽略文件

部分文件需要放在工作目录中却不可以推送提交。

显示untracked不够优雅

**创建[.gitignore](https://github.com/github/gitignore)文件**

忽略文件的原则：

1. 忽略操作系统生成的文件
2. 忽略编译中间产物
3. 忽略敏感信息文件

**失误操作的挽回：**

检查.gitignore文件：`git check-ignore`

强制添加文件：`git add -f <filename>`



