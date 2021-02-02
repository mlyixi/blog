---
title: "vscode远程编辑"
date: 2018-12-30T10:11:52+08:00
tags: ["vscode"]
categories: ["Linux"]
toc: true
---

# remote-ssh
微软官方出的remote-ssh用于连接远程服务器，进行远程编辑。

## 要求
服务端有openssh-server, 客户端有openssh-client.(win/linux/osx)其实都有了。

1. 客户端上安装remote-ssh插件，左侧菜单栏多了个remote-explorer按钮；

2. 选择新建个ssh target，会要求提供服务端信息，提供并输入密码，要多次输入（连接、安装服务端）

3. 然后就是连接主机了，输入密码后直接就能查看用户目录。

该方式使用ssh建立连接，客户端其实还是需要安装vscode的。使用比较简单，就是详细说明了。

# code-server
与remote-ssh不同，code-server提供web远程编辑，从而不需要客户端安装vscode了。

## ubuntu16.04 +
1. 下载deb包
<https://github.com/cdr/code-server>

2. 直接安装deb
`dpkg -i *.deb`

3. 配置
主要是提供局域网访问，就不进行认证了。
```yaml
# ～/.config/code-server/config.yaml 
bind-addr: 0.0.0.0:7070
auth: none
user-data-dir: /home/was/code
extensions-dir: /home/was/code/extensions
password: 9929d59c774c97f6450990ab
cert: false
```
一般使用普通用户开启服务：

```zsh
sudo systemctl enable --now code-server@was
```

4. 插件
code-server的插件安装就比较麻烦了，不能在线安装（v3.8.0)。

需要先下载对应的vsix，然后在网页上安装。

注意：如果不搭建https，好多功能受限。感觉使用有限，不如remote ssh。