---
title: OpenCL clCreateBuffer占用太多时间
date: 2018-06-07 23:36:14
tags: [OpenCL, trick]
categories: OpenCL
---
最近在做一个图像处理的算法，跑在高通平台上，需要使用OpenCL加速。代码分为三个部分：
1. 初始化
2. 处理图像
3. 释放资源

为了尽可能地减少算法的运行时间，我将一切可以预处理的内容都放到了**初始化**中，其中就包括了创建buffer。在初始化中，我调用`clCreateBuffer()`创建了9个buffer，共计约占用600MB内存。然后在**处理图像**中重复使用这些buffer，最后在**释放资源**中释放所有buffer。

但是在实际测试后发现，每调用一次`clCreateBuffer()`，都会花费大约70ms的时间，这样一来，创建所有buffer就花费了约600ms的时间。同样地，在释放这些buffer时，每个也会花费几十毫秒的时间。如此，**初始化**和**释放资源**的时间就令人比较难以接受。

经同事提醒，我想到了高通文档《Qualcomm® Snapdragon™ Mobile Platform OpenCL General Programming and Optimization》中提到的ION内存。文档中说，使用ION可以避免内存拷贝。但是对缩短创建buffer的时间会不会有帮助呢？毕竟在我的印象中，使用`new`创建一段几百MB的内存也才花费几毫秒的时间。

> 使用ION内存来创建OpenCL buffer需要`cl_qcom_ion_host_ptr`扩展(在上述文档中有提到)，其说明及示例代码在OpenCL官网可以查到，为了方便，我直接贴到这里：[cl_qcom_ion_host_ptr](https://www.khronos.org/registry/OpenCL/extensions/qcom/cl_qcom_ion_host_ptr.txt)。

然后我在高通平台上进行了测试。测试结果让人半忧半喜。令人忧的是，使用ION内存来创建OpenCL buffer时，`clCreateBuffer()`只需要几毫秒的时间，但是创建ION内存却需要使用数十毫秒的时间，等于创建buffer的时间转移到了创建ION内存上，最终花费的时间差别不大。令人喜的是，上述现象只发生在第一次调用**初始化**时。后面再次运行算法，再次调用**初始化**，创建buffer和创建ION内存的时间便会都变为几毫秒。另外，无论是第几次调用**释放资源**，速度都很快，只需要数毫秒。

我只是观察到了这个现象，对于其背后的原理却是知之不详。如果了解原理的话，是否可以真正地缩短创建OpenCL buffer的时间呢？

另外我还观察到一个现象。在不使用ION内存时，我使用高通的性能分析工具Snapdragon Profiler观察到，运行算法时系统内存会上升约700MB。但是使用ION内存时，系统内存只会上升200多MB。这可能也跟ION的原理有关吧。

如果有哪位朋友知道上述两个现象的原因，可以发邮件告诉我。多谢！

