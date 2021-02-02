---
title: "wordpress的markdown"
date: 2014-08-26T14:08:52+08:00
tags: ["markdown"]
categories: ["CS"]
toc: true
---

## Markdown快速参考
* 强调

```zsh
	*Emphasize* _emphasize_
	**Strong** __Strong__
```
* 超链接

```zsh
	A [link](http://example.com "Title").
```
或

```zsh
	Some text with [a link][1] and another [link][2].
	[1]: http://example.com/ "Title"
	[2]: http://example.org/ "Title"
```
* 图片

```zsh
	Logo: ![Alt](/wp.png "Title")
```
或

```zsh
	Smaller logo: ![Alt][1]
	[1]: /wp-smaller.png "Title"
```

* 脚注

```zsh
	I have more [^1] to say up here.
	[^1]: To say down here.
```
* 换行

wordpress不支持双空格换行,只能支持回车换行

* 列表

```zsh
	* Item
	- Item
```
或

```zsh
	1. Item
	2. Item
   		* Mixed
   		* Mixed  
	3. Item
```
* 区块


```zsh
	> Block 1
	> Block 2
	> Block 3
```
* 原样输出


当每行行头空出2个以上空格时则原样输出(不能有命令符)

* 代码


```zsh
	`This is code`
```
* 代码块


```zsh
	~~~~
	This is a 
	piece of code 
	in a block
	~~~~

	```lang
	This too
	```
```
* 语法高亮


```zsh
	```zsh
	# hello world
	```
```
* 标题


```zsh
	# Header 1
	## Header 2
	### Header 3 
	#### Header 4 ####
	##### Header 5 #####
	###### Header 6 ######
```
* 定义


```zsh
	WordPress
	:  A semantic personal publishing platform 
```
* 缩写


```zsh
	Markdown converts text to HTML.
	*[HTML]: HyperText Markup Language
```