---
layout: post
title: 【算法学习】均值模糊（续）
category: 科研
tags: 均值模糊 单通道
keywords: 均值模糊，float double
description: 均值模糊
---

##实现功能：
单通道图像均值模糊（float,double类型图像）
##算法实现：

```
#include "stdafx.h"
#include "malloc.h"

/// <summary>
/// 实现功能：	用周边的平均来代替当前值,浮点版本
/// <param name="Src">需要均值模糊的数据在内存的地址。</param>
/// <param name="Dest">保存均值模糊后的结果。</param>
/// <param name="Width">数据的宽度。</param>
/// <param name="Height">数据的高度。</param>
/// <param name="Radius">用于计算平均值的矩形半径。</param>

void  BoxBlur(float* Src, float *Dest, int Width, int Height, int Radius)
{
	int X, Y, Z, MoveIn, MovOut  ;
	float Sum, InvertAmount = 1.0 / (( 2 * Radius + 1) * (2 * Radius + 1));
	float *Temp = (float *)malloc(Height * Width * sizeof(float));
	float *LinePS, *LinePD;

	//	BoxBlur是一种行列可以分离计算的过程，因此先进行行方向的BoxBlur，
	//	得到中间结果，然后再对中间结果进行列方向的BoxBlur，则可以得到最终结果。
	
	for (Y = 0; Y < Height; Y++)			//	对每行的数据进行BoxBlur，注意这个过程是行方向无关的，即每行的结算结果不会相互影响，因此和适合于并行化
	{
		LinePS = Src + Y * Width;			//	定位到第Y行第一个像素的内存地址
		LinePD = Temp + Y * Width;
		Sum = (Radius + 1) * LinePS[0];		//	计算行第一个像素左右两边 Radius范围内的累加值，超过边缘的用边缘值代替, 注意这里的加1是为了后续的编码方便，请仔细体味这个+1，很有相反的	
		for (Z = 0; Z < Radius; Z++)		// 	按理说这里的循环内部是要进行判断Z是否大于等于宽度，否则会出现访问非法内存的错误，但是这种情况只在Radius>Width时出现，这个是没有意义的。
			Sum += LinePS[Z];				//	计算第一个点右侧的像素累加值
		for (X = 0; X < Width;X++)			
		{	
			MovOut = X - Radius - 1;						//	从左侧移出去的那个像素的坐标
			if (MovOut < 0) MovOut = 0;						//	用边缘值代替
			MoveIn = X + Radius;							//	从右侧移入的那个像素的坐标
			if(MoveIn >= Width) MoveIn = Width - 1;			//	用边缘值代替
			Sum = Sum - LinePS[MovOut] + LinePS[MoveIn];	//	新的累加值 = 旧的累加值 - 移出的值 + 移入的值
			LinePD[X] = Sum;								//	保存到临时内存中
		}
	}

	for (X = 0; X < Width; X++)				//	接下来对临时数据进行列方向的BoxBlur，得到最终结果，编码方式其实一样的，只是列更改为行
	{
		LinePS = Temp + X;					//	定位到第X列的第一个像素的内存地址
		LinePD = Dest + X;
		Sum = (Radius + 1) * LinePS[0];		//	以下是同样的道理，无需注释了
		for (Z = 0; Z < Radius; Z++)
			Sum += LinePS[Z * Width];
		for (Y = 0; Y < Height; Y++)
		{	
			MovOut = Y - Radius - 1;
			if (MovOut < 0) MovOut = 0;
			MoveIn = Y + Radius;
			if(MoveIn >= Height) MoveIn = Height - 1;
			Sum = Sum - LinePS[MovOut * Width] + LinePS[MoveIn * Width];			//	注意这里的*Width了
			LinePD[Y * Width] = Sum * InvertAmount;					//	求平均值，浮点的乘法要比除法快多了，这样为什么不用乘法呢。
		}
	}
	free(Temp);			//	注意释放内存
}
}


int main(int argc, char* argv[])
{
	const char*imagename="E:\\InputName.bmp";  //此处需要根据要显示图片文件的实际文件位置更改
	//从文件中读入图像
	Mat img=imread(imagename);
	Mat img1=imread(imagename);
	Mat Src,Dest;
	cvtColor(img,Src,CV_BGR2GRAY);
	cvtColor(img1,Dest,CV_BGR2GRAY);
	BoxBlur(Src.data,Dest.data,Src.cols,Src.rows,1,1,1); 
	//BoxBlur(img.data,img1.data,img.cols,img.rows,3,3,1); 
	imshow("均值模糊",img1);
	waitKey();
	while(1);
	return 0;
}


```
