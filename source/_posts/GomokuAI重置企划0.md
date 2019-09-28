---
title: GomokuAI重置企划0
categories: GomokuAI重置企划
date: 2019-05-05 20:16:03
tags:
- 五子棋
---

五子棋AI重置企划以及准备工作

<!-- more -->

现在Yaa的GomokuAI有两个版本：
1. xl-engine弱化版 [Code](https://gitee.com/zyayoung/codes/dps4krwle2v790gn1qto615)
2. 手撸菜鸡版


## 准备工作

除了准备好四个基本方向之外，还需要用到每个方向所有可能的起始位置

定义四个方向：

```cpp
const vector<Position> DIRS = {
    {1, 0},    // below / above
    {0, 1},    // right / left
    {1, 1},    // below right / above left
    {1, -1}    // below left / above right
};
```

寻找每个发现所有的起始位置：

```cpp
vector<vector<Position>> DIRS_START;
void AI::genDIRS_START() {
    vector<Position> d1;
    vector<Position> d2;
    vector<Position> d3;
    vector<Position> d4;

    for (int i = 0; i < BOARD_SIZE; i++) d1.push_back({0, i});
    DIRS_START.push_back(d1);

    for (int i = 0; i < BOARD_SIZE; i++) d2.push_back({i, 0});
    DIRS_START.push_back(d2);

    for (int i = 0; i < BOARD_SIZE - 4; i++) d3.push_back({i, 0});
    for (int i = 1; i < BOARD_SIZE - 4; i++) d3.push_back({0, i});
    DIRS_START.push_back(d3);

    for (int i = 1; i < BOARD_SIZE - 4; i++) d4.push_back({i, BOARD_SIZE - 1});
    for (int i = 4; i < BOARD_SIZE; i++) d4.push_back({0, i});
    DIRS_START.push_back(d4);
}
```

按方向遍历棋盘：

```cpp
for (int k = 0; k < DIRS.size(); k++)
    for (int j = 0; j < DIRS_START[k].size(); j++)
        for (Position pos = DIRS_START[k][j]; pos.isValid(); pos += DIRS[k])
```
