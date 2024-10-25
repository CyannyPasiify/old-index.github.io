---
title: [Review] Gabor Convolutional Networks
description: a scale rotation robust conv variance (GoF)
---

<!--KaTeX-->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.css" integrity="sha384-AfEj0r4/OFrOo5t7NnNe46zW/tFgW6x/bCJG8FqQCEo3+Aro6EYUG4+cU+KJWu/X" crossorigin="anonymous">
  <script defer src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.js" integrity="sha384-g7c+Jr9ZivxKLnZTDUhnkOnsh30B4H0rpLUpJ4jAIKs4fnJI+sEnkvrMWph2EDg4" crossorigin="anonymous"></script>
  <script defer src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/contrib/auto-render.min.js" integrity="sha384-mll67QQFJfxn0IYznZYonOWZ644AWYC+Pt2cHqMaRhXVrursRwvLnLaebdGIlYNa" crossorigin="anonymous"></script>
  <script>
      document.addEventListener("DOMContentLoaded", function() {
          renderMathInElement(document.body, {
              // ...options...
          });
      });
  </script>

# [Review] Gabor Convolutional Networks

本文结合Gabor导向滤波器思想将其与DCNN结合，构建了Conv算子与Gabor滤波器结合的Gabor卷积方向滤波器（Convolutional Gabor Orientation Filter，GoF）参数化算子，GoF能够增强模型对方向和尺度的健壮性。与早期工作只在浅层使用不可学习的Gabor滤波器提取手工特征不同，本文首创在模型参数级别将Gabor滤波器与Conv算子耦合的方案。以GoF构建Gabor Convolutional Networks（GCN）模型，在ResNet基座中替换Conv算子后，在许多任务上展现出优于标准DCNN的性能。

**提出了**一种基于标准Conv算子并根据Gabor导向滤波器思想改进的GoF算子，此算子能够对多个方向和多个尺度信息进行建模，将此算子嵌入DCNN或替换Conv算子即可显著增强模型对方向和尺度的健壮性，进一步提升模型在某些对空间变换建模有较高需求的任务上的性能，例如大目标检测和基于多方向纹理的匹配或分类。

## 收录

IEEE Transactions on Image Processing 2018

Luan S, Chen C, Zhang B, et al. Gabor convolutional networks[J]. IEEE Transactions on Image Processing, 2018, 27(9): 4357-4366.

[[source]](https://ieeexplore.ieee.org/document/8357578)

## 贡献点

1. 首次提出将Gabor滤波器与Conv算子结合的方法思路，以提升DCNN模型对图像平移、尺度和旋转变换的健壮性。
2. 构建了GoF算子，此算子便于嵌入各种网络，并且在许多任务的性能指标上达到SOTA。
3. 提出针对GoF算子优化的梯度反向传播更新方法。

## 研究问题

DCNN对旋转、缩放等空间变换的健壮性不足（DCNN在应对不同方向尺度对象时容易失效），在以往解决方案中，增加DCNN对空间变换建模能力要么依赖于数据增强，要么采用大模型或定制模块，尚缺乏一种通用且能够良好建模空间变换的模型内部方法。

## 思想方法

### 相关工作

Gabor小波是一组用于傅里叶变换的复数域基函数，其重要性质是这组小波标准差的乘积在时域和空域都是最小的。

[TI-Pooling](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Laptev_TI-Pooling_Transformation-Invariant_Pooling_CVPR_2016_paper.pdf)设计了一个并行网络结构用于表示变换集，并在顶层之前应用变换不变性池化算子。

空间变换网络（Spatial Transformer Networks，STN）。使用一个局部子CNN对变换矩阵进行估计来修正特征图。

方向响应网络（Oriented Response Network，ORN），主动旋转滤波器（Actively Rotating Filter，ARF）

可变形卷积网络（Deformable Convolutional Network）。使用可变形卷积和可变形RoI池化算子来提升CNN对变换的建模能力，使得网络对几何变换变得健壮，但这些可变形滤波器更倾向于以小规格滤波器的形式进行应用。

Gabor Filters + CNN杂交。一些方法使用Gabor滤波器提取手工特征然后送给CNN继续处理，在这些方法中，Gabor滤波器提取特征被当成一个离线预处理过程。另一些方法将Gabor滤波器融入网络，将第一个卷积层替换为Gabor滤波器以提取特征。

### 从Conv参数调制生成GoF算子

对一个$ C_{out}\times C_{in}\times N\times W \times W$的参数化Conv算子，给定一组包含$U$个方向的Gabor滤波器，对每一个Conv算子$C_{in}\times N\times W \times W$，采用空域逐元素乘方式调制Conv算子为$U$个新算子，算子总计具有$ C_{out}\times C_{in}\times U\times N\times W \times W$个参数（论文中约定$N=U$），构成$C_{out}$个GoF组。对于$ C_{in}\times N\times H \times W$的具有$C_{in}$个通道$N$维特征图输入，同一组的$C_{in}$个$U$个方向的$N\times W\times W$卷积核，将每个方向的输出特征图求和聚合，经GoF组卷积后产生$ C_{out}\times N\left(=U\right)\times H \times W$规格的特征图输出。

![Modulation process of GoFs](Assets/Modulation%20process%20of%20GoFs.png)

图中为一个$N=4,W=3$的Conv算子，调制到$U=4$个方向。

### GoF优化更新

GoF的可学习参数仅限于其内含的卷积核参数$C_{i,o}$，它通过多方向尺度的Gabor调制生成GoF，所以实际可学习参数更少。梯度计算只需在每个通道内将各方向GoF子滤波器的梯度按Gabor滤波器权重加和。

$$
\delta=\frac{\partial L}{\partial C_{i,o}} = \sum_{u=1}^{U}{\frac{\partial L}{\partial C_{i,o}}\circ G\left(u,v\right)}
$$

$$
C_{i,o}=C_{i,o}-\eta\delta
$$

式中，$L$表示损失函数，$G\left(u,v\right)$表示$u$方向$v$尺度的Gabor模板。

### 实验

论文使用MNIST、随机旋转后的MNIST、SVHN(The Street View House Numbers)、CIFAT-100、Food-101数据集研究了关于Gabor模板的方向U、尺度V数量的选择，以及GCN网络各阶段的核宽度和层数设置。

**Gabor模板的方向U、尺度V数量的选择**。随U数值增长，错误率（Error Rate）呈现先下降后回升的趋势。这说明U设置过小会导致提取特征信息不足，而过大又会导致模型复杂难以有效训练。因此需要交互式地选取适中的U值，论文推荐值为U=4,5。论文使用V=1和V=4进行了对比实验，V=1表示每个阶段均使用尺度V=1的Gabor模板，V=4表示每个阶段依次使用V=1,2,3,4的Gabor模板；V=4时模型错误率有略微下降。

**GCN网络各阶段的核宽度和层数设置**。相关实验表明，相较于CNN、STN等其它方法，GCN每阶段只需设置较少的核数即可达到较低的错误率，具有良好的性能伸缩率。

此外，将Gabor模板替换为Gaussian模板依然能够取得相较于相当参数量ResNet的提升，这证明了将导向滤波器嵌入Conv算子的有效性。

## 讨论

### 调制与正则化

GoF的本质是将标准卷积调制到各个方向尺度上去，这是一个正则化过程，可引导卷积层学习更一般的特征（方向性尺度性被削弱），而不是强化某一个方向尺度的特征，从而避免过拟合。

### 多尺度和尺度感知

使用不同尺度V的Gabor模板相当于提取多尺度特征，但依旧不能称之为尺度感知的，因为GoF并没有利用也不知晓图像的物理距离，所以想要利用上医学图像提供的间距信息依然需要其它方法。
