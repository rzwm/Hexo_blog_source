---
title: OpenCV两个Mat相减的隐藏秘密
date: 2018-06-08 23:01:32
tags: [OpenCV]
categories: 图像处理
---

## 起因
今天在看同事写的代码时，发现一个“错误”：
他的原意是实现以下功能：

```cpp
cv::Mat absDiff;
cv::absdiff(mat1, mat2, absDiff);
```

其中`mat1`和`mat2`均为`CV_8UC1`类型。

但是可能是一时没想起这个函数，于是他写成了这个样子：

```cpp
cv::Mat absDiff = cv::abs(mat1 - mat2);
```

## 问题

于是我认真地告诉他，这样做是错的。假设`mat1`为`[0]`，`mat2`为`[255]`，那么`mat1 - mat2`将会得到`[0]`，因为`cv::saturate_cast<uchar>(0 - 255) == 0`。则`cv::abs([0])`自然就是`[0]`，而他的期望是得到`[255]`。

并且我写出如下代码证明他是错的：

```cpp
cv::Mat diff = mat1 - mat;
```

结果`diff`确实是`[0]`。那么`cv::Mat absDiff = cv::abs(diff);`肯定就是`[0]`了。

但是他坚持让我用`cv::Mat absDiff = cv::abs(mat1 - mat2);`测试。结果。。。。`absDiff`竟然真的是`[255]`！我当时就震惊了，同时隐隐有一种感觉：这其中一定隐藏着一个天大的秘密。

## 秘密

于是我进入调试模式，认真观察每一步，终于明白了玄机所在。

### 为什么我得到了`[0]`
先来分析我的测试代码：

```cpp
cv::Mat diff = mat1 - mat;
```

在我以前的观念里，`mat1 - mat`是两个`cv::Mat`互相作用，实际上调用的是`cv::subtract()`。但是事实上，`mat1 - mat2`调用的是以下函数：

```cpp
CV_EXPORTS MatExpr operator - (const Mat& a, const Mat& b);

MatExpr operator - (const Mat& a, const Mat& b)
{
    MatExpr e;
    MatOp_AddEx::makeExpr(e, a, b, 1, -1);
    return e;
}
```

可以看到，`mat1 - mat2`实际上是生成了一个`MatExpr`对象。在生成过程中，并没有进行实际的相减操作，而只是保存了`a`和`b`，并记录了它们之间期望进行的操作：相减。实际的相减操作是在类型转换时进行的：

```cpp
//cv::Mat diff = mat1 - mat; // 在这里调用了operator Mat()

MatExpr::operator Mat() const
{
    Mat m;
    op->assign(*this, m);
    return m;
}
```

在`assign`中最终调用了`cv::subtract()`。所以`mat1 - mat2`得到了`[0]`。

### 为什么同事得到了`[255]`

再来分析我同事的测试代码：

```cpp
cv::Mat absDiff = cv::abs(mat1 - mat2);
```

从上面我们知道，`mat1 - mat2`生成了一个`MatExpr`对象。而`cv::abs()`调用的是

```CPP
MatExpr abs(const MatExpr& e)
{
    CV_INSTRUMENT_REGION()

    MatExpr en;
    e.op->abs(e, en);
    return en;
}
```

这个函数再次生成了一个`MatExpr`对象，其中记录了操作对象`e`和期望进行的操作：取绝对值，而并没有进行实际的运算。然后同上面一样，它是在赋值给`absDiff`时调用`operator Mat()`进行运算的。神奇的地方来了，它把前面的相减操作与这里的取绝对值操作组合到了一起(而不是依次运算)，最终调用的正是`cv::absdiff()`！

于是谜底揭开了，一切的原因都是因为`MatExpr`这个中间对象实现了延迟运算和操作组合。

## 感叹

我还是太年轻！