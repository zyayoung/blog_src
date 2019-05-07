---
title: GomokuAI重置企划1
categories: GomokuAI重置企划
date: 2019-05-05 20:30:54
tags:
- 五子棋
---

咕价函数

<!-- more -->

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

## 第0步：连续5格的估值表

对于五个连续空位（空或包含己方棋子）中$2^5$种情况，对每个位置落子后能形成的棋形估值。

[Code](https://gitee.com/zyayoung/Gomoku/blob/c3891f5114ec1f3deff165d26f05b6a992e864c0/main.cpp)

在ai.pattern5stone[p][i]存储了五个连续空位中所有情况p中位置i落子后能形成的棋形。其中p为7位二进制数，存储五个连续空位有无己方棋子以及两边有无遮挡。

```cpp
bool AI::isP5(int p) { return (p & 0B0111110) == 0B0111110; }

bool AI::isP4(int p) {
    if (p == 0B0111101 || p == 0B0111100) return true;
    if (p == 0B0011110 || p == 0B1011110) return true;
    if (p == 0B0111010 || p == 0B0101110) return true;
    if (p == 0B0110110) return true;
    return false;
}

bool AI::isP4d(int p) {
    if (p == 0B1111101 || p == 0B1111100) return true;
    if (p == 0B1011111 || p == 0B0011111) return true;
    if (p == 0B1101110 || p == 0B0101111) return true;
    if (p == 0B1101111) return true;
    if (p == 0B1111010 || p == 0B0111011) return true;
    if (p == 0B1111011) return true;
    if (p == 0B0110111 || p == 0B1110110) return true;
    if (p == 0B1110111) return true;
    return false;
}

bool AI::isP3(int p) {
    for (int i = 1; i <= 5; i++) {
        if (isP4(p | 1 << i)) return true;
    }
    return false;
}

bool AI::isP3d(int p) {
    for (int i = 1; i <= 5; i++) {
        if (isP4d(p | 1 << i)) return true;
    }
    return false;
}

bool AI::isP2(int p) {
    for (int i = 1; i <= 5; i++) {
        if (isP3(p | 1 << i)) return true;
    }
    return false;
}

bool AI::isP1(int p) {
    for (int i = 1; i <= 5; i++) {
        if (isP2(p | 1 << i)) return true;
    }
    return false;
}

void AI::genPattern5Stone() {
    for (int j = 0; j < 1 << 7; j++) {
        for (int k = 1; k <= 5; k++) {
            pattern5stone[j][k - 1] = P0;
            if (j & 1 << k) {
                pattern5stone[j][k - 1] = P0;
            } else if (isP5(j | 1 << k)) {
                pattern5stone[j][k - 1] = P5;
            } else if (isP4(j | 1 << k)) {
                pattern5stone[j][k - 1] = P4;
            } else if (isP4d(j | 1 << k)) {
                pattern5stone[j][k - 1] = P4d;
            } else if (isP3(j | 1 << k)) {
                pattern5stone[j][k - 1] = P3;
            } else if (isP3d(j | 1 << k)) {
                pattern5stone[j][k - 1] = P3d;
            } else if (isP2(j | 1 << k)) {
                pattern5stone[j][k - 1] = P2;
            } else if (isP1(j | 1 << k)) {
                pattern5stone[j][k - 1] = P1;
            }
        }
    }
}
```

## 第1步：单方向估值表

找到每个方向所有可能成五个子的空位，查表索引求和

[xl文档](https://www.aiexp.info/files/introduction_to_xl.pdf)提供了很大的帮助：

1. 对于多余五个子的连续空间，可以使用大小为5的窗口滑动，对每个空位取最大值即可
2. 对于长度为5-15的连续空间的所有情况可以预先计算完成

生成所有长度的单方向估值表

```cpp
void AI::genPatternCache() {
    // length < 5
    for (int length = 0; length < 5; length++) patternCache.push_back({{}});
    // length == 5
    vector<vector<int>> patternCacheAtLength5;
    for (int pattern = 0; pattern < 1 << 5; pattern++) {
        vector<int> patternCacheAtPattern;
        for (int i = 0; i < 5; i++) {
            patternCacheAtPattern.push_back(pattern5stone[0B1000001 | pattern << 1][i]);
        }
        patternCacheAtLength5.push_back(patternCacheAtPattern);
    }
    patternCache.push_back(patternCacheAtLength5);
    // length >= 5
    for (int length = 6; length <= BOARD_SIZE; length++) {
        vector<vector<int>> patternCacheAtLength;
        for (int pattern = 0; pattern < 1 << length; pattern++) {
            vector<int> patternCacheAtPattern(length, 0);

            // left
            for (int i = 0; i < 5; i++)
                patternCacheAtPattern[i] = pattern5stone[(pattern & 0B11111) << 1 | 1][i];

            // right
            for (int i = length - 5; i < length; i++)
                patternCacheAtPattern[i] = max(
                    patternCacheAtPattern[i],
                    pattern5stone[(pattern >> (length - 5) & 0B11111) << 1 | 1 << 6][i - length + 5]);

            // center
            for (int from = 1, to = 5; to < length - 1; from++, to++) {
                for (int i = from; i <= to; i++)
                    patternCacheAtPattern[i] = max(
                        patternCacheAtPattern[i],
                        pattern5stone[(pattern >> (from) & 0B11111) << 1][i - from]);
            }
            patternCacheAtLength.push_back(patternCacheAtPattern);
        }
        patternCache.push_back(patternCacheAtLength);
    }
}
```

生成当前棋盘的单方向估值表

```cpp
void AI::updatePMap(int side) {
    for (int k = 0; k < DIRS.size(); k++) {
        for (int j = 0; j < DIRS_START[k].size(); j++) {
            int p = 0;  // p - Pattern 0B00000
            int len = 0;
            Position pos = DIRS_START[k][j];
            for (; pos.isValid(); pos += DIRS[k]) {
                if(board[pos.x][pos.y] == side)
                    p = p<<1 | 1, len++;
                else if (board[pos.x][pos.y] == EMPTY)
                    p = p << 1, len++;
                else{
                    if (len >= 5) {
                        Position tmpPos = pos;
                        for (int i = 0; i < len; i++) {
                            tmpPos += -DIRS[k];
                            assert(tmpPos.isValid());
                            PMap[tmpPos.x][tmpPos.y][k][side] = patternCache[len][p][i];
                        }
                    }
                    len = p = 0;
                }
            }
            if (len >= 5) {
                Position tmpPos = pos;
                for (int i = 0; i < len; i++) {
                    tmpPos += -DIRS[k];
                    PMap[tmpPos.x][tmpPos.y][k][side] = patternCache[len][p][i];
                }
            }
        }
    }

    // Update PnCount
    memset(PnCount, 0, sizeof(PnCount));
    for (int i = 0; i < BOARD_SIZE; i++) {
        for(int j=0; j<BOARD_SIZE; j++){
            for(int k=0; k<4; k++){
                PnCount[PMap[i][j][k][side]][side]++;
            }
        }
    }
}
```

## 未来的工作：多方向估值表、增量式估值

多方向
局部更新
