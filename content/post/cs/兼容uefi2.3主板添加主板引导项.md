---
title: "兼容UEFI2.3主板添加主板引导项"
date: 2014-08-25T14:08:52+08:00
tags: ["UEFI"]
categories: ["CS"]
toc: true
---

win和黑苹果双系统的引导其实是没有必要用clover的efi替换wbm的efi的。当然，如果你的主板支持uefi的情况比较好的，比如有图形界面的uefi引导，那么可以不用看下面。  添加主板的efi引导项(nvram)可以用UEFISHELL的bcfg, Linux的efibootmgr,苹果的bless, mactel-boot, M$的XXX－－我也不知道。
## 使用bcfg
CLOVER自带UEFISHELL,所以用bcfg比较方便

* 进入UEFISHELL
* 确定位置
如果使用当前硬盘的UEFISHELL,则`fs0:`就是了。如果用的是U盘，那么请确认是在哪个`fsX:`里，可以输入`fsX:`进行确认。
* 添加引导项
```bash
bcfg boot add 3 fsX:EFICLOVERCLOVERX64.efi "CLOVER"
```
* 补充修改项
```bash
bcfg boot dump -v
bcfg boot remove 3
bcfg boot mv 3 0
```

## 使用efibootmgr
以archlinux为例，使用refind作为UEFI引导

* 引导后加载efi变量
```bash
# modprobe efivars
```
* 挂载efi分区

```bash
# mount /dev/sda1 /mnt
```
* 安装引导项

```bash
# efibootmgr -c -g -d /dev/sdX -p Y -w -L "clover" -l '/EFI/Clover/CloverX64.efi' 
其中X指你的EFI分区所有的硬盘号，Y为分区号，如EFI分区在sda1(第1块硬盘第1分区），则X为a,Y为1

```

* 补充修改主板uefi引导项：

```bash
efibootmgr -v
efibootmgr -b xxxx -B #删除引导项xxxx, 如0000,0001.以上面命令显示的id号为准
efibootmgr -o x,x,x,x #改变引导顺序，如 3,2,1,4,0,以id号为准
```
## Windows加载EFI分区
```bash
mountvol z: /s
```