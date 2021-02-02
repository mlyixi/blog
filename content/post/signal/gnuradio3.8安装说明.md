---
title: 'Gnuradio'
date: "2021-01-11T11:42:12+08:00"
tags: ["gnuradio"]
categories: ["signal"]
toc: true
---
https://wiki.gnuradio.org/index.php/TutorialsGDB

# gnuradio 3.9
gnuradio 3.9竟然这么快就发布了。主要更新：

* 使用C++14
* 使用PyBind11替代SWIG
* Volk作为依赖而不作为子模块存在了

依赖：
1. python3.6.5
2. numpy 1.13.3
3. volk 2.4.1
4. cmake 3.10.2
5. boost 1.65
6. Mako 1.0.7
7. PyBind11 2.4.3
8. GCC 8.3.0 / Clang 11.0.0 / MSVC 1910（VS2017 15.0)
9. UHD 3.9.7


# gnuradio 3.8
gnuradio 3.8同时支持python2和python3, 基于QT5，所以源码编译有些区别。
* 使用C++11
* 新依赖：MPIR/GMP, Qt5, gsm, codec2
* 删除依赖：libusb, Qt4, CppUnit
* 同时支持python2和python3，也是最后一个支持python2的gnuradio版本了。
* 现代CMake
* VOLK版本从1.4上升到v2.0.0
* YAML替代XML

由于python2已经停止维护，所以就使用python3了。
## uhd deps
老版本的UHD默认是使用python2的，在这里改成python3
```zsh
sudo apt install libboost-all-dev libusb-1.0-0-dev doxygen python3-docutils python3-mako python3-numpy python3-requests python3-ruamel.yaml python3-setuptools cmake build-essential libgps-dev
```
python3的文档编译不通过，也没使用用，直接禁用掉：

```zsh
cmake -DENABLE_PYTHON3=ON -DPYTHON_EXECUTABLE=/usr/bin/python3 -DENABLE_GPSD=ON -DENABLE_MAN_PAGES=OFF -DENABLE_DOXYGEN=OFF -DENABLE_MANUAL=OFF ..
```

## gnuradio 3.8 deps
```zsh
sudo apt install git g++ libgmp-dev swig python3-sphinx python3-lxml doxygen libfftw3-dev libsdl1.2-dev libgsl-dev libqwt-qt5-dev libqt5opengl5-dev python3-pyqt5 liblog4cpp5-dev libzmq3-dev python3-yaml python3-click python3-click-plugins python3-zmq python3-scipy python3-pip python3-gi-cairo
```
```zsh
sudo -H pip3 install pyqtgraph numpy scipy
```
貌似gnuradio的安装目录没有默认进系统环境变量,添加上:
```zsh
echo 'export PYTHONPATH=/usr/local/lib/python3/dist-packages:/usr/local/lib/python3/site-packages:$PYTHONPATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/user/local/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export PYTHONPATH=/usr/local/lib/python3/dist-packages:/usr/local/lib/python3/site-packages$PYTHONPATH' >> ~/.profile
echo 'export LD_LIBRARY_PATH=/user/local/lib:$LD_LIBRARY_PATH' >> ~/.profile
source .bashrc
```
后面的编译环境差不多
```zsh
git checkout maint-3.8
git submodule update --init # gnuradio v3.8使用volk v2.0
mkdir build && cd build
cmake 
make -j $(nproc --all)
make test
sudo make install
sudo ldconfig
```