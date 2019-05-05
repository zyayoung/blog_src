---
title: GomokuAI重置企划1
date: 2019-05-05 20:30:54
tags:
---

## xl-engine弱化版分析

[xl-engine](https://github.com/accreator/xl-engine)主要提供如下思路：

1. 置换表
2. 历史启发
3. alpha-beta 或 MTD(f)
4. 增量式棋盘
5. 增量式估值
6. 真假禁手识别
7. VCF 扩展

在弱化版中实现(chao)了3-6, 其中估值使用的是所有位置落子后能形成的棋形加权求和。

## 第0步：估值表

对于五个连续空位（空或包含己方棋子）中$2^5$种情况，对每个位置落子后能形成的棋形估值。

[Code](https://gitee.com/zyayoung/Gomoku/blob/c3891f5114ec1f3deff165d26f05b6a992e864c0/main.cpp)

在ai.pattern5stone[p][i]存储了五个连续空位中所有情况p中位置i落子后能形成的棋形。其中p为7位二进制数，存储五个连续空位有无己方棋子以及两边有无遮挡。

## 第1步：估价函数

找到每个方向所有可能成五个子的空位，查表索引求和

## 未来的工作：增量式估值

局部更新
