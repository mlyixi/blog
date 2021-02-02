---
title: "vimscript笔记"
date: 2014-12-05T10:11:52+08:00
tags: ["vim", "latex"]
categories: ["Linux"]
toc: true
---

用vim基本上就是熟悉各种按键(相信如五笔一样已经习惯),真正要用好它的话还得各种插件,插件多了后不自己手动改改就是各种冲突.所以最终,你得学会vimscript,至少各种概念要熟悉.

这里为[笨方法学Vimscript](http://learnvimscriptthehardway.onefloweroneworld.com/)的摘要
## 设置vim变量
### 全局
设置(set number),切换(set number!),查看(set number[?])
### 局部
setlocal
## 设置映射
### 基本映射
设置(map key toKey),查看(map keys),删除(unmap keys)
### 模式映射
nmap,vmap,imap
### 精确映射
模式映射是递归的,也就是链式的.这是就要非递归映射了

nnoremap,vnoremap,inoremap

### 引导符
自定义插件时,为了避免与别人定的冲突,总要设置一个引导符.

`let map leader = ","`

这样你就可以使用`nnoremap <leader>d dd`

映射了.当然我们不会这样干.我们一般这样映射一些函数.

当然这样也会导致冲突,所以又有了`local leader`.

`let maplocalleader= "\\"

这样我们就可以用`<localleader>`了.
### 组合键

### 局部映射
`h map-local`

## 设置缩写
`iabbrev string toString`

## 自动命令和自动命令组
`autocmd event pattern command`

```zsh
augroup testgroup
    autocmd BufWrite * :echom "Foo"
    autocmd BufWrite * :echom "Bar"
augroup END
```

## 操作符-位移映射
经常有一些位移操作,如dw,ci(,yt等操作+位移的组合,我们可以对位移进行映射
`onoramp key keys`

## 变量和选项
`let foo="bar"`
`set nowrap`