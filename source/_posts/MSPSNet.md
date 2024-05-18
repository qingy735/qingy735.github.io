---
title: >-
  《Deep Multiscale Siamese Network With Parallel Convolutional Structure and
  Self-Attention for Change Detection》笔记
tags:
  - RS
  - change detection
  - self attention
  - parallel convolutional structure
categories: 论文
abbrlink: 53542
date: 2022-10-11 10:32:35
---

> 论文地址：[Deep Multiscale Siamese Network With Parallel Convolutional Structure and Self-Attention for Change Detection](https://ieeexplore.ieee.org/document/9632564)



## 摘要

### 动机

**现有的许多变化检测算法在实际应用中仍需要进一步改进，特别是在增加特征提取的有效性和降低模型计算成本方面。**

本文提出的方法在可接受的计算成本下具有出色的特征提取和特征集成能力。该网络主要包含三个子网络：深度多尺度特征提取、PCS进行特征集成、基于self attention特征细化。第一个子网络中设计了一个基于卷积块的深度多尺度孪生网络来描述不同时间图像在不同尺度上的特征；第二个子网络提出了一种PCS模型来综合不同时间图像的多尺度特征；第三个子网络构建SA模型，进一步增强图像信息的表示。



### 贡献

作者从特征提取和特征集成的有效性以及降低计算存储成本的角度出发，设计并提出了一种新颖的CD深度神经网络。

- 为了描述多尺度图像特征并提高对不同时间图像的特征提取能力，提出了一种基于设计的卷积块的深度多尺度孪生神经网络（SNN）
- 为了提升特征融合的有效性，提出了一种并行卷积结构（parallel convolutional structure，PCS）来整合不同时间的特征；此外构建了一个自注意力（SA）模型来进一步提高特征表示能力
- 提出了一种新的CD模型，在保证检测精度的同时，具有比之前更广的应用范围，大量使用3×3和1×1卷积层来减少参数数量和计算成本





## 网络结构

整个网络结构分为三部分：共享权重的特征提取模块、基于PCS的特征聚合模块、基于SA的特征细化模块。为了获得图像信息的最优表示，设计了深度多尺度共享权重特征提取器，从不同的时间图像中提取特征。PCS的提出是为了实现不同时相图像的有效特征融合，SA模块为了使特征更具有判别性。网络首先生成多尺度多深度的特征，然后从底向上分别输入PCS和SA模块处理，最后经过卷积层输出变化图。

![image-20221123190808555](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123190808555.png)





### 基于深度多尺度SNN的特征提取

设计网络的初衷是保证提取特征的有效性和非冗余性，使模型能够以较低的参数量和存储成本实现图像的最优表示。因此，作者提出了一个具有新卷积块和大量3×3和1×1卷积层的深度多尺度SNN。多尺度提取通过引入自适应平均池化操作来实现。通过引入双卷积层和BN层，我们可以在每一层中挖掘更多的特征，这比单卷积层更适合复杂背景下的特征表示。

![image-20221123192156194](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123192156194.png)

![image-20221123192210652](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123192210652.png)



### PCS模块

时态特征融合是CD框架中非常关键的部分，它实现了不同时态图像在特征层面的关联，从而最大限度地提高了融合特征对变化的区分度。因此，时间特征整合的有效性决定了 CD 的性能。本文中提出了一种并行卷积结构（PCS）来完成多尺度特征的集成和图像信息的表示。首先将双时相特征进行concat处理，然后输入PCS模块；PCS主要是分组卷积构成，将卷积分为c组，并且每组的dilation参数设置不同。

![image-20221123192425473](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123192425473.png)



### SA模块

PCS生成的集成特性可以粗略地定位更改，但是它缺乏突出显示区域细节的能力。为了解决这个问题，我们进一步引入了SA模块，我们将 PCS 的输出输入到 SA 中以生成精炼的集成特征。





## 实验

### 对比实验

![image-20221123193345274](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123193345274.png)

![image-20221123193413102](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123193413102.png)

![image-20221123193427283](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123193427283.png)

![image-20221123193448349](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123193448349.png)



![image-20221123193502622](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123193502622.png)





### 消融实验

![image-20221123193521196](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123193521196.png)

![image-20221123193535930](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123193535930.png)





### 特征可视化

![image-20221123193610816](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123193610816.png)





## 总结

self attention模块处理特征有必要性，同时PCS模块对于特征的聚合有一定的提升效果。



> 想要进一步了解可以点击顶部链接查看原论文！



















