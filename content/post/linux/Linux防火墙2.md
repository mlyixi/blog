---
title: "Linux防火墙2"
date: 2014-10-04T10:08:52+08:00
tags: ["iptables"]
categories: ["Linux"]
toc: true
---

在[上一节](../Linux防火墙1/)我们提到,我们配置的防火墙可能已经在别人配置的防火墙内了. 即使自己设置了数据包全部通过的规则,数据包也可能在外层防火墙被滤掉了.那么如何扫描别人给我们开放的端口呢?
这里我们介绍nmap.
# 安装
* osx: brew install nmap
* linux: 用你最喜欢的包管理安装吧
* windows: zenmap

# 功能架构图
![Alt](/linux/nmap.jpg "nmap功能架构图")

* 主机发现(Host Discovery)
* 端口扫描(Port Scanning)
* 版本侦测(Version Detection)
* 操作系统侦测(Operating System Detection)

## 主机发现
用于发现目标主机是否在线(Alive,处于开启状态),原理与 Ping 命令类似,发送探测包到目标主机,如果收到回复,那么说明目标主机是在线的。默认情况下NMAP会发送四种不同的数据包来探测:

* ICMP echo request
* TCP SYN to 443
* TCP ACK to 80
* ICMP timestamp request

### 参数
* -sL: 仅将指定的目标的IP列举出来,不进行主机发现。
* `-sn`: 只进行主机发现,不进行端口扫描。
* `-Pn`: 指定主机视作开启
* `-PS/PA/PU/PY`: 使用TCP SYN/ACK 或SCTP INIT/ECHO 方式进行发现
* `-PE/PP/PM`: 使用ICMP echo,timestamp和netmask方式进行发现
* -PO 使用IP协议包探测对方主机是否开启
* -n/-R: -n表示不进行DNS解析;-R表示总是进行DNS解析
* --dns-servers \<serv1[,serv2],...\>: 指定 DNS 服务器
* --system-dns: 指定使用系统的 DNS 服务器
* --raceroute: 追踪每个路由节点

### 示例
* nmap -sn -PE -PS80,135 -PU53 scanme.nmap.org
* nmap -sn 192.168.1.100-120

## 端口扫描
用于确定目标主机的TCP/UDP端口的开放情况,默认情况下,Nmap 会扫描 1000 个最有可能开放的 TCP 端口。
### 端口划分
* open:端口是开放的。
* closed:端口是关闭的。
* filtered:端口被防火墙 IDS/IPS 屏蔽,无法确定其状态。
* unfiltered:端口没有被屏蔽,但是否开放需要进一步确定。
* open|filtered:端口是开放的或被屏蔽。
* closed|filtered :端口是关闭的或被屏蔽。

### 扫描方式
* TCP SYN scanning(默认半开放扫描)
* TCP connection scanning: 全连
* TCP ACK scanning: ACK扫描
* TCP FIN/Xmas/NULL scanning: 秘密扫描方式
* UDP scanning: 用于UDP端口
* 其它

### 扫描方式参数
* -sS/sT/sA/sW/sM: 指定使用TCP SYN/Connect/ACK/Window/Maimon的方式来对目标主机进行扫描
* -sU: 指定使用 UDP 扫描方式确定目标主机的 UDP 端口状况
* -sN/sF/sX: 指定使用TCP Null,FIN和Xmas秘密扫描方式来协助探测对方的TCP端口状态

### 端口参数与扫描顺序
* -T0-5: 扫描时间,影响扫描成功率.默认T3,一般T4够用,T2会成N倍加长扫描时间
* -p ranges: 扫描指定的端口,如-p22; -p1-65535; -p U:53,111,T:21-25,80,S:9
* -F: 快速模式,仅扫描 TOP 100端口
* -r: 不进行端口随机打乱的操作
* --top-ports 1000: 扫描开放概率最高的1000个端口
### 示例
* nmap -sS -sU -T4 --top-ports 300 192.168.1.100

## 版本侦测
用于确定目标主机开放端口上运行的具体的应用程序及版本信息
### 参数
* -sV: 指定让 Nmap 进行版本侦测
* --version-intensity 7: 指定版本侦测强度(0-9),默认为 7。数值越高,探测出的服务越准确,但是运行时间会比较长
* --version-light: 指定使用轻量侦测方式 (intensity 2)
### 示例
* nmap -sV 192.168.1.100

## OS侦测
用于确定目标主机的OS
### 参数
* -O: 指定 Nmap 进行 OS 侦测
* --osscan-limit: 限制Nmap只对确定的主机的进行 OS 探测(至少需确知该主机分别有一个 open 和 closed 的端口)
* --osscan-guess: 大胆猜测对方的主机的系统类型。由此准确性会下降不少,但会尽可能多为用户提供潜在的操作系统
### 示例
* nmap -Pn -O 192.168.1.100


# 高级用法

# 实战
针对我们配置的网关,外网地址a.b.c.d,首先
`nmap a.b.c.d`
其返回的结果是
```zsh
PORT STATE filtered ports
80/tcp open http
7777/tcp open cbt
8888/tcp open suntanswerbook
9999/tcp open abyss
```
说明: 我们使用默认参数,对网关进行top1000个端口进行扫描,返回了4个开放端口80,7777,8888,9999. 但是这种默认配置有时不能返回正确的完全结果,比如某次扫描只返回了80,8888端口. 所以必须多扫描几次或加-T2参数.

`nmap -sV a.b.c.d`
其返回的结果是
```zsh
PORT STATE SERVICE VERSION
80/tcp open http nginx1.4.0
7777/tcp open http-proxy polipo http proxy
8888/tcp open ssh openSSH6.2
9999/tcp open http Transmission BitTorrent Management httpd
```

说明: 我们使用版本探测参数,成功扫描出网关配置的服务