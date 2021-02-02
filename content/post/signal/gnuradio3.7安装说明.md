---
title: 'Gnuradio3.7安装说明'
date: "2021-01-11T11:40:41+08:00"
tags: ["gnuradio"]
categories: ["signal"]
toc: true
---


# 环境要求
* ubuntu  16.04/18.04
* gnuradio 3.7.11-14
* uhd 003.010.001
* python 2.7


# 前置工具和库
```zsh
sudo apt install -y cmake git g++ libboost-all-dev libgps-dev \
python-dev python-mako python-numpy python-wxgtk3.0 python-sphinx python-cheetah swig \
libzmq3-dev libfftw3-dev libgsl-dev libcppunit-dev doxygen libcomedi-dev libusb-1.0-0-dev \
libqt4-opengl-dev python-qt4 libqwt-dev libsdl1.2-dev python-gtk2 python-lxml \
pkg-config python-sip-dev liblog4cpp5-dev python-zmq libcanberra-gtk-module
```
# uhd安装
```zsh
git clone https://github.com/EttusResearch/uhd
cd uhd
git tag -l
git checkout release_003_010_001_000
cd host
mkdir build && cd build
cmake ../ -DENABLE_GPSD=ON
make -j $(nproc --all)
make test
sudo make install && sudo ldconfig
echo "export LD_LIBRARY_PATH=/usr/local/lib">> ~/.bashrc
source ~/.bashrc
uhd_find_devices
```

# gnuradio安装
```zsh
## gnuradio3.9不将volk作为子模块了，而我们还在使用3.7,所以还需要同步子模块。
cd $HOME
git clone https://github.com/gnuradio/gnuradio
cd gnuradio
git checkout v3.7.11  # 支持到v3.7.14, maint-3.7
git submodule --init --recursive # 使用 volk v1.4
mkdir build && cd build
cmake ../
make -j $(nproc --all)
make test
sudo make install && sudo ldconfig
gnuradio-config-info --version
gnuradio-config-info --prefix
gnuradio-config-info --enabled-components
```

注意： `make test`一般会产生两个错误，qa_skiphead和qa_tag_share，目测是qa文件写错了,不影响使用：
<https://github.com/gnuradio/gnuradio/commit/d6e217cbc00ec0333ba030ab2d84c31b7dbb77d5>

## 添加系统关联及配置
```zsh
/usr/local/libexec/gnuradio/grc_setup_freedesktop  install
```
取消view->show variable editor

## gr-ieee802-11优化
* 系统
```zsh
printf 'net.core.rmem_max=33554432' >> /etc/sysctl.conf # 适配千兆网
printf "@$USER - rtprio 99" >> /etc/security/limits.conf # 适配优先级权限
```
重启

* 运行volk_profile
```zsh
volk_profile
```

# gr-foo安装（由于gnuradio更新了文件名，所以gr-foo要使用上一次提交）
```zsh
git clone https://github.com/bastibl/gr-foo.git
cd gr-foo
git checkout 2ba97c8
mkdir build
cd build
cmake ..
make -j $(nproc --all)
sudo make install
sudo ldconfig
```
# gr-ieee802-11安装
```zsh
git clone git://github.com/bastibl/gr-ieee802-11.git
cd gr-ieee802-11
mkdir build
cd build
cmake ..
make -j $(nproc --all)
sudo make install
sudo ldconfig
```

# Overruns and Underruns

Overruns and underruns(缓冲区过载和欠载）显示PC不能足够快地处理或产生数据，它产生的原因可能有以下几个：

* 数据传输带宽过小，直连USRP而不要连个10/100M交换机.
* PC性能不过关，不要用虚拟机.
* 运行`volk_profile`来确保采用CPU特定的指令加速.
* 使用不拥挤的信道，如5G信道.
* 检查帧检测是否工作，特别是当没有帧时它被触发（这经常是由直流偏置或本振泄漏导致的）