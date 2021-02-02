---
title: "iOSframe和bound的理解"
date: 2014-08-27T14:08:52+08:00
tags: ["iOS"]
categories: ["OSX"]
toc: true
---

翻译自[此文](http://ashfurrow.com/blog/you-probably-dont-understand-frames-and-bounds/)

在iOS中,viewController是一个控制view层次的对象.主要负责MVC中Model和View之间的交互.

viewController中有一个根视图self.view. self.view的范围是从状态栏下面到属于该控制器的下界(iOS7中已经改成透明状态栏了)。如下图，其下界就是tabbar上方。这是因为tabbar是属于tabbarcontroller的，层次上算是该视图控制器的superviewcontroller. 而上界为什么是状态栏下面就可以理解了。如果在interfacebuilder里看self.view的属性，则可以看出其frame是（0，20，宽度，高度）。注意，由于xib显示不了父控制器，就算是在ib中选择了bottombar，高度也是460.

![Alt](http://ashfurrow.com/img/import/blog/you-probably-dont-understand-frames-and-bounds/EC87F23A25274175B83A06630EDFABFB.png "在tabbarcontroller中的viewcontroller")

在ios4及以前,你只能在一个视图控制器中加载子视图,这样写代码管理这些视图显示就有些难了.

![Alt](http://ashfurrow.com/img/import/blog/you-probably-dont-understand-frames-and-bounds/1EA3627758A3487791BD14E0CF65502B.png "iOS4以前")

在ios5以后,你可以在一个视图控制器中加载子视图控制器中的视图.

![Alt](http://ashfurrow.com/img/import/blog/you-probably-dont-understand-frames-and-bounds/24979633EED54A5D9C25EFB6023C9EA6.png "iOS5以后")

下面说下frame与bounds的区别

![Alt](http://ashfurrow.com/img/import/blog/you-probably-dont-understand-frames-and-bounds/B8B143BB6681476794386EFDB88201A7.png "bound与frame区别")

蓝色视图是经过变换换视图,但是它真正的范围是虚线框所示的范围.所以frame是指视图在父视图坐标系中的坐标,也就是包括这个视图的最小竖直长方形,而bounds是指在自已坐标系中的坐标, 也就是蓝色原点相对于虚线框的位置,所以加载子视图的过程就了然了.