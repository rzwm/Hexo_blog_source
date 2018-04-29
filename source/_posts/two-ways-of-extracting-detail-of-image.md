---
title: 提取图像细节的两种方法
date: 2018-04-29 21:50:14
tags: [image-processing]
mathjax: true
categories: 图像处理
---

一幅图像可以分解为两层：底层(base layer)和细节层(detail layer)。底层包含图像的低频信息，反映了图像在大尺度上的强度变化；细节层包含图像的高频信息，反映了图像在小尺度上的细节。分解图像有两种方式，以下分别进行解释。

## 1. 加性分解

要获取图像的底层，即图像的低频信息，使用低通滤波(如均值滤波(mean filter)，高斯滤波(gaussian filter)，导向滤波(guided filter))对图像进行滤波即可：
$$B = f(I) $$
其中$I$表示要分解的图像，$f(\cdot)$表示低通滤波操作，$B$为提取的底层。

提取底层后，使用源图像减去底层，即为细节层：
$$D = I - B$$
其中$D$表示提取的细节层。

因为底层加上细节层即为源图像，所以我称此种分解方法为加性分解，对应于加性噪声。关于此种方法的应用，可以参见[1]。

## 2. 乘性分解

获取底层的方法与\textbf{加性分解}相同。然后使用源图像除以底层，即可得到细节层：
$$D = \frac{I + \epsilon}{B + \epsilon}$$
其中$\epsilon$为一个很小的常数，以防止除零错误。

因为底层乘以细节层即为源图像，所以我称此种分解方法为乘性分解，对应于乘性噪声。关于此种方法的应用，可以参见[2]。在其他文章中，此处得到的细节层也称为商图像(quotient image)[3]或比例图像(ratio image)[4]。

## 3. 代码及效果

```cpp
// 图像细节提取。
// 编程环境：Visual Studio Community 2015 + OpenCV 3.3.0
#include "opencv2/core/core.hpp"
#include "opencv2/imgcodecs/imgcodecs.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/highgui/highgui.hpp"

int main()
{
	cv::Mat I = cv::imread("im.png");
	if (I.empty())
	{
		return -1;
	}
	
	I.convertTo(I, CV_32FC3);
	
	cv::Mat B;
	cv::boxFilter(I, B, -1, cv::Size(31, 31));
	
	// 1. 加性分解
	cv::Mat D1 = I - B;
	
	// 2. 乘性分解
	const float epsilon = 1.0f;
	cv::Mat D2 = (I + epsilon) / (B + epsilon);
	
	// 显示图像
	I.convertTo(I, CV_8UC3);
	cv::imshow("源图像", I);
	
	B.convertTo(B, CV_8UC3);
	cv::imshow("Base layer", B);
	
	D1 = cv::abs(D1); // 因为包含负数，所以取绝对值
	D1.convertTo(D1, CV_8UC3);
	cv::imshow("Detail layer 1", D1);
	
	cv::normalize(D2, D2, 0.0, 255.0, cv::NORM_MINMAX); // 归一化
	D2.convertTo(D2, CV_8UC3);
	cv::imshow("Detail layer 2", D2);
	
	cv::waitKey();
	return 0;
}
```

![图1：源图像](http://o96d382wn.bkt.clouddn.com/two-ways-of-extracting-detail-of-image-im.png)
![图2：Base Layer](http://o96d382wn.bkt.clouddn.com/two-ways-of-extracting-detail-of-image-B.png)
![图3：Detail Layer1](http://o96d382wn.bkt.clouddn.com/two-ways-of-extracting-detail-of-image-D1.png)
![图4：Detail Layer2](http://o96d382wn.bkt.clouddn.com/two-ways-of-extracting-detail-of-image-D2.png)

## 4. 应用
提取图像的细节层后，可以进行细节增强(detail enhancement)或细节转移(detail transfer)[2]等。

## 5. 参考文献

[1] S. Li, X. Kang, and J. Hu. Image fusion with guided fltering. IEEE Transactions on Image Processing, 22(7):2864–2875, July 2013.

[2] Georg Petschnigg, Richard Szeliski, Maneesh Agrawala, Michael Cohen, Hugues Hoppe, and Kentaro Toyama. Digital photography with ﬂash and no-ﬂash image pairs. In ACM transactions on graphics (TOG), volume 23, pages 664–672. ACM, 2004.

[3] Amnon Shashua and Tammy Riklin-Raviv. The quotient image: Class-based re-rendering and recognition with varying illuminations. IEEE Transactions on Pattern Analysis and Machine Intelligence, 23(2):129–139, 2001.

[4] Zicheng Liu, Ying Shan, and Zhengyou Zhang. Expressive expression mapping with ratio images. In Proceedings of the 28th annual conference on Computer graphics and interactive techniques, pages 271–276. ACM, 2001.