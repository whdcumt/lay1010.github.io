---
layout: post
title: 【算法学习】均值模糊模拟高斯模糊半径大小的确定
category: 科研
tags: 均值模糊 高斯模糊 半径
keywords: 均值模糊，高斯模糊
description: 均值模糊模拟高斯模糊半径大小的确定
---

##实现功能：
均值模糊模拟高斯模糊半径大小的确定
##算法实现：

```
// boxesForGauss.cpp : 定义控制台应用程序的入口点。
/// <summary>  
/// 实现用均值模糊模拟高斯模糊时，根据需要均值模糊的次数，计算所需要的均值模糊的半径依次大小.  
/// 参考博文：http://blog.ivank.net/fastest-gaussian-blur.html  
///modify:whd qq:710433980  最新整理时间 2016.9.26。
#include "stdafx.h"
#include <cmath>
//函数功能：四舍五入函数。
/// <param name="r"> 要四舍五入的数值</param>  
/// <remarks> 返回值为四舍五入后的结果</remarks>  
double round(double r)  
{  
    return (r > 0.0) ? floor(r + 0.5) : ceil(r - 0.5);  
}
//函数功能：计算均值模糊模拟高斯模糊的各半径的值。
/// <param name="sigma"> 高斯模糊参数sigma</param>  
/// <param name="n"> 需要用到均值模糊的数量，默认值：3</param>  
/// <remarks> 返回值为依次进行均值模糊的半径</remarks>  
int *boxesForGauss(float sigma, int n)  // standard deviation, number of boxes
{
	 float wIdeal = sqrt((12*sigma*sigma/n)+1);  // Ideal averaging filter width 
     int wl = floor(wIdeal);  if(wl%2==0) wl--;
     int wu = wl+2;		
     float mIdeal = (12*sigma*sigma - n*wl*wl - 4*n*wl - 3*n)/(-4*wl - 4);
     int m = round(mIdeal);

     int *sizes=new int[3];  
	 for(int i=0; i<n; i++)
	 {
		 sizes[i]=i<m?wl:wu;
	 }
     return sizes;
}
int _tmain(int argc, _TCHAR* argv[])
{  
	double sigma=1;
	int *sizes=boxesForGauss(sigma,3);
	printf("高斯模糊的参数sigma=%f\n",sigma);
	printf("三次均值模糊的半径分别为：%d,%d,%d\n",sizes[0],sizes[1],sizes[2]);
	return 0;
}
```

![处理结果](http://a1.qpic.cn/psb?/V10ctZwO4GqAC4/Q73TR2FTq7t4Vnuyj2OMFgCavfMCD81PVkYliQe3d2A!/b/dHwBAAAAAAAA&ek=1&kp=1&pt=0&bo=kAYEBJAGBAQDCSw!&sce=60-2-2&rf=viewer_4)