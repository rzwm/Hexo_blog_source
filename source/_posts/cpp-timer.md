---
title: C++11高精度计时器
date: 2018-06-05 22:26:41
tags: [C++]
categories: C++
---

做图像处理算法时，免不了要测量函数的运行时间。以前我都是使用OpenCV的计时函数`cv::getTickCount()`和`cv::getTickFrequency()`，但是这样一来，在不使用OpenCV的项目中就没法用了。幸好C++11增加了`std::chrono`库，可以很方便地实现跨平台的时间测量。于是我封装了一个简单的计时器类，这样只要将其简单地添加到项目中，就可以直接使用了。此计时器单位为毫秒，但可以精确到微秒级。

```cpp
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

测试代码如下：

```cpp
int main()
{
	Timer timer;
	std::cout << "1" << std::endl;
	timer.toc("output 1");

	timer.tic();
	std::cout << "2" << std::endl;
	timer.toc("output 2");
	
	system("pause");

	return 0;
}
```

输出如下：

```cpp
1
output 1 time: 0.26ms.
2
output 2 time: 0.039ms.
请按任意键继续. . .
```