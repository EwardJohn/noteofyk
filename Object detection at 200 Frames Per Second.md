---
title: Object detection at 200 Frames Per Second
tags: 2020-6-5
renderNumberedHeading: true
grammar_cjkRuby: true
---


---
 ## 引言
  
1. 使用蒸馏损失函数将精确的teacher网络的知识转移到学生网络
2. 目标尺度蒸馏损失，特征图非最大抑制，单阶段目标检测归一化蒸馏损失函数
3. 只在teacher阶段使用soft lable ,训练的时候使用没有标签的数据
4. 最终参数少了10多倍，帧率达到了200FPS，在PASCAL数据集上面实现了比baseline多14map的检测精度。
5. 我们意识到了网络架构，损失函数，训练数据影响最终模型的大小和速度
6. **网络结构**：决定了检测器的速度和精度，Learning deeply supervised object detectors from scratch.实现了17M的参数，帧率达到17FPS；
*原则*：速度至上，更深的结构但是更浅的网络，实现了在VGG网络中15M的参数，速度达到200FPS
*本文的创新点*：
1. 第一个在单阶段检测器上面使用蒸馏方法；
2. 提出了特征图非极大抑制函数
3. 提出了目标比例蒸馏函数
 *数据的作用*：使用半监督学习，使用大量的未被注释的数据，使用soft-label 方法处理未被注释的数据，使用标注和未标注地数据训练网络；
 4. 提出的训练损失函数允许我们根据目标尺度和权重来控制给teacher标签的权重；该蒸馏损失函数无缝地整合检测损失和蒸馏损失函数


## 结构定制
*yolov2 作为teacher网络，tiny-yolo作为student网络*；

*将前面几层地特征图与最后主要的卷积层混合在一块由于特征尺度不一样，我们使用特征堆叠，大的特征被resize,以使分布能够沿着不同的特征层；当混合特征时，我们密集地使用bottleneck层这样能够在少许层压缩信息。

*Deep but narrow*:在Tiny-Yolo结构中在最后几层由1024个通道地特征图组成，在实验中减少了滤波器的数量，这样可以提高速度；但是这样的深度没法提高精度，所以我们在后面几个主要的卷积层，我们添加了许多1x1卷积层，这样可以加深网络的深度，但没有增加计算复杂度；
![整体框架图](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202066/distill结构图.png)
*Distillation loss 在有标签和没有标签的数据上面使用，FM-NMS在最后teacher网络的最后几层被使用来抑制重叠的标记框
![F-YOLO网络结构图](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202066/检测器的网络架构图.png)
为了使网络结构更简单，我们限制了网络的深度，使特征图变少并且使用了3x3和1x1的滤波器核

*Disstillation loss for training*:
从teacher 网络获得的知识以soft labels的形式被转移
1. 在YOLO架构中，最后层特征图预测N个边框，特征图的数量就被设置成*Nx(K+5),K是类别的数量，5是边框坐标核目标的可能性值*因此在每个cell中，网络学习去预测类别可能性，边框坐标以及scores，整体的损失可以由三部分损失组成，回归损失，目标损失，分类损失
2. 我们要是使用蒸馏的话可以直接将teacher 网络最后一层的输出作为groud_truth，损失将把教师网络的激活传递给学生网络；*这里存在一个问题：由于单阶段检测器的密集采样引入了一些问题使得直接前向运用蒸馏无效，解决办法在下面体现*
  
  *目标按尺度进行蒸馏*
  1. 由于的RCNN族蒸馏算法使用最后一层的预测知识传递给学生，这样学生就会将这些背景中的边框进行学习，最后影响了精度和速度，RCNN通过使用RPN减少这些背景候选框;本文使用的单阶段检测器也存在这样的问题，本文通过把蒸馏损失函数作为目标尺度函数，只有当教师网络的预测值高于某个值的话，才会将边框坐标和类别概率传递给学生网络继续训练；
     ![解释](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202066/网络损失函数.png)
	 ![enter description here](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202066/网络蒸馏损失方程.png)
     
*Feature Map-NMS*:
由于但阶段网络在预测的时候都是在一个cell中的anchor box中预测重叠率最高的那个box，但是单阶段检器中的许多cell和anchor box都不会预测一张图片中的相同目标，因此NMS是有效的，但是这里我们使用了蒸馏损失函数使得表现下降；原因：如下图
![FM-NMS](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202066/FM-NMS.png)

*训练数据的策略*：
1. 当数据有注释时，损失函数由教师的损失函数和gt损失函数组成；
2. 当数据没有标签时，损失函数只由教师的损失函数组成；
3. 所以这种训练方法是由label 和soft-label组成的；
4. 使用DarkNet-19 YOLOV2作为teacher,使用F-yolo作为学生网络；
5. 教师网络还有数据对精度的影响，如图：
   ![enter description here](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202066/label数据的影响.png)
  
  
## 实验
*实验设置*
1. 使用Darknet深度学习框架
2. Tiny-Darknet 是在ImageNet上面训练进行初始化，这里移除网络的最后两层，并添加额外的卷积层；
3. SGD 在前120个epoch学习率为0.001,接下来的20epoch学习率为0.0001，最后的20个epoch的学习率为0.00001
4. λD被设置成1，也就是说检测损失和蒸馏损失作用是相当的，但是蒸馏损失是正比于目标的，蒸馏损失一直就会小于检测损失；

*实验结论*
  1.  *特征融合结论*：融合的高级特征层越多，精度越高，因为初始层包含很多初级特征；精度比较图如下
     ![特征融合精度比较图](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202066/特征融合精度图.png)
  2.  *速度比较*：如图
    ![检测器的速度比较图](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202066/速度比较图.png)
   

