---
title: OpenGL导入3D模型
categories: 游戏相关
date: 2019-05-05 23:25:29
tags:
- OpenGL
- Blender
---

Blender模型导入OpenGL(freeglut)踩坑经历

<!-- more -->

## 思路

先将.blend中需要的模型导出成包含基本顶点和面的信息的.obj和.mtl文件，再在程序中读取。

## 核心

ObjLoader类，可读取外部.obj文件并绘制。参考ChenJY的博文[OpenGL读取obj文件模型](https://chenjiayang.me/2016/12/07/OpenGL/) (December 7, 2016)

## 改

- 原博文的ObjLoader只加载三角形而忽视四边形；解决方案：Blender导出时选择导出成三角形
- 对于复杂的blender模型可利用Decimate修改器减少三角形数量
- 加载.mtl文件，并处理好每个面的材质（颜色），以加入色彩
- 将.obj和.mtl文件写入.h文件中，以便编译进可执行文件
