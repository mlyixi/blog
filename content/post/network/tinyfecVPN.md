---
title: 'TinyfecVPN'
date: "2021-01-15T11:25:24+08:00"
tags: ["vpn"]
categories: ["network"]
toc: true
---

编译无限制版

```zsh
#makefile中改变目标TARGETS=amd64
make nolimit_release
```
```zsh
#Server enable ip forward:
echo 1 >/proc/sys/net/ipv4/ip_forward

#setup SNAT rule:
iptables -t nat -A POSTROUTING -s 10.222.0.0/16 ! -d 10.222.0.0/16 -j MASQUERADE

#run tinyfecVPN server
./tinyvpn_amd64 -s -l 0.0.0.0:8855 --sub-net 10.222.2.0 --tun-dev tun100 --report 10 -k 1234
```

```zsh
#run tinyFecVPN client
./tinyvpn_x86 -c -r 44.55.66.77:8855 --sub-net 10.222.2.0 --tun-dev tun100 --keep-reconnect  --report 10 -k 1234

#add route rules
ip route add 44.55.66.77/32 via 192.168.99.1     #change 44.55.66.77 to your server ip; 192.168.99.1 to your default gateway.
ip route add 0.0.0.0/1 via 10.222.2.1 dev tun100
ip route add 128.0.0.0/1 via 10.222.2.1 dev tun100
```



