---
title: 《HFA-Net：High frequency attention siamese network for building change detection in VHR remote sensing images》笔记
tags:
  - RS
  - change detection
  - attention
  - High frequency enhancement
categories: 论文
abbrlink: 41400
date: 2022-11-21 09:41:02
---



> 论文地址：[HFA-Net: High frequency attention siamese network for building change detection in VHR remote sensing images](https://www.sciencedirect.com/science/article/abs/pii/S0031320322001984)



## 摘要

### 动机

虽然基于深度学习技术可以很好地处理建筑物变化检测（BCD），但是也存在以下问题：

**在对具有更清晰的边界的对象的分割和识别上仍然受高频信息获取不足的影响，导致在建筑物变化检测中建筑物边界的检测效果并不理想。**



## 工作贡献

- 提出了一种新的基于孪生网络的框架：**HFA-Net**，用于更好地识别VHR遥感图像中的变化建筑物。
- 提出了一个**空间注意力（SA）和高频增强（HF）结合的HFAB模块**，首先通过SA引导模型关注建筑物，然后通过HF来突出建筑物的高频部分即边界。HFAB使模型获得更好的特征表示能力。
- 证实**在深度神经网络中全局高频信息的增强有益于BCD任务**，同时对比之前的基于CNN的方法，该方法能够在BCD任务中达到SOTA效果。



## 网络结构

网络整体结构如下图所示。该方法采用孪生网络+U-Net结合的架构，其中Encoder部分共享权重，在最下面一层进行concat操作。

![image-20221121095256549](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121095256549.png)



### HFAB

作者为了实现高频信息的提取的增强，设计了这个HFAB模块。首先通过空间注意力模块给予特征图中建筑物更多的关注，随后通过高频信息增强模块对建筑物边界进行增强。具体网络结构如下图所示。

![image-20221121095752800](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121095752800.png)



同时根据HFAB的效果示意图可以很好地理解这一过程，其中先进行空间注意力模块再进行高频信息增强是为了在空间注意力模块中过滤掉一些无关信息，避免HF模块过后输出的特征图包含过多的高频噪声。

![image-20221121095952361](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121095952361.png)





#### SA模块

可以在上面流程图中看到，作者采用的空间注意力增强模块时使用了一个U-Net结构作为空间掩码的提取。具体为：上分支部分采用U-Net网络进行处理，而与一般的U-Net不同的是特征通道数一直是降低的，最后输出时通道数为1，便得到了空间掩码$mask(H \times W \times 1)$，然后将它与输入进行相乘实现空间维度的增强

![image-20221121100420741](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121100420741.png)

#### HF模块

作者在提取高频信息的时候采用的是使用Sobel算子。而又因为通常建筑物的形状各异，所以转而使用各向同性的Sobel算子。其中各向同性Sobel算子包含八个方向，于是首先计算每个方向的高频信息，然后进行**求mean**操作。具体操作如下图所示。

![image-20221121100745080](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121100745080.png)



![image-20221121100817024](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121100817024.png)

同时HF模块不仅仅是进行了高频信息的提取，还进行了通道注意力处理。即上分支部分首先进行全局平均池化操作，得到特征图（$1 \times 1 \times C$），随后经过两个全连接层（FCL）得到通道掩码（$1 \times 1 \times C$），最后与输入进行通道级别相乘，得到使用通道注意力后的特征。最后与高频分支（即下分支）进行**concat**，然后经过1×1卷积的处理得到最终的特征。

![image-20221121100439760](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121100439760.png)



### 训练细节

**Loss**：**BCELOSS**

使用Pytorch框架实现了所提出的方法，并在GeForce RTX 3090 GPU和24-GB VRAM上进行了训练。Epoch设置为200，Batchsize设置为8；优化器选用SGD，初始学习率设置为0.01，momentum设置为0.9，weight decay设置为1e-5；学习率在10、15、30、40代时进行衰减，decay rate设置为0.1（MultiStepLR策略）。





## 实验

### 数据集



**WHU-CD**：航拍图像，空间分辨率为0.075m，尺寸32507×15354



**LEVIR-CD**：是一个公共的大规模构建CD数据集。它包含637对大小为1024 × 1024的高分辨率(0.5m) RS图像



**Google Dataset**：大规模多光谱卫星图像变化检测数据集，空间分辨率为0.55m。包含19个季节变化的VHR图像对，大小1006×1168到4936×5224不等



> **在实验中统一裁剪为256×256大小进行处理**





### 对比实验

![image-20221121101459430](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121101459430.png)

![image-20221121101511056](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121101511056.png)



![image-20221121101519444](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121101519444.png)

#### 可视化比较

**WHU-CD**：

![image-20221121101622113](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121101622113.png)



**LEVIR-CD**：

![image-20221121101730948](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121101730948.png)



**Google dataset**：

![image-20221121101754453](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121101754453.png)





### 消融实验

![image-20221121101849808](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121101849808.png)

可以看出单独的HF模块对方法也有不错的提升，证明高频信息提取和增强处理对变化检测任务是有效的。同时作者还提出SA和HF模块的处理顺序对结果也有影响，先进行SA处理有利于过滤掉不属于建筑物的对象，从而减少输出中的高频噪声。



### 边界精度评估实验

由于提出的方法是为了更好地预测建筑物边界，所以做了相应实验验证提出的方法能够更好地预测变化建筑边界，具体结果如下图所示：

![image-20221121101935623](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221121101935623.png)







## 总结

**在BCD任务中，针对高频信息进行增强确实有利于结果中边缘的识别**，同时空间注意力和高频信息增强相结合能够一定程度上抑制高频噪声的产生。



























