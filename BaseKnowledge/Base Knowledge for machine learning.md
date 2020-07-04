---
title: Base Knowledge for machine learning  
tags: 2020-7-4
renderNumberedHeading: true
grammar_cjkRuby: true
---


## 机器学习

**感知器**：

 1. 介绍：感知器主要是用来进行二分类任务的（1或者-1），其结构和神经网络结构比较次相近，包含输入层和输出层，主要用于解决线性可分问题；
 2. 单层感知器由一个线性组合器和一个二至阈值元件组成。
    其结构图入下所示：![单层感知器结构图](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202074/单层感知器结构图.png) 
	可以理解为：将x~0~,x~1~,------x~n~的变量输入，经过组合器的整合，输出为1或者-1，通过组合器对输入变量判断其正确与否，判断的依据就是权重w~0~,w~1~------w~2~,所以以上公式的输入值为：
	![感知器公式](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202074/感知器公式.png)

3. 感知器的目标函数如下：
     ![实现类别判断的函数](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202074/目标函数.png) 
	 
4. 对其结果判断的几种情况：
     ![几种情况](https://raw.githubusercontent.com/EwardJohn/noteofyk/master/img/202074/几种情况.png)，感知机和神经网路相似，但是缺少激活函数和loss函数，但也是通过梯度下降的方法实现超平面的划分，但是容易造成过拟合。