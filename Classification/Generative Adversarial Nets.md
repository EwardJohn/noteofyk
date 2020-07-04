---
title:  Generative Adversarial Nets
tags: 2020-7-4
renderNumberedHeading: true
grammar_cjkRuby: true
---

##  论文简介

 1. 设计一种模型，使得包含两种网络，一种的生成网络，生成和训练样本分布一致的样本；另外一种是判别网络，判断判别器网络输入的数据是属于真实数据还是生成器生成的数据；
2. 假设生成器网络为G，由多层感知器组成，其从随机噪声中随机挑选数据z输入，输出为G(z),G的分布为p(g);
3. 判别器D也是一个多层感知机，输出D(x)代表着判定判别器的输入x属于真实数据而不是生成器的概率。

**本文的核心问题-对抗**

 1. 生成网络G的目标是让生成的数据的分布p(g)和p(data)足够接近，那么判别器网络就无法将来自G的数据判断出来，即D(G(z))足够大；
 2. 判别网络D的目标是能够正确识别来源于真实数据D(x)生成器网络的数据G(z),即需要使D(x)尽可能的大，使D(G(z))尽可能地小，这样对抗就产生了；
 3. 将判别器地目标：D(G(z))尽可能地小转换成问题：1-D(G(z))足够大；
    
**模型的目标问题**


1. 对于生成网络G来说，目标函数为：
          ![生成网络的目标函数](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202074/判别器的目标函数.png)使用梯度下降求出该公式的极小值；

2. 对于判别网络D来说，目标函数为：
         ![判别器网络的目标函数](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202074/生成器的目标函数.png)需要使用随机梯度上升求出其极大值；

3. 这样，模型就成了优化两个目标函数的问题，只需要进行反向传播来对模型进行训练即可优化这两个目标函数，不需要像传统的生成网络那样需要复杂的计算最大似然函数、马尔可夫链或其他推理。
   
4. 整个网络的主要目标是一个最大最小问题，如图所示：
         ![模型优化公式](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202074/模型优化过程图.png) *最小化这个公式就是在训练训练生成器网络，最大化这个公式就是在训练判别器网络*

5. 整个模型的优化过程如图所示：
       ![模型优化过程示意图](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202074/模型优化公式.png)  图中黑线表示真实数据的分布，绿线表示生成器对初始样本的估计，蓝线表示判别器对于样本估计的准确率，z轴指向x轴的线表示生成器G生成样本的分布趋势。（d）中蓝线表示生成器和判别器达到平衡，对应的值应该是0.5，表示生成器和判别器达到平衡，双方不在发生变化。
	   
6. 由于这两个模型是串联的，在训练中没有办法同时训练，采用优化k次D，优化一次G。
  
*论文的tensorflow代码链接*
[GANs代码详解网站](https://blog.aylien.com/introduction-generative-adversarial-networks-code-tensorflow/)