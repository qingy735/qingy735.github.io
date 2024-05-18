---
title: >-
  《A Deeply Supervised Attention Metric-Based Network and an Open Aerial Image
  Dataset for Remote Sensing Change Detection》笔记
tags:
  - RS
  - change detection
  - CBAM
  - deeply supervised layers
  - metric learning
categories: 论文
abbrlink: 2293
date: 2022-11-23 10:24:44
---

> 论文地址：[A Deeply Supervised Attention Metric-Based Network and an Open Aerial Image Dataset for Remote Sensing Change Detection](https://ieeexplore.ieee.org/document/9467555)



## 摘要

### 动机

尽管近几年深度学习方法在变化检测中取得了很多实质性突破，但是**变化检测的结果容易受到光照、噪声、和尺度等外部因素的影响，从而导致检测图中出现伪变化和噪声**





## 主要工作

- 提出了一种基于 **深度监督（DS）** 注意力度量的网络（DSAMNet）



- 集成CBAM模块，在空间和通道方面获得更具有辨别力的特征；使用深度监督以实现更好的特征提取



- 提出了一个新的变化检测数据集SYSU-CD，并且模型在CDD和SYSU-CD上取得了很好的效果





## 网络结构

DSAMNet整体结构图如下所示，主要可以分为三部分：特征提取、度量学习、深度监督。特征提取部分主要采用ResNet网络进行多尺度多深度特征的提取；度量学习部分首先将输入特征经过CBAM模块处理得到更具有辨别性的特征，然后计算双时相特征之间的距离，最后根据**BCL Loss**进行优化；深度监督部分主要由一个深度监督层和损失计算构成，深度监督层负责将特征上采样到输入图像大小。

![image-20221123103155809](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123103155809.png)



### CBAM

CBAM模块主要进行通道注意力和空间注意力处理，空间注意力和通道注意力均采用最大池化和平均池化处理，不过一个在空间维度，另一个在通道维度。

![image-20221123104338589](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123104338589.png)

![image-20221123104548679](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123104548679.png)



### 度量学习模块

**为了充分利用低层特征中丰富的空间信息和高层特征中的语义信息**，首先将ResNet提取到的每一层特征进行通道统一（全部变成96，并且统一尺寸为输入图像的一半），然后将每一层的特征进行concat后输入到CBAM模块计算双时相特征之间的距离，最后和GT进行计算BCL Loss。其中BCL Loss计算如下图所示

![image-20221123104632659](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123104632659.png)



![image-20221123110522900](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123110522900.png)



### 深度监督模块

由于网络的中间层的训练是不透明的而且缺乏监督，所以中间层并不能很好地学习有效特征，所以使用深度监督模块来增强浅层特征的表征能力。主要针对前两层特征进行深度监督处理，文章中提到这两层同时具备丰富的空间信息和语义信息。在输出变化图后与GT进行Dice Loss计算来优化网络。

![image-20221123121048547](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123121048547.png)



## 实验

### 对比实验

![image-20221123122838424](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123122838424.png)





![image-20221123122847259](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123122847259.png)





### 可视化对比实验



![image-20221123122911508](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123122911508.png)





![image-20221123122929541](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123122929541.png)



### 消融实验

![image-20221123123000141](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123123000141.png)



![image-20221123123021835](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123123021835.png)



### 超参$\lambda$实验

![image-20221123123122491](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221123123122491.png)



## 总结

- 本文提出了一种新的深度学习处理变化检测的方法：使用CBAM模块直接从特征中学习到变化图，同时使用辅助深度监督模块。用于生成具有更多空间信息的变化图。



- CBAM可以有效地使特征更具有判别性，从而辅助度量模块的学习；同时深度监督模块可以很好地利用中间块特征中包含的信息，从而进一步改进度量模块学习到的变化图。



- 通过在CDD、SYSU-CD数据集上实验表示该方法结果优于其他最先进的方法。















