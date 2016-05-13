title: 那些年踩过的OpenCV 2.4.X的坑
date: 2016-01-12 11:43:51
tags:
- OpenCV
categories:
- Engineering
---
(last updated: 2016.01.12)
由于前几天又被坑了一下，在这里记录一下OpenCV 2.4.X里的一些坑和比较隐藏的东西，以防以后又踩到。
<!--more-->
## VideoCapture
1. VideoCapture里的CV_CAP_PROP_POS_FRAMES和CV_CAP_PROP_FRAME_COUNT得到的数与实际利用read加上帧数统计的结果并不一致（偏小）。
所以建议不要利用set和get来实现视频回放或跳帧的功能。

## Mat
1. Mat在赋值的时候是采用引用计数的方式。复制的时候需要使用clone()。
2. 有多少人不知道Mat可以表示超过4个通道：CV_32FC(100)。
3. CV_32FC3的元素类型为Vec3f，CV_8UC3的元素类型为Vec3i。
