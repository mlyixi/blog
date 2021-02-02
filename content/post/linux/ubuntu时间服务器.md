---
title: "ubuntu时间服务器"
date: 2017-11-29T10:11:52+08:00
tags: ["ubuntu", "ntp"]
categories: ["Linux"]
toc: true
---
# ubuntu ntp服务器配置与同步
1. sudo apt install ntp
2. 配置/etc/ntp.conf，如conf/ntp.conf所示
3. sudo systemctl restart ntp

# ubuntu ntp客户端配置
1. 现在一般使用systemd-timesyncd与ntp服务器同步
2. 配置/etc/systemd/timesyncd.conf中的NTP=192.168.10.13
3. sudo timedatectl set-ntp true
4. 重启后ntp同步,可通过timedatectl status查看

# windows ntp客户端配置
由于windows使用对称主动模式发送同步请求，在与非windows ntp服务器同步时会出现问题，需要将windows改为使用客户端对等模式发送请求。所以要重新配置windows time服务。

具体操作如timedate.bat所示，下载后管理员运行。

或设置成开机启动,流程如下:
1. win+r 启动“运行”界面
2. 输入shell:common startup 或shell:startup进入启动目录（前者为所有用户的启动目录，后者为自己的启动目录）
3. 将bat文件放入即可

