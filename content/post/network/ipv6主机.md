---
title: "IPv6主机"
date: 2020-11-04T14:08:52+08:00
tags: ["IPv6"]
categories: ["network","Linux"]
toc: true
---

## prefixpolicies
发现双栈网络下优先使用ipv6访问网站，查了一下是因为prefixpolicies的配置，默认为RFC3484标准配置。

### Windows
netsh interface ipv6 show prefixpolicies

### Linux
cat /etc/gai.conf