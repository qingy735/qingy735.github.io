---
title: >-
  《TransUNetCD：A Hybrid Transformer Network for Change Detection in Optical
  Remote-Sensing Images》笔记
tags:
  - RS
  - change detection
  - Transformer
categories: 论文
abbrlink: 28607
date: 2022-10-10 19:37:26
---



> 原文地址：[TransUNetCD: A Hybrid Transformer Network for Change Detection in Optical Remote-Sensing Images](https://ieeexplore.ieee.org/document/9761892/)



## 摘要

### 问题

1. UNet由于卷积神经网络内在的限制，对全局上下文和长距离的空间联系的获取是不充足的
2. Transformer虽然能够获取长距离的特征依赖关系，但是缺乏low-level的细节而导致结果中位置信息的缺失



### 工作



> 提出UNet和Transformer相结合的方法TransUNetCD

1. 对通过卷积神经网络提取的特征进行encoder操作提取丰富的全局上下文信息
2. 经过decoder的连续上采样，通过连续的跳跃连接将其与高分辨率的多尺度特征连接去学习局部-全局语义特征，并恢复特征的全分辨率，实现精确地像素定位
3. 引入差异增强模块，生成包含丰富变化信息的差异图，通过对每个像素的加权和选择性地聚合特征，提高网络的有效性和准确性



## 引言

### 模型设计动机

1. 引入Transformer模块：获取全局上下文信息，提取出更具有代表性的深度特征，减少伪变化的影响
2. 提出差异增强模块DEM：通过对change map进行加权得到更具有判别力的特征表示



### 其他方法不足

- RNN：高分辨率图像光谱信息不频繁和时间信息不足
- Attention机制（channel、spatial）：只是在channel或者spatial维度增强特征，忽略了浅层特征的增强，不能充分地学习全局上下文并且计算量大





## 研究方法

### 网络结构

UNet通过跳跃连接机制实现深层特征和浅层特征的融合，实现特诊提取和细节恢复。将CNN特征提取部分的最后一层进行concat后输入到Transformer中进行high-level特征的进一步增强，获取更具有代表的深度特征，之后经过decoder部分进行连续下采样并与之前的浅层特征进行concat以补充丢失的浅层信息。最后与加权的差异图进行相乘得到最终的变化特征并输出change map

![image-20221011092006773](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011092006773.png)



#### Cascading Upsampling Decoder（CUD）

论文中提到的是由于连续的上采样会丢失low-level细节，所以为了弥补浅层特征的丢失，提出了连续上采样decoder。其本质就是将encoder产生的浅层特征与上采样后的深层特征进行concat后进行一系列卷及操作实现浅层信息的补充

![image-20221011092737068](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011092737068.png)



#### Difference Enhancement Module（DEM）

双时相图像的差异图能够清晰的展示出图像的变化，但是由于使用了无差异的像素计算，忽略了图像局部特征的差异，也就是不能识别出一些伪变化的情况，所以不能被直接使用。因此本文提出了一种基于SA的差异增强模块，用来识别差异图中的深度特征变化和背景

![image-20221011093314509](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093314509.png)





### 损失函数

> 使用简单的Dice loss和加权交叉熵损失$L = L_{wce} + L_{dice}$

![image-20221011093450683](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093450683.png)

![image-20221011093516106](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093516106.png)



## 实验分析

### CDD数据集

![image-20221011093610319](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093610319.png)

![image-20221011093718440](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093718440.png)

### DSIFN数据集

![image-20221011093807998](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093807998.png)

![image-20221011093831369](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093831369.png)



### LEVIR-CD数据集

![image-20221011093858075](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093858075.png)

![image-20221011093913833](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093913833.png)



### WHU-CD数据集

![image-20221011093944668](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011093944668.png)

![image-20221011094002616](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011094002616.png)



### Params和FLOPs

从图中可以看出虽然该模型参数量比较高，但是相比于其他方法，计算量并没有增加太多。*文中提到参数量太大的原因是使用ViT作为预训练导致的*

![image-20221011094106585](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011094106585.png)



### 学习曲线

可以看出来该模型相较于BiT方法，收敛速度更快并且具有更好的准确度。同时观察验证集实验可以看出来，该模型的泛化能力更强，具有很好的鲁棒性

![image-20221011094458650](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011094458650.png)





### 消融实验

从下图中可以看出来在进行特征融合操作中，concat对于结果的提升更加明显，**后续实验可以优先考虑concat**。同时也可以看出来单独的DEM模块对于模型提升并不是很大，但是和Transformer结合后提升效果有所增强，说明经过Transformer处理后的特征具有更好的判别效果。Transformer模块可以聚合全局信息，准确定位变化区域，并且对纹理、形状和大小特征更加鲁棒。DE模块的存在可以提高模型的精度，使模型在前期快速收敛。

![image-20221011094824489](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011094824489.png)

从下图对比可以看出来，该方法的鲁棒性比较好，在四倍缩放的情况下依然能够优于BiT。同时观察可视化结果可以看出来该方法对于细长目标的检测更加有效

![image-20221011095451796](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011095451796.png)

![image-20221011095437782](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221011095437782.png)





## 总结

1、模型性能好

2、模型的鲁棒性和泛化较于其他模型更加好（个人认为是UNet的跳跃机制导致的，使得深层特征包含更多的浅层信息，同时由于Transformer的存在深层特征更具有代表性，BiT缺少了UNet提供丰富浅层特征的部分）

3、对于有些颜色变化很大的物体变化检测误差比较大（个人认为是由于差异增强模块，单一的依靠像素级别的直接差异判别还是有所弊端，应该修改到特征级别）

![img](https://gitee.com/qingy735/blogimg/raw/master/img/wps1.jpg) 

4、小物体检测性能较差（道路、汽车等）（**浅层特征和深层特征的处理上需要给予更多地关注**，CUD模块似乎并不能很好的处理这方面问题）