---
title: 《Optical Remote Sensing Image Change Detection Based on Attention Mechanism and Image Difference》笔记
abbrlink: 49056
date: 2022-11-21 10:41:12
tags:
  - RS
  - change detection
  - attention
  - difference enhance
categories: 论文
---

> 论文地址：[Optical Remote Sensing Image Change Detection Based on Attention Mechanism and Image Difference](https://ieeexplore.ieee.org/document/9254128)



## 摘要

### 动机

**为了建模高层和低层特征之间的内在相关性**

提出了一种由多个上采样注意力单元组成的密集注意力方法。该单元同时采用了上采样空间注意力和上采样通道注意力。该单元可以利用具有丰富类别信息的高层特征来指导低层特征的选择，可以利用空间上下文信息来捕捉地物变化的特征。而且还引入差异增强模块促使特征选择性地聚合。



## 主要工作

- 提出一种端到端的网络架构用于变化检测
- 提出了一种密切关注特征融合每个阶段的稠密注意力方法。在每个上采样注意力（UA）单元中，采用上采样空间注意力（SA）和通道注意力（CA）来同时捕捉空间和通道维度的变化信息。**UA单元可以很好地定位细节信息和纹理特征**
- 提出差异增强模块，可以直接将差异图像直接映射到新的特征空间，充分挖掘变化信息，生成变化稠密图





## 网络结构

该方法整体结构与U-Net类似，采用U-Net和DenseNet结合的方式，多的部分为上面一个分支：差异增强模块（该作者似乎很看好这个方法，在[TransUNetCD](https://qingy735.gitee.io/posts/28607.html)这篇论文中也是使用了相同的增强方法，具体介绍可以阅读文章）。在图像输入之初便将双时相图像进行concat处理，然后再Decoder部分进行**UA**模块处理，最后和差异增强模块部分的特征进行相乘得到最终的差异图。

作者解释在解码器中使用稠密注意力机制是因为可以更好地保留双时相遥感图像中变化区域的纹理和细节信息，不仅增加了抗干扰能力，而且提高了检测小物体变化的能力。

![image-20221121175405265](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121175405265.png)

### UA模块

论文中提到：高级特征具有丰富的语义类别信息，能够很好地为低级特征类别定位作指导。同时高级特征与用于对低级特征进行加权来获取精确的像素级合并。其中UA模块结构如下图所示。首先看左边，高级特征上采样后与低级特征进行**add**后经过一个卷积层输出空间维度的掩码 $mask_{SA}(1,W,H)$ ，之后与低级特征进行空间维度相乘得到经过空间注意力处理的特征；其次是右边，高级特征首先进行通道注意力处理，然后经过一个卷积层输出通道维度的掩码 $mask_{CA}(C,1,1)$ ，最后和上面的特征进行相乘；最后将上采样后的高级特征与上述输出进行concat处理，后得到最终输出。



![image-20221121180719472](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121180719472.png)



> 注：以下为个人理解

首先分析空间注意力部分：因为高级特征具有很丰富的语义信息，所以在上采样后与低级特征进行add操作是为了指导变化物体的位置信息，最后输出空间维度掩码指导低级特征是为了更好地定位变化目标。

而通道注意力部分：由于高级特征具有很好地语义信息，所以不同通道中可以很好地识别出背景和前景，于是进行通道注意力处理再与浅层特征相乘有利于区分变化区域和非变化区域（这里指背景）。

而为什么最后还要进行concat操作，是因为高级特征包含信息很丰富，上述操作只是利用了一部分，进行cancat操作可以更好地利用高级特征信息。



### DE模块

只是简单的求差分之后残差卷积处理，个人认为会包含很多无关变化（受光照、拍摄角度、季节等影响）。

![image-20221121183042531](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183042531.png)







## 实验

### 消融实验

![image-20221121183508942](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183508942.png)

### 对比实验



![image-20221121183522653](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183522653.png)





![image-20221121183603827](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183603827.png)



![image-20221121183622304](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183622304.png)

![image-20221121183758543](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183758543.png)

![image-20221121183731917](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183731917.png)

### 可视化实验

![image-20221121183639818](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183639818.png)



![image-20221121183658373](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183658373.png)



![image-20221121183816898](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183816898.png)

![image-20221121183834453](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121183834453.png)

> 注：原论文包含更详细的介绍，如果感兴趣可以阅读原文，链接见上方





















