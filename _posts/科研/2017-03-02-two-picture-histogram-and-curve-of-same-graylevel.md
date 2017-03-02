---
layout: post
title: 【算法学习】将两幅图像的累积灰度直方图绘制并画出相同灰度级的相同像素点个数曲线
category: 科研
tags: 灰度直方图 直方图曲线 相同灰度级
keywords:  灰度直方图 直方图曲线 相同灰度级
description: 将两幅图像的累积灰度直方图绘制并画出相同灰度级的相同像素点个数曲线
---

##程序：
		%matlab显示原图和灰度直方图以及具有重叠灰度值的数量
		%读取第一幅图像
		I = imread('C:\4.jpg');%图必须是灰度图，或者转换为灰度图I=rgb2gray(I);
		[count,x] = imhist(I,256);  %256个灰度级，count为每个灰度级的像素个数
		count(1)=0;  %将灰度值为零的像素点清零。
		%读取第二幅图像
		I1 = imread('C:\13.jpg');%图必须是灰度图，或者转换为灰度图I=rgb2gray(I);
		[count1,x1] = imhist(I1,256);  %256个灰度级，count为每个灰度级的像素个数
		count1(1)=0;  %将灰度值为零的像素点清零。
		figure;
		subplot(131);
		plot(x,count);     %绘制第一幅图的灰度直方图
		subplot(132); 
		plot(x1,count1);   %绘制第二幅图像的灰度直方图
		for i=1:256
		count2(i)=min(count(i),count1(i));  %获得每灰度级重叠的像素个数
		end
		subplot(133);
		plot(x,count2);

##图像与结果：
![原始图像](http://a1.qpic.cn/psb?/V10ctZwO1IoAMt/9z9TxGrrvhGypolNpkFjzf5vgySPLMpJv8ob*lxd3TQ!/b/dG4BAAAAAAAA&bo=YAUAAgAAAAABAEA!&rf=viewer_4)
![最终结果](http://a2.qpic.cn/psb?/V10ctZwO1IoAMt/MyDrV7MyCUn4GiY0*EsdDncLLzRiqSLQ3ZFikTtsg3w!/b/dCUAAAAAAAAA&bo=PAZpAgAAAAABAHY!&rf=viewer_4)
##涉及知识点
		1.subplot的使用方法，先subplot后绘图。
        2.可以通过imhist函数修改某个灰度级的数值。
