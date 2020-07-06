---
title: MobileNetv3
tags: 2020-7-5
renderNumberedHeading: true
grammar_cjkRuby: true
---


## MobileNetV3 的改进

 - 继续使用了MobileNetV1的深度可分离卷积；
 - 继续使用了MobileNetv2的反向残差Block;
 - 使用了squeeze-excitation机制；
 - 使用了空间结构搜索技术(NAS)和NetAdapt算法用来网络层的微调；
    
*NAS介绍*

 - 使用的空间结构搜索技术是应用于硬件平台上的空间结构搜索技术，首先在论文MnasNet中提出的；
 - 核心思想：将CNN模型分解成预先定义的块，然后在每个块中搜索运算和链接，允许不同的块使用不同的层结构；
 - 操作过程如下图所示：
    ![enter description here](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202075/1593921639292.png)
	通过上图可以看出，每个块中包含的属性以及相应的选择有如下：
	*本论文是通过使用mobileNetv2的网络结构作为子种子，然后再在上面进行搜索生成MobileNetv3的网络结构*
	  - 卷积操作的选择(ConvOp)：常规的卷积操作；深度卷积操作；倒置瓶颈卷积操作；
	  - 卷积核的大小：3x3,5x5
	  - squeeze-and-excitation ratio(SE ratio):0;0.25
	  - 跳跃操作(skip ops)：pooling；residual;no skip；
	  - 输出卷积核通道数：Fi
	  - 块Block的数量：Ni
	 卷积操作、卷积核的大小、SE ratio、跳跃操作核输出通道大小决定了一个层的架构，Ni决定了这个层将为块重复多少次，上图种的block4中的每一层都有一个反向瓶颈5x5卷积一个单位剩余跳跃路径，并且同一层重复N4次。Block的数量决定之后，该优化问题的搜索空间大小也就确定了。
	 

 - 在上述网络模型确定之后，通过强化学习的方式对建立的多目标优化问题进行求解，因为强化学习的方式很方便，奖励reward机制很容易制定。强化学习种的奖励机制如下式：
         ![强化学习的奖励公式](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202076/强化学习的奖励机制公式.png)
	其中，m表示由一系列action a1:T决定的采样模型，R(m)是建立的多目标优化模型中的目标函数值。搜索框架由三个部分组成：一个是基于递归神经网络（RNN）的控制器，一个获取模型精度的训练器核一个基于移动设备的推理引擎，用来测量延迟。对于每个采样模型m，在目标任务上对其进行训练以获得其精度ACCA(m)，并在真实的移动设备上运行它，从而获得其推理延迟LAT(m)。之后，便可以计算收益值R(m)（也就是目标函数）。最后，通过上式定义的最大化收益被更新，如图所示：
	![enter description here](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202076/NAS流程图.png)。
	

**H-Swish的要用**：
   

 - 对于在移动设备上面使用sigmoid函数会产生昂贵的计算消耗；
 - 使用swish 替换掉ReLu，因为ReLu当输入为一个比较大的值时，输出也为一个比较大的值，会造成精度误差。但是ReLu可以在任何软硬件平台上进行计算，量化的时候消除了潜在的精度损失，使用h-swish 替换掉swish，在量化模式下大概提高15%的效率，h-swish在深层网络中更加明显。公式如下：
        ![非线性公式](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202076/swish和h-swish公式.png)

		  