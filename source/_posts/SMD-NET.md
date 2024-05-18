---
title: 《SMD-Net：Siamese Multi-Scale Difference-Enhancement Network for Change
  Detection in Remote Sensing》笔记
tags:
  - RS
  - change detection
  - high-resolution images
categories: 论文
date: 2022-09-26 19:37:26
abbrlink: 41292
---



> 原文地址：[SMD-Net: Siamese Multi-Scale Difference-Enhancement Network for Change Detection in Remote Sensing](https://www.mdpi.com/2072-4292/14/7/1580)



## 摘要

### 问题：

由于双时像图像的环境差异和复杂的成像条件，在变化检测结果中通常会存在一些问题：**丢失小物体**、**物体不完整**、**边缘粗糙**等。现有的变化检测方法通常对这方面缺少关注


### 工作：

提出孪生变化检测方法**SMD-NET**用于双时相遥感图像

1. 使用多尺度差异图逐步增强变化区域的信息来获得更好的变化检测结果

2. 提出用于高层次特征的孪生残差多核池化模块**SRMP**提升模型中的high-level变化信息

3. 对于低级特征提出特征差异模块**FDM**，该模块使用特征差异来完全提取变化信息并帮助模型生成更准确的细节



## 引言

### 模型设计动机：

1. **孪生网络**：单分支结构存在双时相图像空间特征纠缠和不对应问题，在特征融合过程中会影响模型性能。而孪生网络分别从双时相图像中提取特征，然后融合他们生成变化检测结果
2. **多尺度特征差异图**：由于位置偏移、光照条件、季节变化等因素，很难检测像素较弱的变化区域，同时孪生网络缺少对相对脆弱变化区域信息的关注和连续的下采样和卷积造成这些变化区域的丢失，此外当前的RS变化检测模型并没用充分的利用特征差异
3. **孪生残差多核池化模块（SRMP）**：low-level特征包含细粒度位置信息，如：坐标和图像纹理；high-level特征包含粗粒度语义信息，如：物体范围、土地属性、语义等。SRMP可以提升对象的完整性和基于高级语义信息的小物体检测能力
4. **特征差异模块（FDM）**：提供多尺度详细的变化信息来增强变化区域局部细节，FDM能够提升模型在物体边缘上的性能



### 先前工作不足：

没有充分利用差异图**difference map**或者只是使用差异图而没有保留原始的特征图。这将使变化信息没有被完全挖掘导致丢失小物体和变化检测结果不完整



## 研究方法

### 网络结构

**Siamese U-shaped network**：Siamese encoders、a feature difference map processing module、a decoder

> 特征差异图处理模块由一个**SRMP**和四个**FDM**模块构成

为了提高模型的性能，Encoder第一个部分采用在Imagenet上预训练的ResNet-34，并且取除了最后的池化层和全连接层，之后的四个encoder块由几个残差块构成（加速收敛、防止梯度消失）。Decoder部分采用卷积-转置卷积-卷积结构构成





![image-20220926210608417](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220926210608417.png)





### SRMP模块

基于[The Residual Multi-kernel Pooling module (RMP)](https://ieeexplore.ieee.org/document/8662594/)改进，能够进一步提取上下文信息并且扩大感受野。

具体操作流程如下：

1. 计算深层特征图的绝对差异值来得到特征差异图D
2. 使用四个不同stride的最大池化层进行下采样来获得具有不同细节层次的多个特征图（2x2、3x3、4x4、5x5）
3. 由于不同感受野的融合同时会产生不同细节的高层次变化信息，所以采用1x1卷积和上采样将四个单通道特征图恢复到原特征差异图大小即D，同时采用最近邻近差值防止信息丢失
4. 将原始的两个特征图和上采样的特征图进行concat操作作为SRMP的输出



![image-20220927093014912](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220927093014912.png)





### FDM模块



使用FDM从低层次差异图中提取变化信息增强变化区域的特征细节，能够使模型生成更准确的边缘。并且由于特征差异图不仅包含变化信息而且包含伪变化信息（光照、季节等原因导致），FDM在提取变化信息时可以去除在低层次特征差异图中的一些伪变化。

对于每个浅层特征，使用FDM提取特征并且生成多尺度和多深度的特征图，随后这些高分辨率特征图和深层变化信息一起送入Decoder中。在融合几阶段，该模型使用高级特征指导这些高分辨率变化特征图，以至于能够进一步优化变化区域的边缘并且最终生成更好的变化检测结果。



具体操作流程如下：

1. 根据生成的特征图计算差异图D
2. 将D输入进Conv-BN-ReLU模块以减少维度（1/8）
3. 降维后的差异图送入一个残差模块获得浅层变化特征图
4. 将得到的浅层变化特征图和最初的两个特征图进行concat作为FDM的输出





![image-20220927095340306](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220927095340306.png)



### 损失函数设计



一般的，在像素级别的任务中使用Dice Loss或者交叉熵损失函数。但由于在遥感图像变化检测中通常正样本会远小于负样本，即正负样本数量是不平衡的。此时如果直接使用上述损失函数计算会导致准确率很高，但是会导致大量正样本被误判并且召回率recall会很低。为了解决正负样本不平衡问题，论文中采用BCE Loss和Tversky Loss进行训练。
$$
L_{loss} = L_T + L_{BCE}
$$
其中$L_T$为Tversky loss：
$$
L_T(A,B)=\frac{(A \cap B)}{((A \cap B) + \alpha |A - B| + \beta |B - A|)}
$$
$L_{BCE}$为二元交叉熵损失：
$$
L_{BCE} = -(B \times lnA + (1 - B) \times ln(1 - A))
$$

> 其中A为模型的预测结果，B为实际结果；|A-B|为false positive（FP），|B-A|为false negative（FN）；$\alpha$和$\beta$用来控制FP和FN的权重





## 实验结果

### 数据集选取：

采用三种数据集：CDD、BCDD、OSCD



### 比较方法和评估指标

#### 比较方法：

- FC-EF
- FC-Siam-Conc
- FC-Siam-Diff
- DASNet（VGG16、ResNet50）
- SNUNet-CD
- RDP-Net



#### 评估指标

- Precision（P）

  $$
  P = \frac {TP} {TP + FP}
  $$



- Recall（R）
  $$
  R = \frac {TP} {TP + FN}
  $$
  
- F1-score
  $$
  F1 = 2 \times \frac {P \times R}{P + R}
  $$
  
- Overall Accuracy（OA）
  $$
  OA = \frac {TP + TN}{TP + FN + FP + TN}
  $$
  
- Intersection-over-Union（IoU）
  $$
  IoU = \frac {TP}{TP + FP + FN}
  $$
  
- Kappa
  $$
  Kappa = \frac {OA - PRE}{1 - PRE}
  $$
  

> 其中PRE为：
> $$
> PRE = \frac{(TP+FP)(TP+FN)+(FN+TN)(FP+TN)}{(TP+FP+TN+FN)^2}
> $$



### 消融实验

#### SRMP & FDM

> 对比了特征差异图进行最大池化后的不同插值结果，最近插值和双线性插值

可以从图中看出经过最大池化后由于增加了感受野，大物体更加完整、细长物体更加连续（橙色圈出部分）。并且当采用双线性插值进行上采样时会模糊大物体的边缘（绿色圈出部分），同时还会对小物体变化检测造成严重损害（红色圈出部分）。从图7.c中可以看出虽然更粗糙但是却保留了相当高的变化像素强度（个人理解为区分度），以便后续对浅层变化特征进行指导

![image-20220928100947577](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928100947577.png)

从表中可以看出SRMP模块能够从深层特征差异图中提取有效的深层变化信息并且能够更好地解决物体多尺度问题和物体完整性问题。同时最近插值能够保护小物体和物体边缘的变化像素强度。

对于FDM模块从表中可以看出对比baseline，添加FDM后的各个指标均有增加，说明FDM模块是具有作用的



![image-20220928102333044](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928102333044.png)



### 对比实验

#### CDD

![image-20220928122118095](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928122118095.png)

![image-20220928122825113](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928122825113.png)



#### BCDD

![image-20220928124109221](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928124109221.png)

![image-20220928124129688](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928124129688.png)



#### OSCD

![image-20220928124402230](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928124402230.png)

![image-20220928124421832](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928124421832.png)



#### 参数总数和运行速度

![image-20220928124757222](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928124757222.png)

![image-20220928124813971](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220928124813971.png)



当然原论文做了更多更详细的实验分析，感兴趣可以阅读原论文！



### 结论

采用孪生网络、SRMP模块和FDM模块为解码器提供多尺度和多深度信息，以增强变化强度和保留原始特征。该网络模型能在物体完整性、小物体检测和物体边缘检测中取得很好的性能。并且在和其他模型在以上三个数据集中比较是同样取得了很好的效果，在模型体量和运行速度上有一个很好的权衡。然而该模型还有一些不足的地方，如模型的推理速度需要进一步缩短等。









