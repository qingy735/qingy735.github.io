---
title: 《SwinSUNet：Pure Transformer Network for Remote Sensing Image Change Detection》笔记
abbrlink: 6529
date: 2022-11-24 17:55:38
tags:
  - RS
  - change detection
  - Transformer
  - Swin Transformer
categories: 论文
---

> 论文地址：[SwinSUNet: Pure Transformer Network for Remote Sensing Image Change Detection](https://ieeexplore.ieee.org/document/9736956)





## 摘要

### 动机

 **虽然CNN在CD领域取得了很大的成就，但由于卷积运算固有的局域性，无法捕获时空中的全局信息。** 而Transformer可以有效地提取全局信息，因此被用来解决计算机视觉任务。



### 主要工作

- 设计了一个具有孪生U-Net结构的纯Transformer网络来解决CD问题。SwinSUNet包含Encoder、Decoder、Fusion三个部分，均以Swin Transformer block为基本单元

  Encoder具有分层的Swin Transformer孪生网络结构，因此可以并行处理双时相图像并提取其多尺度特征

  Fusion模块主要负责将Encoder生成的双时相特征进行融合

  Decoder结构与Encoder不同的地方在于多了upsampling and mreging（UM）模块，用来恢复变化信息的细节





## 网络结构

Encoder首先使用patch partition和linear embedding将输入图像转为image token；随后，Encoder生成多尺度特征；最后Decoder使用Swin Transformer逐步预测变化信息。

![image-20221124180737892](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124180737892.png)



### Encoder

patch partition模块负责将图像转换成多个token，linear embedding模块负责将每个token的通道号映射到指定维度。



### Fusion

将双时相特征进行concat和现行映射后输入Swin Transformer block得到变化特征。



### Decoder

首先将输入特征进行上采样，然后与Encoder生成的对应尺度特征进行concat，然后为了更好地识别前景和背景使用通道注意力，最后经过一个线性映射处理得到细化后的变化特征。

![image-20221124181559106](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124181559106.png)

其中上采样部分采用Patch reshaping方法，没有扩充特征通道，从而减少了特征的冗余同时见少了计算量，有利于变化信息的提取。

![image-20221124181611531](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124181611531.png)



## 实验

### 对比实验

![image-20221124181942362](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124181942362.png)

![image-20221124181957008](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124181957008.png)





![image-20221124182009306](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124182009306.png)



![image-20221124182020672](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124182020672.png)



![image-20221124182033820](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124182033820.png)



![image-20221124182051293](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124182051293.png)





![image-20221124182105146](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124182105146.png)





### 消融实验

![image-20221124182152509](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124182152509.png)



![image-20221124182213838](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221124182213838.png)





## 总结

总的看下来最大的贡献应该就是使用出Swin Transformer模块了，至于参数量相比于纯CNN实现的孪生U-Net网络如何论文中没有提到。如果参数量能够大量下降的话应该算不错的方向，其次也证明了纯Swin Transformer能够处理CD任务。另一方面Path reshape算半个贡献点吧，后续实验上采样可以尝试使用该方法。











