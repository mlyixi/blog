---
title: 'n2n使用'
date: "2021-01-04T14:03:05+08:00"
tags: ["n2n"]
categories: ["network"]
toc: true
---

# 基本概念
n2n是一个L2上基于P2P技术的VPN，实现了IPv4互联网上的一个overlay网络，该网络构建了一个虚拟网桥（虚拟交换机），从而形成一个虚拟局域网。

## 组织架构
* Supernode超级节点：它在edge节点间建立握手，或为位于防火墙之后的节点中转数据。它的基础作用是注册节点的网络路径，并为不能直通的节点做路由，能够直通的节点间通信，是P2P的。
* Edge节点：用户PC机上安装的用于建立n2n网络的软件。几乎每个edge节点都会建立一个tun/tap设备，作为接入n2n网络的入口。
* community社区：多个相同网段地址的edge节点形成的社区。

Edge节点间通过虚拟的tap网卡交互。每个tap网卡都是一个n2n edge节点。每台PC机可以有多个tap网卡，所以，在n2n网络中，同一台PC机可以属于多个网络。
![image](/network/n2n_arch.png "N2N网络架构图")

## 命令参数
下面是n2n 2.8及以上的命令行参数，注意2.8以下(2.6)没有-n参数，需要使用ip route命令添加静态路由实现。
```zsh
Welcome to n2n v.2.8.0 for Debian buster/sid
Built on Jan  5 2021 11:28:16
Copyright 2007-2020 - ntop.org and contributors

edge <config file> (see edge.conf)
or
edge -d <tun device> -a [static:|dhcp:]<tun IP address> -c <community> [-k <encrypt key>]
    [-s <netmask>] [-u <uid> -g <gid>][-f][-T <tos>][-n cidr:gateway] [-m <MAC address>] -l <supernode host:port>
    [-p <local port>] [-M <mtu>] [-D] [-r] [-E] [-v] [-i <reg_interval>] [-L <reg_ttl>] [-t <mgmt port>] [-A[<cipher>]] [-H] [-z[<compression algo>]] [-h]

-d <tun device>          | tun device name
-a <mode:address>        | Set interface address. For DHCP use '-r -a dhcp:0.0.0.0'
-c <community>           | n2n community name the edge belongs to.
-k <encrypt key>         | Encryption key (ASCII) - also N2N_KEY=<encrypt key>.
-s <netmask>             | Edge interface netmask in dotted decimal notation (255.255.255.0).
-l <supernode host:port> | Supernode IP:port
-i <reg_interval>        | Registration interval, for NAT hole punching (default 20 seconds)
-L <reg_ttl>             | TTL for registration packet when UDP NAT hole punching through supernode (default 0 for not set )
-p <local port>          | Fixed local UDP port.
-u <UID>                 | User ID (numeric) to use when privileges are dropped.
-g <GID>                 | Group ID (numeric) to use when privileges are dropped.
-f                       | Do not fork and run as a daemon; rather run in foreground.
-m <MAC address>         | Fix MAC address for the TAP interface (otherwise it may be random)
                         | eg. -m 01:02:03:04:05:06
-M <mtu>                 | Specify n2n MTU of edge interface (default 1290).
-D                       | Enable PMTU discovery. PMTU discovery can reduce fragmentation but
                         | causes connections stall when not properly supported.
-r                       | Enable packet forwarding through n2n community.
-A1                      | Disable payload encryption. Do not use with key (defaulting to Twofish then).
-A2 ... -A5 or -A        | Choose a cipher for payload encryption, requires a key: -A2 = Twofish (default),
                         | -A5 = Speck-CTR.
-H                       | Enable full header encryption. Requires supernode with fixed community.
-z1 ... -z2 or -z        | Enable compression for outgoing data packets: -z1 or -z = lzo1x (default=disabled).
-E                       | Accept multicast MAC addresses (default=drop).
-S                       | Do not connect P2P. Always use the supernode.
-T <tos>                 | TOS for packets (e.g. 0x48 for SSH like priority)
-n <cidr:gateway>        | Route an IPv4 network via the gw. Use 0.0.0.0/0 for the default gw. Can be set multiple times.
-v                       | Make more verbose. Repeat as required.
-t <port>                | Management UDP Port (for multiple edges on a machine).

Environment variables:
  N2N_KEY                | Encryption key (ASCII). Not with -k.
```


# 基本配置
基本配置保证edge与edge间能够点对点正常通信。

## supernode
超级节点需要有公网IP，或云上的弹性公网IP(EIP)。
```zsh
#/etc/n2n/supernode.conf
-l=7777     # 监听端口
```
## edge
```zsh
#/etc/n2n/edge.conf
-d=n2n0
-c=my
-k=密码
-a=10.0.0.*     # n2n地址
-l=1.2.3.4:7777 # supernode的公网IP:端口
```
一台主机即可以做supernode也可以做edge.

# 网关模式配置
通过n2n隧道到达远程网络或传输所有Internet通信是两项最常见的任务，实现这些功能需要正确的主机配置和路由配置。

可参考<https://github.com/ntop/n2n/blob/dev/doc/Routing.md>和<https://github.com/ntop/n2n/blob/dev/doc/n2n_gateway.sh>

## 基本概念
在这种情况下，首先要保证点对点能正常通信，即两个edge节点能相互访问。然后根据要到达的远程网络或Internet代理节点，确定哪个edge是server,哪个edge是client.

server是提供对远程网络/互联网的访问的edge节点，而client是连接edge节点。
## n2n配置
server和client均开启n2n转发
```zsh
#/etc/n2n/edge.conf
-d=n2n0
-c=my
-k=密码
-a=10.0.0.*     # n2n地址
-l=1.2.3.4:7777 # supernode的公网IP:端口
-r
```
## 主机配置
server和client均开启ip转发和NAT
```zsh
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -j MASQUERADE
```
上面的命令只能保证当前有效，重启后不再生效，可考虑永久化：
```zsh
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
```
iptables的永久化需要安装iptables-persistent
```zsh
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
```
它会将iptables配置保存在/etc/iptables/rules.v4和/etc/iptables/rules.v6中，可以通过如下命令查看是否生效：
```zsh
iptables -L -n -t nat
```
## 路由配置

为了简单起见，以下面的网络拓扑进行配置说明：

cloud: 局域网地址172.16.0.153， n2n地址10.0.0.1，出口ip 1.2.3.4

office: 局域网192.168.10.10/24, n2n地址10.0.0.10，出口ip a.b.c.d

现在由于office网络出口处加了多个防火墙（不受控，配置乱，下载易出错，网速不稳定），希望用n2n打通隧道，穿透防火墙。cloud节点为server, office主机192.168.10.10为client.

### n2n v2.8以上配置
1. server
/etc/n2n/edge.conf中添加下列配置，注意要在-r之前。
```zsh
-n=192.168.10.0/24:10.0.0.10
```
该配置是回程路由。

2. client
/etc/n2n/edge.conf中添加下列配置，注意要在-r之前。
```zsh
-n=0.0.0.0/0:10.0.0.1
```
该配置是去程路由。通过`route -n`我们可以看到
```zsh
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.1        128.0.0.0       UG    0      0        0 n2n0
0.0.0.0(default)192.168.10.1    0.0.0.0         UG    0      0        0 ens33
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 n2n0
1.2.3.4         192.168.10.1    255.255.255.255 UGH   0      0        0 ens33
128.0.0.0       10.0.0.1        128.0.0.0       UG    0      0        0 n2n0
192.168.10.0    0.0.0.0         255.255.255.0   U     0      0        0 ens33
```
即添加了第1条、第4条和第5条。第1条和第5条使用hack方式，即0.0.0.0/1覆盖default路由项而不删除default.第4条见下面说明。

### 使用ip route命令进行路由配置
对于n2n v2.8以下的版本，只能手动配置路由：
1. server:
```zsh
ip route add 192.168.10.0/24 via 10.0.0.10 dev n2n0 src 172.16.0.153
```

2. client:
```zsh
ip route add 1.2.3.4 via 192.168.10.1 dev ens33 src 192.168.10.10
ip route del default
ip route add default via 10.0.0.1 dev n2n0 src 192.168.10.10
#ip route add 192.168.10.0/24 dev ens33 src 192.168.10.10
```
第4条一般默认即有，如没有的话可考虑加上。
## 测试
使用如下命令可测试出口IP:
```zsh
curl ipinfo.io/ip
```

## 问题
在这种模式上，好像转发流量都是经过supernode节点进行的，而没有进行p2p,无法提升速度。

同时对丢包的容忍度不强，无法过长城。
> 参考链接
<https://www.meirenji.info/2018/02/03/N2N%E7%BB%84%E7%BD%91-%E5%AE%9E%E7%8E%B0%E5%AE%B6%E9%87%8C%E8%AE%BF%E4%B8%8E%E5%85%AC%E5%8F%B8%E7%BD%91%E7%BB%9C%E4%BA%92%E8%AE%BF-%E7%B2%BE%E7%BC%96%E7%89%88/>
<https://www.ntop.org/n2n/using-n2n-to-steer-your-internet-traffic-and-circumvent-restrictions/>