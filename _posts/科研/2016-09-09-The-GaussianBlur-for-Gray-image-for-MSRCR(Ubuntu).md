---
layout: post
title: 【算法学习】将MSRCR中的模糊处理由FFT修改为时域纯高斯模糊
category: 科研
tags: 高斯模糊 灰度图像 Ubuntu MSRCR
keywords: 高斯模糊算法，Ubuntu MSRCR
description: 将MSRCR中的模糊处理由FFT修改为时域纯高斯模糊
---

##实现功能：
将MSRCR中的模糊处理由FFT修改为时域纯高斯模糊
##算法实现：
MSR_original_lib.c中的程序代码：


```
/*
 *
 * Copyright 2013 IPOL Image Processing On Line http://www.ipol.im/
 *
 *
 * This file implements an algorithm possibly linked to the patents:
 *
 *  - US 5991456, "Method of improving a digital image," Issued Nov 23, 1999
 *  - US 6834125, "Method of improving a digital image as a function of its
 *  dynamic range," Issued Dec 21, 2004
 *  - US 6842543 B2, "Method of improving a digital image having white
 *  zones," Issued Jan 11, 2005
 *  - US 8111943, "Smart Image Enhancement Process," Issued Feb 7, 2012
 *  - EP 0901671, "Method of improving a digital image," 
 *  Issued September 3, 2003
 *  - AUS 713076, "Method of improving a digital image," 
 *  Issued February 26, 1998
 *  - WO 1997045809 A1, "Method of improving a digital image," July 4, 2006
 *  - JPO 4036391 B2, "Method of improving a digital image"
 *
 * This file is made available for the exclusive aim of serving as
 * scientific tool to verify the soundness and completeness of the
 * algorithm description. Compilation, execution and redistribution of
 * this file may violate patents rights in certain countries. The
 * situation being different for every country and changing
 * over time, it is your responsibility to determine which patent rights
 * restrictions apply to you before you compile, use, modify, or
 * redistribute this file. A patent lawyer is qualified to make this
 * determination. If and only if they don't conflict with any patent
 * terms, you can benefit from the following license terms attached to this
 * file.
 *
 */

/**
 *  @file MSR_original_lib.c
 *
 *  @brief Libraries using in the MSR_original.cpp
 *
 *
 *  @author Catalina Sbert Juan
 */



#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <fftw3.h>
#include "MSR_original_lib.h"


#ifndef M_PI
#    define M_PI 3.14159265358979323846
#endif
#define PI2  6.283185307179586  /* 2*pi*/
/**
 * @compute the Coefficient of the Gaussian
 * @param winSize the size of Gaussian window
 * @param sigma the param of Gaussian function
 * @return ikern the Coefficient of the Gaussian
 */
int* buildGaussKern(int winSize, float sigma)   /* [1]删除incline[2]修改int sigma为float sigma */
{
    int wincenter, x;
    float   sum = 0.0f;
    wincenter = winSize / 2;
    float *kern = (float*)malloc(winSize*sizeof(float));
    int *ikern = (int*)malloc(winSize*sizeof(int));
    float SQRT_2PI = 2.506628274631f;
    float sigmaMul2PI = 1.0f / (sigma * SQRT_2PI);
    float divSigmaPow2 = 1.0f / (2.0f * sigma * sigma);
    for (x = 0; x < wincenter + 1; x++)
    {
        kern[wincenter - x] = kern[wincenter + x] = exp(-(x * x)* divSigmaPow2) * sigmaMul2PI;
        sum += kern[wincenter - x] + ((x != 0) ? kern[wincenter + x] : 0.0);
    }
    sum = 1.0f / sum;
    for (x = 0; x < winSize; x++)
    {
        kern[x] *= sum;
        ikern[x] = kern[x] * 256.0f;
    }
    free(kern);
    return ikern;
}
/**
 * @The Gaussian blur function of processing the gray image,and the function is simulating the downward function of Convolution
 * @param pixels the input gray image
 * @param pixelsout the output blured gray image
 * @param width  the width of the gray image
 * @param height  the height of the gray image
 * @param sigma  the param of the gaussian function
 * @return pixelsout the output blured gray image
 */
double* GaussBlur1D(double*  pixels,double*  pixelsout, unsigned int  width, unsigned int  height, float sigma)  /*删掉unsigned  int channels,因为是单通道没有用*/
{                                                                                                                             
 /*[1]修改返回值类型void为double类型,函数最后,return pixelsout,这个空间需要开辟和释放*/ 
 /*[2]修改灰度图像数据pixels和模糊后图像pixelsout的数据类型为double类型,因为图像分通道后就是double类型的,后面的tmpBuffer开辟内存需要修改*/ 

    width = 1 * width;  /*3修改为1，因为三个通道变为了1个通道，存储每行数据的宽度变为了原来的1/3.*/
    if ((width % 4) != 0) width += (4 - (width % 4));

    unsigned int  winsize = (1 + (((int)ceil(3 * sigma)) * 2));  /*窗的大小*/
    int *gaussKern = buildGaussKern(winsize, sigma); /*构建高斯核，计算高斯系数*/
    winsize *= 1; /*3改为1，高斯窗的宽度变为原来的1/3*/
    unsigned int  halfsize = winsize / 2;  /*窗的边到中心的距离*/

    double *tmpBuffer = (double*)malloc(width * height* sizeof(double));  /*开辟新的内存存储处理高斯模糊后的数据*/
    int h,w,k;
    for (h = 0; h < height; h++)    /*外层循环，图像的高度*/
    {
        unsigned int  rowWidth = h * width;     /*当前行的宽度为图像的高度乘以每行图像的数据所占的宽度。因为是按行存储的数组。*/

        for (w = 0; w < width; w++) /*w+=channels，可以修改为w++，因为是单通道数据，而不是三通道数据*/
        {
            double rowR = 0.0;  /*存储r分量的数据*/
            int * gaussKernPtr = gaussKern;/*将高斯系数赋值给gaussKernPtr*/
            int whalfsize = w + width - halfsize;
            unsigned int  curPos = rowWidth + w;  /*当前位置*/
            for (k = 1; k < winsize;k++) /* k += channels修改为k++*/
            {
                unsigned int  pos = rowWidth + ((k + whalfsize) % width);
                int fkern = *gaussKernPtr++;
                rowR += (pixels[pos] * fkern);  /*当前像素值乘以高斯系数，rowR这了泛指单通道的当前像素点高斯处理后的数 */
            }

            tmpBuffer[curPos] = rowR/256; /*除以256,((unsigned char)(rowR >> 8))修改为/256*/

        }
    } 
    halfsize = winsize / 2;
    for (w = 0; w < width; w++)
    {
        for (h = 0; h < height; h++)
        {
            double col_all = 0;
            int hhalfsize = h + height - halfsize;
            for (k = 0; k < winsize; k++)
            {
                col_all += tmpBuffer[((k + hhalfsize) % height)* width + w] * gaussKern[k];
            }
            pixelsout[h * width + w] = col_all/256;    /*(unsigned char)(col_all >> 8)修改为/256*/
        }
    }
    free(tmpBuffer);
    free(gaussKern);
    return pixelsout;
}
/**
 * @brief Convolution with a Gaussian kernel using FFT.
 *
 *
 * @param input double array
 * @param scale the size  of the gaussian kernel
 * @param nx x-size of the array
 * @param ny y-size of the array
 *
 * @return output the convolved array
 */

double *convolution(double *input, double scale, double *output,
                    size_t nx, size_t ny)
{
    double *out;
    fftw_plan p;
    int image_size, image_size4;
    int i,j,index;
    double sigma,normx, normy;

    out = (double*) fftw_malloc(sizeof(double) * (nx*ny));

    /*compute the Fourier transform of the input data*/

    p= fftw_plan_r2r_2d((int)ny, (int)nx, input, out, FFTW_REDFT10,
                        FFTW_REDFT10,FFTW_ESTIMATE);
    fftw_execute(p);
    fftw_destroy_plan(p);

    /*define the gaussian constants for the convolution*/

    sigma=scale*scale/2.;
    normx=M_PI/(double)nx;
    normy=M_PI/(double) ny;
    normx*=normx;
    normy*=normy;

    image_size=(int)nx * (int)ny;
    image_size4=4*image_size;

    for(j=0; j<(int)ny; j++)
    {
        index=j*(int)nx;
        for(i=0; i<(int)nx; i++)
            out[i+index]*=exp((double)(-sigma)*(normx*i*i+normy*j*j));
    }

    /*compute the Inverse Fourier transform */

    p=fftw_plan_r2r_2d((int)ny, (int)nx, out, output, FFTW_REDFT01,
                       FFTW_REDFT01, FFTW_ESTIMATE);
    fftw_execute(p);

    for(index=0; index<image_size; index++)
        output[index]/=image_size4;

    fftw_destroy_plan(p);
    fftw_free(out);

    return output;
}

/**
 * @brief The main part of the Multiscale Retinex
 *
 * @f$ MSRout= \sum w (\log(input)-\log(input* G_\sigma))\f$
 *
 * @param input input color channel
 * @param scale[nscales] the  scales for the convolution
 * @param nscales number of scales
 * @param w the weight for each scale
 * @param nx x-size of the image
 * @param ny y-size of the image
 *
 * @return out output of the multiscale retinex
 */

double *MSRetinex(double *out, double *input, double *scale, int nscales,
                  double w, size_t nx, size_t ny)
{
    int i, image_size, n;
    double *pas;

    image_size=(int)nx* (int)ny;

    pas=(double*) malloc(image_size*sizeof(double));

    /* initialization of the output*/
    
    for(i=0; i<image_size; i++)
        out[i]=0.;

    /* Compute Retinex output*/

    for(n=0; n<nscales; n++)
    {
       /*  GaussBlur1D(input,pas,nx,ny,scale[n]/3);*/ 
      convolution(input,scale[n],pas,nx,ny);
        printf("尺度：%f\n",scale[n] );
        for(i=0; i<(int)image_size; i++)
            out[i]+=w*(log(input[i])-log(pas[i]));
    }

    free(pas);

    return out;
}

/**
 * @brief Color restoration for the multiscale Retinex
 *
 * Consists of multiplying the output of the multiscale Retinex by a
 * color restoration function (CRF).
 *
 * @f$ CRF= (\log(125 input)-\log(gray))\f$
 *
 * @f$ MSRCRout=CRF MSRout\f$
 *
 * @param input input color channel
 * @param gray intensity channel of the input color image
 * @param image_size size of the image
 *
 * @return out output of the multiscale retinex with color restoration
 */


double *Color_Restoration(double *out,  double *input, double *gray,
                          size_t image_size)
{

    int i;
    double A;

    for(i=0; i<(int)image_size; i++)
    {
        A=log(3*gray[i]);
        out[i]*=(log(125.*input[i])-A);
    }

    return out;

}

/**
 * @brief Gain/offset
 * is a linear transformation to transform the logarithmic domain into the
 * display domain [0,255]
 *
 * @f$ out=G(input-b) \f$
 *
 * @param input input color channel
 * @param G the gain
 * @param b the offset
 * @param image_size size of the image
 *
 * @return out  output color channel
 */


double *Gain_offset(double *out, double *input, double G, double b,
                    size_t image_size)
{
    int i;

    for(i=0; i<(int)image_size; i++)
    {
        out[i]=G*(input[i]-b);
        if(out[i] < 0.) out[i]=0.;
        if(out[i] > 255.) out[i]=255.;
    }
    return out;
}

```
MSR_original_lib.h中的代码：

```
/*
 *  MSR_original_lib.h
 *  
 *
 *  Created by Catalina Sbert Juan on 12/07/13.
 *
 * Copyright 2013 IPOL Image Processing On Line http://www.ipol.im/
 *
 *
 * This file implements an algorithm possibly linked to the patents:
 *
 *  - US 5991456, "Method of improving a digital image," Issued Nov 23, 1999
 *  - US 6834125, "Method of improving a digital image as a function of its
 *  dynamic range," Issued Dec 21, 2004
 *  - US 6842543 B2, "Method of improving a digital image having white
 *  zones," Issued Jan 11, 2005
 *  - US 8111943, "Smart Image Enhancement Process," Issued Feb 7, 2012
 *  - EP 0901671, "Method of improving a digital image,"
 *  Issued September 3, 2003
 *  - AUS 713076, "Method of improving a digital image,"
 *  Issued February 26, 1998
 *  - WO 1997045809 A1, "Method of improving a digital image," July 4, 2006
 *  - JPO 4036391 B2, "Method of improving a digital image"
 *
 * This file is made available for the exclusive aim of serving as
 * scientific tool to verify the soundness and completeness of the
 * algorithm description. Compilation, execution and redistribution of
 * this file may violate patents rights in certain countries. The
 * situation being different for every country and changing
 * over time, it is your responsibility to determine which patent rights
 * restrictions apply to you before you compile, use, modify, or
 * redistribute this file. A patent lawyer is qualified to make this
 * determination. If and only if they don't conflict with any patent
 * terms, you can benefit from the following license terms attached to this
 * file.
 *
 */

#ifndef _MSR_ORIGINAL_LIB_H
#define _MSR_ORIGINAL_LIB_H

#ifdef __cplusplus
extern "C" {
#endif

int* buildGaussKern(int winSize, float sigma);

double* GaussBlur1D(double*  pixels,double*  pixelsout, unsigned int  width, unsigned int  height, float sigma);
double *convolution(double *input, double scale, double *output, size_t nx, size_t ny);


double *MSRetinex(double *out, double *input,double *scale, int nscales, double w, size_t nx, size_t ny);

double *Color_Restoration(double *out,  double *input, double *gray, size_t dim);

double *Gain_offset(double *out, double *input, double G, double b , size_t dim);

#ifdef __cplusplus
}
#endif

#endif /* _MSR_ORIGINAL_LIB_H*/

```

算法实现效果：三尺度，尺度大小使用默认值

![原始图片](http://a3.qpic.cn/psb?/V10ctZwO1cFN12/DBpUUbAQKb2NNhWoCj9nlx2utm1LuTryb.m9.TnPQMs!/b/dAoBAAAAAAAA&bo=gALgAQAAAAADB0E!&rf=viewer_4)
![MSRCP(卷积)处理后图片](http://a2.qpic.cn/psb?/V10ctZwO1cFN12/rp13711zKMSnQAQSTYtYyZ0EOV11szhiVYRYfMGW9mw!/b/dAkBAAAAAAAA&bo=gALgAQAAAAADB0E!&rf=viewer_4)
![MSRCP(时域纯高斯)处理后图片](http://a3.qpic.cn/psb?/V10ctZwO1cFN12/mhdonSyJH7nLUQDxLL*STDlbscyb.bMY1DZto1tzB3U!/b/dOMAAAAAAAAA&bo=gALgAQAAAAADB0E!&rf=viewer_4)
![MSRCR(卷积)处理后图片](http://a3.qpic.cn/psb?/V10ctZwO1cFN12/luK.XLUhwBTcApHEVXt1ATBueHroeX9jTAFaVeuYXR0!/b/dHABAAAAAAAA&bo=gALgAQAAAAADB0E!&rf=viewer_4)
![MSRCR(时域纯高斯)处理后图片](http://a2.qpic.cn/psb?/V10ctZwO1cFN12/uSyPYB1E2xRBBHq2*nBGKTM9Xr3uXP1zPAmJleWkdfc!/b/dAkBAAAAAAAA&bo=gALgAQAAAAADB0E!&rf=viewer_4)


