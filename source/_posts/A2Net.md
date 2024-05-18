---
title: >-
  《Lightweight Remote Sensing Change Detection With Progressive Feature
  Aggregation and Supervised Attention》笔记
tags:
  - RS
  - change detection
  - light weight
categories: 论文
abbrlink: 38331
date: 2023-02-22 14:07:37
---



> 原文地址：[Lightweight Remote Sensing Change Detection With Progressive Feature Aggregation and Supervised Attention](https://ieeexplore.ieee.org/document/10034814)



## 摘要

### 问题

尽管之前的基于CNN的方法能够取得不错的结果，但是模型的内存和计算成本太高，很难在实际中应用



### 工作

提出一种新颖的轻量级网络，它通过渐进式特征聚合和监督注意力，根据移动网络提取的特征识别变化

- 设计了一个neighbor aggregation module（NAM）来融合主干网络邻近阶段的特征，用来增强时间特征的表示能力
- 提出progressive change identifying module（PCI）从双时相特征中提取时间差异信息
- 设计了supervised attention模块（SAM）来对特征重新加权，以有效地从高层到低层聚合多级特征



## 引言

### 先前方法缺点

参数量和计算成本太大，不利于模型在真实世界的部署实现



### 解决方法

1. 使用轻量的特征提取网络（MobileNetV2），同时为了弥补特征表示能力的不足，构建NAM来增强特征表示能力
2. 设计一个具有轻量化模块的decoder，PCI、SAM



### 主要贡献

1. 提出轻量化模型A2Net（3.78 M parameters and 6.02 G FLOPs），并且取得了sota
2. 提出NAM增强backbone特征表示能力
3. 提出PCI逐步找到不同特征级别的时间变化信息以准确识别变化对象，并且使用SAM来重新加权特征以实现渐进的特征聚合



## 方法

### 网络结构

首先通过骨干网络提取多尺度特征，然后经过**NAM**对特征进行增强，最后获得差分特征。后面先将差分特征输入**PCIM**中进行差异特征增强然后经过**SAM**对特征进行重新加权，输出预测图。

![image-20230224103752830](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224103752830.png)



### NAM

由于backbone采用的是移动网络MobileNetV2，所以提取到的特征表征能力较弱，于是提出**NAM**对特征进行增强。具体操作为：所提取特征上下阶段特征分别进行最大池化和上采样后经过卷积与当前阶段特征进行**concat**，而后当前特征通过1×1卷积构成一个残差网络，保持当前阶段特征的主要信息。

![image-20230224113134285](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224113134285.png)





### PCIM

PCIM主要是为了成分挖掘差异特征中的变化信息，输入特征从大感受野到小感受野逐步挖掘变化信息。

![image-20230224113608045](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224113608045.png)



### SAM

在上一步通过**PCIM**已经充分挖掘了变化信息，但是由于高级特征缺少上下文信息的指导，容易造成噪声等无关信息。所以**SAM**旨在对特征进行重新加权，确保变化特征能够得到更多的关注，忽略无关特征。该模块首先经过1×1卷积然后通过激活层。最后得到变化图（值为0-1）和非变化图（用全1矩阵减去变化图），将两者进行**concat**，通过1×1卷积后作为权重map与原特征进行点乘，最后得到重加权的特征。

![image-20230224114240434](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224114240434.png)







## 实验

### 对比实验

该模型相比于其他的方法具有更好的效果。如图7第三行所示，相同的建筑物在不同的时间显示不同的颜色，从而导致一些与真实建筑物变化相反的无关变化，该方法能够较好地检测出变化区域；同时图8第六行，许多方法无法区分包含与建筑物外观相似的道路的伪变化。相反，所提出的方法可以很好地识别建筑物变化的边界和主体。

![image-20230224114942640](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224114942640.png)



![image-20230224114957987](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224114957987.png)



![image-20230224115022493](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224115022493.png)



![image-20230224115046949](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224115046949.png)



### 消融实验

![image-20230224115119249](https://gitee.com/qingy735/blogimg/raw/master/img/image-20230224115119249.png)



## 总结

1. **NAM**对于特征有较好的增强效果，特征交互类似思路可行
2. **PCIM**中利用不同感受野挖掘特征潜在信息也能够较好地运行，在对于较小分辨率特征是否可行？
3. **SAM**类似于自注意力机制，主要是通过变化图和非变化图生成特征权重图。











