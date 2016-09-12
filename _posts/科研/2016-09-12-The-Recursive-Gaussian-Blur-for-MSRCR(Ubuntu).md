---
layout: post
title: 【算法学习】递归高斯模糊
category: 科研
tags: 高斯模糊 递归高斯 Ubuntu MSRCR
keywords: 高斯模糊算法，Ubuntu MSRCR
description: 将MSRCR中的模糊采用递归高斯
---

##实现功能：
递归高斯模糊
##算法实现：

```
 * @Recurisive   Gaussian Blur
 * @param src    the original image
 * @param dst    the blurred image
 * @param width  the width of image
 * @param height the height of image
 * @param sigma  the Coefficient of the Gaussian
 * @param chan   the number of channels
 * @return dst   the output blured gray image
 */
double * IMG_GaussBlur(double* src, double * dst, int width, int height, float sigma, int chan) /*(1)unsigned char*修改为double(2)unsigned char*&修改为double*/
{
	int    i         = 0;
	int    row       = 0;
	int    col       = 0;
	int    pos       = 0;
	int    channel   = 0;
	int    n         = 0;
	int    bufsize   = 0;        
	int size         = 0;
	int rowstride    = 0;
	int itemp0       = 0;
	int itemp1       = 0;    
	float temp       = 0;
	int    channelsize = width*height;

	if (width>height)
	{
		bufsize = width;
	}
	else
	{
		bufsize = height;
	}    

	double* w1    = (double *) malloc (bufsize * sizeof (double));     /*float修改为double*/
	double *w2    = (double *) malloc (bufsize * sizeof (double));     /*float修改为double*/
	double *in    = (double *) malloc (channelsize * sizeof (double));  /*float修改为double*/
	double *out   = (double *) malloc (channelsize * sizeof (double));  /*float修改为double*/

	/****************************计算高斯核***************************/
	double  q= 0;  /*float修改为double*/
	double  q2, q3; /*float修改为double*/  
	double b0;
	double b1;
	double b2;
	double b3;
	double B    = 0;
	int    N    = 3;

	if (sigma >= 2.5)
	{
		q = 0.98711 * sigma - 0.96330;
	}
	else if ((sigma >= 0.5) && (sigma < 2.5))
	{
		q = 3.97156 - 4.14554 * (double) sqrt ((double) 1 - 0.26891 * sigma);
	}
	else
	{
		q = 0.1147705018520355224609375;
	}

	q2 = q * q;
	q3 = q * q2;
	b0 = (1.57825+(2.44413*q)+(1.4281 *q2)+(0.422205*q3));
	b1 = (        (2.44413*q)+(2.85619*q2)+(1.26661 *q3));
	b2 = (                   -((1.4281*q2)+(1.26661 *q3)));
	b3 = (                                 (0.422205*q3));
	B = 1.0-((b1+b2+b3)/b0);

	/*加速方法 减少循环多次/b0*/
	b1 /= b0;
	b2 /= b0;
	b3 /= b0;
	/*计算高斯核结束*/

	/* 处理图像的多个通道*/
	for (channel = 0; channel < chan; channel++)
	{
		/*获取一个通道的所有像素值*/
		for (i = 0, pos = channel; i < channelsize ; i++, pos += chan)
		{
			/* 0-255 => 1-256 */
			in[i] = (double)(src[pos] + 1.0);
		}

		/*纵向处理*/
		for (row=0 ;row < height; row++)
		{
			pos =  row * width;            
			size        = width;
			rowstride    = 1;    
			bufsize        = size;
			size        -= 1;

			temp =  (in + pos)[0]; 
			w1[0] = temp;  /*！！！计算需要相同初值（否则噪声），但是赋值的时候不能用相同的（否则像素重复或缺失）！！！*/
			w1[1] =temp;
			w1[2] =temp;



			for (  n=3; n <= size ; n++)
			{
				w1[n] = (double)(B*(in + pos)[n*rowstride] +    ((b1*w1[n-1] +     b2*w1[n-2] + b3*w1[n-3] )));

			}
			w1[0] =  (in + pos)[0];  /*左边产生噪声（使用不同初始值时产生），右边有3像素黑色不变（out最后3个像素未设定）*/
			w1[1] =(in + pos)[1];
			w1[2] =  (in + pos)[2];

			

			(out + pos)[size]= w2[size]= (double)w1[size];   /*float修改为double*/
			(out + pos)[size-1]=w2[size-1]=(double)(1.0*w1[size-1]) ; /*float修改为double*/
			(out + pos)[size-2]=w2[size-2]= (double)((1.0)*w1[size-2]); /*float修改为double*/
			w2[size]= (double)w1[size-2]; /*float修改为double*/
			w2[size-1]= (double)w1[size-2]; /*float修改为double*/
			w2[size-2]= (double)w1[size-2]; /*float修改为double*/
			for (n = size-3; n>= 0; n--) 
			{
				(out + pos)[n * rowstride] = w2[n] = (double)(B*w1[n] +    ((b1*w2[n+1] +    b2*w2[n+2] + b3*w2[n+3] )));  /*float修改为double*/

			}    

		}    


		/*横向处理*/
		for (col=0; col < width; col++)  /*wbp 在纵向处理的基础上继续横向处理*/
		{                
			size        = height;
			rowstride    = width;    
			bufsize        = size;
			size        -= 1;

			temp  = (out + col)[0*rowstride];  /* wbp 第col列的第一个数据，复制3份，开始前向滤波*/
			w1[0] = temp;
			w1[1] = temp;
			w1[2] = temp;
			for ( n=3; n <= size ; n++)
			{
				w1[n] = (double)(B*(out + col)[n*rowstride] + ((b1*w1[n-1] +    b2*w1[n-2] + b3*w1[n-3] ))); /*float修改为double*/

			}
			w1[0] =  (out + col)[0];
			w1[1] =  (out + col)[rowstride];
			w1[2] =  (out + col)[2*rowstride];


			temp        = w1[size];
			w2[size]    = temp;
			w2[size-1]    = temp;
			w2[size-2]    = temp;
			(in + col)[size * rowstride]=w1[size];
			(in + col)[(size-1) * rowstride]=w1[size-1];
            (in + col)[(size-2) * rowstride]=w1[size-2];

			for (n = size-3; n >= 0; n--)
			{
				(in + col)[n * rowstride] =w2[n]= (double)(B*w1[n] +    ((b1*w2[n+1] +     b2*w2[n+2] + b3*w2[n+3] )));  /*float修改为double*/

			}                
		}
		/*修正偏移的拷贝方法, 但是修正后图像右边及下边会丢失数据？？？// wbp 并且图像左边及上边多出数据*/
                int x,y;
		for(y=0; y<height; y++)
		{
			itemp0 = y*width*chan;
			itemp1 = (y)*width;                                /*+3  数据没丢失，但是是拷贝而不是原数据*/
			for (x=0; x<width; x++)
			{            
				dst[itemp0+x*chan + channel]=in[itemp1+x]-1;    
			}
		}         
	}

	free (w1);
	free (w2);
	free (in);
	free (out);
    return dst;
}

```

算法实现效果：三尺度，尺度大小使用默认值

![原始图片](http://a3.qpic.cn/psb?/V10ctZwO1cFN12/DBpUUbAQKb2NNhWoCj9nlx2utm1LuTryb.m9.TnPQMs!/b/dAoBAAAAAAAA&bo=gALgAQAAAAADAEY!&rf=viewer_4)
![MSRCP(递归模糊)处理后图片](http://a2.qpic.cn/psb?/V10ctZwO1cFN12/JPZuBfSulUgzhI9siSzXJ13TsyOUkPuFWPIv3gcA.RA!/b/dAkBAAAAAAAA&bo=gALgAQAAAAADAEY!&rf=viewer_4)
![MSRCR(递归模糊)处理后图片](http://a2.qpic.cn/psb?/V10ctZwO1cFN12/EwLSUdHQHrSW3i1fMWZ87Qa8Wmr.XNNJonfS8KgY87A!/b/dAkBAAAAAAAA&ek=1&kp=1&pt=0&bo=gALgAQAAAAADB0E!&sce=60-3-3&rf=viewer_4)



