---
title: pytorch训练模型时出现文件类型报错
tags:
  - bug
categories: Pytorch
abbrlink: 91b0e005
date: 2021-12-30 22:16:58
---
## RuntimeError: received 0 items of ancdata

### 错误原因

>在dataloader加载数据时出现的错误，原因是pytorch多线程共享tensor是通过打开文件的方式实现的，而打开文件的数量是有限制的

### 解决方法

1. 增加open files的限制数量 `ulimit -SHn 51200`
2. 修改多线程的tensor方式为file_system（默认方式为file_descriptor，受限于open files数量）`torch.multiprocessing.set_sharing_strategy('file_system')`

>**注**：如果方法二还报错`unable to open shared memory object </torch_3394728_xxxxxxx> in read-write mode`则参考以下链接
https://github.com/pytorch/fairseq/issues/1560
>https://github.com/pytorch/pytorch/issues/2706
