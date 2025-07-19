---
title: dp优化整合
author: PengDave
date: 2025-07-14 19:30:04
tags:
---
# 斜率优化

## P3195 [HNOI2008] 玩具装箱

考虑 dp，令 $s_i=\sum^{i}_{k=1}c_k$，$dp_i$ 为 $1$ 到 $ i$ 装箱的最小代价，则转移方程为 $dp_i=\min dp_j+(i-j-1+s_i-s_j-L)^2(1\le j<i)$。

此时，为了方便，我们将所有的 $s_i$ 加 $1$，将 $L$ 也加 $1$，那么式子简化为 $dp_j+(s_i-s_j-L)^2=dp_j+s_i^2-2s_is_j-2s_iL+(s_j+L)^2$。

此时，假设有 $j_0<j_1$ 且由 $j_1$ 转移更优，则 $dp_{j_0}+s_i^2-2s_is_{j_0}-2s_iL+(s_{j_0}+L)^2\ge  dp_{j_1}+s_i^2-2s_is_{j_1}-2s_iL+(s_{j_1}+L)^2$，即 $(s_{j_1}-s_{j_0})\times 2s_i\ge [dp_{j_1}+(s_{j_1}+L)^2]-[dp_{j_0}+(s_{j_0}+L)^2]$，由于显然 $s_{j_1}\ge s_{j_0}$，令 $Y_i=dp_i+(s_i+L)^2$，因此 $2s_i \ge \frac{Y_{j_1}-Y_{j_0}}{s_{j_1}-s_{j_0}}$。
