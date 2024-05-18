---
title: 《SNUNet-CD：A Densely Connected Siamese Network for Change Detection of VHR Images》笔记
tags:
  - RS
  - change detection
  - siamese network
categories: 论文
abbrlink: 23553
date: 2022-10-03 19:00:00
---



> 原文地址：[SNUNet-CD: A Densely Connected Siamese Network for Change Detection of VHR Images](https://ieeexplore.ieee.org/document/9355573)



## 摘要

### 问题：

一直专注于深层变化语义特征的提取而**忽视了浅层信息的重要性（包含高分辨率和细粒度特征）**，于是经常导致**变化目标边缘像素的不确定性**和**小物体的缺失**。

### 工作

> 提出了一个稠密连接孪生网络**SNUNet-CD**来进行变化检测工作

**SNUNet-CD**通过encoder-decoder和decoder-decoder之间紧密的信息传输缓解了深层神经网络位置信息的丢失。并且提出了集成通道注意力模块**ECAM**来实现不同层次的特征的最终提炼。



## 引言

### 先前工作不足

**基于U-Net的网络结构在连续的下采样后会丢失准确的空间位置信息，这往往会导致变化目标物体的边缘像素不确定和小物体的丢失。**



### 动机

> 先前的研究已经表明浅层网络包含细粒度的位置信息，深层网络包含更多的粗粒度语义信息。

基于上述提到的先验知识，可以使用稠密孪生网络作用于变化检测。首先通过encoder获取high-level特征，然后经过decoder时添加浅层特征进行位置信息的补充。同时由于不同层次的特征还是有差距的，所以提出了**ECAM**模块来解决语义鸿沟问题

以下为该论文主要工作和目的：

1. 提出基于NestedUNet的稠密连接网络SNUNet-CD：缓解深层网络位置信息丢失
2. 提出ECAM：聚合和细化不同语义层次特征，一直语义鸿沟和定位误差
3. 通过大量实验对比：模型性能（F1-Score）和计算复杂度都优于其他取得SOTA的方法



## 研究方法

### 网络结构

由于使用孪生网络分别提取双时相图像特征，所以采用concat的方法对特征进行融合来保证信息的完整性。为了保存高分辨率和细粒度的位置信息，该模型在encoder和decoder之间使用dense skip connection机制。如图显示的，在对图像对进行下采样时将两个分支的特征进行融合，并将融合之后的高分辨率、细粒度特征通过跳跃连接传递和相应的decoder部分来实现对深层特征位置信息的补充。



![image-20221010183145012](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221010183145012.png)



![image-20221010183430449](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221010183430449.png)



### ECAM

生成的特征图$X^{0,1}、X^{0,2}、X^{0,3}、X^{0,4}$大小是相同的，但是具有不同的语义层次和空间位置表示。具体来说就是浅层输出特征具有更细粒度和精确的位置信息，深层输出特征具有更粗粒度和丰富的语义信息。由于语义鸿沟的存在，必须采用相应的方法来解决融合问题。ECAM通过调整对不同层次特征的关注度来实现对语义鸿沟的抑制（采用Channel Attention，CAM），具体计算流程如下：
$$
CAM(F) = \sigma(MLP(AvgPoll(F)) + MLP(MaxPool(F)))
$$

$$
M_{intra} = CAM(x^{0,1}+x^{0,2}+x^{0,3}+x^{0,4})
$$

$$
F_{ensemble} = [x^{0,1}+x^{0,2}+x^{0,3}+x^{0,4}]
$$

$$
M_{inter} = CAM(F_{ensemble})
$$

$$
ECAM(F_{ensemble}) = (F_{ensemble} + repeat_{(4)}(M_{intra})) \bigotimes M_{inter}
$$

![image-20221010185122202](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221010185122202.png)

最后经过一个1×1卷积输出2×H×W的change map



### 损失函数

> 采用加权交叉熵损失和dice loss结合

$$
L = L_{wce} + L_{dice}
$$

其中加权交叉熵损失为：
$$
L_{dice} = \frac{1}{H × W} \sum_{k=1}^{H×W}weight[class]·(log(\frac{exp(\hat{y}[k][class])}{\sum_{l=0}^{1}exp(\hat{y}[k][l])}))
$$

> 其中class表示0 or 1

Dice loss：
$$
L_{dice} = 1 - \frac{2 · Y · softmax(\hat{Y})}{Y + softmax(\hat{Y})}
$$

> 其中$\hat{Y}$代表change map，$Y$表示ground truth



## 实验分析

从图中可以看出SNUNet-CD相比于其他的方法能够取得很好的性能，并且在通道数不是很大时参数的大小相较于其他方法也处于不错的位置。

![image-20221010190005902](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221010190005902.png)

![image-20221010190451016](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221010190451016.png)



对于可视化实验分析，可以明显的看出，SNUNet-CD方法能够更加有效地描述变化目标的边缘，如图第三行对于道路的检测；同时对小目标的检测更加有效，如图中第一行车辆部分。

![image-20221010190359114](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221010190359114.png)



## 总结

该方法采用稠密连接网络的主要出发点是为了弥补多次下采样后浅层信息丢失情况，通过不断的浅层特征的叠加实现最终特征图的信息完整性，这里指大物体边缘信息和小物体自身。实验上也证明了通过特征叠加的有效性，但是可能会出现特征过多导致模型的泛化能力降低，论文中并没有给出相应的实验。同时，本文中提出的解决不同层次语义鸿沟的策略：ECAM，主要是通过通道注意力的方法对浅层和深层特征的关注部分进行自适应性调整后再融合，在上述的消融实验中也证明了ECAM模块的有效性。



**核心**：

1. 重视浅层特征的主要性，主要为**位置信息**等
2. 对于多尺度特征融合需要进行一定的手段处理，消除或抑制语义鸿沟















