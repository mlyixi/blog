---
title: 'NFS服务器'
date: "2021-01-14T15:42:57+08:00"
tags: ["ubuntu", "NFS"]
categories: ["Linux"]
toc: true
---

# 服务端
## 安装
```zsh
sudo apt update
sudo apt install nfs-kernel-server
```
配置文件一般不需要更改：
```zsh
/etc/default/nfs-kernel-server
/etc/default/nfs-common
```

## 准备共享目录
```zsh
|- /srv/nfs4
    |- backups
    |- www
```

```zsh
sudo mount --bind /opt/backups /srv/nfs4/backups
sudo mount --bind /var/www /srv/nfs4/www
```

```zsh
#/etc/fstab
/opt/backups /srv/nfs4/backups  none   bind   0   0
/var/www     /srv/nfs4/www      none   bind   0   0
```

## 共享
```zsh
#/etc/exports
/srv/nfs4         192.168.33.0/24(rw,sync,no_subtree_check,crossmnt,fsid=0)
/srv/nfs4/backups 192.168.33.0/24(ro,sync,no_subtree_check) 192.168.33.3(rw,sync,no_subtree_check)
/srv/nfs4/www     192.168.33.110(rw,sync,no_subtree_check)
```
指定哪些IP可以进行什么操作，如:
* 192.168.33.0/24下的所有主机可以在nfs4下读写(rw),同步写入硬盘(sync),不检查父目录权限(no_subtree_check)
* 192.168.33.0/24下的所有主机可以在backups下只能读，只有192.168.33.3才能写文件
* 192.168.33.110才能在www目录下读写

## 执行
```zsh
sudo exportfs -ra
sudo service nfs-kernel-server restart
```
如果使用防火墙，注意关掉或打开对应端口。如ufw防火墙：
```zsh
sudo ufw allow from 192.168.33.0/24 to any port nfs
```

# 客户端
## 安装
```zsh
sudo apt update
sudo apt install nfs-common
```
## 挂载
```zsh
sudo mount -t nfs -o vers=4 192.168.33.10:/backups /backups
sudo mount -t nfs -o vers=4 192.168.33.10:/www /srv/www
```
```zsh
# /etc/fstab
192.168.33.10:/backups /backups   nfs   defaults,timeo=900,retrans=5,_netdev	0 0
192.168.33.10:/www /srv/www       nfs   defaults,timeo=900,retrans=5,_netdev	0 0
```
sudo mount -t nfs -o vers=4 192.168.10.7:/home/was/nfs4 /home/so/share

## Windows客户端

1. 在`启用或关闭Windows功能`中打开`NFS客户端和管理工具`；
2. 在`区域`中的管理标签中设置`使用Unicode UTF-8提供全球语言支持`；
3. 重启；
4. 在`此电脑`的标签栏中`映射网络驱动器`添加nfs位置`\\ip(domain)\path\to\nfs`。
