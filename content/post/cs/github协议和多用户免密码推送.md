---
title: "github协议和多用户免密码推送"
date: 2014-12-22T10:08:52+08:00
tags: ["github"]
categories: ["CS"]
toc: true
---

今天发现github推出了MAC下的图形管理软件,果断试用了下,发现蛮不错的.可惜好像不支持多帐号. 想着用多帐号模拟分布式合作练习git操作的 = =.

昨天新建了个用户,打算把练习代码放到上面去, 结果总是显示`empty reply from server`. 可能的原因是:

1. 用的https协议,被墙了.
2. 没有配置当前仓库下的user.name和user.email.被全局名覆盖了.(虽然https登录会问你要用户名和密码).

想想这一块还是要详细了解下,所以就有了这篇日志.

## Github
### 协议
github支持两种协议,也就是远程URL的格式.

1. https: 官方建议使用的协议,每次都要认证,有可能被墙.
2. git: 通过ssh协议传输, 可以实现免密推送.

### 协议查看与修改
可以通过两种方式查看或设置远程URL

* git命令

```zsh
git remote -v
git remote set-url origin https/git.git
```
* 直接修改仓库下的.git/config

## 免密推送
其实github不推荐这种方式,因为推送太过频繁并不有利.但是,我们就想不输密码!!!没钱也要任性.

如上面所述,我们只能使用git协议.所以,先把`各个仓库的远程URL改成git协议的`吧.

* 通过ssh生成密钥

```zsh
ssh-keygen -t rsa -C "your_email@example.com"
# Creates a new ssh key, using the provided email as a label
# Generating public/private rsa key pair.
# Enter file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
```

果断输入你要保存钥匙的文件名吧(这里ssh好像不认主目录`~`符号),并输入一个passphrase(下面步骤要用,增加本地安全性的,不设也行).

* 加入私钥

```zsh
ssh-agent -s  # 开始授权中心
ssh-add ~/.ssh/你的私钥 # 加入私钥
```
私钥不会自动添加(默认只检测id_rsa,id_dsa等),如果你出现如下错误,基本上就是没有添加私钥了.

```zsh
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

或
Warning: Permanently added the RSA host key for IP address '192.30.252.128' to the list of known hosts.

```

每次添加多麻烦啊,怎么办?可以在profile中加入`ssh-add`,但是很不幸,这样也不好: 如果多次登录,系统就会生成很多ssh-agent进程了. 也可以考虑更换授权中心ssh-agent.

想想折腾这些不更麻烦,自己需要时添加下不是更方便,也就释然了.

* 上传公钥
上github,点`settings->SSH keys->add SSH key`. 完工

## 多帐号
未经过详细测试,不过,理论上推想对各个帐号进行上面的设置应该就好了.

也可以全局一个帐号作为主号,其它小号的仓库再用git config设置.

## 保存ssh配置
额,忘了说了,换了台电脑怎么办? 当然是把当前的.ssh文件夹复制一份了.