---
layout: post
title: 【算法学习】基于物理(形状)的图像分割算法(RAD分割算法)
category: 科研
tags: RAD图像分割 物理分割 图像分割
keywords:  RAD图像分割 物理分割 图像分割
description: 基于物理(形状)的图像分割算法(RAD分割算法)
---

##主页翻译：
		基于物理的图像分割方法(RAD分割方法)
		文中所提到的方法是基于这样的观察（对于一个单一的物理反射率可以具有不同的图像值。）我们把这些值都叫做主颜色，这些值之所以不同主要是因为阴影，遮挡和高光（这些主要是因为几何特征造成的）。有一种图像分割的类型是根据这些形状的检测，也就是基于物理的分割方法，这个颜色反射模型是1985年提出来的，跟这个模型最相关了。然而这个模型需要许多的假设，其结果就是不能在理论框架中找到这种结构，同时也不适用于现实的图像。在我们的方法中，我们认为这种结构形成了很好连接的山脊在颜色直方图中。
		为了获得这些结构，我们提出一种新的基于脊分布分析（RAD）方法，来找到占主导地位的颜色。为了找到他们，首先，使用MLSEC-ST技术，这种技术不受山脊中小的违规和间隙的影响，然后我们使用简单的山脊提取算法，最后，一个洪水过程被提出来寻找直方图中的主颜色。
		定性结果显示我们的方法能够获得很好的效果对于存在阴影和高亮边缘的情况，定量的分析结果从伯克利的数据集中显示，我们的方法优于目前最先进的轮廓提取方法，并具有较低的计算成本。

##流程与结果：
算法流程
![算法流程](http://a2.qpic.cn/psb?/V10ctZwO1IoAMt/ifSJypCQri6NwC135FYl1Cn2WTsNR6Dx98dgj.QjCUg!/b/dBYAAAAAAAAA&bo=rwiAAgAAAAADAAA!&rf=viewer_4)
	算法效果
	通过伯克利数据集和基准测试获得的和目前最先进的算法的比较。
![处理效果](http://a1.qpic.cn/psb?/V10ctZwO1IoAMt/0OQFS*51nTtE6jXITgQS*3R6QpQUTH*d7lIka3Dr*0U!/b/dG4BAAAAAAAA&bo=VQWAAgAAAAAFAPE!&rf=viewer_4)
##文献：
		Image Segmentation in the Presence of Shadows and Highlights Using a Ridge based Histogram Analysis. E Vazquez, J. van de Weijer and R. Baldrich. Proc. ECCV08, Marseille, France ( Oral presentation, Slides) .
##代码：
	    Matlab code. For both Windows and Linux. 