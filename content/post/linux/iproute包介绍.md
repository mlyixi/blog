---
title: "iproute包介绍"
date: 2014-12-08T14:08:52+08:00
tags: ["iproute"]
categories: ["network", "Linux"]
toc: true
---

你现在看的linux书籍是否还在介绍linux下用ifconfig,windows下用ipconfig查询ip,route查看路由？这完全out了。现在主流linux发行版一般不再默认安装这些unix上的`net-tools`了，而安装`iproute2`包。

## iproute
iproute提供了ip命令供完整地查询网络相关信息.

```zsh
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  
OBJECT := { link | addr | addrlabel | route | rule | neigh | ntable | tunnel | tuntap | maddr | mroute | mrule | monitor | xfrm | netns | l2tp | tcp_metrics | token | netconf }

OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] | -f[amily] { inet | inet6 | ipx | dnet | bridge | link } | -4 | -6 | -I | -D | -B | -0 | -l[oops] { maximum-addr-flush-attempts } | -o[neline] | -t[imestamp] | -b[atch] [filename] | -rc[vbuf] [size]}
```
这里当然不能把所有的讲完,仅列举一些以供熟悉该命令.

## 构成
我们看到,ip与原有的linux命令有一些区别,主要加入了object和command这一个两项, 从而使它变得更复杂,但同时,其实也更好理解.

### 显示链路
`ip link list`或简写`ip l`

这里的link与interface是一样的. 它会显示所有定义的接口的状态,最大传输单元,队列,MAC等信息.

### 显示ip地址
`ip addr show`或简写`ip a`

这样就会追加显示ip地址了. 结合接口的状态,你可以获得更多信息,当然得有相关网络基础知识了.

### 显示路由
`ip route show`或简写`ip r`

这样就显示了内网的路由和通往外网的路由.

### 显示arp表
`ip neigh show`或简写`ip n`

reachable指在线的,stale指在册但下线了.
### 路由策略
`ip rule list`或简写`ip ru`

### 总结
从上面我们可以看出,对于object和command,在不引起歧义的条件下,可以省略后面的字母.

## 多重路由问题
我们假设一个局域网有两个isp出口(电信和教育网). 你当然可以架设两个软路由器,到时手动指定网关即可--我实验室用的就是这种(用的是windows服务器).但是不方便啊,不能有dhcp服务等问题很麻烦.

那么如何通过一个软路由器实现呢?假设$IF1是接电信网的接口名,$IF2是教育网的,对应的ip是$IP1,$IP2,连接到对应的$P1,$P2.
### 流量分割
* 我们得保证外网数据包从哪个isp进来就得从相同的isp出去
> 在/etc/iproute2/rt_tables中加入
> 
> ```zsh
> ip route add $Net1 dev $IF1 src $IP1 table T1
> ip route add default via $P1 table T1
> ip route add $Net2 dev $IF2 src $IP2 table T2
> ip route add default via $P2 table T2
> ```

* 我们得设置"main"表
> ```zsh
> ip route add $Net1 dev $IF1 src $IP1
> ip route add $Net2 dev $IF2 src $IP2
> ```

通过以上的学习,你是否发现这些命令和`net-tools`差别蛮大呢? `net-tools`的思想是unix的KISS原则,但是`iproute`是更像`iptables`,`tc`等复杂命令,所以`systemd`这种的产物的出生也就不足为奇了. Linux随时间将更远离unix!