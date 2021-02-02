---
title: "MATLAB基础"
date: 2014-09-06T14:08:52+08:00
tags: ["MATLAB"]
categories: ["CS"]
toc: true
---

## 预定义名

* eps: 最小浮点数
* pi: $\pi$
* i/j: 虚数单位
* Inf/inf: 无穷大
* intmax/intmin: 最大、最小正数
* realmax/realmin: 最大、最小实数

## 函数
* accuumarray
* cross
* cumsum
* cumprod
* dot
* prod
* sum
* tril, triu

## 变量管理
* clear
* clc
* clf
* 工作空间图形化操作
* who/whos
* save/load
* ans

## 工作目录
* cd  
* dir
* doc 
* help
* which
 
## 运算
* 加法: +
* 减法: -
* 乘法: *
* 点乘: .*
* 左除: / 右除\
* 点除: ./
* 幂: ^
* 转置: .'
* 共轭转置: '
* 关系运算: ==, ~=,>,>=,<,<=
* 逻辑运算: &,|,~
* all: 是否为全0;  B=all(A),B=all(A,dim)
* any: 是否有非0; B=any(A),B=any(A,dim)

## 矩阵生成
* 冒号生成: a : inc : b
* 线性函数: linspace(a,b,n)等差,logspace(a,b,n)等比
* 随机函数: rand(n,m), randn(),randi(),randperm(),randsrc()
* 矩阵函数: diag(),eye(),gallery(),magic(),ones(),zeros()