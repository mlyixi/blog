---
title: "iOS视图控制器初始化总结"
date: 2014-08-27T18:08:52+08:00
tags: ["iOS"]
categories: ["OSX"]
toc: true
---

首先,区别viewcontroller初始化的三种方式:完全代码化,半代码化初始和IB初始的区别

1. 代码化:完全没有Xib之类的东西,通过纯代码实现加载.
2. 半代码化:设计xib,然后在程序中用代码调用xib来初始化.
3. 完全IB化:设计xib之类,然后加入到其它xib里.如MainWindow.xib中加入rootviewController,而rootviewcontroller通过xib设计.

好了,了解了这些,来看看各种方法的使用.

1. viewDidLoad:这个方法在三种方式下都会调用,而且是加载完view后调用.
2. loadView:代码化时调用. 半代码化和完全IB化时亦调用,但会重写xib中的view,调用在initwithNibName之后,viewDidLoad之前.
3. initWithNibName:半代码化初始时使用. 完全IB化初始时不调用. 代码化初始时会通过init调用,且调用在[super init]中. 
4. awakeFromNib:awakeFromNib这个方法是一个类在IB中被实例化时被调用的.看了帖子发现大家都推荐使用viewDidLoad而不要使用awakeFromNib,应为viewDidLoad会被多次调用,而awakeFromNib只会当从nib文件中unarchive的时候才会被调用一次.实际测试中发现,当一个类的awakeFromNib被调用的时候,那么这个类的viewDidLoad就不会被调用了,这个感觉很奇怪.
5. initWithCoder,涉及到序列化,即对象自己解析文件时用到的,也可以用在视图控制器类半代码化实例时.比如,通过IB创建一个controller的nib文件,然后在xocde中通过initWithNibName来实例化这个controller,那么这个controller的initWithCoder会被调用.
6. 调用顺序: `[super init](可选)` -> initWithNibName(其实在super init中) -> int after self created -> loadView -> ViewDidLoad