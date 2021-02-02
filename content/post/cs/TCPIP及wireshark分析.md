---
title: "TCPIP及wireshark分析"
date: 2014-09-26T14:08:52+08:00
tags: ["wireshark"]
categories: ["CS"]
toc: true
---

# TCP
## 三次握手
### 开始:
* 客户发送Syn,Seq=X
* 服务发送Syn,Ack=X+1,Seq=Y
* 客户发送Ack=Y+1,Seq=Z
### 结束
* 服务发送Fin,Seq=X
* 客户发送Ack,Seq=X+1
* 客户发送Fin,Seq=Y
* 服务发送Ack,Seq=Y+1

# wireshark分析使用
## 过滤器
* 协议过滤:如TCP,UDP,ARP等
* IP过滤:如ip.src==<ip>,ip.dst==<ip>
* 端口过滤:如tcp.port==80,tcp.srcport=-98
* Http过滤:http.request.methed=="GET"
* 逻辑:AND,OR,==
## 封包详细信息
* Frame:物理层数据帧结构
* EthernetII:数据数据链路层以太网帧头部信息
* InternetProtocolVersion4:IP包头部信息
* TransmissionControlProtocol:传输层数据段头部信息
* HypertextTransferProtocol:应用层信息