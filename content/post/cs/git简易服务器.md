---
title: "git简易服务器"
date: 2021-01-24T10:08:52+08:00
tags: ["git"]
categories: ["CS"]
toc: true
---

最近在整理资料，纠结于代码如何备份，最终觉得还是放到git里比较好。

之前使用的gitlab，但web界面太占资源，而且实验室除了我管理外估计也没有其它人用了，所以权限也不用管理(排除gitolite)。所以直接`git init --bare`走起。

<https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server>

# 服务端
## 添加用户
```zsh
sudo adduser git
su git
cd
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```
将自己的公钥加到authorized_keys文件中：

```zsh
cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
```
## 创建空仓库
```zsh
mkdir project.git
cd project.git
git init --bare
```
OK, 服务器端就搭建好了。唯一的麻烦就是客户端第一次push的时候要先建立空仓库。

# 客户端
```zsh
cd myproject
git init
git add .
git commit -m 'Initial commit'
git remote add origin git@ip_address:~/project.git
git push origin master
```

至于服务端切换bash到git-shell以禁止登陆，也就没什么必要了，而且麻烦。