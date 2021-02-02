---
title: "Linux防火墙1"
date: 2014-09-25T10:08:52+08:00
tags: ["iptables"]
categories: ["Linux"]
toc: true
---

## iptables与netfilter
经常混用,不过iptables指的是防火墙的管理工具而netfilter指的是linux实现防火墙的内核模块.

## iptables结构
* 表(Tables)
现在一般有5种内建表,Filter表, NAT表,Mangle表,Raw表和security表,分别用于包过滤,网络地址转换,包重构,数据跟踪处理和MAC层安全处理.
* 链(Chains)
链可以看成是数据流结构中的节点,表和链其实是iptables的两个维度,而不是表包含链,链包含规则这种关系.不同的内建表针对不同的功能,所以包含不同的链.脱离数据流结构谈链是不切实际的,所以具体见下一段.
* 规则(Rules)
对同时流经表和链的数据包的一些规定,称为目标值,包括accept,drop, queue,return.

## 数据流结构
![Alt](/linux/iptables_dataflow.png "数据流结构")
1. 首先,当一个数据包进入网卡时,首先进入PREROUTING链,内核根据数据包目的IP判断是否需要转发(NAT)。
2. 如果数据包是发往本机的,它就不需要转发,所以进入INPUT链, 然后到达本机处理,再经OUTPUT和POSTROUTING输出.
3. 如果数据包不是发往本机的,它就会转发,这里的条件是内核开启转发.这是实现NAT的必要条件.在/etc/sysctl.conf文件中设置`net.ipv4.ip_forward=1`然后运行`sysctl -p`.

## 优先顺序
### 表
Raw>Mangle>NAT>Filter
### 链
* 入站数据流
从外界到达防火墙的数据包，先被PREROUTING规则链处理（是否修改数据包地址等），之后会进行路由选择（判断该数据包应该发往何处），如果数据包的目标主机是防火墙本机（比如说Internet用户访问防火墙主机中的web服务器的数据包），那么内核将其传给INPUT链进行处理（决定是否允许通过等），通过以后再交给系统上层的应用程序（比如Apache服务器）进行响应。
* 转发数据流
来自外界的数据包到达防火墙后，首先被PREROUTING规则链处理，之后会进行路由选择，如果数据包的目标地址是其它外部地址（比如局域网用户通过网关访问QQ站点的数据包），则内核将其传递给FORWARD链进行处理（是否转发或拦截），然后再交给POSTROUTING规则链（是否修改数据包的地址等）进行处理。
* 出站数据流
防火墙本机向外部地址发送的数据包（比如在防火墙主机中测试公网DNS服务器时），首先被OUTPUT规则链处理，之后进行路由选择，然后传递给POSTROUTING规则链（是否修改数据包的地址等）进行处理。

## 策略
当所有规则都匹配完后所要执行的规则,是ACCEPT或DROP掉.默认是ACCEPT.所以在制定规则时有两种考虑,第一种是DROP掉,只让想要的数据包通过.另一种是全部ACCEPT,只过滤掉不想要的数据包.一般而言实际中都采用第一种策略.

## iptables常用命令
* `iptables -t filter --list`: 显示filter表的规则
* `iptables -F`: 清空所有规则或`iptables -t NAT -F`
* `iptables-save>/etc/iptables.rules`: 保存现有规则
* `iptables-restore < /etc/iptables.rules`: 加载规则
* `iptables -S`: 显示某链的所有规则.
* `iptables -t filter -P INPUT ACCEPT`: 定义filter表的INPUT链的策略为ACCEPT.

## 管理和设置iptables规则
![Alt](/linux/iptables_cmd.jpg "iptables命令")
![Alt](/linux/iptables_filter.jpg "iptables过滤条件")
### command说明
* `-A, --append`: 新增规则到某个规则链中，该规则将会成为规则链中的最后一条规则。
* `-D, --delete`: 从某个规则链中删除一条规则，可以输入完整规则，或直接指定规则编号加以删除。
* `-I, --insert`: 插入一条规则，原本该位置上的规则将会往后移动一个顺位。
* `-R, --replace`: 取代现行规则，规则被取代后并不会改变顺序。
* `-L, --list`: 列出某规则链中的所有规则。
* `-F, --flush`: 删除某规则链中的所有规则。
* `-Z, --zero`: 将封包计数器归零。封包计数器是用来计算同一封包出现次数，是过滤阻断式攻击不可或缺的工具。
* `-N, --new-chain`: 定义新的规则链.
* `-X, --delete-chain`: 删除某个规则链。
* `-P, --policy`: 定义过滤政策。 也就是未符合过滤条件之封包，预设的处理方式。
* `-E, --rename-chain`: 修改某自订规则链的名称。

### parameter说明
* `-p, --protocol`: 比对通讯协议类型是否相符，可以使用 ! 运算子进行反向比对，例如：`-p ! tcp`，意思是指除 tcp 以外的其它类型，包含udp、icmp ...等。如果要比对所有类型，则可以使用 all 关键词，例如:`-p all`。
* `-s, --src, --source`: 用来比对封包的来源 IP，可以比对单机或网络，比对网络时请用数字来表示屏蔽，例如：-s 192.168.0.0/24，比对 IP 时可以使用 ! 运算子进行反向比对，例如：-s ! 192.168.0.0/24。
* `-d, --dst, --destination`: 用来比对封包的目的地 IP，设定方式同上。
* `-i, --in-interface`: 用来比对封包是从哪片网卡进入，可以使用通配字符 + 来做大范围比对，例如：-i eth+ 表示所有的 ethernet 网卡，也以使用 ! 运算子进行反向比对，例如：-i ! eth0。
* `-o, --out-interface`: 用来比对封包要从哪片网卡送出，设定方式同上。
* `--sport, --source-port`: 用来比对封包的来源端口，可以比对单一端口，或是一个范围，例如：--sport 22:80，表示从 22 到 80 端口之间都算是符合件，如果要比对不连续的多个端口，则必须使用 --multiport 参数，详见后文。比对端口时，可以使用 ! 运算子进行反向比对。
* `--dport, --destination-port`: 用来比对封包的目的端口，设定方式同上。
* `--tcp-flags`: 比对 TCP 封包的状态旗号，
* `--syn`: 用来比对是否为要求联机之 TCP 封包，与 iptables -p tcp --tcp-flags SYN,FIN,ACK SYN 的作用完全相同，如果使用 !运算子，可用来比对非要求联机封包。
* `-m multiport --source-port`: 用来比对不连续的多个来源端口.
* `-m multiport --destination-port`: 用来比对不连续的多个目的端口.
* `--icmp-type`: 用来比对 ICMP 的类型编号
* ...

### 目标`-j`
* `accept`: 将封包放行，进行完此处理动作后，将不再比对其它规则，直接跳往下一个规则链.
* `reject`: 拦阻该封包，并传送封包通知对方
* `drop`: 丢弃封包不予处理，进行完此处理动作后，将不再比对其它规则，直接中断过滤程序
* `redirect`: 将封包重新导向到另一个端口（PNAT），进行完此处理动作后，将 会继续比对其它规则。 这个功能可以用来实作通透式porxy 或用来保护 web 服务器。例如：`iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080`
* `MASQUERADE`: 改写封包来源 IP 为防火墙 NIC IP，可以指定 port 对应的范围，进行完此处理动作后，直接跳往下一个规则链.  SNAT的一种特殊形式，适用于像adsl这种临时会变的ip上.
* `LOG`: 将封包相关讯息纪录在 /var/log 中，详细位置请查阅 /etc/syslog.conf 组态档，进行完此处理动作后，将会继续比对其规则
* `SNAT`: 改写封包来源 IP 为某特定 IP 或 IP 范围，可以指定 port 对应的范围，进行完此处理动作后，将直接跳往下一个规则链. 扮演网关角色的机器和其他机器共享一个网络连接时使用,内网匿名访问外网.
* `DNAT`: 改写封包目的地 IP 为某特定 IP 或 IP 范围，可以指定 port 对应的范围，进行完此处理动作后，将会直接跳往下一个规则链. 用于将网络内部服务器掩藏起来.
* ...

## 实战
这里以处于防火墙内的网关服务器为例:

假设内网段192.168.1.0/24,内网卡eth0, 192.168.1.1,外网卡eth1, a.b.c.d.新建sh文件写入:

```zsh
# 清空表和链
iptables -F
iptables -X 
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -t raw -F
iptables -t raw -X
iptables -t security -F
iptables -t security -X

# 设置默认策略为接受
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -t nat -P PREROUTING ACCEPT
iptables -t nat -P OUTPUT ACCEPT
iptables -t nat -P POSTROUTING ACCEPT
iptables -t mangle -P PREROUTING ACCEPT
iptables -t mangle -P OUTPUT ACCEPT

# 启动回环
iptables -A INPUT -i lo -j ACCEPT

# 不过滤---filter表全通过
ptables -A OUTPUT -j ACCEPT
iptables -A INPUT -j ACCEPT
iptables -A FORWARD -j ACCEPT

# SNAT
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth1 -j SNAT --to a.b.c.d
```

# 说明
经过以上的配置,内网机器应该可以访问Internet并能访问网关上的各种服务了,这说明我们写的规则成功了.但是是不是意味着外网机器也能访问网关呢? 当然不是啦,网关上面还有网关呢,那边设置了防火墙阻止包进入的话我们是不能让外网机器访问到网关的.