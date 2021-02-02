---
title: "OSX下的PATH"
date: 2014-11-29T18:08:52+08:00
tags: ["PATH"]
categories: ["OSX", "Linux"]
toc: true
---

如果你在OSX下使用跨平台的工具(git,macvim,zsh等),反正就是homebrew安装的,你可能已经发现一大堆源于PATH变量的奇怪问题了:我的vim用的好好的,怎么到了gvim就出错了呢?

全部问题来自于OS X想要解决PATH设置混乱的局面,采用了一个Path_helper,它会读取/etc/paths.d里的路径,并解决重复设置的问题.

引入这个的同时也导致了一些新的问题,特别是苹果貌似之前(Lion,MLion)采用了shell和图形界面程序不同设置PATH的方法,如enviroments.plist的设置(10.9以后应该抛弃了这个),更使得环境变量的设置变得一团糟.

## 系统层次上:

```zsh
/etc/paths -> /etc/profile ->/etc/paths.d
              |(if has bash)
           -> /etc/bashrc
```

看到了么, 如果你用了bash,那好,苹果还是支持你的.

但是如果你用的是zsh,那它就只加载paths.d下面的路径了(/etc/zshenv).

## 用户层次上的login shell(登录时用)
### sh(bsh)
.profile
### bash
.bash_profile->.bash_login->(if both not exists).profile
### zsh
.zshenv->.zprofile

## 用户层次上的interactive shell(交互时用,就是你打开的各种终端)
### bash
.bashrc
### zsh
.zshrc

基本上PATH是在login shell时加载的,而在interactive shell时加载的是一些别名和设置.

## 问题来了
在终端中打开一个zsh,经过系统层和用户层的profile,执行命令行时它的环境变量符合我们的预期

但是当你打开一个图形界面时,特别是像macvim这样内部调用mvim,用open macvim打开时环境变量貌似只执行到系统这一层(也可能是本身的bug--如使用latex-suite时,能执行xelatex,但找不到xdvipdfmx及其它一些程序).

现在的解决方法五花八门, 不过基本上就两个思路(系统的zshenv还是不要改了):

1. 用上bash
2. 用上Path_helper

反正不用考虑跨平台问题,我果断选用2.