---
title: "Doxygen安装和使用"
date: 2015-01-12T14:08:52+08:00
tags: ["Doxygen"]
categories: ["CS"]
toc: true
---

## Doxygen
在[Xcode插件管理](/post/osx/Xcode插件管理)这一文章中列出了OSX下常用的文档生成器,这里我们重点介绍Doxygen
### 安装
* 到官网下直接下载图形界面
* 通过homebrew安装`brew install doxygen`

## 使用 doxygen 生成文档
### 生成配置文件
`doxygen -g`会在当前目录下生成配置文件`Doxyfile`.

### 编辑配置文件
`Doxyfile`与makefile类似:

* OUTPUT_DIRECTORY
* INPUT
* FILE_PATTERNS
* RECURSIVE
* EXTRACT_ALL
* EXTRACT_PRIVATE
* EXTRACT_STATIC

### 生成文档
`doxygen Doxyfile`

## 文档格式
* UNIX手册: GENERATE_MAN
* RTF: GENERATE_RTF
* latex: GENERATE_LATEX
* chm: GENERATE_HTMLHELP
* XML: GENERATE_XML

## Xcode文档位置
* 全局

`/Applications/Xcode.app/Contents/Developer/Documentation/DocSets/***.docset`

* 个人

`~/Library/Develooper/Shared/Documentation/DocSets/***.docset`

## 图形与图表
在默认情况下，`Doxyfile` 把 `CLASS_DIAGRAMS` 标记设置为 Yes来生成类层次结构图.

要想生成更好的视图，可以下载 `Graphviz`下并在Doxyfile 中使用以下标记用来生成美化图表：

* CLASS_DIAGRAMS
* HAVE_DOT
* CLASS_GRAPH
* COLLABORATION_GRAPH

Graphviz是个不错的脚本作图工具,有空学习下.