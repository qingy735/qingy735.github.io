---
title: 《Remote Sensing Image Change Detection With Transformers》笔记
tags:
  - RS
  - change detection
  - attention
  - Transformer
categories: 论文
abbrlink: 14776
date: 2022-10-21 09:08:25
---

> 原文地址：[《Remote Sensing Image Change Detection With Transformers》](https://ieeexplore.ieee.org/document/9491802)



## 摘要

### 问题

对于场景中具有复杂目标的情况，高分辨率遥感变化检测仍具有挑战性。具有相同语义概念的物体在不同时间和空间位置上可能表现出截然不同的光谱特征。最近大多数纯CNN模型仍然很难将时空中的长距离概念联系起来，而Nonlocal self-attention方法虽然可以通过建模像素间稠密关系来实现，但是计算效率很低。



### 工作

> 提出双时相图像Transformer（BiT）来高效地建模时空域上下文



**核心观点**：感兴趣的高层次概念可以用几个视觉单词来表示，即**语义token**

为了实现这一观点，将双时相图像表示为几个语义token，并使用Transformer解码器在紧凑时空中建模上下文。然后将学习到的丰富的上下文token反馈给像素空间，通过Transformer解码器对元是特惠总能进行优化。BiT相比于纯卷积的baseline模型，计算时间和参数量都大大减少。



## 引言

**【当前面临的挑战】**

1. 场景中存在复杂的目标
2. 不同的成像状态

**【模型需要的能力】**

1. 识别场景中感兴趣变化的高层语义信息
2. 区分真实变化和伪变化



### 当前任务不足

**【纯卷积】**

存在内在的限制，即感受野的大小。通常通过堆叠卷积层或者使用空洞卷积、引入注意力机制来解决。

**【注意力机制】**

例如通道注意力、空间注意力、self-attention，在对全局的建模中是有效的。但是比较难将时空中的的长范围概念进行联系，主要因为他们要么是将注意力单独应用于每个时态的图像用来增强特征，要么是简单地重加权通道或空间维度融合的双时相特征或者图像中。对于self-attention来说，虽然可以对时空中每一对像素的语义联系进行建模，但是计算量很大。



### 具体工作

1. 使用CNN（ResNet）提取输入图像中的高级语义特征，并且使用空间注意力将每个图像的特征转换为一组紧凑的语义token
2. 使用Transformer编码器对两组语义token进行上下文建模
3. 通过Transformer解码器将富含丰富上下文信息的token映射到像素空间
4. 计算两个特征的差异图并且输入到浅层的CNN中进行像素级的变化预测





## 网络结构

整体模型结构由三部分组成：CNN特征提取、Transformer、CNN预测，流程图如下所示

![image-20221021130108341](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021130108341.png)

![image-20221021130300735](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021130300735.png)

### Semantic Tokenizer

为了获得紧凑的token，标记器学习一组空间注意力图，将特征图空间池化为一组特征（个人认为可以理解为特征压缩类似，将深度特征在空间维度进行压缩，可以看做是空间注意力的一个变种）

![image-20221021130701509](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021130701509.png)



![image-20221021130738390](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021130738390.png)

由上面公式可以看出最后输出的语义token维度被压缩为了$L \times C $ ，由后面的实验可以看出$L$的取值不宜过大。



### Transformer Encoder



本文中采取了和ViT类似的方法，将Norm层放在了MSA/MLP前面执行，PreNorm已经被证明有更稳定更有能力。而对于输入的两组token（双时相图像）首先进行concat，然后添加位置编码后输入Encoder部分。

![image-20221021131722563](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021131722563.png)

### Transformer Decoder

作者将最初的提取的深层特征作为query，Encoder输出部分作为key和value，同时Decoder部分为一个孪生网络，分别作用于两种图像，输入部分从Encode输出一分为二得到



### CNN Backbone

使用ResNet18提取双时相图像特征图，同时对最后一层做出了相应修改（详细见原论文）



### Prediction Head

使用一个非常浅的FCN对差异图进行变化预测，即首先通过相减得到两个特征的差异图，之后输入到浅层卷积模块得到通道数为2的变化检测图（分别表示变化or未变化）

![image-20221021132745003](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021132745003.png)



### Loss Function

使用简单的交叉熵损失进行模型优化

![image-20221021132847841](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021132847841.png)





## 实验分析

一下为用来比较的模型的详细说明：

1. **Base**: our baseline model that consists of the CNN backbone (ResNet18_S5) and the prediction head.
2. **BIT**: our BIT-based model with a light backbone (ResNet18_S4).
3. **Base_S4**: a light CNN backbone (ResNet18_S4) + the prediction head.
4. **Base_S3**: a much light CNN backbone (ResNet18_S3) + the prediction head.
5. **BIT_S3**: our BIT-based model with a much light backbone (ResNet18_S3).



在基于**LEVIR-CD**、**WHU-CD**、**DSIFN-CD**数据集上的相关实验如图所示，可以看出BiT在三个数据集上都达到了最好的水平，尤其是在DSIFN-CD数据集上的F1超出其他最好的算法近五个点。同时在Base部分也同样具有很不错的效果，这可能归功于对全局特征的高度抽象，增强了网络的学习能力。

![image-20221021133246859](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021133246859.png)

![image-20221021133649615](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021133649615.png)

![image-20221021133700422](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021133700422.png)

![image-20221021133706244](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021133706244.png)





### 模型参数和计算量

![image-20221021133731563](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021133731563.png)

可以看出BiT在计算量和参数上面也更占优势，在保持高准确度的同时还保证了参数和计算量很小。

同时可以从下面两张图看出来相比于不使用Transformer而言，BIT模型具有更好的稳定性和泛化能力

![image-20221021133913831](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021133913831.png)

![image-20221021133942528](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021133942528.png)



### 消融实验



将深度语义特征提炼为密度更高的语义token能够更好地提升模型性能，使用高度提炼的语义token作为Transformer输入既能减少复杂度又可以减少深度特征中的信息冗余，使得有用的语义信息更加紧凑

![image-20221021134103555](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021134103555.png)

其他消融实验：

![image-20221021134233689](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021134233689.png)

### 语义token可视化

![image-20221021135718537](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021135718537.png)



![image-20221021135745032](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221021135745032.png)



## 总结

本文只使用了比较简单的结构就实现了比较好的效果，其中有几个点值得以后着重思考和实验：

1. 语义token，即压缩凝练深层特征使网络学习能力增强
2. Transformer，Transformer确实可以提高网络的鲁棒性等，并且能够大大增强网络的学习能力
3. BiT还有很多值得改进的方法，当然也可以直接将他看做是一个Baseline，例如：损失函数（正负样本不均衡问题）、特征提取部分（采用特征提取能力更强的网络）等









