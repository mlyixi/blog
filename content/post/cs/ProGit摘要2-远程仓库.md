---
title: "ProGit摘要2-远程仓库"
date: 2014-10-24T10:18:52+08:00
tags: ["ProGit"]
categories: ["CS"]
toc: true
---

[上一节](/post/cs/ProGit摘要1-本地仓库)中我们介绍了git原理和本地仓库的使用,现在介绍远程仓库的使用
### 克隆仓库
远程

```zsh
git clone git://uri
```
本地

```zsh
git clone /path/to/repo/
```
这样就克隆了远程仓库origin的所有分支,如origin/master. 注意,本地仓库也有master分支,当然还会有其它分支,但是它们没有仓库名(默认为本地嘛).

### 查看远程库
```zsh
git remote
```
这样就列出了所有的仓库.克隆下来的仓库被全名为origin,可用`-o`指定仓库名

查看仓库下所有分支

```zsh
git remote show [repoName]
```

### 添加远程仓库
我们的本地仓库可能没有和远程仓库建立连接,这时使用如下命令添加关联:

```zsh
git remote add repoName git://uri
```

### 重命名远程仓库

```zsh
git remote rename oldRepoName repoName
```

### 删除远程仓库

```zsh
git remote rm repoName
```

### 远程抓取分支

```zsh
git fetch [repo-name] [branch]
```
抓取数据,但并不合并(你可能要手动解决冲突)

### 同步仓库

```zsh
git pull [repo-name]
```
相当于fetch && merge,适用于可以快速合并的仓库.

### 推送分支
将本地master分支推送到origin主机上

```zsh
git push origin master
```
注意, 这时你需要设置push的策略push.default

```zsh
git config --global push.default simple
```
一般simple够用了,然后它会提示你输入用户名和密码.

### 删除分支

```zsh
git push origin :remoteBranchName
```

### 打标签
我们总会在某一时间点上的提交打上标签作为软件版本号
* 显示已有标签

```zsh
git tag
```
* 新建标签

```zsh
git tag v1.4 -m 'my version 1.4'
```

* 查看标签

```zsh
git show v1.4
```
更多的标签内容就不在这里罗列了.