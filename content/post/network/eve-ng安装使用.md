---
title: 'EVE-NG安装使用'
date: "2021-01-08T15:42:08+08:00"
tags: ["eve-ng"]
categories: ["network","Linux"]
toc: true
---

eve-ng是基于ubuntu开发的网络仿真OS.由于eve-ng直接使用dynamips,qemu等虚拟化技术运行官方的固件,所以其仿真能力是最强的,和实物操作没什么分别.特别是网上有很多镜像,包括防火墙,vpn等,应有尽有,作为内网渗透环境来说最好不过了.
## 版本
1. eve-ng pro专业版,但需要购买,目前破解只到2.0.4-20/21
2. eve-ng ce社区版
eve-ng 2基于ubuntu 16.04开发, eve-ng 3基于ubuntu 18.04开发.
## 安装方式
1. OVA安装.可以部署到VMware Workstation/vSphere/VirtualBox等虚拟平台上,缺点是多了层虚拟化,性能不强.但优点是灵活.特别是Workstation,适合新手学习.
2. ISO安装.直接安装在物理机上,减少一层虚拟化.有两种安装方式,一是使用eve-ng提供的iso安装,二是使用ubuntu添加源安装.其实两者的镜像差不多都是一样的,首先是安装ubuntu server,然后运行eve-ng初始化脚本.
下面说明下第二步的操作命令:
```zsh
sudo su
passwd
printf "eve-ng-ce\n" |tee /etc/hostname
sed -i -e 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
systemctl restart  sshd
sed -i -e 's/GRUB_CMDLINE_LINUX_DEFAULT=.*/GRUB_CMDLINE_LINUX_DEFAULT="net.ifnames=0 noquiet"/' /etc/default/grub
update_grub
# 可能需要更改网卡名为eth0
# auto eth0
# iface eth0 inet dhcp
reboot
# 如果有多个网卡,重启后对应关系会出错,可以换换接口.
wget -O - http://www.eve-ng.net/repo/install-eve.sh | bash -i
# 国内下载比较慢,可以考虑代理下载deb到/var/cache/apt/archives/下
reboot
# 根据提示操作eve-ng配置
apt install gnome gnome-shell chromium-browser
# 如果磁盘空间有限,可考虑安装xfce
# apt install xfce4 lightdm
# printf "user-session=xfce\nallow-guest=false\n|tee -a /usr/share/lightdm/lightdm.conf.d/50-unity-greeter.conf
# 自动登录还可以添加autologin-user=<your-username>   autologin-user-timeout=0
add-apt-repository ppa:smartfinn/eve-ng-integration
apt-get update
apt-get install eve-ng-integration
```
### 引导和磁盘格式
1. 服务器考虑使用uefi安装,lvm磁盘(ubuntu要划分一个fat32分区并挂载到/boot/efi才能实现uefi引导,直接分个ESP分区不启作用).
<https://zanshin.net/2019/10/15/how-i-setup-arch-linux-using-uefi-and-an-encrypted-lvm/>
2. U盘或交付盘考虑使用uefi安装, ext4磁盘.
3. 也可以考虑将ova安装到硬盘(目前看没什么用)
<https://www.howtoing.com/converting-a-vmware-image-to-a-physical-machine>

### 确保安装完后有kvm_intel
lsmod|grep kvm
否则不能运行qemu虚拟机.
## 登录
在主界面登录上,除了帐号外,还有Native console, html5 console和html5 desktop.
* native console使用xdg-open打开,需要安装一些工具:
> linux: <https://github.com/SmartFinn/eve-ng-integration>
> windows: Windows client side pack.
* html5 console: 集成在eve-ng中,比较方便.console功能不太好
* html5 desktop: 图形化界面OS要求.

## 镜像
eve-ng主要是仿真cisco设备的,包括IOS, IOL,由于使用QEMU,所以也可以仿真*nix.
* Dynamips   .image /opt/unetlab/addons/dynamips
* IOL        .bin   /opt/unetlab/addons/iol/bin
* QEMU       .qcow2 /opt/unetlab/addons/qemu
### 官方维护列表
<https://www.eve-ng.net/index.php/documentation/supported-images/>
<https://www.eve-ng.net/index.php/documentation/howtos/howto-create-own-linux-host-image/>

## 基本操作
eve-ng图形化的基本操作比较简单,直接参考pdf书籍. 

比较麻烦的是对镜像的配置,对于不同厂商,配置指令都不一样,所以要多实验多记录.

## IOL路由器配置示例
### IOL的一个问题
今天用Console口连上路由器，结果不停地出现类似地如下错误信息：
%Error opening tftp://255.255.255.255/ciscortr.cfg (Timed out)
解决方法是在全局配置模式下输入以下命令停止AutoInstall
Router#enable
Router#conf t
Router(config)#no service config
Router(config)#exit
Router#copy running-config startup-config
Router#reload

### DHCP
```zsh
enable   //进入特权模式
conf t   //进入配置模式
interface ethernet 0/1    //选择e0/1接口
no shutdown      //开启接口
ip address 192.168.1.1 255.255.255.0    //配置ip
ip nat inside      //配置nat
exit
ip dhcp pool lan   //配置dhcp
default-router 192.168.1.1
dns-server 114.114.114.114
network 192.168.1.0 255.255.255.0
lease 2
exit
ip dhcp excluded-address 192.168.1.1 192.168.1.10
interface ethernet 0/0        //选择e0/0接口
ip nat outside    //配置nat
exit
ip route 0.0.0.0 0.0.0.0 172.31.0.1    //配置缺省路由，最后一个ip需要指定management的网关
access-list 199 permit ip 192.168.1.0 0.0.0.255 any   //配置动态nat
ip nat inside source list 199 interface ethernet 0/0 overload
end
write

sh ip interface brief
```

https://www.secshi.com/14378.html
### pfsense软路由
1. 关闭NAT功能
2. 设置静态路由表
3. checksum offload的一个bug
在system-advanced/networking中关闭hardware checksum offload.
## 自制镜像
1. ubuntu 18.04 desktop卡在/dev/sdaX clean上: 
一般是显卡驱动问题,使用virtio驱动: 
-vga virtio
`但还是有问题.`
使用非hwe内核好像就可以了: apt install linux-generic

2. 模板
模板文件在/opt/unetlab/html/templates/intel/中
比如在要修改linux默认使用virtio驱动,可以修改linux.yml中的qemu_options:
-vga std  改为 -vga virtio

3. 优化与软件
一般Windows镜像模板要禁用杀软,防火墙和自动更新.
win10比较麻烦,可参考https://blog.csdn.net/weixin_44545251/article/details/101017904
Ubuntu自身不带防火墙和杀软,更新可考虑(一般linux版本内可更新)
一般安装openssh-server, netcat, net-tools(winscp, putty)等工具方便测试.
分辨率设为1280x800
4. 压缩

/opt/qemu/bin/qemu-img convert -c -O qcow2 /opt/unetlab/tmp/0/45a13c01-6faa-465a-b90b-ec44c979685b/1/hda.qcow2 /opt/unetlab/addons/qemu/win-10-ltsc/hda.qcow2

5. 修复
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions


## 模板列表
没有对应镜像目录的下拉框也会显示,太长了,可以考虑不显示.
cp /opt/unetlab/html/includes/config.php{.distributed,}
## 显卡
https://www.jianshu.com/p/ffc37624e5ae
https://wiki.archlinux.org/index.php/QEMU/Guest_graphics_acceleration


## ping/udp/tcp
ping/ping6 ip
echo "This is my data" > /dev/udp/127.0.0.1/3000
telnet ip port
ss -lu
ss -lt
