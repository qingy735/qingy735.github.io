---
title: 模型迁移时的动态shape问题
tags:
  - pytorch
  - 动态shape
  - 模型迁移
categories:
  - Summary
abbrlink: 8603
date: 2022-01-04 14:22:29
---

## 模型迁移到npu时处理动态shape问题

### 1、为什么要处理动态shape

>因为当前NPU是在线编译算子，对动态shape场景会反复编译算子从而导致性能较差(*本人测试跑一个step都需要接近一个小时，根本跑不起来*)

### 2、如何处理动态shape

#### a. 提取动态shape

>- 动态shape一般由**nonzero**操作、**tensor[index]** 切片操作引起的
>- 可以提取模型运行时涉及到算子，然后通过分析动态shape算子出现位置再进行解决，本人采用[Pytorch op记录提取](https://gitee.com/wangjiangben_hw/ascend-pytorch-crowdintelligence-doc/blob/master/pytorch-train-guide/%E6%A8%A1%E5%9E%8B%E7%AE%97%E5%AD%90%E6%8F%90%E5%8F%96%E6%8C%87%E5%8D%97.md)

---
>提取完算子就可以进行处理动态shape了，一般的话动态shape出现在**模型输入input、target**和**loss**部分。

#### b. 处理模型输入处动态shape

>一般的话对于目标检测算法，图片输入都是不固定的，模型本身如果需要处理的话也是在forward函数中进行，我们需要将该处理移到**model(input, target)**以前，即在传入模型之前固定输入shape。这里以 *ssdlite320* 为例说明

一般的图片处理需要进行*normalize*和*resize*两部分，处理resize部分就ok了。如果模型中有transform方法处理input那就不需要过多操作，但是如果模型没有固定shape就需要图片**填0处理**  

```python
def fix_input(images):
    '''
    固定input
    '''
    # 固定input image
    batch_shape = (3, 1024, 1024)
    # 填充值
    pad_value = 0

    images_pad = []
    for image in images:
        image = image.to(device)
        padding_size = [0, batch_shape[-1] - image.shape[-1],
                            0, batch_shape[-2] - image.shape[-2]]
        padding_size = [0, batch_shape[-1] - image.shape[-1],
                            0, batch_shape[-2] - image.shape[-2]]
        image = F.pad(image, padding_size, value=pad_value)
        images_pad.append(image)
    images = images_pad
    return images
```

>注意这里填零是拓展图片右边和下边，因为这样不会破坏其他参数中对于图片坐标的记录

如果传参还有targets，则还需要固定ground_box数量。操作和上述类似也是补0  

```python
def fix_target(targets):
    '''
    固定targets
    '''
    # 填充值
    pad_value = 0
    # 固定ground_box数量
    max_boxes = 20
    classes = 91

    targets_pad = []
    for target in targets:
        boxes_num = target['boxes'].shape[0]
        if boxes_num < max_boxes:
            diff_num = max_boxes - boxes_num
            # box对齐
            target['boxes'] = torch.cat([target['boxes'], torch.zeros([diff_num, 4])], dim=0)
            # label对齐
            padding_label = np.zeros(diff_num) + classes
            target['labels'] = torch.cat([target['labels'], torch.from_numpy(padding_label).long()], dim=0)
            # mask对齐
            # padding_mask = torch.zeros(diff_num, target['masks'].shape[1], target['masks'].shape[2])
            padding_mask = target['masks'][0].unsqueeze(0)
            target['masks'] = torch.cat([target['masks'], padding_mask], dim=0)
            # area对齐
            padding_area = torch.zeros(diff_num)
            target['area'] = torch.cat([target['area'], padding_area], dim=0)
            # iscrowd对齐
            padding_iscrowd = torch.zeros(diff_num)
            target['iscrowd'] = torch.cat([target['iscrowd'], padding_iscrowd.long()], dim=0)
        else:
            select_idx = torch.randperm(boxes_num)[:max_boxes]
            target['boxes'] = target['boxes'][select_idx]
            target['labels'] = target['labels'][select_idx]
            target['masks'] = target['masks'][select_idx]
            target['area'] = target['area'][select_idx]
            target['iscrowd'] = target['iscrowd'][select_idx]
        target['boxes'] = target['boxes'].to(device)
        target['labels'] = target['labels'].to(device)
        target['masks'] = target['masks'].to(device)
        target['image_id'] = target['image_id'].to(device)
        target['area'] = target['area'].to(device)
        target['iscrowd'] = target['iscrowd'].to(device)
    return images, targets
```

#### c. 处理loss部分的动态shape

>一般情况下loss部分的动态shape产生于按需取值和赋值等操作，详见[文档](https://gitee.com/wangjiangben_hw/ascend-pytorch-crowdintelligence-doc/blob/master/pytorch-train-guide/固定动态shape范例文档.md#计算类修改固定shape)
>
>主要用的的思想就是保持shape不变，将不需要的部分赋值为0或其他操作，在最后计算时抵消掉前面多余的shape值，需要注意的时不能改变原先语义，记得加上的要reduce



### 3、npu测试

>处理完动态shape后记得去npu上测试，一般的话前几个step还是会很慢，需要跑几个step后才会恢复正常，所以不要看到前面很慢就以为没固定好。



---

> 本文章主要参考 [PyTorch训练调试优化和工具使用指南](https://gitee.com/wangjiangben_hw/ascend-pytorch-crowdintelligence-doc/blob/master/pytorch-train-guide/PyTorch训练调试优化和工具使用指南.md#31-动态shape算子提取和解决思路) 和 [固定动态shape范例文档](https://gitee.com/wangjiangben_hw/ascend-pytorch-crowdintelligence-doc/blob/master/pytorch-train-guide/固定动态shape范例文档.md) 这两篇文章，主要结合自身处理流程进行讲述。如有错误，还望指正！