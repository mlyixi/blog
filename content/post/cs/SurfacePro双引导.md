---
title: "SurfacePro双引导"
date: 2018-09-03T14:08:52+08:00
tags: ["UEFI"]
categories: ["CS"]
toc: true
---

# 安装
首先，禁用可信计算模块TPM和修改安全引导为信任第三方签名。
其次，查看磁盘结构，将Raid-0(Intel Rapid ...)修改为AHCI。
然后，UEFI重装windows和安装ubuntu即可。

# 选择internal drive
否则会出现不能引导Windows的问题。
