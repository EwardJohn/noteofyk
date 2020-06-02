---
title: Like What You Like:Knowledge Distill via Neuron Selectivity Transfer
tags: 2020-6-1
renderNumberedHeading: true
grammar_cjkRuby: true
---


## 摘要
### teacher-student model
#### teacher model 通常比较大，速度也慢，那么通过学习一个student model 将teacher上面有用地特征学习到，其余地知识删除掉，那么在不影响精度地情况下，速度也能得到保证

### MMD
#### 设计了一个知识迁移(KT)损失函数，将分布之间的最大平均损失(Maximum Mean Discrepancy)降到最小
#### 将新设计的损失函数和原来的损失函数联合起来能够提高学生网络的表现；
#### 联合其他的KT方法探索更好的结果，尝试了目标检测算法证明该方法的有效性

## 引言

### 模型压缩的方法

#### 网络修剪--通过修剪低重要性的神经元和权重（需要特别的硬件和操作）
##### 论文都有（1）Optimal brain damage.（2）Learning both weights and connections for efficient neural network.（3）Pruning convolutional neural networks for resource efficient inference.（4）Channel pruning for accelerating very deep neural networks.（5）A filter level pruning method for deep neural network compression.（6）Pruning filters for efficient ConvNets.

#### 网络量化--减少权重和特征的准确性（需要特别的硬件和操作）

##### 论文都有：（1）Binarized neural networks: Training deep neural networks with weights and activations constrained to +1 or -1. （2）ImageNet classification using binary convolutional neural networks.

#### 知识迁移---直接训练一个student model

##### 论文都有：（1）Model compression.（2）Distilling the knowledge in a neural network.（3）Hints for thin deep nets.（4）Paying more attention to attention:Improving the performance of convolutional neural networks via attention transfer.（5）Accelerating convolutional neural networks with dominant convolutional kernel and knowledge pre-regression.（6）Face model compression by distilling knowledge from neurons.

##### 本论文的思想：（1）如果一个神经元被某些样本和区域激活，那么就说明这些区域和样本可能拥有一些与任务相关的特性；（2）本文提出teacher 和 student 模型的神经元选择模式的分布匹配，如图：![enter description here](./images/teacher-student_1.png)（3）student 网络不仅从gt学习相应的知识，而且模仿teacher 网络中介层中的激活分布

#### 知识蒸馏（KD）

##### （1）论文：Distilling the knowledge in a neural network.（2）从一个大的teacher model 中通过teacher model 提供的类别分布根据softmax 蒸馏出一个student model （3）知识就是teacher 网络的软化输出，提供类间和类内相似性的监督，softened label 能够将一个类别中的样本投影到一个连续的空间，这样类别内和类别间中的相似性就能够得到计算；![enter description here](./images/KD_softened-softmax.png) student network 是通过联合Softened softmax 和 一般的softmax 进行训练的（4）缺点：只能用于softmax函数并且严重依赖于类别数量。

## 背景知识

#### 条件定义：

##### （1）T------teacher network; S---student network ;假设CNN中一层输出特征图为F <![enter description here](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202061/特征图.png) C 为特征图的通道数，HxW是空间维度，假设teacher 和student 特征层空间维度相同，若是特征层的维度不同，进行插值使得维度相同。


#### MMD（最大平均插值）

1. 可以考虑成基于数据采样的可能性分布距离矩阵；
2. 假设我们从分布p和q采样得到X和Y，然后在p和q之间的平方MMD距离可以被计算成![enter description here](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202061/MMD.png)
3. 公式可以化简成如下：![enter description here](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202061/MMD2.png)
4. 当特征空间对应于全局RKHS（核希尔伯特空间），那么最小化MMD相当于最小化p和q之间的距离；
 
 
 ----
 #### 神经元选择性迁移
 #####  **动机**:
 1. 左图显示神经元对猴的脸比较敏感，右图显示神经元被强的特征激活---显示出被同一个神经元激活的区域具有与任务相关的相似性，在学生网络中也应该模仿这些激活模式
 2. 在教师网络中定义一种新知识：神经元选择或者叫相关激活，然后将其迁移到学生网络中；
    
	
 ##### **不能直接匹配特征图**：
 
 1. 把空间中每个位置的激活都考虑成一个特征图，*每个卷积核的单一激活特征图是神经元选择维度空间的样本*,这个样本分布反映CNN如何解释一张输入图片；
 2. 直接匹配样本不很好，因为 忽略了空间中的样本密度；


#### 方程
1. **公式**：![Selectivity Transfer loss](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202062/迁移损失函数.png)
   