---
title: 分段问题的算法实现
date: 2018-07-19 23:06:29
tags:  [algorithm, segment]
categories: 算法
---

## 分段问题
有一个长度为L的数组，其每个元素的索引分别是0，1，2，...，L-1。现欲将其分段，每段的长度为S(S <= L)。求所有段的起始位置的索引。

例如，L为8，S为4时，输出：0，4。

当L不能被S整除时，一般会有如下三种选择：
1. 独立。即不能被整除的部分作为独立的一段。例如，L为9，S为4时，输出：0，4，8。
2. 合并。即不能被整除的部分合并到上一段。例如：L为9，S为4时，输出：0，4。
3. 分条件独立。即当满足一定条件时，不能被整除的部分作为独立的一段，否则合并到上一段。在本文中，我们只考虑一种条件：不能被整除的部分大于S的一半。例如，L为9，S为4时，输出：0，4；而L为11，S为4时，输出：0，4，8。

## 算法实现
### 独立
最直接的实现，就是预先求出所分的段数，然后计算索引：
```cpp
void segment(int L, int S, std::vector<int>& vec)
{
	int count = 0;
	if (L % S == 0)
	{
		count = L / S;
	}
	else
	{
		count = L / S + 1;
	}

	for (int i = 0; i < count; ++i)
	{
		vec.push_back(i * S);
	}
}
```
除此之外，也可以使用迭代的方式，更加简洁：
```cpp
void segment(int L, int S, std::vector<int>& vec)
{
	for (int index = 0; index < L; index += S)
	{
		vec.push_back(index);
	}
}
```
### 合并
同样可以使用两种方式实现：
```cpp
void segment(int L, int S, std::vector<int>& vec)
{
	const int count = L / S;

	for (int i = 0; i < count; ++i)
	{
		vec.push_back(i * S);
	}
}
```
```cpp
void segment(int L, int S, std::vector<int>& vec)
{
	for (int index = 0; index <= L - S; index += S)
	{
		vec.push_back(index);
	}
}
```

### 分条件独立
同样可以使用两种方式实现：
```cpp
void segment(int L, int S, std::vector<int>& vec)
{
	int count = 0;

	if (L % S == 0)
	{
		count = L / S;
	}
	else
	{
		count = L / S;
		if (L - count * S > S / 2)
		{
			count += 1;
		}
	}

	for (int i = 0; i < count; ++i)
	{
		vec.push_back(i * S);
	}
}
```
```cpp
void segment2(int L, int S, std::vector<int>& vec)
{
	for (int index = 0; index < L - S / 2; index += S)
	{
		vec.push_back(index);
	}
}
```

可以看到，用迭代的方式，代码更加简洁。