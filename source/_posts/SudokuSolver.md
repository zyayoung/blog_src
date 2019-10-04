---
title: Sudoku Solver
categories: CV
date: 2019-09-28 12:04:05
tags:
  - Detection
---

# 大致思路

1. 读取图像数据，寻找数独在图像中的位置
2. 数字识别
3. 解数独
4. 将结果映射回原图

# Prerequest

- Python
- Opencv
- Numpy
- Mnist
- DFS
- 

# Steps

## 读取图像数据

读取图像使用Opencv库

```python
import cv2
```

读取电脑内置摄像头的图像：
```python
cap = cv2.VideoCapture(0)
```

也可使用手机app将手机作为网络摄像头，从手机上读取图像推流：
```python
cap = cv2.VideoCapture("http://192.168.43.1:8080/video")
```

然后使用
```python
ret, frame = cap.read()
```
将

## 寻找数独在图像中的位置

先用自适应二值化获得


https://android.jlelse.eu/a-beginners-guide-to-setting-up-opencv-android-library-on-android-studio-19794e220f3c
https://github.com/opencv/opencv/blob/master/samples/python/squares.py
