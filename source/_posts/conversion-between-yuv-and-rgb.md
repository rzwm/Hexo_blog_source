---
title: YUV与RGB之间的转换公式推导
date: 2018-08-23 23:02:45
tags: [image-processing, YUV]
mathjax: true
categories: 图像处理
---

第一次接触YUV格式，是要把一张YUV420格式的图像转换为RGB格式。于是，我就上网去搜索YUV与RGB之间的转换公式。但是，看了几篇博客之后，我迷惑了：为什么有这么多不同的转换公式？到底哪个才是正确的呢？后来，我找到了一份相对来说比较权威的代码，然后使用了里面所用的转换公式，而那份疑惑也就深深埋于心底。

前两天，我又遇到了YUV与RGB之间转换的问题。这次我决定，一定搞清楚问题的答案不可，于是就有了这篇博客。如果你也遇到了同样的问题，相信读过这篇博客之后，你会和我一样找到答案。

我主要参考了维基百科关于YUV的解释：[YUV](https://en.wikipedia.org/wiki/YUV)。相对于博客而言，维基百科还是比较权威的。

# 五个常数

YUV信号一般是从RGB信号源采样而来的，所以首先有一个RGB转YUV的公式，然后由此公式再反过来推导YUV转RGB的公式。

在定义RGB转YUV的公式之前，先要认识公式所需要的五个常数：$W_R$，$W_G$，$W_B$，$U_{max}$，$V_{max}$。

其中$W_R$，$W_G$，$W_B$分别表示R，G，B通道的权重。根据这三个权重，求R，G，B通道的加权和，即得到Y通道。由于$W_R + W_G + W_B = 1$，所以一般只需定义$W_R$和$W_B$，$W_G$即由$1 - W_R - W_B$给出。

而$U_{max}$和$V_{max}$分别表示Y通道与B通道和R通道的最大偏移系数。

# 根转换公式

认识了所需要的5个常数之后，就可以定义RGB转YUV的公式，如下：

$$
\begin{multline}
\shoveleft Y = W_RR + W_GG + W_BB \\\\
\shoveleft U = U_{max} \frac{B - Y}{1 - W_B} \\\\
\shoveleft V = V_{max}\frac{R - Y}{1 - W_R} \\\\
\end{multline}
$$

由此公式逆推，就可以得到YUV转RGB的公式：

$$
\begin{multline}
\shoveleft R = Y + V\frac{1 - W_R}{V_{max}} \\\\
\shoveleft G = Y - U\frac{W_B(1 - W_B)}{U_{max}W_G} - V\frac{W_R(1 - W_R)}{V_{max}W_G} \\\\
\shoveleft B = Y + U\frac{1 - W_B}{U_{max}} \\\\
\end{multline}
$$

这是YUV与RGB之间最基本最根源的转换公式，所以我称它为根转换公式。

# 量化转换公式

分析上述根转换公式，可以得到如下结论：

若R，G，B的取值范围都是$[0, 1]$，则Y的取值范围为$[0, 1]$，U的取值范围为$[-U_{max}, U_{max}]$，V的取值范围为$[-V_{max}, V_{max}]$。

而一般在图像处理中的R，G，B的取值范围为$[0, 255]​$，则Y的取值范围为$[0, 255]​$，U的取值范围为$[-255U_{max}, 255_{max}]​$，V的取值范围为$[-255V_{max}, 255V_{max}]​$。

可以看到，U和V的值可能为负值，不太符合我们的习惯。所以我们可能期望将其量化到某个正的范围，如$[0, 255]$。添加了量化操作的公式，我称之为量化转换公式。

要量化U和V很简单。比如要从$[-255U_{max}, 255U_{max}]$量化到$[0, 255]$，乘以$\frac{255}{2 \cdot 255U_{max}}$，再加上$127.5$即可。

在实际应用中，一般常用的量化范围是：$Y \in [16, 235], U \in [16, 240], V \in [16, 140]$和$Y \in [0, 255], U \in [0, 255], V \in [0, 255]$。

它们的推导过程如下：

## Y -> [16, 235], U -> [16, 240], V -> [16, 240]

$$
\begin{align}
Y &= (W_RR + W_GG + W_BB) \frac{235 - 16}{255} + 16 \\\\ 
&= (W_RR + W_GG + W_BB) \frac{219}{255} + 16 \\\\
U &= U_{max} \frac{B - Y}{1 - W_B} \frac{240-16}{2 \cdot 255U_{max}} + 128 \\\\
&= \frac{B - Y}{1 - W_B} \frac{112}{255} + 128  //注意：此处的Y是量化前的Y \\\\
V &= V_{max}\frac{R - Y}{1 - W_R} \frac{240-16}{2 \cdot 255V_{max}} + 128 \\\\
&= \frac{R - Y}{1 - W_R} \frac{112}{255} + 128 //注意：此处的Y是量化前的Y \\\\
R &= (Y - 16)\frac{255}{219} + (V - 128)\frac{255V_{max}}{112}\frac{1 - W_R}{V_{max}} \\\\
&= (Y - 16)\frac{255}{219} + (V - 128)\frac{255(1 - W_R)}{112} \\\\
G &= (Y - 16)\frac{255}{219} - (U - 128)\frac{255U_{max}}{112}\frac{W_B(1 - W_B)}{U_{max}W_G} - (V - 128)\frac{255V_{max}}{112}\frac{W_R(1 - W_R)}{V_{max}W_G} \\\\
&= (Y - 16)\frac{255}{219} - (U - 128)\frac{255}{112}\frac{W_B(1 - W_B)}{W_G} - (V - 128)\frac{255}{112}\frac{W_R(1 - W_R)}{W_G} \\\\
B &= (Y - 16)\frac{255}{219} + (U - 128)\frac{255U_{max}}{112}\frac{1 - W_B}{U_{max}} \\\\
&= (Y - 16)\frac{255}{219} + (U - 128)\frac{255(1 - W_B)}{112} \\\\
\end{align}
$$

## Y -> [0, 255], U -> [0, 255], V -> [0, 255]
$$
\begin{align}
Y &= W_RR + W_GG + W_BB\\\\
U &= U_{max} \frac{B - Y}{1 - W_B} \frac{255}{2 \cdot 255U_{max}} + 128 \\\\
&= \frac{B - Y}{2(1 - W_B)} + 128 // 此处推算应为127.5，但别人都是128，不知为何。。。下同 \\\\
V &= V_{max}\frac{R - Y}{1 - W_R} \frac{255}{2 \cdot 255V_{max}} + 128 \\\\
&= \frac{R - Y}{2(1 - W_R)} + 128 \\\\
R &= Y + 2(V - 128)V_{max}\frac{1 - W_R}{V_{max}} \\\\
&= Y + 2(V - 128)(1 - W_R) \\\\
G &= Y - 2(U - 128)U_{max}\frac{W_B(1 - W_B)}{U_{max}W_G} - 2(V - 128)V_{max}\frac{W_R(1 - W_R)}{V_{max}W_G} \\\\
&= Y - 2(U - 128)\frac{W_B(1 - W_B)}{W_G} - 2(V - 128)\frac{W_R(1 - W_R)}{W_G} \\\\
B &= Y + 2(U - 128)U_{max}\frac{1 - W_B}{U_{max}} \\\\
&= Y + 2(U - 128)(1 - W_B) \\\\
\end{align}
$$

# BT.601和BT.709

我们已经推导出了所有公式，但是我们还没有确定一开始认识的那五个常数。那么这五个常数是如何确定的呢？

事实上，对于这五个常数，不同的标准有不同的推荐值。而最常用的标准有BT.601（for SDTV，标清）和BT.709（for HDTV， 高清）。

BT.601的推荐值是：

$$
\begin{multline}
\shoveleft W_R = 0.299 \\\\
\shoveleft W_G = 0.587 \\\\
\shoveleft W_B = 0.114 \\\\
\shoveleft U_{max} = 0.436 \\\\
\shoveleft V_{max} = 0.615 \\\\
\end{multline}
$$

BT.709的推荐值是：

$$
\begin{multline}
\shoveleft W_R = 0.2126 \\\\
\shoveleft W_G = 0.7152 \\\\
\shoveleft W_B = 0.0722 \\\\
\shoveleft U_{max} = 0.436 \\\\
\shoveleft V_{max} = 0.615 \\\\
\end{multline}
$$

将这五个值代入上述公式，就可以得到具体的转换公式了。

这样一来，我们就有了六个公式。大多数的博客所提出的公式都逃不出这个范围（包括下面的定点化）。不信你可以去对照一下。

# 定点化

可以看到，上述六个公式的系数一般都是浮点数。而浮点数在运算时，速度一般没有定点数快。所以，常用的处理是将所有系数乘以一个常数，从而转化为定点数来运算，最后再将结果除以此常数。为了优化最后除以常数的运算，一般选取此常数为2的次幂，这样就可以将除法运算转化为位移运算。此常数一般选取256或16384等。显然，定点化会使转换结果有一定的精度损失，而此常数越大，精度损失就越小。

# 公式生成器

根据上述原理，我写了一个简单的公式生成器。Github地址：[YUVRGBFormulaGenerator](https://github.com/rzwm/YUVRGBFormulaGenerator)。你可以下载此代码，然后生成自己想要的转换公式。

# 具体转换公式

下面我将我推导出的具体的转换公式贴出来，供大家参考。

## BT.601

### 常数

$$
\begin{multline}
\shoveleft W_R = 0.299 \\\\
\shoveleft W_G = 0.587 \\\\
\shoveleft W_B = 0.114 \\\\
\shoveleft U_{max} = 0.436 \\\\
\shoveleft V_{max} = 0.615 \\\\
\end{multline}
$$

### 根公式
$$
\begin{multline}
\shoveleft Y = 0.299R + 0.587G + 0.114B \\\\
\shoveleft U = -0.14714R - 0.28886G + 0.436B \\\\
\shoveleft V = 0.615R - 0.51499G - 0.10001B \\\\
\shoveleft R = Y + 1.13984V \\\\
\shoveleft B = Y - 0.39465U - 0.58060V \\\\
\shoveleft B = Y + 2.03211U \\\\
\end{multline}
$$

### Y -> [16, 235], U -> [16, 240], V -> [16, 240]

$$
\begin{multline}
\shoveleft Y = 0.25679R + 0.50413G + 0.09791B + 16 \\\\
\shoveleft U = -0.14822R - 0.29099G + 0.43922B + 128 \\\\
\shoveleft V = 0.43922R - 0.36779G - 0.07143B + 128 \\\\
\shoveleft R = 1.16438(Y - 16) + 1.59603(V - 128) \\\\
\shoveleft G = 1.16438(Y - 16) - 0.39176(U - 128) - 0.81297(V - 128) \\\\
\shoveleft B = 1.16438(Y - 16) + 2.01723(U - 128) \\\\
\end{multline}
$$

### Y -> [0, 255], U -> [0, 255], V -> [0, 255]

$$
\begin{multline}
\shoveleft Y = 0.299R + 0.587G + 0.114B \\\\
\shoveleft U = -0.16874R - 0.33126G + 0.5B + 128 \\\\
\shoveleft V = 0.5R - 0.41869G - 0.08131B + 128 \\\\
\shoveleft R = Y + 1.402(V - 128) \\\\
\shoveleft G = Y - 0.34414(U - 128) - 0.71414(V - 128) \\\\
\shoveleft B = Y + 1.772(U - 128) \\\\
\end{multline}
$$

## BT.709

### 常数

$$
\begin{multline}
\shoveleft W_R = 0.2126 \\\\
\shoveleft W_G = 0.7152 \\\\
\shoveleft W_B = 0.0722 \\\\
\shoveleft U_{max} = 0.436 \\\\
\shoveleft V_{max} = 0.615 \\\\
\end{multline}
$$

### 根公式

$$
\begin{multline}
\shoveleft Y = 0.2126R + 0.7152G + 0.0722B \\\\
\shoveleft U = -0.09991R - 0.33609G + 0.436B \\\\
\shoveleft V = 0.615R - 0.55861G - 0.05639B \\\\
\shoveleft R = Y + 1.28033V \\\\
\shoveleft G = Y - 0.21482U - 0.38059V \\\\
\shoveleft B = Y + 2.12798U \\\\
\end{multline}
$$

### Y -> [16, 235], U -> [16, 240], V -> [16, 240]

$$
\begin{multline}
\shoveleft Y = 0.18259R + 0.61423G + 0.06201B + 16 \\\\
\shoveleft U = -0.10064R - 0.33857G + 0.43922B + 128 \\\\
\shoveleft V = 0.43922R - 0.39894G - 0.04027B + 128 \\\\
\shoveleft R = 1.16438(Y - 16) + 1.79274(V - 128) \\\\
\shoveleft G = 1.16438(Y - 16) - 0.21325(U - 128) - 0.53291(V - 128) \\\\
\shoveleft B = 1.16438(Y - 16) + 2.11240(U - 128) \\\\
\end{multline}
$$

### Y -> [0, 255], U -> [0, 255], V -> [0, 255]

$$
\begin{multline}
\shoveleft Y = 0.2126R + 0.7152G + 0.0722B \\\\
\shoveleft U = -0.11457R - 0.38543G + 0.5B + 128 \\\\
\shoveleft V = 0.5R - 0.45415G - 0.04585B + 128 \\\\
\shoveleft R = Y + 1.5748(V - 128) \\\\
\shoveleft G = Y - 0.18732(U - 128) - 0.46812(V - 128) \\\\
\shoveleft B = Y + 1.8556(U - 128) \\\\
\end{multline}
$$

