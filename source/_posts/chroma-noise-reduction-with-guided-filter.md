---
title: 使用Guided Filter去除彩色噪声
date: 2018-07-19 23:56:27
tags: [image-processing, denoise, chroma noise, guided filter]
categories: 图像处理
---

手机在暗光环境下拍照时，由于进光量不足，拍出的照片上经常会出现严重的彩色噪声，如下图所示：
![彩色噪声严重的图像](http://o96d382wn.bkt.clouddn.com/chroma-noise-reduction-with-guided-filter-2.jpg)


这种彩色噪声极大地影响了图像的观感，所以必须去除。而要在手机上运行，算法也一定要轻量化。这里提出一种轻量级而且比较实用的去彩噪的方法：使用Guided Filter去噪。

算法的步骤如下：
1. 将需要去噪的RGB图像(记为I)转换为YUV格式，记为I_YUV；
2. 对I进行guided filter，I自己作为引导图像，得到滤波后的图像，记为I_gf；
3. 将I_gf转换为YUV格式，记为I_gf_YUV；
4. 使用I_YUV的Y通道替换I_gf_YUV的Y通道，得到新的YUV图像，记为I_Y_gf_UV;
5. 将I_Y_gf_UV转换为RGB格式，即为去噪后图像。

去噪后的效果如下：
![去除彩色噪声后的图像](http://o96d382wn.bkt.clouddn.com/chroma-noise-reduction-with-guided-filter-result.jpg)
可以看到，除了颜色有一点冲淡外，去彩噪的效果相当显著。

测试代码如下：
```cpp
// Win10 + Visual Studio Community 2017 + OpenCV 3.3.0
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/ximgproc.hpp"
#include <iostream>

int main()
{
	cv::Mat I = cv::imread("2.jpg");
	if (I.empty())
	{
		std::cout << "Couldn't open file." << std::endl;
		return -1;
	}

	// step 1. convert I to YUV format
	cv::Mat I_YUV;
	cv::cvtColor(I, I_YUV, cv::COLOR_BGR2YUV_I420);

	// step 2. do guided filtering on I
	cv::Mat I_gf;
	cv::ximgproc::guidedFilter(I, I, I_gf, 30, 0.03 * 255 * 255);

	// step 3. convert I_gf to YUV format
	cv::Mat I_gf_YUV;
	cv::cvtColor(I_gf, I_gf_YUV, cv::COLOR_BGR2YUV_I420);

	// step 4. replace I_gf_YUV's Y channel with I_YUV's Y channel
	cv::Mat I_Y_gf_UV = I_gf_YUV.clone();
	memcpy(I_Y_gf_UV.data, I_YUV.data, I.cols * I.rows);

	// step 5. convert I_Y_gf_UV to BGR, which is the result
	cv::Mat result;
	cv::cvtColor(I_Y_gf_UV, result, cv::COLOR_YUV2BGR_I420);

	cv::imwrite("result.jpg", result);

	return 0;
}

```