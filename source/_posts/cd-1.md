---
title: 关于变化检测的相关介绍（一）
tags:
  - cv
  - RS
  - change detection
categories: Summary
abbrlink: 14212
date: 2022-09-28 16:19:38
---



> [《Deep Learning-Based Change Detection in Remote Sensing Images: A Review》](https://www.mdpi.com/2072-4292/14/4/871)

 

Change Detection（CD）：识别不同时期获取的图像中的差异

用途：fire detection, environmental monitoring, disaster monitoring, urban change analysis, and land management, etc

 

### 传统CD方法：

- pixel-based CD（PBCD）

1. 通过图像差异、主成分分析（PCA）、变化向量分析（CVA）等方法生成差异图像

2. 采用基于阈值的方法生成变化检测结果

- object-based CD（OBCD）

1. 提取双时相图像的特征并将他们划分为不同的语义信息

2. 分析双时相之间的差异

 

缺点：

对于传统方法而言很难挖掘大量高分辨率图像之间的潜在联系

![image-20220928162415577](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928162415577.png) 

 



### 基于深度学习的变化检测：

1. 单分支结构：直接将双时相图像concat作为模型输入。如基于U-Net

 

2. 双分支结构：共享权重，孪生网络

 

### 常用数据集：

- SAR Images

![img](https://gitee.com/qingy735/blogimg/raw/master/img/wps2.jpg) 

- Multi-Spectral Images：

  1. Wide-Area Datasets： 
![image-20220928162623433](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928162623433.png)

  2. Local-Area Datasets：
![image-20220928162924043](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928162924043.png)

 

- Hyperspectral Images
  ![image-20220928163017512](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928163017512.png)

  Primary challenges for HSI-CD：

  - Limited Labeled Data

  - High-Dimensionality
  - Mixed Pixels Problem

 

- Very High Spatial Resolution (VHR) Images
  ![image-20220928163201290](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928163201290.png)

  Technological challenge：

  - Limited Spectral Information

  - Spectral Variability
  - Information Loss

 

- Heterogeneous Datasets：
![img](https://gitee.com/qingy735/blogimg/raw/master/img/wps7.jpg) 

 

### Change Detection Architecture：

工作流程： 
![image-20220928163259801](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928163259801.png)

 

- Pre-Processing：有利于图像特征提取和图像分析结果。数学归一化

- Geometric Registration：修复图像几何失真

- Radiometric Correction：absolute calibration、relative calibration

- Despeckling

- Denoising