---
title: "Linux多网卡"
date: 2018-12-04T14:08:52+08:00
tags: ["多网卡"]
categories: ["network", "Linux"]
toc: true
---

# Linux多网卡问题
## 问题描述
对于多块网卡的电脑，一个使用场景就是：将该电脑视作多个独立的收发系统，但共享一个cpu(多任务OS)。但这对于linux默认是不成立的：linux是一个弱终端系统模型（Weak End System Model)，上述使用场景的OS是一个强终端系统模型(SESM)。

区别是：
WESM, route(dst IP, TOS)-> gateway, interface
SESM, route(src IP, dst IP, TOS)-> gateway

即强终端系统强制规定从哪个网口进的数据包就从哪个网口出，而弱终端系统则可以换个网口发数据（即ip_forward, NAT网关服务器必需,参见《gateway.sh》）。

所以现在的问题是如何将linux从弱终端系统切换到强终端系统。

## 流程
1. 配置内核arp
sysctl net.ipv4.conf.all.arp_filter=1
sysctl net.ipv4.conf.all.arp_ignore=1
sysctl net.ipv4.conf.all.arp_announce=2

2. 配置内核路由
在/etc/iproute2/rt_tables中添加N个路由表，N是网卡个数，如：
100 rtable0
101 rtable1

ip route add $ETH0_NET dev eth0 proto static src $ETH0_IP table rtable0
ip route add default via $ETH0_GW dev eth0 proto static src $ETH0_IP table rtable0

ip route add $ETH1_NET dev eth1 proto static src $ETH1_IP table rtable1
ip route add default via $ETH1_GW dev eth1 proto static src $ETH1_IP table rtable1

ip rule add from $ETH0_IP table rtable0
ip rule add from $ETH1_IP table rtable1
可以将这些写入对应的文件，以实现开机启动，如/etc/network/ifup-pre or ifup-post(视Linux发行版的网络管理器而定）。
iptables -A INPUT -m addrtype --dst-type UNICAST -i eth0 ! -d $ETH0_IP -j DROP
iptables -A INPUT -m addrtype --dst-type UNICAST -i eth1 ! -d $ETH1_IP -j DROP




