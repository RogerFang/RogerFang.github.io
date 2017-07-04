---
title: branch upstream
date: 2017-04-26 15:44:03
categories:
- Git
---

# upstream设置
当前工作目录工作分支，跟远程的仓库，分支之间的链接关系。

## 基本设置
```
git branch --set-upstream-to=origin/develop
或
git branch -u origin/develop
```
将当前分支的upstream设为远程仓库的develop分支。

## 推送时设置
```
git push -u origin develop
```
推送本地develop分支到远程origin仓库的develop分支，并建立本地分支develop的upstream为origin/develop。

# 取消upstream
`git branch --unset-upstream [unset的其他分支]`

# 查看upstream
通过`cat .git/config`查看


* * *
感谢:
https://higoge.github.io/2015/07/06/git-remote03/