---
top: true
title: "Git常见操作以及版本开发规范"
date:  2020-03-21T15:33:03
categories: git
tags:
- git
---

Git工具的使用是作为一个合格码农必备技能。对Git工具的使用有助于提升我们IT从业人员项目开发效率以及管理工作。Git 的工作就是创建和保存你项目的快照及与之后的快照进行对比。

## 版本开发规范

在我们IT人员开发项目的流程中，经常会按照一定规范定义我们的版本。所谓代码版本管理，是指对代码开发、测试、发布、维护过程中的分支与tag的命名、创建、使用、删除、合并等生命周期进行管理的过程。

### 分支的分类

+ **版本主分支：master分支**

  **定义**：可用稳定版本分支，包含了所有稳定版本的历史

  **使用**：只有正式版本发布或者线上问题修复版本发布时，才可以往该分支中合并代码。任何其它情况不可以直接操作该分支，必须通过Pull Request的形式合并代码

  **操作人员**：项目负责人

+ **开发主线分支：develop分支**

  **定义**：开发主分支，包含了所有的代码提交历史 

  **使用**：不可以直接往该分支上提交代码，可基于该分支派生出其他分支进行开发，开发完成后，再合入该分支。合并的方式只能通过pull request 

  **操作人员**：项目负责人

+ **功能分支：feature分支**

  **定义**：功能分支，临时为开发某一个具体的功能所建的分支 

  **使用**：当需要开发某一个功能时，临时基于develop派生出来基于该分支进行迭代工作，当迭代结束时，将代码合并到develop分支中，然后删除该分支

  **操作人员**：开发人员

+ **发布分支：release分支**

  **定义**：发布分支，当开发完成转测的时候用到的分支

  **使用**：当某一个版本的开发工作结束，将要转测试时，基于develop派生出来该类型的分支。基于该分支进行bug修复工作。当测试完成，所有的bug都修复完成，将该分支合并到master分支并打tag，同时合并到develop分支。最后，将该分支删除。在bug修复时，也可以基于release分支创建临时的bug分支进行开发，如bug/<redmine_id>

  **命名**：以release/进行开头，如release/v1.0.0 

  **操作人员**：开发人员

## Git 介绍

git架构如图所示。分为工作区、暂存区、本地仓库和远程仓库。具体操作如图所示。

![img](http://images.cnblogs.com/cnblogs_com/wupeiqi/662608/o_git.png)

## Git常见命令

```go
开始一个工作区（参见：git help tutorial）
   clone      克隆仓库到一个新目录
   init       创建一个空的 Git 仓库或重新初始化一个已存在的仓库

在当前变更上工作（参见：git help everyday）
   add        添加文件内容至索引
   mv         移动或重命名一个文件、目录或符号链接
   reset      重置当前 HEAD 到指定状态
   rm         从工作区和索引中删除文件

检查历史和状态（参见：git help revisions）
   bisect     通过二分查找定位引入 bug 的提交
   grep       输出和模式匹配的行
   log        显示提交日志
   show       显示各种类型的对象
   status     显示工作区状态

扩展、标记和调校您的历史记录
   branch     列出、创建或删除分支
   checkout   切换分支或恢复工作区文件
   commit     记录变更到仓库
   diff       显示提交之间、提交和工作区之间等的差异
   merge      合并两个或更多开发历史
   rebase     在另一个分支上重新应用提交
   tag        创建、列出、删除或校验一个 GPG 签名的标签对象

协同（参见：git help workflows）
   fetch      从另外一个仓库下载对象和引用
   pull       获取并整合另外的仓库或一个本地分支
   push       更新远程引用和相关的对象
```

接下来，将基于项目实际操作具体的git命令。

### 初始化项目

当我们新建一个项目后，需要执行`git init`命令，创建一个 Git 仓库。

```go
➜  hyperManager git init 
已初始化空的 Git 仓库于 /Users/eggsy/go/src/github.com/hyperManager/.git/
```

然后执行`git remote add`，将远程仓库和本地仓库绑定。

```go
➜  hyperManager git:(master) ✗ git remote add origin https://github.com/Eggsyz/hyperManager.git
➜  hyperManager git:(master) ✗ git remote -v
origin  https://github.com/Eggsyz/hyperManager.git (fetch)
origin  https://github.com/Eggsyz/hyperManager.git (push)
```

### 提交修改命令

接下来，将介绍项目开发的一些命令

1. 将修改添加到缓存

执行`git add`命令添加修改的文件到缓存区

```go
➜  hyperManager git:(master) ✗ git add README.md  
```

2. 将缓存数据保存到仓库

执行`git commit`命令将缓存区文件写入本地仓库

```go
➜  hyperManager git:(master) ✗ git commit -m "first committed" 
```

3. 将本地仓库文件推送到远程仓库

执行`git push`命令将本地仓库文件推送到远程仓库

```go
➜  hyperManager git:(master) ✗ git push origin  master  
枚举对象: 5, 完成.
对象计数中: 100% (5/5), 完成.
使用 8 个线程进行压缩
压缩对象中: 100% (3/3), 完成.
写入对象中: 100% (5/5), 450 bytes | 450.00 KiB/s, 完成.
总共 5 （差异 0），复用 0 （差异 0）
To https://github.com/Eggsyz/hyperManager.git
 * [new branch]      master -> master
```

4. 查看提交记录

执行`git log`查看提交记录

5. 创建并切换新分支

```go
git checkout -b develop master
```

6. 出现问题，执行回滚操作

```go
用命令git reset HEAD可以把暂存区的修改撤销掉（unstage），重新放回工作区：
$ git reset HEAD 
```

### 具体开发流程

1. **项目负责人**会基于master派生develop分支

```go
git checkout -b develop master
git push -u origin develop
```

2. **开发人员**根据develop分支派生新的功能/特性分支

```go
git checkout -b feature/p2p develop
git push -u origin feature/p2p
```

3. 功能完成开发后**开发人员**创建*PR*，**项目负责人**会review代码并合并到develop分支

```go
在github或者gitlab会看到new pull request选项，可通过这个来请求合并分支
```

4. 删除feature/p2p分支

```go
git branch -d feature/p2p
git push -d origin feature/p2p
```

5. 当版本开发结束时，基于develop分支派出生release分支，如release/v1.0.0。

```go
git checkout -b release/v1.0.0 develop
git push -u origin release/v1.0.0
```

6. 发布内测版本，即：基于release分支打tag，发布给**测试人员**。

```go
git tag v1.0.0-alpha.1 release/v1.0.0
git push origin v1.0.0-alpha.1
```

7. **测试人员**如果测出bug，**开发人员**需要基于release派生bug分支，进行修复，当修复完成后提*PR*请求合并到release分支

```go
git checkout -b bug/1234 release/v1.0.0
git push -u origin bug/1234
// 合并请求
...
// 删除bug分支
git branch -d bug/1234
git push -d origin bug/1234
```

8. 当全部测试结束，所有bug都修复完成后，开始正式代码发布：**项目负责人**将release分支合并到master分支和develop分支中，并删除release分支。

```go
在gitlab或github上创建pull request进行代码合并
```

9. **项目负责人**基于master分支打tag，名字一般为版本号，如v1.0.0

```go
git tag v1.0.0 master
git push origin v1.0.0
```

10. **项目负责人**可根据需要，将测试版本的tag号删除。也可以留着，以防后续需要。

```go
git tag -d v1.0.0-alpha.1
git tag -d v1.0.0-alpha.2
git push --delete origin v1.0.0-alpha.1
git push --delete origin v1.0.0-alpha.2
```

