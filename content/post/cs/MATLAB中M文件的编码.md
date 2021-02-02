---
title: "MATLAB中M文件的编码"
date: 2014-12-06T14:08:52+08:00
tags: ["MATLAB"]
categories: ["CS"]
toc: true
---

试着把windows下写的M文件拿到黑苹果下,结果打开后注释乱码了.

尝试着把文件改成UTF8编码,但是MATLAB还是乱码.看了下官方文档,发现MATLAB用的是windows那一套:根据你的首选语言确定编码,而不用unicode编码.

windows下我们都是安装的中文系统,默认编码GB2312.

OSX(Linux)下我喜欢用英文作为首选语言,资料国外的多啊,反正是unicode的,都能识别中文.

但是MATLAB就不干了,非得编码成US-ASCII等英文编码,于是就乱码了.

知道了原因,解决方法也很简单:把系统的首选语言换成中文.

别再相信feature函数,startup.m之类的了...