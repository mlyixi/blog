---
title: "Xcode子工程及cocoapods管理"
date: 2014-08-30T17:08:52+08:00
tags: ["iOS"]
categories: ["OSX"]
toc: true
---

iOS应用总会用到很多库,而苹果只允许使用第三方的静态库.所以,开发者不能制作简单的framework供其它开发者调用. 而且,静态库加入到自己的工程也是个比较麻烦的事情,特别是有些工程没有制作静态库的target,这时还要自己制作静态库目标.

# 自定义静态库

1. 新建自己的工程,git化.
2. 添加子工程: `git submodule add git://github.com/gitname/gitname.git`. 如果对应的工程有submodule, 同步git时可以用`git clone XXXX --recurcive`
3. 如果子工程没有制作静态库目标,制作:
	* 添加静态库目标,将创建以目标名为名的头文件和实现文件,删除实现文件,可以将子工程的头文件包含进目标头文件中.
	* 在`public headers folder`中设置`include/$(TARGET_NAME)`实现 `target/target.h`的引用. 其实不这样引用也可以.
	* 将头文件加入到外部可见或工程可见(私有可见无意), pch文件亦加入.
	* 将实现文件加入到`compile source`中.
	* 可以考虑将资源文件加入到`copy files`中
	* 编译
4. 将工程文件拖入父工程
5. 在父工程中加入静态库目标
6. 在`link binary with library`中加入.a文件
7. 在`build setting`中更改链接标志 `-ObjeC -all_load`
8. 在`header search paths`中添加头文件搜索路径: 也许不知道各个环境变量的值很难搞定这块,但是我会告诉你可以在`build phases`中加入script打印各环境变量的值么(env)? 注意,`TARGET_BUILD_DIR=BUILT_PRODUCTS_DIR`. 知道了这些变量值,再查看实际路径值,对应起来就是了.
9. 添加子工程的framework

好了,完成以上步骤,一个子工程就添加完毕了. 要添加多个子工程?重复之.

## cocoapods
上面的方法的确比较麻烦,不过所幸的是有了cocoapods可以管理子工程. cocoapods是用ruby写的,而在ml上的ruby比较老了,所以先安装版本控制器rvm,现在的10.9就不用了:

```zsh
$ curl -L https://get.rvm.io | bash -s stable --ruby
```
安装完后它会提示你运行命令启动新版本.

接着安装新版ruby,如:

```zsh
rvm install 1.9.3 --with-gcc=clang
```
要有option,不然安装cocoapods报错.ruby安装好后就是安装和使用cocoapods了:

* 安装
```zsh
sudo gem install cocoapods
```
* cocoapods支持的[Specs](https://github.com/CocoaPods/Specs/tree/master/Specs)
* 创建Podfile

```zsh
pod 'AFNetworking', '~> 2.0'  
pod 'ObjectiveSugar', '~> 0.5'
```
* 导入Specs

```zsh
pod install
```
* 用Xcode打开workspace文件