---
title: "Xcode插件管理"
date: 2014-12-25T17:08:52+08:00
tags: ["iOS"]
categories: ["OSX"]
toc: true
---

## 插件也要包管理(Alcatraz)
之前(一年前吧), 关注了下Xcode插件,但是看到其包管理软件Alcatraz还在更新中,所以就没安装了.今天突然想起插件的事,就去看了下,果断支持最新版Xcode了.果断安装:

```zsh
curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh
```

## VVDocumenter
文档是一件麻烦事,果断选用VVDocumenter吧,在函数/类前加上注释(输入```///```),然后用文档生成器生成.
```zsh
Xcode->Window->Package Manager->VVDocumenter
```

`下面是注释规范`

### 类的注释
```zsh
/// 文档A.
@interface DocA : NSObject
```
### 属性的注释
```zsh
/// 数值属性.
@property (nonatomic,assign) NSInteger num;
```

### 方法的注释
```zsh
/// 方法.
- (void)someMethod;
```

`下面是文档生成器`
### HeaderDoc
苹果自己开发的,集成在xcode5及以上. 只要你不导出文档,足够用了

### Appledoc
和Xcode自身的文档兼容,但是只能限定于苹果系统/OC语言.

### Doxygen
支持较广泛.

### XVim
Vim重度用户必选.