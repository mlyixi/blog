---
title: 'IOL_DHCP配置'
date: "2021-01-11T14:34:15+08:00"
tags: ["IOL","DHCP"]
categories: ["network","Linux"]
toc: true
---

# 初始化设置
刚进入系统会要求设置密码和管理端口，按提示操作即可。

# 配置网络接口
```zsh
Router> enable
Router# configure terminal
Router(config)# hostname Router
Router(config)# enable secret eve
Router(config)# no ip domain-lookup

Router(config)# interface ethernet 0/1 #这里接口名可能不同，按?查询。
Router(config-if)# ip address 192.168.5.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# end
Router# write
```

# 查看网络接口
```zsh
Router# show interfaces
```

# DHCP Server
```zsh
dhcp# configure terminal
dhcp(config)# service dhcp
dhcp(config)# ip dhcp excluded-address 192.168.5.1 192.168.5.100  //配置dhcp不分配的地址 
dhcp(config)#ip dhcp pool e01
dhcp(dhcp-config)#network 192.168.5.0 255.255.255.0  
dhcp(dhcp-config)# default-router 192.168.5.1 
dhcp(dhcp-config)#dns-server 192.168.5.1
dhcp(dhcp-config)# lease 3
dhcp(dhcp-config)# end
dhcp# write
```

```zsh
dhcp# show ip dhcp binding
```

# DNS Server
```zsh
Router(config)# ip dns server
Router(config)# ip domain-lookup
Router(config)# ip name-server 8.8.8.8
Router(config)# ip host 10.1.1.1
Router# clear hosts *
```
# SLAAC
```zsh
Router(config)# ipv6 unicast-routing
Router(config)# interface ethernet 0/1
Router(config-if)# description IPv6-SLAAC
Router(config-if)# ipv6 address 2048:5:5:5::1/64
Router(config-if)# ipv6 address FE80::1 link-local
Router(config-if)# ipv6 enable
Router(config-if)# end
Router# write
```
# DHCP V6 Server
## 早期概念

众所周知，在有关网络的研究中，我们从 IPv4 协议开始。我们了解设备上的静态 IP 寻址，然后了解主机上使用 DHCP 协议的动态 IP 寻址。现在在 IPv6 网络中，全局单播地址可以手动或动态配置。但是，对于 IPv6 网络，我们有两种动态分配方法：

* 无状态地址自动配置 (SLAAC)

* IPv6 的动态主机配置协议 (DHCPv6)

 

如果简单概括 SLAAC，它是指主机无需 DHCPv6 服务器即可获得 IPv6 全局单播地址的一种方法。SLAAC 的基础在于 ICMPv6，它比 IPv4 的 ICMP 更加可靠。基本上，SLAAC 使用以下 ICMPv6 消息来提供 IPv6 寻址：

* 路由器请求报文 (RS)：当客户端被配置通过 SLAAC 自动获取其寻址信息时，它将向路由器发送 RS 消息。该消息发送到所有 IPv6 路由器 FF02::2 的多播地址。这是 ICMPv6 消息类型 133。

* 路由器通告报文 (RA)：路由器发送 RA 消息以向客户端提供 IPv6 寻址信息。该消息包括前缀和局部段的前缀长度。路由器会定期（可在 4~1800 秒范围内配置）发送 RA 消息，或为响应 RS 消息而发送 RA 消息。默认情况下，思科路由器每 200 秒发送一次 RA 消息。RA 消息始终发送到所有 IPv6 节点 FF02::1 的多播地址。这是 ICMPv6 消息类型 134。


![image](/network/rs_ra.jpeg "RS和RA消息")


顾名思义，SLAAC 是一种无状态服务。这意味着没有服务器维护网络地址信息。它还不知道正在使用哪些 IPv6 地址以及哪些可用。而这正是 DHCPv6 发挥作用的时候。关于客户端如何能够自动获取 IPv6 寻址信息的决定将取决于 RA 消息中确定的信息。为此，我们将使用两个标志，即受管地址配置标志（M 标志）和其他配置标志（O 标志）。

通过使用 M 和 O 标志的不同组合，RA 消息确定以下三个寻址选项之一：

* SLAAC（仅使用 RA 消息）

* DHCPv6 无状态（RA 和 DHCPv6 消息）

* DHCPv6 有状态（仅使用 DHCPv6 消息）

请注意，尽管 RA 消息定义了客户端如何动态获取 IPv6 地址，但是客户端的操作系统可以选择忽略 RA 消息，而仅使用 DHCPv6 服务器的服务。

SLAAC 是思科路由器上的默认选项。RA 消息中的 M 标志和 O 标志都设置为 0（位）。在客户端上，IPv6 全局单播地址的创建方式是 RA 消息提供的前缀加上使用 EUI-64 的接口 ID，或加上装有 Windows 操作系统的 PC 中出现的随机生成值。

如果先前对设备中的 M 和 O 标志进行了修改，我们可以在接口模式下通过以下配置将接口重置为仅通过 SLAAC 工作：
```zsh
Router (config-if) # no ipv6 nd managed-config-flag
Router (config-if) # no ipv6 nd other-config-flag
```

# DHCPv6 无状态

DHCPv6 无状态选项通知客户端使用 RA 消息中的信息来获取 IPv6 寻址，但是 DHCPv6 服务器中提供其他配置参数。（例如，DNS 服务器的 IPv6 地址）。使用该名称进行定义是因为 DHCPv6 服务器不维护任何客户端状态信息（例如可用和已分配的 IPv6 地址的列表）。

对于 DHCPv6 无状态，O 标志设置为 1（位），M 标志保留为默认设置 0（位）。O 标志为 1 可通知客户端可以从 DHCPv6 服务器获得其他配置信息。

要修改从路由器接口发送的 RA 消息以指示 DHCPv6 无状态，请使用以下命令：
```zsh
Router (config-if) # ipv6 nd other-config-flag
```

# DHCPv6 有状态

此选项与我们在 IPv4 网络中研究的 DHCP 最相似。在这种情况下，RA 消息会通知客户不应使用其消息信息，并且必须从 DHCPv6 有状态服务器获取所有 IPv6 寻址信息和其他配置参数。因为 DHCPv6 服务器维护 IPv6 状态信息，所以使用此名称进行定义。（例如，分配的 IPv6 地址列表）

M 标志指示是否应使用 DHCPv6 有状态。这里不使用 O 标志，可以忽略。以下命令用于将 M 标志从 0 更改为 1，以指示 DHCPv6 有状态：
```zsh
Router (config-if) # ipv6 nd managed-config-flag
```
DHCPV6 - 其他特征：

* DHCPv6 具有 4 路协商过程。使用以下消息：

    - 请求：客户端使用多播地址 FF02::1:2 发送此消息以查找 DHCPv6 服务器，该地址是所有 DHCPv6 服务器的多播地址。

    - 通告：服务器用通告消息（单播）响应请求消息，以向客户端提供寻址信息。

    - 请求：客户端将此消息发送到服务器，以确认提供的地址和任何其他参数。

    - 应答：服务器以该消息结束该过程，该消息包含分配的 IPv6 地址和相应的配置参数。

* DHCPv6 服务器使用 UDP 端口 547，DHCPv6 客户端使用 UDP 端口 546

* DHCPv6 可以以两种形式实现：

    - 快速提交：DHCP 客户端通过快速交换两条消息（征求和应答）从服务器获取配置参数。

    - 正常提交：DHCP 客户端交换四条消息（征求、通告、请求和应答）。

    - 默认情况下，使用正常提交。

* 综上所述，DHCPv4 和 DHCPv6 之间的消息对比如下：
![image](/network/dhcp46.jpeg "DHCPv4和DHCPv6消息类型")

![image](/network/dhcp_options.jpeg "DHCPv6消息参数")

## DHCPv6 有状态分析拓扑

在此示例中，我们使用的是带有路由器和真实 PC 的以下拓扑。请注意，路由器 R1 上需要“IPv6 unicast-routing”命令，因为需要发送 ICMPv6 RA 消息。

![image](/network/dhcp_stateful.jpeg "DHCPv6 RA有状态")

```zsh
Device# configure terminal
Device(config)# ipv6 unicast-routing
Device(config)# ipv6 dhcp pool LAN1
Device(config-dhcpv6)# address prefix 2001:2:2:2::/64 lifetime infinite 86400
Device(config-dhcpv6)# dns-server 2001:2:2:2::1
Device(config-dhcpv6)# domain-name example.com
Device(config-dhcpv6)# exit
Device(config)# interface g1
Device(config)# ipv6 address 2001:2:2:2::1/64
Device(config)# ipv6 address FE80::1 link-local
Device(config)# ipv6 dhcp server LAN1
Device(config)# ipv6 nd managed-config-flag
Device(config)# no shutdown
Device(config)# end
Device# write
```
请注意，在我们的 DHCPv6 池配置中，未指定默认网关，就像使用“default-router”命令的 IPv4 示例一样。这是因为路由器通过我们在拓扑中所看到的 RA 消息自动将其自己的本地链接地址 (FE80::1) 作为默认网关发送。

在下面的“show”命令中可以看到，第一个位置处显示的是 DHCPv6 池的名称，第二个位置处显示已配置的参数，我们并没有看到任何输出，因为没有已收到 IPv6 地址的 DHCPv6 客户端。

```zsh
Device# show ipv6 dhcp pool
Device# show ipv6 dhp binding
```
DHCPv6 使用 DHCPv6 唯一标识符 (DUID) 来识别 DHCPv6 客户端和服务器。每个客户端只有一个 DUID，并且每个服务器也只有一个 DIUD。

在这个示例中，我们还看到其他“show”命令，其中显示有 DHCPv6 服务器的 DUID 以及 M 标志设为 1 的效果，M 标志设为 1 指示主机必须使用 DHCPv6 来获得具有可路由地址特征的全局单播 IPv6 地址。

Windows 系统 PC 默认情况下会生成一个随机的接口 ID 值，用于使用 SLAAC（而非 EUI-64）方法自动配置 IPv6。但是，我们可以使用以下命令在管理员模式下通过 Windows CMD 禁用：“netsh interface ipv6 set global randomizeidentifiers=disabled”

http://sd.ourvip.net:7000/CCNARoutingandSwitchingEssentials/course/module10/index.html#10.2.1.2

http://sd.ourvip.net:7000/
