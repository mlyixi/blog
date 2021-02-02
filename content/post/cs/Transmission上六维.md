---
title: "Transmission上六维"
date: 2014-12-26T14:08:52+08:00
tags: ["PT","IPv6"]
categories: ["CS"]
toc: true
---

重装了实验室网关(archlinux 3.17.4),发现transmission连不上六维的tracker了.之前(netcfg)明明是可以的.网络配置都是copy的哈,为什么换成netctl就不行了呢. 更奇怪的是我的ipv6-nat下的局域网电脑是可以连上的(ipv6好好的).

在六维上查下,发现都是老帖子,主要是说transmission要支持ipv6和六维的torrent hash. 尝试去看了下源码,发现完全outdated了.

查transmission官网,发现transmission连接tracker用的是curl,于是果断`curl -6 -v http://bt.neu6.edu.cn`,发现能找到其ipv6,但是连不上,ping6其它网站也是这样.基本确定是网络设置的问题.

netctl配置文件一般默认是外网:

```zsh
Description='internet link'
Interface=eth0
Connection=ethernet
IP=static
Address=('192.168.1.23/24' '192.168.1.87/24')
#Routes=('192.168.0.0/24 via 192.168.1.2')
Gateway='192.168.1.1'
DNS=('192.168.1.1')

## For IPv6 static address configuration
IP6=static
Address6=('1234:5678:9abc:def::1/64' '1234:3456::123/96')
#Routes6=('abcd::1234')
Gateway6='1234:0:123::abcd'
```

内网:

```zsh
Description='local link'
Interface=eth1
Connection=ethernet
IP=static
Address=('192.168.1.23/24' '192.168.1.87/24')
#Routes=('192.168.0.0/24 via 192.168.1.2')
Gateway='192.168.1.1'
DNS=('192.168.1.1')

## For IPv6 static address configuration
IP6=static
Address6=('1234:5678:9abc:def::1/64' '1234:3456::123/96')
#Routes6=('abcd::1234')
Gateway6='1234:0:123::abcd'
```
以前的典型静态ip配置是这样的.为什么routes可以注释掉? 因为理论上来讲,如果不指定routers,数据包会自动去找指定的default route--gateway,并把它当作一个route.但是这里貌似出问题了.

用`ip -6 r`查询发现外网的default route变成了一个奇怪的ip. 为什么会这样呢? 看了下内网的设置,发现内网的gateway6地址就是这个奇怪的ip.果断注释掉.重启OK