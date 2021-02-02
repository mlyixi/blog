---
title: "latex-suite的安装和使用手记"
date: 2014-10-24T14:08:52+08:00
tags: ["latex"]
categories: ["CS"]
toc: true
---

换了hosts终于用上各种国外服务了,感谢imouto
(imouto已经不可用了,而且我现在只用texstudio,简单)

翻到了以前的资料,转到博客中来

## 安装
1. 下载安装包。
2. 解压到~/.vim下
3. 编辑~/.vimrc，按说明来吧。
```vim
filetype plugin on
set shellslash
set grepprg=grep\ -nH\ $*
filetype indent on
let g:tex_flavor='latex'
```
4. 编辑~/.vim/ftplugin/tex.vim
```vim
set sw=2 
set iskeyword+=:
```
5. 在vim中添加helptags.
```vim
:helptags ~/.vim/doc
```

话说现在都用vundle管理vim包了吧,N年前的事了,将就着看吧.


## 使用
1. 输入document后按F5,出现框架documentclass
2. 在\begin{document}前输入包名,按F5,插入包.
3. 在\begin{document}环境中输入环境名,按F5插入环境.
4. \rf折叠所有.空格解开,za也是解开.
5. \ll \lv \ls编译,查看,反查命令.
6. 在输入\`号后输入a-z,A-Z可能出现相应的字母和数学表达式(\`I), (\`M),同时,可以输入(\`/)

其实还是蛮喜欢latex-suite的.它的一些设定还是很方便的,比sublime text的latex插件好用.不过配置起来有点麻烦.