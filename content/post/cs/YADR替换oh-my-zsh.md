---
title: "YADR替换oh-my-zsh"
date: 2014-09-26T14:09:52+08:00
tags: ["zsh"]
categories: ["CS"]
toc: true
---

由于喜欢agnoster主题,但是oh-my-zsh默认还是解决不了这个问题.所以考虑换个配置包.终于找到另一个不用配置的的配置包YADR,基本不用自己手动一点点搞了,除了自己下载iterm2,其它依赖会直接搞定: 它会安装homebrew(记得回车输入密码), 然后安装必要的软件,如zsh,macvim,下载powerline字体,下载vim插件,真是傻瓜到家了,更主要的是还可以更新.

## 安装

```zsh
sh -c "curl -fsSL https://raw.github.com/skwp/dotfiles/master/install.sh";
```
注意,如果安装有macvim或vim的要用homebrew卸载掉,所以最好是原版安装.
## 更新

```zsh
cd ~/.yadr
git pull --rebase
rake update
```
## 手动要改的
* /etc/pashs中将/usr/local/bin放到第一行(会自己改了)
* 记得运行`brew linkapps`加入图形化macvim到applications.
* 在item中更改主题Profiles => Colors => Load Presets=> Solarized Dark
* 如果使用苹果电脑,安装[seil](https://pqrs.org/osx/karabiner/seil.html),将caps键改成esc键.
* 将spotlight/Alfred的快捷键设为`Ctrl-Cmd-Space`,以使vim中的autocomplete能用`Cmd-Space`.
* 设置item的hotkey(preference=>keys=>hotkey)
* iterm中取消Use Lion-style full screen: 好像没什么用
* macvim中取消Advanced settings下的Prefer native fullscreen: 好像没什么用
* 在.zsh.after内新建文件prompt.zsh,加入


```zsh
DEFAULT_USER=$USER;
prompt agnoster
```
* vim不自动换行,加上

```zsh
   set formatoptions+=1
   set wrap
   set nolist
   nnoremap j gj
   nnoremap k gk
   vnoremap j gj
   vnoremap k gk
```
# vim
同时它还给vim安装了很多插件,就不一一介绍了,可以参考yadr官方说明.