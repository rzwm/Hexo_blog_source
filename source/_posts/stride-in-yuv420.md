---
title: YUV420中的stride
date: 2018-08-21 21:50:14
tags: [image-processing, YUV, stride]
categories: 图像处理
---

一幅图像除了宽度和高度，有时还有stride，详情参见[Image Stride](https://docs.microsoft.com/zh-cn/windows/desktop/medfound/image-stride)。对于RGB图像，stride很好理解。但是对于YUV图像，由于Y、U、V三个通道的宽度和排列方式不同，所以stride的样子不是很直观。最近我在工作时就遇到了这个问题。经过一番搜索，我在libyuv中找到了答案。下面就分享我找到的YUV420中的stride的答案。

YUV420一般分为四种格式：
```cpp
// 1. I420

   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   U U U U U U U U
   V V V V V V V V

// 2. YV12

   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   V V V V V V V V
   U U U U U U U U

// 3. NV12

   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   U V U V U V U V
   U V U V U V U V

// 4. NV21

   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   Y Y Y Y Y Y Y Y
   V U V U V U V U
   V U V U V U V U
```
可以看到，所有四种格式的Y通道排列是相同的。对于I420和YV12格式而言，它们的U和V通道都是分开排列的，所以这两种格式称为平面格式，即YUV420p（p，即planar）。而对于NV12和NV21格式而言，它们的U和V通道是交错排列的，所以这两种格式称为半平面格式，即YUV420sp（sp，即semi-planar）。

由于平面格式和半平面格式排列的不一致，stride的表现也就不一致。下面我就平面格式和半平面格式各举一例，来说明它们的stride的样子。以“-”表示由于stride大于图像宽度而添加的额外的空白像素。需要注意的是，没有“Y”、"U"、"V"或“-”的地方不代表任何像素。即：
```cpp
Y Y - -
Y Y - -
U -
V -
```
在内存中的实际排列为：
```cpp
Y Y - - Y Y - - U - V -
```

1. I420

   平面格式有：Y stride，U stride和V stride。
```cpp
   // 图像宽度：8
   // 图像高度：4
   // Y stride: 16
   // U stride: 6
   // V stride: 8
   Y Y Y Y Y Y Y Y - - - - - - - -
   Y Y Y Y Y Y Y Y - - - - - - - -
   Y Y Y Y Y Y Y Y - - - - - - - -
   Y Y Y Y Y Y Y Y - - - - - - - -
   U U U U - - U U U U - -
   V V V V - - - - V V V V - - - -
```

2. NV12

   半平面格式有：Y stride，UV stride。
```cpp
   // 图像宽度：8
   // 图像高度：4
   // Y stride: 16
   // UV stride: 12
   Y Y Y Y Y Y Y Y - - - - - - - -
   Y Y Y Y Y Y Y Y - - - - - - - -
   Y Y Y Y Y Y Y Y - - - - - - - -
   Y Y Y Y Y Y Y Y - - - - - - - -
   U V U V U V U V - - - -
   U V U V U V U V - - - -
```