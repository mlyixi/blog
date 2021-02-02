---
title: "Archlinux网关安装与配置"
date: 2014-11-30T14:08:52+08:00
tags: ["Archlinux", "网关"]
categories: ["network", "Linux"]
toc: true
---

Archlinux不是滚挂还是滚挂. 试着升级了一下网关(好久没更新了),尝试了`pacdiff`,不过我显然没有注意到问题所在,虽然还是可以启动,但是好多systemctl service是红的,网络也不行了(netcfg被netctl/networkd替换了),一些服务也不能启动了(估计是group,passwd这些权限问题改乱了). 想想这多问题,还是重装吧.

## U盘安装
usbwriter直接写入U盘,也可以dd.

## 建立网络连接
### dhcp
不用配置
### 静态

```zsh
ip a
ip link set extern up
ip addr add externIP dev extern
ip route add default via gatewayIP
vi resolv.conf
```
啥,还是ping不通? 看看有没有解析出dns,解析了的话应该是网关禁用ping了,当然也可能是软件的一些bug,比如systemd的dhcpd服务,stop掉.

## 分区格式化
### mbr

```zsh
cfdisk /dev/sda
mkfs.ext4 /dev/sda1
mkswap /dev/sdaX
swapon /dev/sdaX
```

### GPT/UEFI

```zsh
cgdisk /dev/sda
...
```
## 挂载

```zsh
mount /dev/sda1 /mnt
mkdir /mnt/home, /mnt/boot
mount /dev/sda2 /mnt/home

# UEFI必须持载EFI分区到boot
mount /dev/sda3 /mnt/boot
```
## pacman
mirrorlist-&gt;china

## pacstrap安装

```zsh
pacstrap -i /mnt base base-revel
```

## fstab

```zsh
genfstab -U -p /mnt >> /mnt/etc/fstab
```
检查(比如我就检查出把win挂载到/mnt/home了,果断重新生成.

## 进入系统

```zsh
arch-chroot /mnt /bin/bash
```
## 系统配置

```zsh
vi /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc --utc
echo myhostname > /etc/hostname
vi /etc/hosts
mkinitcpio -p linux
passwd
useradd -m -G wheel -s /bin/bash guys
passwd guys
```
## 网络配置

现在systemd的networkd还在开发,果断不用(动不动改来改去,怎么受得了).用netctl(其实和netcfg差不多)

不喜欢自动分配的网卡名,果断放入/etc/udev/rules.d/10-network.rules

```zsh
cp /etc/netctl/examples/ethernet-static /etc/netctl/extern
vi extern
cp /etc/netctl/examples/ethernet-static /etc/netctl/intern
vi intern
netctl enable extern intern
```

## 引导

### MBR

```zsh
pacman -S grub os-prober
grub-install --target=i386-pc --recheck /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```
### UEFI

```zsh
pacman -S gummiboot
gummiboot --path=$esp install # $esp为EFI分区所在的设备名

# nano $esp/loader/entries/arch.conf
title          Arch Linux
linux          /vmlinuz-linux
initrd         /initramfs-linux.img
options        root=/dev/sdaX rw

# nano $esp/loader/loader.conf
default  arch
timeout  5
```
## 重启
```zsh
exit
unmount -R /mnt
reboot
```

## NAT

```zsh
sysctl -a | grep forward
sysctl net.ipv4.ip_forward=1
cat>/etc/sysctl.d/30-ipforward.conf<<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.default.forwarding=1
net.ipv6.conf.all.forwarding=1
EOF

iptables -t nat -A POSTROUTING -o internet0 -j MASQUERADE
iptables -A FORWARD -i net0 -o internet0 -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables-save > /etc/iptables/iptables.rules
systemctl enable iptables.rules
```
## ssh

```zsh
pacman -S openssh
vi /etc/ssh/sshd_conf
```

OK,基本配置好了.