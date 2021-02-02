---
title: "ubuntu创建deb包"
date: 2020-09-31T10:11:52+08:00
tags: ["ubuntu"]
categories: ["Linux"]
toc: true
---
工作中经常要自己编译源代码，对于大型开源工程，每安装一台电脑就要重新编译一下，即浪费时间又容易出问题（git代码总在更新）。

即然我是一个喜欢用archlinux而不喜欢gentoo的人，就很想找到即可以改源代码，又可以分发的方法，对，就是打包---------我即要做一个码农，也要做个打包人。

由于主要操作系统是ubuntu，所以主要是deb打包。查了些资料，发现打包资料国内又少又乱，其实原因应该主要是上游乱--语言、编译工具链，这些就导致开发人员使用最合适项目的打包方式，于是乎，几乎每个项目的打包方式都不一样。

但其实打包的原理很简单，就像make install一样将文件复制到系统指定目录，打包就是将文件复制到一个`chroot`根目录，到时安装的时候复制再复制过去。最大的问题的，对于复杂点的工程，手动这样操作不能忍受= =。

# autotools工具链
Autotools 的组件是：
* automake
* autoconf
* automake
* make

使用autoconf工具链的项目最明显的特征就是configure.ac文件了（主要是c语言吧）。以n2n项目为例：
```zsh
n2n项目
├── doc
├── legacy
├── packages
│   ├── centos -> rpm
│   ├── debian
│   │   └── debian
│   ├── etc
│   │   ├── n2n
│   │   └── systemd
│   │       └── system
│   ├── openwrt
│   │   └── etc
│   │       └── init.d
│   ├── rpm
│   └── ubuntu -> debian
├── tools
├── win32
│   └── DotNet
└── wireshark
```

可以看到除了源代码，还有packages文件夹(名字可能是packages/packaging/debian啥的，都有可能)，打包文件夹下一般都有一些针对特定系统的配置文件和操作描述文件。

进到ubuntu目录，打开README，发现需要的依赖是:
```zsh
apt-get install debhelper fakeroot dpkg-sig
```
再看其它文件，发现有configure文件，好了，直接装上依赖并`./configure`吧。

## deb打包工具链
在autotools工具链上除了上面用到的，总结起来有以下一些：
1. debmake
2. debuild和dpkg-buildpackage
3. debhelper和dh-make

# cmake工具链
好了，除了autotools，我们知道常用的还有c/c++的cmake工具链。最明显的特征就是CMakeLists.txt文件了。

以ETTUS/UHD工程为例：
```zsh
uhd/host
├── cmake
│   ├── debian
│   │   ├── patches
│   │   └── source
│   ├── Modules
│   ├── msvc
│   │   ├── amd64
│   │   └── x86
│   ├── redhat
│   └── Toolchains
├── docs
├── include
│   └── uhd
├── lib
├── python
├── tests
└── utils
```
我们发现在cmake文件里有debian这样的发行版目录，说明编译应该是依赖于cmake的。cmake编译一般是新建一个build目录，然后在这里面编译：
```zsh
mkdir build && cd build
cmake ..
make
make install 
```
编译好后其实就可以打包了：
```zsh
cmake --build .
cpack
```
可能还要加下一些参数，可参考：
```zsh
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -DBUILD_TIFF=ON \
-DPYTHON_LIBRARY=$ANACONDA_HOME/lib/libpython2.7.so \
-DPYTHON_INCLUDE_DIR=$ANACONDA_HOME/include/python2.7/ \
-DPYTHON_EXECUTABLE=$ANACONDA_HOME/bin/python
```
# checkinstall
还有一个工具就是checkinstall，它好像就是采用最根本的原理，将`make install`的文件操作放到虚拟根目录中并打包：
```zsh
./configure
make
checkinstall
```
但是Makefile一般只是将二进制文件放到系统里，缺少一些配置，对cmake打包工具链的项目也不好用。

一般还是推荐使用开发者写的打包代码进行打包。

# deb包的一些操作
## 查看deb包
* dpkg -c *.deb
* dpkg -I *.deb

## 离线安装deb包
1. 方式1
* dpkg -i *.deb
* apt-get -f install #修复依赖
2. 方式2
如果是安装包不易下载,可手动下载至下面目录:
/var/cache/apt/archives
3. 方式3
使用gdebi *.deb安装.


# 创建rpm包
yum install rpm-build automake autoconf

## 编译源码
进入源码目录：
./autogen.sh
./configure
make

## 生成rpm
进入packages打包工程：
./configure
make
就会在~/rpmbuild下生成rpm文件，可忽略签名。

## 安装rpm
rpm -i *.rpm



参考链接：
1. <https://www.debian.org/doc/manuals/debmake-doc/ch08.en.html>
2. <https://zhuanlan.zhihu.com/p/141956373>
3. <https://blog.usejournal.com/creating-debian-packages-cmake-e519a0186e87>
4. <https://blog.hosiet.me/blog/2016/09/15/make-debian-package-with-git-the-canonical-way/>
5. <https://www.debian.org/doc/manuals/maint-guide/build.zh-cn.html>




