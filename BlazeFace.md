---
title: BlazeFace: Sub-millisecond Neural Face Detection on Mobile GPUs
tags: 2020-6-8
renderNumberedHeading: true
grammar_cjkRuby: true
---


**摘要**
1. 能够在移动设备上面实现200-1000+fps;
2. 应用广
3. 网络使用的是MobileNetV1/V2的变种
4. 根据SSD中的anchor机制进行GPU-friendly修正
5. 为了解决手机拍摄中或者录像中的抖动问题，引入blending nms
  
**引言**
1. 由于手机前后摄像头不同的聚焦长度和抓拍的不同的目标尺寸；
2. BlazeFace产生6个脸部关键点坐标（眼中心，眼眶，嘴中心，鼻子点）这些允许我们进行脸部旋转；
 
 
**结构和设计**
*扩大视野域*：
1. ***深度可分离卷积***，通过对不同通道分别进行卷积操作，如下图：![深度可分离卷积层](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202068/可分离卷积.png)  卷积部分的参数个数为：N_depthwise = 3 × 3 × 3 = 27 DC完成后的Feature map 数量与输入层的通道数相同，无法扩展Feature map，而且这种运算对输入层的每个通道独立进行卷积运算，没有有效利用不同通道在相同空间位置上的feature信息，因此需要Pointwise convolution 来将这些Feature map进行组合生成新的Feature map;
   *在深度可分离卷积上面*参数量为s*s*c*k*k
2. ***逐点卷积***
   该运算与常规卷积运算非常相似，它的卷积核尺寸为1x1xM,M为上一层的通道数，这里的卷积运算会将上一步的map在深度方向上进行加权组合，生成新的Feature map,如下图所示：![逐点卷积](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202068/逐点卷积.png)
   常规卷积计算的参数量为108 = 4x3x3x3
   分离卷积的参数量为39，所以在参数量相同的前提下，采用分离卷积的神经网络层数可以做的更深；
   *逐点卷积的参数*：s*s*c*d 
   那抹这两部分的参数量的比为k^2 : d，然而 k一般为1，d为非常大的数，所以参数量主要由这部分决定
   
2. 在本文中通过将3x3卷积核换成5x5卷积核，卷积核数量的增加换取bottleneck（用来获取感受野）的减少，如图所示：
   ![BottleNeck](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202068/Bottleneck_BlazeFace.png)
     在MobileNet的残差结构中将3x3卷积核换成5x5卷积核形成BlazeBlock，并将两个BlazeBlock叠加起来形成double BlazeBlock；
	 
*特征提取器*
 对于前置摄像头，由于更小范围的目标尺寸和更低的计算需求，特征提取器采取一个128x128像素的RGB输入，并且由5个单BlazeBlocks和6个double BlazeBlocks组成的2D卷积；
   ![满足前置摄像头的结构](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202068/前置摄像头的结构.png) 最高的像素为96，最低的像素为8x8（不同于SSD，它的像素一直下降到1x1）；
   
 *Anchor scheme*
如图所示：
![anchor_scheme](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202068/anchor_scheme.png)
在SSD的anchor机制下，只在特征层维度下采样到8x8像素；我们将88、44和22分辨率的每个像素的2个锚点替换为88分辨率的6个锚点。由于人脸层次率变量的限制，anchors被设置成1：1有利于高精度的实现。

*普通anchor的计算公式*:如图：
![普通anchor的计算公式](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202068/anchor的计算公式.png)

*后处理*
1. 由于特征图的下采样只降到8x8，所以随着目标的尺寸的变化，目标上面的anchor的数量会增加。
2. blending nms :在重叠的预测之间把bounding box的回归参数作为权重平均值；

*结果展示*
![性能展示](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202068/性能展示.png)