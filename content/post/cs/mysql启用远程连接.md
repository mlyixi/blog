---
title: 'Mysql启用远程连接'
date: "2021-01-11T14:33:06+08:00"
tags: ["Mysql"]
categories: ["CS"]
toc: true
---
# windows下mysql启用远程连接
打开workbench,在左侧的Users and Privileges标签中将对应用户(root)的From Host值改为%
# ubuntu下mysql启用远程连接
同上，但还要配置/etc/mysql/mysql.conf.d/mysqld.cnf,注释掉bind-address这一行

