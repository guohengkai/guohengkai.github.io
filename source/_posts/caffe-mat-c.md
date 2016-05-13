title: 调用Caffe时Matlab与C++混用的坑
date: 2016-01-22 00:02:35
tags:
- Caffe
- OpenCV
- Matlab
categories:
- Engineering
---
因为C的[HDF5接口](http://www.hdfgroup.org/HDF5/doc/cpplus_RM/index.html)各种不给力，比如不支持increment插入等，因此经常利用Matlab来生成数据进行训练。在混用Matlab和C++调用Caffe时，有两点需要注意，否则会入坑。
<!--more-->
1. Matlab读取图片为RGB格式，而OpenCV为BGR格式。
2. Matlab的矩阵为列优先，而OpenCV为行优先。

为了使得C++的test代码易用，建议利用Matlab进行数据写入时对图像进行转置，即：
``` matlab
img = flip(permute(img, [2, 3, 1]), 3);
```

等以后有空且有需求的时候试着用C++写一个HDF5的库，支持Matlab的h5create、h5write和h5read功能。【挖坑】
