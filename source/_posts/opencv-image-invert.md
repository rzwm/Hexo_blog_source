---
title: OpenCV图像取反
date: 2018-06-07 21:24:10
tags: [OpenCV]
categories: 图像处理
---

最近在做一个基于去雾的图像亮度增强的算法[IBEABFHR](https://github.com/rzwm/IBEABFHR)，其中用到了图像取反的操作。所谓图像取反，就是将RGB图像的每个像素点(r, g, b)，使用(255 - r, 255 - g, 255 - b)替换。对于灰度图像而言，则是将(g)使用(255 - g)替换。
如下图所示：
![RGB图像](http://o96d382wn.bkt.clouddn.com/opencv-image-invert-rgb.jpg)
![RGB取反图像](http://o96d382wn.bkt.clouddn.com/opencv-image-invert-rgb_inverse.jpg)
![灰度图像](http://o96d382wn.bkt.clouddn.com/opencv-image-invert-gray.jpg)
![灰度取反图像](http://o96d382wn.bkt.clouddn.com/opencv-image-invert-gray_inverse.jpg)

在OpenCV中要实现此操作，可以遍历每个像素，用255去减，此方法不再赘述。更直接地，可以对图像(`cv::Mat`)整体做减法：

```cpp
cv::Mat image = cv::imread("rgb.jpg");
cv::Mat image_inverse = cv::Scalar(255, 255, 255) - image;
```
也可以使用`cv::subtract`，效果是一样的:

```cpp
cv::Mat image = cv::imread("rgb.jpg");
cv::Mat image_inverse;
cv::subtract(cv::Scalar(255, 255, 255), image, image_inverse);
```

但是我在网上搜索到了更好的方法，那就是使用位运算中的取反操作(~)：

```cpp
cv::Mat image = cv::imread("rgb.jpg");
cv::Mat image_inverse = ~image;
```

原理是，对于一个`unsigned char`类型的变量`c`，`255 - c`与`~c`是相等的。

此方法在`opencv2\core\mat.hpp`中声明如下：

```cpp
CV_EXPORTS MatExpr operator ~(const Mat& m);
```

需要注意的是，此方法仅对整数型的`cv::Mat`有效。

一般来说，位运算的速度都是比较快的，事实也是如此，我使用一张4160x2340的图像来做测试，两种方法各运算100次取平均时间，结果如下:
+ 相减法：14.1862ms
+ 位运算法：11.8158ms

但是在测试过程中发现一个问题：第一次取反操作(无论是相减法还是位运算法)竟然需要耗时100多毫秒，而后面再进行取反操作速度就变正常了，只要10几毫秒。现在还不知道原因是什么。希望有知道的朋友可以告诉我。可以发我邮箱。下面附上出现此问题的代码。

```cpp
// Timer.h
#pragma once
#include <iostream>
#include <chrono>

class Timer
{
public:
	Timer()
		: t1(res::zero())
		, t2(res::zero())
	{
		tic();
	}

	~Timer()
	{}

	void tic()
	{
		t1 = clock::now();
	}

	void toc(const char* str)
	{
		t2 = clock::now();
		std::cout << str << " time: "
			<< std::chrono::duration_cast<res>(t2 - t1).count() / 1e3 << "ms." << std::endl;
	}

private:
	typedef std::chrono::high_resolution_clock clock;
	typedef std::chrono::microseconds res;

	clock::time_point t1;
	clock::time_point t2;
};
```

```cpp
// main.cpp
#include <iostream>
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "Timer.h"

int main()
{
	cv::Mat image = cv::imread("5.jpg");
	if (image.empty())
	{
		std::cout << "Couldn't open file." << std::endl;
		system("pause");
		return -1;
	}

	Timer timer;
	timer.tic();
	cv::Mat temp = cv::Scalar(255, 255, 255) - image;
	timer.toc("single ");

	timer.tic();
	for (int i = 0; i < 100; ++i)
	{
		cv::Mat temp1 = cv::Scalar(255, 255, 255) - image;
	}
	timer.toc("subtract");

	timer.tic();
	for (int i = 0; i < 100; ++i)
	{
		cv::Mat temp1 = ~image;
	}
	timer.toc("operator~");
	
	system("pause");
	return 0;
}
```

注：本文使用OpenCV 3.3.0。