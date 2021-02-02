---
title: "GPT修复记"
date: 2015-10-27T14:08:52+08:00
tags: ["UEFI"]
categories: ["CS"]
toc: true
---

现在用的SSD+HDD混合硬盘, GPT分区(OSX+Win7+ARCH Linux---好久没上Linux了). 

一真用的好好的. 不过好像有次当机后OSX不认HDD分区了.Win下没问题,我也就不当回事,接着用了.

今天升级OSX到10.11, 发现问题依旧,打开磁盘实用工具后发现系统认为HDD是MBR分区表了.哪里出错了呢?

尝试到win下用磁盘精灵,发现没法修复.还是到*nix下找答案吧.

```zsh
brew install gptfdisk;
gdisk /dev/disk1;
```
发现GPT: damaged.

开始修复:

r 进入修复模式

b 恢复

w 写入

OK,就是这么简单.