---
title: "homebrew日常操作"
date: 2016-09-12T14:08:52+08:00
tags: ["homebrew"]
categories: ["OSX"]
toc: true
---
国内安装：
```zsh
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```
## 基本概念
`brew command options formula`

一个formula就是一个安装包.
安装位置在/usr/local/Cellar下.

常用command:
install, remove, list, search

## 更新
```zsh
brew update
brew upgrade
brew outdated
brew cleanup
```

整体操作虽然方便,但是有些formula还是不常更新比较好,比如数据库.这时就需要:

```zsh
brew pin
brew unpin
```

## 仓库
homebrew完全支持github上的第三方代码仓库, 使用tap管理
```zsh
brew tab caskroom/cask
```
homebrew默认安装homebrew/core仓库, cask仓库主要包含Mac下的图形界面软件.
当然还有其它一些仓库.
