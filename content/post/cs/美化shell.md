---
title: "美化shell"
date: 2014-08-26T14:08:52+08:00
tags: ["zsh"]
categories: ["CS"]
toc: true
---

现在发现oh-my-zh并不是很好,更好的方法见[这里](http://mlyixi.byethost32.com/blog/?p=135)

## MAC:

* 安装 `homebrew`
* 安装软件: `brew install zsh, vim , macvim etc...`
* 注册环境变量:修改`/etc/paths`,将`/usr/local/bin`设为第一行。系统将优行查找`/usr/local/bin`中的程序
* 注册 shell:修改`/etc/shells`,添加`/usr/local/bin/zsh`.
* 下载ohmyzsh


```bash
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
chsh -s /usr/local/bin/zsh
```
* 下载iTerm2,这时已经切换到zsh.
* 使用agnoster主题: 打开`.zshrc`,设置


```bash
SH_THEME="agnoster"
DEFAULT_USER=$USER
```
* 设置[配色方案](http://ethanschoonover.com/solarized),然后打开iTerm的首选项，打开profiles中的Colors,load Presets,import "Solarized Dark.itermcolors",然后在load Presets中选中Solarized Dark。
* 打字体补丁:添加[系统字体](https://gist.github.com/qrush/1595572)
* 补充: iTerm2要配合新版的 theme 文件，而 oh-my-zh 的 theme 文件是旧版(172行),需要更新成115行的 [theme文件](https://gist.github.com/ashebanow/9a02751a82058efaf0d8/)才可以正常显示如下特殊字符

## KDE4
* 安装zsh后切换到zsh
* 启用配色方案，在最新的kconsole下已经有solarized配色方案了。不知道是不是oh-my-zsh的原因。所以在settings--manage profiles--edit profile--appearance下修改就好。
* 安装字体