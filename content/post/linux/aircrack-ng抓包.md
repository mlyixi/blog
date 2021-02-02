---
title: "aircrack-ng抓包"
date: 2018-12-30T10:11:52+08:00
tags: ["aircrack-ng"]
categories: ["Linux"]
toc: true
---

# optional rename interface name:
echo 'SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="02:01:02:03:04:05", ATTR{dev_id}=="0x0", ATTR{type}=="1", NAME="wlx0"'|sudo tee -a  /etc/udev/rules.d/70-persistent-net.rules

# 安装
sudo apt install aircrack-ng

# start monitor mode
sudo airmon-ng start wlx0 [channel] #在信道channel上监听(mon0)，不指定信道全监听但不能用于aireplay

sudo airodump-ng mon0 # 监听所有ESS

sudo airodum-ng mon0 -c 1 --bssid bssid #监听ESS下的station,得到mac

sudo aireplay-ng -0 1000 -a 7E:F7:E6:C8:0B:E7 -c BC:83:85:EE:35:AB mon0 # 在mon0的信道上进行解关联解认证攻击

# start wireshark and select mon0


# scan all wifi
sudo iw dev wlx0 scan|grep SS


# airpcap+wireshark说明
radio only: 加FCS字段后wireshark解析插件报格式错误，可能原因在于radiotap已经成为事实上的标准
radiotap: 可加FCS
ppi: 可加FCS

# 操作系统fcs相关
## linux 3.10.11以后可以通过ethtool查看和更改fcs开关(限于有线,无限一般都是不能更改的）
sudo ethtool -k eth0 
sudo ethtool -K eth0 rx-fcs on rx-all on

dev <devname> set monitor <flag>*
none:
fcsfail:
control:
otherbss:
cook:
active:
