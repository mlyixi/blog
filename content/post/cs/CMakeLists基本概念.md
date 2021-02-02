---
title: "CMakeLists基本概念"
date: 2016-11-26T14:08:52+08:00
tags: ["cmake"]
categories: ["CS"]
toc: true
---
## cmake 常用项，注意顺序
```cmake
cmake_minimum_required(VERSION 3.5)
project(pcap_tutorial) # 工程名
```

## 预处理，如变量，查找依赖等
```cmake
set(CMAKE_CXX_STANDARD 11) # 设置cmake变量
```
有些变量具有特别含义，如:

```cmake
set(CMAKE_BUILD_TYPE "Release") # 编译成Release(定义NDEBUG宏),相当于cmake参数-DCMAKE_BUILD_TYPE=Release
set(CMAKE_BUILD_TYPE "Debug") # 不定义NDEBUG宏

list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake/Modules) #cmake模块查找路径，工程优先
find_package(PCAP REQUIRED) # 调用findPCAP.cmake文件，查找libpcap库.

if (PCAP_FOUND) # 如果找到，就会定义头文件目录PCAP_INCLUDE_DIR和运行库PCAP_LIBRARY
    message(${PCAP_INCLUDE_DIR}) # 打印变量
    message(${PCAP_LIBRARY})
    include_directories(${PCAP_INCLUDE_DIR})
endif ()

## add build target(exe/lib): target = add + link
add_executable(pcap_tutorial main.cpp)

#add_library(pcap_tutorial STATIC|SHARED|MODULE main.cpp)

TARGET_LINK_LIBRARIES(pcap_tutorial ${PCAP_LIBRARY}) # 添加链接库一定得在target之后。

add_executable(pcap_tt main.cpp)
#add_library(pcap_tutorial STATIC|SHARED|MODULE main.cpp)
TARGET_LINK_LIBRARIES(pcap_tt ${PCAP_LIBRARY})

## gcc定义的头文件目录：C_INCLUDE_PATH, CPLUS_INCLUDE_PATH（优先级1）, 也可由-I参数指定（优先级0），系统默认(如/usr/local/include,
## /usr/include 优先级3）
## gcc定义的运行库目录：LIBRARY_PATH, 也可由-L参数指定，系统默认（/usr/local/lib, /usr/lib),优先级和头文件一样。

## add sub-projects
#add_subdirectory(sub_project)
```
