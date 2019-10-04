---
title: Object Detection 简单整理
categories: CV
date: 2019-09-28 12:04:05
tags:
  - Detection
---

# Object Detection 任务

输入：图片

输出：图片中各物体实例的类别和位置

![](https://gluon-cv.mxnet.io/_static/object-detection.png)

# Prerequest

- 深度学习基础

# 主流方法

主流的 Object Detection 分为两大类：one-stage和two-stage。

One-stage方法，图片只经过一次神经网络就可以预测出各物体的类别和位置。特点是速度快，但准确率相对低，尤其不擅长预测小而密集的物体，对物体类别的判断较不准确。其代表有SSD, YOLOv3, RetinaNet。

Two-stage方法，图片需要经过两次神经网络，第一次得到物体的proposal，第二次得到最后的预测结果。特点是速度慢准确率高，主要代表为Faster R-CNN及其衍生版本。

上述说法并不绝对，例如one-stage的RetinaNet在MS COCO上的测试结果好于Faster R-CNN，而two-stage的Light-Head R-CNN速度可以超过one-stage。

# Backbone

- Resnet [arxiv](https://arxiv.org/abs/1512.03385)
- FPN [arxiv](https://arxiv.org/abs/1612.03144)
  
# Fundataion


# Reference

- J. Jordan, “An overview of object detection: one-stage methods,” Jeremy Jordan, 11-Jul-2018. [Online]. Available: https://www.jeremyjordan.me/object-detection-one-stage/. [Accessed: 28-Sep-2019].
‌
