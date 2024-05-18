---
title: 《DarkVisionNet：Low-Light Imaging via RGB-NIR Fusion with Deep Inconsistency Prior》笔记
tags:
  - cv
  - image fusion
  - RGB&NIR
categories: 论文
mathjax: true
abbrlink: 48007
date: 2022-03-03 17:01:17
---



## Abstract

由于低光图像中的高强度噪声放大了RGB和NIR图像之间结构不一致的影响，使得现有的算法失效。为了解决这一问题，本文提出了一种新的RGB-NIR融合算法 Dark Vision Net（DVN）。该算法的两个技术新颖点是Deep Structure 和 Deep Inconsistency Prior (DIP)两部分。同时设计了新的由对齐的RGB-NIR图像对组成的数据集Dark Vision Dataset（DVD），也是第一个公开的RGB-NIR融合基准。本文结果显示DVN在PSNR和SSIM指标方面明显优于其他算法，特别是在极低光条件下。



## Introduction

高质量的低光图像是一个具有挑战但是很有意义的任务。一方面他是很多重要任务的基础（24小时监控、手机拍摄等）；另一方面在极低环境下图像中的大量噪声会阻碍算法进行图像恢复。RGB-NIR图像融合技术要做的是：通过相应的近红外图像中的丰富的详细的信息来增强低光环境中具有大量噪声的RGB图像，极大地提高融合后RGB图像的信噪比（SNR）。

![image-20220303222024522](https://gitee.com/qingy735/blogimg/raw/master/img/202203032220790.png)

但是目前存在的RGB-NIR融合算法存在着RGB和NIR图像结构不一致的问题，导致了生成了不自然的图像和关键信息的缺失，这些限制了RGN-NIR融合算法在低光环境下的应用（因为在低光环境下RGB图像会因为大量的噪声极大地加剧和NIR图像的结构不一致性）。

本文主要通过解决低光环境RGB-NIR图像结构不一致问题来改进具有极低信噪比的RGB-NIR图像融合问题。作者认为通过引入深度特征的先验信息可以很好地解决上述问题。下方是整个算法的结构图：

![image-20220303223232068](https://gitee.com/qingy735/blogimg/raw/master/img/202203032232173.png)

其中有两个主要新颖技术点：

- DSEM

  > 即使面对低信噪比，该结构也能很有效地提取和表示可靠的结构信息，对后续的先验信息很重要。

- DIP



## Related Work

### Image Denoising

虽然现在有很多图像去噪的算法，但是在极低光照环境下，被高强度噪声破坏的精细纹理和很难再恢复，与此同时去噪算法会倾向于输出过于平滑的图像。



### RGB-NIR Fusion

现有算法大部分有以下两个问题：

- 处理结构不一致图像时能力不足，导致结果出现严重的伪影。
- 噪声抑制能力不足，特别是在低光环境下。



### DataSets

由于获取RGB-NIR图像对比较困难，所以目前只有少量数据数据可以用于RGB-NIR融合研究。为了解决这一问题，作者团队收集了一个名为Dark Vision Datase（DVD）的数据集，作为第一个公开的RGB-NIR融合基准。



## Approach

### Prior Knowledge of Structure Inconsistency

本文首先从各个特征通道提取出二值边缘图，然后将不一致性用函数 F 来定义。

![image-202203041348775](https://gitee.com/qingy735/blogimg/raw/master/img/202203041348775.png)

其中C和N代表RGB和NIR图像的R/G/B通道，$edge^{C}$和$edge^{N}$分别代表C和N的二值边缘映射。当 F 等于0时则代表$edge^{C}$和$edge^{N}$具有严重的不一致性，相反，在RGB和NIR结构一致的区域 F 为1。利用 F 的输出可以直接通过与NIR相乘来抑制不一致的结构。



### Extraction of Deep Structures

虽然函数 F 能够很好的描述RGB-NIR之间的不一致性，但是却不能直接应用于低光环境下。当面对大量噪声的RGB图像时，计算出的不一致映射只包含非信息噪声。所以，为了避免噪声在结构不一致提取上的影响，本文提出了深度结构提取模块 Deep Structure Extraction Module (DSEM)和深度不一致先验模块Deep Inconsistency Prior (DIP)来计算特征空间上的结构不一致性。

![image-20220304140042125](https://gitee.com/qingy735/blogimg/raw/master/img/202203041400254.png)

### Supervision of DSEM

DSEM从网络R中提取多尺度特征$feat_{i}$（i代表尺度），并且输出多尺度深度结构映射$struct_{i}$。为了使DSEM预测出高质量的深度结构映射，本文引入了一个监督信号$struct^{gt}_{i,c}$，损失函数表示为：$\mathcal{L}_{stru} = \sum_{i=1}^{3}\sum_{c=1}^{Ch_{i}}Dist(struct_{i,c},struct^{gt}_{i,c})$，其中$Ch_{i}$是第i层深度结构的通道数，Dist是Dice Loss，$struct_{i,c}$是第i层第c个通道的预测深度结构，$struct^{gt}_{i,c}$代表相应的ground-truth。其中Dice Loss表示为$Dist(P,G) = (\sum_{j}^{N}p_{j}^{2}+\sum_{j}^{N}g_{j}^{2})/(2\sum_{j}^{N}p_{j}g_{j})$，$p_{j}$和$g_{j}$代表在P和G上的第j个像素。

---

本文根据Deep Image Prior这篇论文的思想建立了监督信号。$struct_{i,c}^{gt}$从而一个预训练的网络AutoEncoder中获得，其中AE网络和R相比只是少了skip connection结构。其中$struct_{i,c}^{gt}$计算公式为：



![image-20220304150109062](https://gitee.com/qingy735/blogimg/raw/master/img/202203041501153.png)

其中$dec_{i,c}$表示多尺度特征，$\bigtriangledown$代表Sobel算子，m代表均值。

![image-20220304150022031](https://gitee.com/qingy735/blogimg/raw/master/img/202203041500142.png)



### Calculation of DIP and Image Fusion

通过DSEM提取出的深度结构包含丰富的结构信息同时对噪声具有鲁棒性，根据RGB和NIR图像产生的struct可以引入上述的结构不一致函数$\mathcal{F}$来计算高质量的结构不一致先验：$M_{i,c}^{DIP} = \mathcal{F}(struct_{i,c}^{C},struct_{i,c}^{N})$，其中C和N分别代表RGB和NIR图像。由于$M_{i,c}^{DIP}$代表的是结构不一致而不是像素强度不一致，所以可以直接用$struct_{i,c}^{N}$代替$feat_{i,c}^{N}$，其中转换公式为：$\hat{struct}_{i,c}^{N} = M_{i,c}^{N} \cdot struct_{i,c}^{N}$，在DIP的引导下$\hat{struct}_{i,c}^{N}$直接舍弃了与RGB图像不一致的结构，从而解决了结构不一致性。为了进一步将NIR中的结构细节融入到RGB中，本文设计了一个多尺度融合模块，详见上图(c)。



### Loss Function

该模型的总损失函数为：$\mathcal{L} = \mathcal{L}_{rec}^{C} + \mathcal{L}_{rec}^{\hat{C}} + \mathcal{L}_{rec}^{N} + \lambda_{1} \cdot \mathcal{L}_{stru}^{C} + \lambda_{2} \cdot \mathcal{L}_{stru}^{N}$ 其中$\mathcal{L}_{stru}^{C}$ 和$\mathcal{L}_{stru}^{N}$是深度结构预测损失函数$\lambda_{1}$和$\lambda_{2}$为对应的系数，分别设置为1/1000和1/3000，$\mathcal{L}_{rec}^{C}$、$\mathcal{L}_{rec}^{\hat{C}}$、$\mathcal{L}_{rec}^{N}$分别表示fused-RGB、coarse-RGB、NIR图像的重构损失函数。其中$\mathcal{L} = sqrt{(\|X - X_{gt}\|^{2} + \varepsilon^{2})}$，$\varepsilon$一般被设置为$10^{-3}$。



## Experiment

>  训练图片随机裁剪为`128*128`，learning rate从2e-4逐渐减少为1e-6

### Performance Comparison

在DVD数据集上PSNR和SSIM指标的比较结果：

![image-20220304211734225](https://gitee.com/qingy735/blogimg/raw/master/img/202203042117440.png)

同时DVN与同领域的其他算法相比还具有更好的去噪、细节恢复和伪影抑制能力。

![image-20220304211939388](https://gitee.com/qingy735/blogimg/raw/master/img/202203042119550.png)

本文还进行了真实低光环境下的性能测试，同样取得了很好的效果

![image-20220304212220048](https://gitee.com/qingy735/blogimg/raw/master/img/202203042124175.png)



### Effectiveness of DIP

本文中作者还验证了DIP这一结构的有效性，为此作者重新训练模型（去除DIP模块）来进行比较。

![image-20220304212741907](https://gitee.com/qingy735/blogimg/raw/master/img/202203042127093.png)

可以看出DIP可以抑制图像伪影和增强细节特征，能够更好地处理结构不一致性问题。


## Code

***注：根据作者`Bingbing Yu`提供的运行demo进行研究。***

---



- 提取深度特征

  作者首先是先通过R网络（个人理解为编码器与解码器结构，代码中用到了RCAB模块）输出RGB和NIR的三层特征结构，然后再计算NIR各层特征的梯度图`nir_structure`。RGB图像的步骤相似，首先也是提取编码器和解码器特征然后再输出梯度图`rgb_structure`。

- DIP模块应用

  根据上述得到的`rgb_structure`与`nir_structure`可以算出$M^{DIP}$

  ```python
  def f(x, y):
      return 1 / 2 * (1 - x) * (1 - y) + x * y
  ```

  ```python
  masks = []
  for i in range(len(rgb_structure)):
  	masks.append(f(rgb_structure[i], nir_structure[i]))
  ```

- 特征融合

  重复上述编码器和解码器步骤，不过在编码时在对应尺度加上相应的NIR特征$\hat{struct}$（抑制了结构不一致性）

  ```python
  # 编码部分
  enc1 = self.encoder_level1(x)
  enc1_fuse_nir = enc1 + self.atten_conv1(mask[0] * encoder_outs[0])
  x = self.down12(enc1_fuse_nir)
  enc2 = self.encoder_level2(x)
  enc2_fuse_nir = enc2 + self.atten_conv2(mask[1] * encoder_outs[1])
  x = self.down23(enc2_fuse_nir)
  enc3_fuse_nir = enc3 + self.atten_conv3(mask[2] * encoder_outs[2])
  ```

## Summary

### 算法基本运行流程

- 分别提取RGB和NIR图像的深度特征$feat^{C}$和$feat^{N}$
- 将$feat^{C}$作为DSEM的输入，在监督信号的指导下训练DSEM，其中监督信号由AutoEncoder输出（因为RGB和NIR第一步结构一样，所以选取一个进行训练DSEM就行）
- 而后将训练好的DSEM的输出$struct^{C}$ 和$struct^{N}$进行DIP模块处理，即生成抑制结构不一致的$\hat{struct}^{N}$作为第三步融合模块的输入
- 第一步获取到的DSEM的输出$struct^{C}$作为输入传入最后的融合模块，$\hat{struct}^{N}$在相应层与RGB进行相加融合，最后通过解码器输出融合图像

### 大致思想

- 首先想的是用NIR图像的丰富细节结构与RGB图像融合，来补充低光环境下RGB图像包含大量噪声的影响
- 融合的话采取的是结构融合，又因为低光RGB包含大量噪声，所以得先进行去噪处理，于是作者提出了DSEM模块用来处理噪声。
- 处理完噪声后RGB和NIR图像又存在结构不一致这个问题，融合后会导致伪影等问题，于是作者又提出了DIP模块，用来生成抑制结构不一致性的NIR结构图
- 后续就直接用生成的RGB结构图与NIR结构图进行融合



