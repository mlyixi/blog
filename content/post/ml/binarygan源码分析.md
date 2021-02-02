---
title: 'Binarygan源码分析'
date: "2021-01-11T11:38:16+08:00"
tags: ["GAN"]
categories: ["ML"]
toc: true
---
<https://github.com/salu133445/binarygan>

# 数据集
使用sharedarray生成数据集，通过
```zsh
python3 ./training_data/load_mnist_to_sa.py ./training_data/mnist/ --merge --binary
```
其实是将文件存放在了/dev/shm里，默认文件为/dev/shm/_binarized_mnist_x

# config开发模式
配置项都存放在config.py中了。

# 版本升级tf1.0->2.0
将import tensorflow as tf替换为

import tensorflow.compat.v1 as tf
tf.disable_v2_behavior()

即可

model: binarygan, gan
gan_type: gan, wgan, wgan-gp

# ld
apt install cuda-nvrtc-dev-10-2 libnvinfer-plugin-dev

# tfds
data, info = tfds.load('datasetn_name',with_info)
注意data中的格式完全由info指定，所以info必看。
data: dict
info: DatasetInfo

data字典格式由info的splits字段定义，如训练集和测试集(train/test)

train: Dataset/DatasetV1Adapter
test: Dataset/DatasetV1Adapter

数据集格式由info的features字段定义，如image和label，如此才得能得到x,y数据。

数据在某些维度上可能是1,需要处理：
tf.squeeze()
np.squeeze()

数据的类型也可能要转换
tf.cast()
np.narray.astype()












