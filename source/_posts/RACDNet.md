---
title: 《RACDNet：Resolution- and Alignment-Aware Change Detection Network for Optical Remote Sensing Imagery》笔记
abbrlink: 35686
date: 2022-11-25 11:52:16
tags:
  - RS
  - change detection
  - super-resolution images
  - feature alignment
  - attention
categories: 论文
---

> 论文地址：[Remote Sensing | Free Full-Text | RACDNet: Resolution- and Alignment-Aware Change Detection Network for Optical Remote Sensing Imagery](https://www.mdpi.com/2072-4292/14/18/4527)



## 摘要

### 动机

变化检测任务是基于等效分辨率的共同配准多时相图像。由于传感器成像条件和重访周期的限制，很难获得所需的图像，特别是在紧急情况下。此外，准确的多时相图像配准在很大程度上受到大量对象变化和匹配算法的限制。

### 主要思路

- 首先，提出了一种基于WDSR的轻量级超分辨率网络。为了更好地恢复高频细节信息，计算梯度权重并将其分配给不同的区域，从而迫使网络集中在难以重建的区域。
- 为了减轻过度平滑的影响，进一步引入了对抗损失和感知损失，以提高重建图像的视觉感知质量
- 在孪生U-Net中为了对齐双时态深度特征，可变形卷积单元 (DCU) 通过使用学习到的偏移量扭曲特征图来使用。
- 解码器中为了弥合编码器和解码器之间的语义鸿沟，嵌入了一个有效的注意单元（AU）。



### 主要工作

- 提出了一种新颖的超分辨率网络，它在恢复 RS 图像中的高频细节方面简单而有效。
- 提出了一种对齐感知 CD 网络，其中可以通过使用 DCU、ACU 和注意机制进一步引入双时态深度特征来显式对齐以提高 CD 性能



## 网络结构

整体网络结构分为两部分，第一部分为将图像转为高分辨率图像来凸显高频部分，使得特征中高频细节信息更加丰富。第二部分为特征提取和变化信息生成部分，主要采用孪生U-Net网络结构进行特征提取，并且在双时相图像融合的过程中采用DCU模块进行图像特征配准，**解决图像配准问题**

![image-20221125120756581](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221125120756581.png)





![image-20221125120956911](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221125120956911.png)







![image-20221125121009521](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221125121009521.png)



### DCU

首先将双时相特征进行concat然后通过卷积层输出各个坐标的位移矩阵（即当前像素位置应当如何调整，分为x、y两个方向），之后将位移矩阵与对应特征相乘便得到了配准之后的特征，后续就可以做对应操作了。

![image-20221125121232605](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221125121232605.png)

### AU

因为深层特征包含丰富的语义信息，能够很好地识别背景和前景，所以通过通道注意力得到深层特征的通道级别注意力来重新校准低级特征，不仅减轻了语义鸿沟而且还能很好地融合高级特征的语义特征和低级特征的细节信息。

![image-20221125121726955](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221125121726955.png)



## 总结

图像高频信息在后续实验中可以着重思考，并且特征对齐、配准等也有实验价值。





