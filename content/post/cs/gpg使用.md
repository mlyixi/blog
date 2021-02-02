---
title: "gpg使用"
date: 2019-12-22T10:08:52+08:00
tags: ["gpg"]
categories: ["CS"]
toc: true
---
PGP和GPG经常搞乱，这里梳理下。

# PGP
英文全称为Pretty Good Privacy, 是一个商业软件，由Phil Zimmermann开发，最终被赛门铁克收购，需要付费。
# OpenPGP
OpenPGP是基于PGP软件的一种协议，定义了加密消息、签名、私钥和用于交换公钥的证书统一标准，RFC4880。

# GPG
GnuPG是符合OpenPGP标准的开源加密软件，PGP的开源实现。Linux下取名就是搞。

# 使用

* gpg -k #列出公钥
* gpg -K #列出私钥
* gpg -ea -r pubkey filename.asc # 用公钥加密
* gpg -o filename -d filename.asc # 解密文件