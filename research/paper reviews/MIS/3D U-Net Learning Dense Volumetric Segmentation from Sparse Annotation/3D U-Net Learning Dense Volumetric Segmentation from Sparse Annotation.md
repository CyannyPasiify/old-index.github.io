---
title: "[review] 3D U-Net: Learning Dense Volumetric Segmentation from Sparse Annotation"
description: 3D U-Net, the milestone
---

# [review] 3D U-Net: Learning Dense Volumetric Segmentation from Sparse Annotation

本文提出了一种从稀疏标注的立体数据中学习三维分割的网络。

## 收录

MICCAI 2016

Çiçek Ö, Abdulkadir A, Lienkamp S S, et al. 3D U-Net: learning dense volumetric segmentation from sparse annotation[C]//Medical Image Computing and Computer-Assisted Intervention–MICCAI 2016: 19th International Conference, Athens, Greece, October 17-21, 2016, Proceedings, Part II 19. Springer International Publishing, 2016: 424-432.

[[source]](https://link.springer.com/chapter/10.1007/978-3-319-46723-8_49)

## 要点

- 提出3D U-Net：从稀疏标注的体素图像中学习的立体分割网络。
- 2种应用
  -  Semi-Automated Segmentation：可以对只进行了稀疏标注的数据集进行密集标注，细化标注的结果。
  -  Fully-Automated Segmentation：对未标注的数据进行预测，生成密集分割结果。


## 研究理由

以往slice-by-slice的标注方式是冗余并且低效的，因为相邻切片显示的信息几乎相同，而且逐切片学习出来的2D模型预测效果很差，没有考虑到空间上的互信息。本文提出一种从一部分2D切片标注生成3D密集分割结果的思路，并给出了两种应用方法。

## 主要内容

### 架构

3D UNet改进于先前的UNet结构，只是输入改为3D样本，并将所有运算符替换为3D运算符，如3D卷积、3D最大池化和3D上采样。本文中尽量避免了瓶颈操作，并使用批归一化加速收敛。

有一条编码路径和一条解码路径，每一条都有4个分辨率级别。

编码路径每一层包含两个3×3×3卷积，每层都后接BN+ReLU，然后是一个2×2×2的每个方向上步长都为2的最大池化层。

在解码路径，每一层包含一个步长为2的2×2×2的反卷积层，紧跟两个3×3×3的卷积层，每层都后接BN+ReLU。

通过shortcut，将编码路径中相同分辨率的层传递到解码路径，为其提供原始的高分辨率特征。

最后一层为1×1×1的卷积层，可以减少输出的通道数，最后的输出通道数为标签的类别数量。

网络的输入为3通道的132×132×116的像素集合。输出的大小为44×44×28。

### 使用部分标注

一个重要的部分是加权softmax损失函数，使得网络可以使用稀疏标注的数据进行训练。

为前景分割标记和背景分配不同权重，将未标记像素的权重设置为零，使得网络可以从有标记的像素中均衡地学习，并推广到整个立体数据。

## 结论

3D U-Net诞生于医学影像分割，针对肾脏的生物医学影像的分割达到了IoU（交并比）=86.3%的结果，很大程度上解决了3D图像逐个切片送入模型进行训练的繁琐过程，大幅度提升了训练效率，保留了FCN和UNet具备的优秀特点。

## 讨论

### 为什么3D-UNet要避免瓶颈，而残差网络等要鼓励使用瓶颈？

因为ResNet的bottleneck是指使用1x1卷积，主要是为了通过降通道数量，来降卷积的参数和计算量，这中间会有信息损失，但影响不大，因为毕竟是负责残差的计算；但是在分割任务中利用池化层获取来多尺度信息，池化操作本身就会损失许多信息，所以反而要在最大池化之前将通道数翻倍，来避免瓶颈。

