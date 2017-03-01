---
layout: post
title: 【算法学习】将图片分割出超像素并把每个超像素区域分割出来
category: 科研
tags: 超像素分割 SLIC Dbscan聚类
keywords: 超像素分割，SLIC Dbscan聚类
description: 将一副图片根据其颜色差异（对于灰度图像来说是亮度差异），分割成不同的超像素区域。
---

##实现功能：
将一副图片根据其颜色差异（对于灰度图像来说是亮度差异），分割成不同的超像素区域
##程序实现：

```
im=imread('11.jpg');
[l, Am, C] = slic(im, 3000, 10, 1, 'median');       %slic超像素处理
%figure(1);
%imshow(drawregionboundaries(l, im, [255 0 0]));
lc = spdbscan(l, C, Am, 4);                        %spdbscan()聚类操作    最后一个参数，这里是4，该值越小分割的越细微，更加微小的差别（颜色和亮度）会被区分出来。
%figure(2);
%imshow(drawregionboundaries(lc, im, [255 0 0]))    %将聚类后的区域用红色边界隔开
Blc=lc;        %备份分割好的图像序列
[nL, minLabel, maxLabel] = renumberregions(Blc);
for m=minLabel:maxLabel;    %遍历所有编号的区域
lc=Blc;
%%****************************************************
for i=1:600     %600*800为图像800*600的图像的大小，遍历
for j=1:800
if lc(i,j)==m     %取其值为5的区域，如果是该区域，则置位1，否则置位0，方便和原图像
%的进行对应位置的相乘运算，获取该区域的值
lc(i,j)=1;
else
lc(i,j)=0;
end
end
end
%%*********************************************************
%以上**之间的部分为获取固定编号的区域。
lcint=uint8(lc);      %将lc由double类型转换为uint8类型
img=lcint.*im(:,:,1);  %将取得固定区域的三个通道的数值，其他归为零
img=lcint.*im(:,:,2);
img=lcint.*im(:,:,3);
%figure(3);
%imshow(img);     %显示分割后的图像。
%********以下部分是将该区域截取出来（最小外接矩形）*********
%bw = im2bw(img);
%[r, c]=find(bw==1);  %1为白色
%flag=isempty(r)
%if flag==0
%[rectx, recty, area, perimeter] = minboundrect(c, r, 'a');  %a为面积最小，p为周长最短
%figure(4);
%imshow(img);hold on
%line(rectx,recty);
%**********得到的矩形可能是倾斜的需要进一步校正。下一步就如何将这一部分从图片
%截取出来形成形成单独的图片。
%imcrop函数截取的矩形只能是平行于坐标轴的矩形，建议使用ROI命令。
%由于最终我们不需要考虑形状的问题，这里我们根据所得到的斜的矩形获得平行于坐标
%矩形。
%xmin=min(rectx);
%xmax=max(rectx);
%ymin=min(recty);
%ymax=max(recty);
%imgcrop = imcrop(img, [xmin, ymin, xmax-xmin, ymax-ymin]); %最最左上角的点，和长和高
%figure; imshow(imgcrop );
imgnum=num2str(m);
imwrite(img,['E:\result0\',imgnum,'.jpg']);
end
%**************************************以上为主程序***************************

```
	原图和超像素分割结果
![处理结果](http://a1.qpic.cn/psb?/V10ctZwO1IoAMt/vw7nDYN9WcZiYS4n3x5q2qYCbTawAVQJZvrr8sPREhQ!/b/dG4BAAAAAAAA&bo=YAUAAgAAAAAFB0M!&rf=viewer_4)
		部分超像素分割结果（图像大小与原图像一致）
![部分分割结果](http://a1.qpic.cn/psb?/V10ctZwO1IoAMt/1KUSPfvC6dFzowIOh8Y8mu*aQ0v03**mEwhWQz6oddg!/b/dG4BAAAAAAAA&bo=DhsAAgAAAAAFADQ!&rf=viewer_4)