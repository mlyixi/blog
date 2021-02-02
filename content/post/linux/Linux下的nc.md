---
title: "Linux下的nc"
date: 2018-11-04T14:08:52+08:00
tags: ["nc"]
categories: ["Linux"]
toc: true
---

# nc
nc(netcat)有多个类似的工具
* netcat:原版,GNU,目前不支持ipv6
* netcat.openbsd:openbsd上的netcat,支持ipv6,需要libbsd0
* ncat:nmap项目中的改进,需要nbase,nsock,libpcap等库,支持ipv6
* socat:SocketCAT,netcat增强版,支持ipv6

# 编译
sudo apt install build-essential
sudo apt install flex byacc

./configure
make