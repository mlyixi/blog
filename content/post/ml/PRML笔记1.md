---
title: "PRML笔记1"
date: 2014-10-15T14:09:52+08:00
tags: ["PRML"]
categories: ["ML"]
toc: true
---
# 

## 概念

* 训练集(training set)
* 测试集(test set)
* 检验集(validation set)
* 目标向量(target vector)，由目标变量组成
* 泛化能力(generalization)
* 预处理(preprocessing)，也叫特征提取(feature extraction)

# 分类 

根据目标向量的有无，我们分为`监督学习(supervised learning)`和`非监督学习(unsupervised learning)`。

监督学习按目标变量值的离散与否分为`分类(classification)`或`回归(regression)`。

非监督学习用于发现相似簇，也称为`聚类(clustering)`或`密度估计(density estimation)`或`可视化(visualization)`。

另外还有一种叫`强化学习(reinforcement learning)`，用于解决开发(exploration)和利用(exploitation)两难的收益最大化的决策问题。

## 例1 多项式曲线拟合(回归)
$\vec{x} =(x_1,\ldots,x_N)^T$, $\vec{t}=(t_1,\ldots,t_N)^T$
我们选择线性模型(多项式)
$$
y(x,\vec w)=\sum_{j=0}^M w_jx^j
$$
定义`误差函数(error function)`
$$
E(\vec w)=\frac{1}{2}\sum_{n=1}^N\{y(x_n,\vec w)-t_n\}^2
$$
其中对于$M$的选择我们称其为`模型比较或模型选择（model comparison/selection)`.模型比较是为了得到泛化能力强的模型,而不产生`过拟合(over-fitting)`或`欠拟合`的情况.为此,需要定义一个与样本量无关的评价指标,这里我们使用`均方根误差（root-mean-square error)`
$$
E_{RMS}=\sqrt{2E(\vec w ^*)/N}
$$

过拟合的处理
1. 随着训练集的增大,模型复杂度($M$)可适当增加而不会产生过拟合现象.
2. 正则化(regularization)
$$
\tilde E(\vec w)=\frac{1}{2}\sum_{n=1}^N\{y(x_n,\vec w)-t_n\}^2+\frac{\lambda}{2}||\vec w||^2
$$
其中$\lambda$是对正则项和误差函数权重的权衡.这种平方正则化的回归被称为岭回归(ridge regression).

## 贝叶斯视角
贝叶斯定理在机器学习下的表示:
$$
p(\vec w|\mathcal D)=\frac{p(\mathcal D|\vec w)}{p(\mathcal D)}p(\vec w)
$$
我们一般称$p(\mathcal D|\vec w)$ 为似然函数,而众所周知的最大似然估计就是最大化该似然函数.在机器学习中,我们也将似然函数的负对数形式称为错误函数,则最大似然估计和最小错误函数是同一概念.

## 模型选择



## 维度诅咒





## 决策理论



## 信息论

