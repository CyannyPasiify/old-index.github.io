---
title: "[review] 3D-UCaps: 3D Capsules Unet for Volumetric Image Segmentation"
description: capsule in enconder layers
---

# [review] 3D-UCaps: 3D Capsules Unet for Volumetric Image Segmentation

本文提出了一种基于3D体素的胶囊网络3D-UCaps，应用于3D医学图像分割任务中。

## 收录

MICCAI 2021

Nguyen T, Hua B S, Le N. 3d-ucaps: 3d capsules unet for volumetric image segmentation[C]//Medical Image Computing and Computer Assisted Intervention–MICCAI 2021: 24th International Conference, Strasbourg, France, September 27–October 1, 2021, Proceedings, Part I 24. Springer International Publishing, 2021: 548-558.

[[source]](https://link.springer.com/chapter/10.1007/978-3-030-87193-2_52)

## 要点

- 提出3D-UCaps：编码路径使用3D胶囊模块，解码路径使用3D CNN模块。
- 为何使用胶囊网络
  -  传统架构中池化层会导致位置信息丢失。
  -  CNN对旋转和仿射变换敏感。
  -  胶囊网络使用动态选路（Dynamic Routing）和步长卷积（Convolutional Strides），可以解决以上问题，更好地保留局部-全局联系。
- 集合优势
  - 既具备胶囊网络保留空域联系的能力
  - 又具备CNN学习视觉表示的能力



## 研究理由

传统架构中使用池化层聚合数据，池化操作会归并特征，导致姿态和对象位置等重要信息丢失。具有连续池化层的CNN无法保留局部对象和整体之间的空间联系。另外，激活层在CNN中扮演重要角色，但它是不可解释的黑盒。CNN对旋转和仿射变换敏感，在MRI扫描中容易因为扫描主体的运动导致其中的某些切片发生位置变换，这些情景对CNN而言属于困难情景，会导致CNN性能下降（CNN对存有位置变换的数据不够鲁棒）。

## 主要内容

### 架构

3D-UCaps改进基于3D UNet。包括3部分：视觉特征提取、胶囊编码器、CNN解码器。

视觉特征提取：使用3个3D 、5x5x5卷积层，后两个为Dilate=3的空洞卷积。

胶囊编码器：使用多组3D 3x3x3 Conv Capsule, Routing 3, Stride 2构成，下采样使用Stride=2的卷积实现，使用短连接将多尺度特征通向解码器。

CNN解码器：使用2x2x2转置卷积上采样，其它为标准3x3x3卷积。

在最后一个胶囊卷积层后使胶囊类型与类目数量一致，从而可以使用边界损失（Margin Loss）进行增强监督。

### 胶囊卷积的编排

在低层级使用更多的胶囊类型，而在高层级使用更少的胶囊类型。因为低层级以表征简单对象为主，而高层级主要表征复杂对象，并且选路算法具有聚类特性。

在解码路径膨胀过程中不使用胶囊卷积，因为实验表明其影响很微弱，使用传统设计可以降低计算成本（不需要在胶囊层间运行选路算法）。

### 训练

将GT下采样后和胶囊编码器输出一起用于计算边界损失（Margin Loss）。

在编码器后使用加权交叉熵损失，并使用一条附加路径重建原始图像，计算Masked MSE Loss。

## 结论

提出3D-UCaps网络，它充分利用3D胶囊卷积对分割特征的建模能力，也充分发挥传统CNN解码器生成分割结果的能力。在医学图像分割领域，探索基于胶囊结构和传统CNN的混合架构是一个很有前景的研究方向，它也有助于将模型复杂度和计算成本保持在合理范围内。

