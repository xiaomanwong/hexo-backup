---
title: git command
date: 2019-04-16 17:43:58
tags: Git
---

# Git 命令

|     修改记录     |        修改时间        |                  备注                  |
| :--------------: | :--------------------: | :------------------------------------: |
| 添加删除远程分支 | 2018年07月26日13:59:22 |            删除远程分支命令            |
|   更新fork仓库   | 2018年08月10日14:16:08 | 添加fork的仓库，从主仓库更新代码的命令 |


整理的还不够完善，以后工作中遇到了会不断补进，如有大神，有可以留言，我们一起来完善。欢迎各位留言^_^

<!--more-->

## Git 文件的三种状态

1. 已提交：表示该文件已经被安全地保存在本地版本库中，执行过git commit。
2. 已修改：表示修改了某个文件，但还没有提交保存
3. 已暂存：表示把已修改的文件房子啊下次提交时要保存的清单中，也就是执行了 git add 命令。

## 远程仓库与本地代码的配置

将本地已经有的项目上传到远程空仓库中

``` git
cd <project path>
git init
git remote add origin <远程仓库地址>
git add .
git commit -m '提交日志'
git push -u origin master
```

## Git 基本配置

1. 配置个人的用户名称和电子邮件地址，每次提交时，都会引用这两条信息，以用来说明是谁提交的更新。


使用 `--global` 选项，更改的未用户主目录下的配置，如果想在某个特定的项目中使用其他的名称和邮件，只需要去掉 `--global` 选项重新配置，新的配置位于当前项目的 `.git/config` 文件中。

```git
 git config --global user.name ''yourname''
 git config --global user.email  yourname@gmail.com
```

2. 基础命令

> git init

通过此命令会在当前目录创建一个`.git`的隐藏目录，这是`git`的第一步。

> git status

`git status`是最为常用的命令之一，用于检查本地项目的状态.仔细阅读红色/绿色部分，可以获得相关文件的操作信息，根据提示，判断是执行`git commit` 还是执行`git add` 操作。


> git add

将一个或多个文件添加到 `git`仓库中，只有通过 `git add` 添加的文件才会被版本控制管理。

添加单个文件 `git add HelloWorld2.java`

添加多个文件 `git add --a`

添加当前目录所有文件 `git add .`


> git rm --cached <file>

将文件从`git`追踪列表中移除，只是逻辑删除（从版本库中移除），并不会将本地文件删除

> git commit

执行过 `git add` 命令后，需要将暂存的文件提交到本地仓库中，此时是真正的提交

带 log 的提交： `git commit -m 'first commit'`

通过编辑器提交: `git commit `

带log 的提交比较方便，但当你的提交信息有一定格式或者需要提交的文字内容较多时，使用编辑器效果会更好.


> git log

查看历史提交记录， 包括**提交人**、**时间**、**信息**、**信息指纹**等.

查看提交记录： `git log`

单行展示记录： `git log --pretty=oneline`

展示全部信息： `git log --pretty=fuller`


> git clone <远程仓库地址>

拉取远程仓库代码,此项目并不一定是你所创建

例如： `git clone git@github.com:bboyfeiyu/AndroidEventBus.git`

执行完成后，会在本地当前目录创建一个AndroidEventBus的目录来存放仓库代码

> git remote

将本地已经`init`过的工程链接到远程的空仓库中，以此来完成远程版本库的创建

链接远程版本库： `git remote add origin <远程地址>`


> git remote set-url <仓库地址>

SSH 和 HTTPS 方式切换


```git
# ssh to https
$ git remote set-url origin https://github.com/USERNAME/repository.git
# https to ssh
$ git remote set-url origin git@github.com:USERNAME/repository.git
```

可以使用 `git remote -v` 来检查当前仓库地址

```
> origin git@github.com:username/repository.git (fetch)
> origin git@github.com:username/repository.git (push)
```

SSH 和 HTTPS 的区别：

使用 SSH 方式需要在本地配置一个密钥，具体可参见[多个 SSH 公钥提交代码到不同平台](https://xiaoman.ren/2019/04/17/%E5%A4%9A%E4%B8%AA-ssh-%E5%85%AC%E9%92%A5%E6%8F%90%E4%BA%A4%E4%BB%A3%E7%A0%81%E5%88%B0%E4%B8%8D%E5%90%8C%E5%B9%B3%E5%8F%B0/)中介绍；
而使用 HTTPS 的方式操作简单，弊端是需要经常输入密码。


> git branch

分支是用来管理代码版本、类型的有效工具，可根据不同的服务对象、不同的上线版本等等，来做代码分离，版本管理操作

查看分支： `git branch`

查看带提交信息的分支信息： `git branch -v`

创建分支： `git branch <分支名称>`

创建并切换分支： `git branch -b <分支名称>`

切换分支： `git checkout <分支名称>`

推送分支： `git push origin <分支名称>`

删除分支： `git branch -d <分支名称>`

删除远程分支： `git push origin :<分支名称>`

合并分支： `git merge <分支名称>`



> git tag 

会列出所有的 `tag` 标签信息

在完成了所有功能、并且经过测试之后，可以封板上线的版本，通常会打一个标签，***这是一个很重要的功能*** 

建议每次上线都要做一次，便于后续的版本检索与维护，通常一个标签就代表了一个正式版本。

查看本地/远程 tag 标签：`git tag`

查看模糊 tag 标签： `git tag -l 'v1.4.2.*'`

查看备注 tag 标签： `git tag -ln`


查看标签信息： `git show <标签名>`


创建本地 tag 标签： `git tag -a <标签名> -m "<标签备注信息>"`

删除本地 tag 标签： `git tag -d <标签名>`


推送远程 tag 标签： `git push origin <标签名>`

推送全部 tag 标签： `git push origin -tags`

删除远程 tag 标签： `git push origin :refs/tags/<标签名>`


**查看远程机状态**
> git remote 用来管理， fork 的项目，与主项目的更新操作

```
git remote -v
git remote add upstream git@github.com:XXX/XXX.git
git fetch upstream
git merge upstream/master
git push
```

## 使用中的一些技巧

**命名别名**
复杂并超长的命令，可以通过起别名的方式方便在终端中书协

```
git config --global alias.ci commit
git config --global alias.ck checkout
git config --global alias.st status
```

然后就可以很愉快的使用git命令了。
```
git st
```

