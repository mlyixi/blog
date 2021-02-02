---
title: "ubuntu服务器初始配置"
date: 2020-09-30T10:11:52+08:00
tags: ["ubuntu"]
categories: ["Linux"]
toc: true
---

```zsh
sudo vi /etc/cloud/cloud.cfg.d/90_dpkg.cfg
sudo apt purge cloud-init snapd nano
sudo rm -rf /etc/cloud/
sudo rm -rf /var/lib/cloud/
sudo apt autoremove
echo "$USER ALL=(ALL:ALL) NOPASSWD: ALL" |sudo tee -a /etc/sudoers
echo "set completion-ignore-case on" |sudo tee -a /etc/inputrc
```

# 禁用systemd-resolved
systemd-resolved不好用，经常将复杂网络配置搞坏，可以考虑禁用掉：
```zsh
systemctl disable systemd-resolved
systemctl stop systemd-resolved
rm /etc/resolve.conf  # 删除符号链接
echo "nameserver 114.114.114.114" > /etc/resolve.conf
```

# 网卡
Ubuntu Server 启动时经常卡在:`a start job is running for the raise network`上。解决方法如下：
将
```zsh
auto eno1
iface eno1 inet dhcp
``` 
改为:
```zsh
allow-hotplug eno1
iface eno1 inet dhcp
```
# dhcpd
```zsh
sudo apt install isc-dhcp-server

/etc/default/isc-dhcp-server 
INTERFACES="eth0"

/etc/dhcp/dhcpd.conf
option domain-name "tecmint.lan";
option domain-name-servers ns1.tecmint.lan, ns2.tecmint.lan;
default-lease-time 3600; 
max-lease-time 7200;
authoritative;

subnet 192.168.10.0 netmask 255.255.255.0 {
        option routers                  192.168.10.1;
        option subnet-mask              255.255.255.0;
        option domain-search            "tecmint.lan";
        option domain-name-servers      192.168.10.1;
        range   192.168.10.10   192.168.10.100;
        range   192.168.10.110   192.168.10.200;
}

sudo systemctl start isc-dhcp-server.service
sudo systemctl enable isc-dhcp-server.service
```