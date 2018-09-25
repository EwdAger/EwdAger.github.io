---
title: Git 常用命令
tags:
  - Git
categories: 知识储备
abbrlink: 5fddf106
date: 2018-09-25 09:27:00
---

# Git 简介
Git - 分布式版本控制系统，每台电脑上都可以离线存一个仓库，但总是设立一台服务器作为远程库
SVN - 集中式版本控制系统，无本地仓库，push pull 必须通过中央服务器

# 基本操作

```bash
git init     # 初始化，创建仓库
git add .    # 添加所有文件（添加单一文件写文件名）到缓存区
git commit -m "message" # 将缓存区所有任务保存到仓库，并添加本次修改的信息（注释）

git status   
# 查看当前仓库状态，会显示无修改、有修改但未提交缓存、提交缓存但未提交仓库多种状态

git diff <filename> # 查看当前文件与最新版本的差异

git log # 显示历史记录
git log --pretty=oneline # 一行显示历史记录（显示全部id）
git log --oneline    # 一行显示历史记录（显示id前7位）
git reflog # 显示所有命令记录
git log --graph # 显示分支合并图

git reset --hard <commit ID> # 退回某一版本，HEAD为最新版本

git checkout -- <filename> # 丢弃缓存区某文件的修改

git rm <filename> # 删除某文件并提交到缓存区

git tag v1.0 # 给最新commit创建tag
git tag v0.9 f52c633 # 给f52c633创建tag v0.9
git tag -d v1.0 # 删除这个tag

```
<!--more-->
# 远程库&分支管理

```bash
git remote add <name(origin)> git@github.com:ewdager/learngit.git 
# 从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联

git push -u origin master
# 由于远程库是空的，我们第一次推送master分支时，加上了-u参数，
# Git不但会把本地的master分支内容推送的远程新的master分支，
# 还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令

git clone git@github.com:ewdager/learngit.git # 克隆版本库

git checkout -b dev # 创建dev分支并切换，等于一下两条命令
git branch dev    # 创建dev分支
git checkout dev  # 切换至dev分支

git branch # 查看当前分支

git nerge dev # 把dev分支合并到master
git branch -d dev # 删除dev分支

git merge --no-ff -m "merge with no-ff" dev 
# 强制禁用Fast forward模式，Git就会在merge时生成一个新的commit。删除分支后，不会丢掉分支信息

git stash # 储存当前“状态”，供后续恢复，不会影响缓存区和仓库
git stash list # 查看储存列表
git stash apply <stashid> # 恢复某次状态
git stash drop <stashid> #删除某次状态
git stash pop <stashid> # 恢复并删除某次状态

git branch -D <name> # 强制删除没被合并过的分支

git rebase # 让分支变得更美观！
```

# 高级一点的操作

- 文件已修改，未add到缓存区:
```bash
git checkout -- <filename>
```

- 文件已修改，并add到缓存区未commit：
```bash
git reset HEAD <filename>
git checkout -- <filename>
```

- 生成SSH KEY
```bash
ssh-keygen -t rsa -C "youremail@example.com"
```
	其中`id_rsa`为私钥，`id_rsa.pub`为公钥。

- 当合并分支出现冲突时
	先用`git status`查看冲突文件，然后**手动**修改冲突文件，最后将冲突文件`add`、`commit`即可合并成功。可用`git log --graph`查看分支合并图。

- 临时 Bug 分支&保存现场
	详见[廖雪峰Git教程-Bug分支](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137602359178794d966923e5c4134bc8bf98dfb03aea3000)

- 多人协作
	1. 首先，可以试图用git push origin <branch-name>推送自己的修改；

	2. 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

	3. 如果合并有冲突，则解决冲突，并在本地提交；

	4. 没有冲突或者解决掉冲突后，再用git push origin <branch-name>推送就能成功！如果git pull提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream-to <branch-name> origin/<branch-name>。

	这就是多人协作的工作模式，一旦熟悉了，就非常简单。

全文参考[廖雪峰老师Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)