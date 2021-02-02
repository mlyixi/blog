---
title: '解析超级虚拟实验室eve Ng的工作模式'
date: "2021-01-11T13:39:02+08:00"
tags: ["eve-ng"]
categories: ["network","Linux"]
toc: true
---

转载自<https://www.douban.com/note/629657625/>

近些年来，基于虚拟环境的网络设备或者实验室层出不穷。其中最为成功的虚拟实验室产品线以iou-web，unetlab，以及后来继承者eve-ng最为成功。因为原UNetlab的环境依旧在更新，因此现有比较吸引眼球的实验室就有UNetlabv2和eve-ng俩个了。

在对eve-ng有了一定了解后，有必要将eve-ng的工作模式进行一个简单的说明，便于大家了解。这个解析从以下几个层面进行：架构，安装与更新，使用。

## eve-ng的安装
首先，安装eve-ng时，需要在网上下载上G的iso或者ova的虚拟机文件。如果我告诉你，无论是iso还是ova，你下载的都只是ubuntu的安装介质，你会相信吗？
事实也是如此，eve-ng的安装，只要依靠第一次开机时的联网更新所完成的。在安装介质里，与eve-ng有关的安装文件只有一个，唯一的一个，eve-setup.sh。
```zsh
eve.iso     /install/eve-setup.sh
eve.ova    /etc/eve-setup.sh

==build-installer-iso.sh====
cp /usr/src/eve-ng-public-dev/debian/txt.cfg /tmp/eve-iso/isolinux/
cp /usr/src/eve-ng-public-dev/debian/eve*.seed /tmp/eve-iso/preseed/
cp /usr/src/eve-ng-public-dev/debian/eve-setup.sh /tmp/eve-iso/install/
```
从eve.iso的封装脚本就可以看出，整版iso，实际有用的文件非常非常少，不超过100k.

在启动虚机或者iso时，会调用eve-setup.sh脚本，然后联网，开始下载并安装必备组件和eve-ng.deb本身。eve-ng.deb里包含了三部分内容，有核心的unetlab的服务器信息，必要的后台服务加载配置文件，还有一部内容是script脚本，这个脚本负责下载并配置dynamips和qemu等各种虚机。

当然了，必要的组件包括eve-ng本身都可以离线下载，上传到ubuntu内，进行单独更新
```zsh
# apt-get download eve-ng 
```

eve-ng的deb是不能单独下载的，必须通过ubuntu的apt-get完成，需要签名验证。
```zsh
wget -O - http://www.eve-ng.net/repo/eczema@ecze.com.gpg.key | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64]  http://www.eve-ng.net/repo xenial main"
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get -y download eve-ng

oot@eve-ng:~# dpkg -i eve-ng_2.0.3-60_amd64.deb
(Reading database ... 92388 files and directories currently installed.)
Preparing to unpack eve-ng_2.0.3-60_amd64.deb ...
Checking MySQL... done
Unpacking eve-ng (2.0.3-60) over (2.0.3-59) ...
Setting up eve-ng (2.0.3-60) ...
Processing triggers for ureadahead (0.100.0-19) ...
root@eve-ng:~#
```

eve-ng的安装文件，都保存在/opt的目录下。

事实上，unetlab，以及unetlabv2的安装也是这么玩的。
```zsh
# wget -q -O- http://www.routereflector.com/unetlab/install.sh
# ./install.sh manager
# ./install.sh worker TOKEN IP
```

作为linux平台上运行的eve-ng，各种平台的依附的不同依赖关系确实比较头疼而烦躁，由此指定ubuntu为基础平台我也能理解。但是，在搭载了ubuntu的介质后，启动时又下载重新和安装一次全新的ubuntu，我就很不能理解，哪你索性在发布时给个简版的ubuntu好了。此外，eve-ng并没有很好的去检测和分析依赖关系，安装和部署了相当多不必要的东西，实际上安装了ubuntu的整张介质。
====注：关于update，这是ubuntu本身的问题。每次系统启动，都会调用apt-get执行update动作，确实很烦，而且没任何好办法。=====

## eve-ng的架构

相比unetlab，eve-ng并没有太多独立开发的部分出来。从结构就可以看出来，它是以集合多种组件为一体为优势存在的。
```zsh
root@eve-ng:~# dpkg -l | grep eve-ng
ii eve-ng 2.0.3-59 amd64 A new generation software for networking labs.
ii eve-ng-dynamips 2.0.2-2 amd64 Dynamips files for Eve-NG.
ii eve-ng-guacamole 2.0.1-60 amd64 Guacamole for UNetLab/EVE-NG
ii eve-ng-qemu 2.0.2-16 amd64 QEMU files for Eve-NG.
ii eve-ng-schema 2.0.1-60 amd64 Database schema for UNetLab/EVE-NG
ii eve-ng-vpcs1.0-eve-ng amd64 vpcs Eve-NG compatible
ii linux-headers-4.4.14-eve-ng-ukms+4.4.14-eve-ng-ukms-brctl amd64  Header files Linux kernel,
ii linux-image-4.4.14-eve-ng-ukms+4.4.14-eve-ng-ukms-brctl amd64 Linux kernel binary image for version 4.4.14-eve-ng-ukms+
```
从架构来看，eve-ng本身。eve-ng本质上就是在unetlab的基础上进行了定制和改进，集成了dynamips,qemu和vcps，以及涵盖了现有的所有类型的模拟器和虚拟设备。其中guacamole和schema都是对unetlab定制和改进的具体措施。
![Alt](/network/eve-ng-files.webp "eve-ng的目录结构")

说明： 
1. control,  包含eve-ng安装的俩个重要脚本，preinstall和postinstall. 
preinstall:  对eve-ng安装必须的mysql进行初始化和用户创建，迁移，以及eve-ng-schema里的sql脚本等。       
postinstall: 安装后配置脚本，包括启动服务，apache2的配置，grub安装，plymouth配置，tap配置等。这是保证系统重启后能正常工作的必要前提。 
2. /opt/ovf:    EVE-NG setup脚本。主要包括密码，hostname及网络配置。该脚本通过/etc/init/ovfconfig.conf和/etc/profile.d/ovf.sh在系统启动时调用。 
3. /opt/unetlab/includes  unetlab基于php的管理控制脚本，诸如节点，网络，试验环境等。 
4. /opt/unetlab/includes/init.php    unetlab的初始化脚本。 
5. /opt/unetlab/includes/templates  不同类型节点的配置模板，如asav, junos,xrv等。 
6. /opt/unetlab/scripts    eve-ng平台的管理脚本。这些脚本主要是做发布所用，而不是实际管理和操作。 
7. /usr   eve-ng的主题目录，plymouth调用。 
8. /etc   各不同组件的配置文件。如apache2,lograte, systemd等。  

## eve-ng的使用
eve-ng缺省登录： http://x.x.x.x   username: admin   passwd: eve
eve-ng的使用确实比iou-web要快捷很多，也稳定很多。所有工作都可以通过web点击完成，速度很快，这点很好。
个人安装完成的eve-ng，不支持ftp传输，连ftp的命令都没有。文件传不上去，也拉不下来，太头疼了。很多必备的工具都需要手动配置
apt-get install ftp nano

unetlab的缺省登录：   root with default passwd  unl

## eve-ng平台迁移
能不能顺利迁移至其他平台，安装和更新脚本都是次要的，如何处理好其中组件的依赖关系才最重要。我们先看看UnetLabv2的依赖关系。
UNetLabv2 is based on:
```zsh
Docker: controller, routers and lab nodes run inside a Docker container;
Python: no more C, PHP or Bash, only Python 3;
Python-Flask + NGINX implement and expose APIs;
Memcached caches authentication for a better user experience;
Celery + Redis manages asynchronous long tasks in the background;
MariaDB stores all data/user and running labs;
Git stores original labs with a version control;
jQuery + Bootstrap will implement the UI as a single page app;
iptables + Linux bridge allow to connect to just started lab nodes via SSH;
IOL, QEMU and Dynamips run lab nodes.
再看看eve-ng的依赖关系
Pre-Depends: eve-ng-schema (>= 2.0.1-23), eve-ng-guacamole (>= 2.0.1-23), mysql-server
Depends: apache2, bridge-utils, build-essential, cpulimit, debconf-utils, dialog, dmidecode, eve-ng-guacamole, eve-ng-schema, genisoimage, iptables, lib32gcc1, lib32z1, libapache2-mod-php, php-xml, libc6, libc6-i386, libelf1, libpcap0.8, libsdl1.2debian, linux-image-4.4.14-eve-ng-ukms+, linux-headers-4.4.14-eve-ng-ukms+, logrotate, lsb-release, lvm2, ntp, openvswitch-testcontroller, openvswitch-switch, php, php-cli, php-imagick, php-mysql, php-sqlite3, plymouth-label, python3-pexpect, sqlite3, tcpdump, telnet, uml-utilities, eve-ng-dynamips (>= 0.8.6-44), eve-ng-qemu (>= 1.0.0-6), eve-ng-vpcs, zip, libguestfs-tools
```

现在可以确认的依赖组件有
必备组件： 
* php
* apache
* mysql/mariadb
* guacamole
* python 3
* bridge-utils/cpulimit/iptables/logrotate/lsb-release/tcpdump/telnet/uml-utilities
* openvswitch
* sqlite3
* libguestfs-tools

除了必要的依赖组件外，比较麻烦是各种组件需要有一个初始化配置，诸如apache, guacamole等需要处理。这些脚本要全抓出来，并梳理出来，比较麻烦。

eve-ng的移植流程
尝试从ubuntu到gentoo的移植，非常有挑战性，经历了重重困难，有所收获但是也有诸多问题。总体来说，这是一个不怎么成功的尝试，从实践的角度来看，收益颇丰，尝试还是非常值得的。
1. 安装必备的依赖程序。诸如sqlite, mysql, php, guacamole等。之后进行必要的配置和启动服务
    * mysql安装完成后，需要对mysql进行初始化。
    `emerge --config dev-db/mariadb    #设置mysql的root密码。`
2. 安装guacamole应用及配置。
    * 运行guacamole的preinst脚本
    *执行guac_install_v1.5.sh安装配置脚本，主要是执行guac的sql设置脚本。
    *运行postinstall脚本，设置guacamole的启动脚本。
    *guacamole要用tomcat提供www服务，因此需要特别设置tomcat的启动进程。

```zsh
#/usr/share/tomcat-8/gentoo/tomcat-instance-manager.bash --create --suffix eve-ng --user root --group root
Successfully created instance 'tomcat-8-eve-ng'
It's strongly recommended for production systems to go carefully through the configuration files at '/etc/tomcat-8-eve-ng'.
The generated initial configuration is close to upstreams default which favours the demo aspect over hardening.
root@anixter init.d # /etc/init.d/tomcat-8-eve-ng start
 * Starting tomcat-8-eve-ng ...                                           [ ok ]
root@anixter init.d # 
root@anixter init.d # rc-update add tomcat-8-eve-ng default
 * service tomcat-8-eve-ng added to runlevel default
root@anixter init.d # cp -a /etc/tomcat8/server-guacamole.xml /etc/tomcat8/server.xml &> /dev/null
```
3. 安装和配置eve-ng
* 运行eve-ng的preinstall脚本，主要设置eve-ng的数据库参数，诸如用户口令等。同时执行schema里的sql脚本，进行数* 据库初始化。
* 解包并复制eve-ng的文件到/opt指定位置
* 运行eve-ng的postinstall脚本，进行权限设置，各组件的自启动进程设定，以及虚拟站点的设置等。
* 解包eve-ng各组件的配置文件到/etc，/usr等目录。

4. 安装配置其他各组件到/opt对应位置。

5. 查找_lab.php和unl_fixpermission里，关于chown的代码并修改为正确的apache启动用户。
改代码太复杂了，不如直接改/etc/passwd和/etc/group
```zsh
#more /etc/passwd
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
unl0:x:32768:32768:Unified Networking Lab TID=0:/opt/unetlab/tmp/0:/bin/bash

#more /etc/group
www-data:x:33:
unl:x:32768:
```
注意：这里切记，www-data的用户一定要和运行apache的用户保持一致。

从理论上来讲，到此为止，所有的安装配置已经完成。事实上来看，到此已经可以通过浏览器访问并登录eve-ng，很多功能已经可以实现。
成功之处：
1. 可以通过PDO连接mysql，实现Eve-ng的登录控制。
2. 可以实现用户的增减等管理操作。
3. 可以实现对文件和目录的操作。
4. 增加或删除lab，编辑lab内的节点并进行配置。
5. 正确显示system status。
不足之处：
1. 在运行iol node时，提示Error when connect to AF_Unix，网上高人指点是iou的license设置不对。但是我的license没问题，单独跑iol都可以。直接挂在iol_wrapper下就过不去，肯定不管license事。
2. qemu采用特定的参数在ubuntu下编译，在gentoo下运行会报缺少动态库。

以上这俩个问题比较难处理，iol_wrapper和qemu都是编译过的二进制代码。qemu可以重新编译，但是iol_wrapper只能重新自己写个wrapper了。否则就只能运行一些windows或者原生的系统，无法做到全兼容。

在对iol_wrapper的源码分析后，可以得知，不同linux下对于connect返回值的不同，导致结果截然不同。
```c
/wrappers/include/afsocket.c
   while (connect(*remote_socket, (struct sockaddr *)&remote_addr, sizeof(struct sockaddr_un)) < 0) {
        rc = 2;
        UNLLog(LLERROR, "Error while connecting local AF_UNIX: %s (%i)\n",strerror(errno), rc);
        return rc;
```
到此为止，就可以清理战场了。

结论：在gentoo下移植eve-ng，也并非完全不可能，只是这个成本太高。如果一定要用eve-ng，后续可以单独分一个分区安装ubuntu，并在此基础上运行eve-ng。如果能处理好ubuntu对grub的复写问题，这将是最为简洁也最为可能的一种方式了。
