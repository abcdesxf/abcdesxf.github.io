---
title: '课程学习笔记+算法介绍 - 轮廓线DP'
date: 2026-07-26 22:06:28
updated: 2026-07-26 22:06:28
tags:
  []
categories:
  []
description: ''
---

# 轮廓线 DP

轮廓线 DP 是状压 DP 的一种。它按照固定顺序扫描棋盘，用状态记录“已处理区域”和“未处理区域”交界处的信息。

我们先看最基础的问题：

> 现有一个 $n \times m$ 的棋盘，用 $1 \times 2$ 的骨牌覆盖，求覆盖满的方案数。

普通的 $dp_{i,j}$ 无法表示当前格子是否已被上一行的竖骨牌占用，因此需要用一个二进制状态记录整条轮廓线。

考虑一行一行转移。设 $mask$ 表示当前行被上一行竖骨牌占用的位置，$nxt$ 表示当前行向下一行放置竖骨牌的位置。

每次转移按以下步骤进行：

1. 枚举下一行状态 $nxt$；
2. 判断 $mask$ 能否转移到 $nxt$；
3. 更新答案。

首先必须满足

$$mask \mathbin{\&} nxt=0,$$

否则同一个格子既被上方的竖骨牌占用，又放置了向下的竖骨牌。

再令

$$s=mask\mathbin{|}nxt.$$

$s$ 中为 $1$ 的格子已经由竖骨牌覆盖，为 $0$ 的格子只能放横骨牌。因此，$s$ 中每一段连续的 $0$ 的长度都必须是偶数。

# 例题讲解

## A. 铺砖问题

### 思路

本题就是上面的直接枚举法。

令 $f_s$ 表示处理完若干行后，下一行被竖骨牌预先占用的状态为 $s$ 时的方案数。初始时 $f_0=1$；枚举当前状态 $s$ 和下一状态 $t$，若二者可以转移，就令 $g_t\mathrel{+}=f_s$。

所有行处理完后不能留下伸出棋盘的竖骨牌，因此答案为 $f_0$。

为减小状态数，可以交换 $h,w$，令较小者作为状态宽度。

时间复杂度为 $O(h4^w)$，空间复杂度为 $O(2^w)$。

### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
int h, w, s, t, i, j, S;
long long f[1 << 11], g[1 << 11];
bool a[1 << 11][1 << 11];
bool ok(int x, int y) {
	if (x & y)return 0;
	int z = x | y, c = 0;
	for (int k = 0; k < w; k++) {
		if (z >> k & 1) {
			if (c & 1)return 0;
			c = 0;
		} else c++;
	}
	return !(c & 1);
}
int main() {
	cin >> h >> w;
	if (w > h) {
		swap(w, h);
	}
	S = 1 << w;
	for (s = 0; s < S; s++) {
		for (t = 0; t < S; t++) {
			a[s][t] = ok(s, t);
		}
	}
	f[0] = 1;
	for (i = 0; i < h; i++) {
		memset(g, 0, sizeof g);
		for (s = 0; s < S; s++) {
			for (t = 0; t < S; t++) {
				if (a[s][t])g[t] += f[s];
			}
		}
		memcpy(f, g, sizeof f);
	}
	cout << f[0];
	return 0;
}
```

## B. 铺砖问题 II

### 思路

本题还要处理障碍和 $1\times1$ 木块。若继续直接枚举相邻两行状态，还需要额外统计每段空位中放置单格木块的方案，转移较难处理。

因此改用**逐格推进的轮廓线 DP**：按照从上到下、从左到右的顺序逐个处理格子。

令 $f_{s,k}$ 表示已经处理完当前格子之前的所有格子，轮廓线状态为 $s$，并且已经使用了 $k$ 个 $1\times1$ 木块的方案数。$s$ 的最低位表示当前格子是否已被前面放置的木块占用；处理完当前格子后，将状态右移一位。

对当前格子分类讨论：

1. 格子已经填充：当前位必须为 $0$，直接右移；
2. 格子未填充且当前位为 $1$：它已被木块占用，直接右移；
3. 格子未填充且当前位为 $0$：可以放 $1\times1$ 木块、向右的 $1\times2$ 木块或向下的 $2\times1$ 木块。

放横木块时，将下一格对应的位设为 $1$；放竖木块时，将右移后状态的最高位设为 $1$。两种操作都必须保证目标格存在、未被预先填充且尚未被其他木块占用。

只需保留 $k\le d$ 的状态。全部格子处理完后，轮廓线必须为 $0$，答案为

$$\sum_{k=c}^{d}f_{0,k}.$$

时间复杂度为 $O(nm2^md)$，空间复杂度为 $O(2^md)$。

### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int P = 1e9 + 7;
int n, m, c, d, i, j, s, k, S, v, f[1 << 10][21], g[1 << 10][21];
char a[110][15];
void add(int &x, int y) {
	x = (x + y) % P;
}
int main() {
	cin >> n >> m >> c >> d;
	for (i = 0; i < n; i++) {
		cin >> a[i];
	}
	S = 1 << m;
	f[0][0] = 1;
	for (i = 0; i < n; i++) {
		for (j = 0; j < m; j++) {
			memset(g, 0, sizeof g);
			for (s = 0; s < S; s++) {
				for (k = 0; k <= d; k++) {
					if (f[s][k]) {
						v = f[s][k];
						if (a[i][j] == '0') {
							if (!(s & 1)) {
								add(g[s >> 1][k], v);
							}
						} else if (s & 1) {
							add(g[s >> 1][k], v);
						} else {
							if (k < d) {
								add(g[s >> 1][k + 1], v);
							}
							if (j + 1 < m && a[i][j + 1] == '1' && !(s & 2)) {
								add(g[(s | 2) >> 1][k], v);
							}
							if (i + 1 < n && a[i + 1][j] == '1') {
								add(g[(s >> 1) | (1 << m - 1)][k], v);
							}
						}
					}
				}	
			}
			memcpy(f, g, sizeof f);
		}
	}
	v = 0;
	for (k = c; k <= d; k++)add(v, f[0][k]);
	cout << v;
}
```

## C. 卡卡颂预备

### 思路

每条边只有 `C`、`R`、`F` 三种状态，因此可以用三进制数记录一整行的边。

令 $f_s$ 表示上一行所有方块的下边组成的三进制状态为 $s$ 时的方案数。处理当前行时，使用 DFS 从左到右枚举每个方块的旋转方向，同时维护：

- 当前行各方块的上边状态 $x$；
- 当前行各方块的下边状态 $y$；
- 前一个方块的右边类型。

相邻方块必须满足前一个方块的右边等于当前方块的左边。一整行枚举完成后，用

$$g_y\mathrel{+}=f_x$$

完成行间转移。

第一行的上边和最后一行的下边不需要与其他方块匹配。可以把第一行之前的所有三进制状态都设为 $1$，最后对所有状态求和。

题目把四次旋转视为四种方案，因此即使一个方块旋转后外观不变，也必须重复计数。例如 `RRRR` 有四种旋转方案。代码先合并边界完全相同的旋转，并记录其出现次数，既保证计数正确，也能减少 DFS 分支。

这里使用的是**搜索生成行转移**，而不是前文的二进制状态直接枚举法。设一行合法的不同旋转序列数为 $T$，时间复杂度为 $O(nT)$，最坏为 $O(n4^m)$；状态数组的空间复杂度为 $O(3^m)$。将棋盘整体旋转，使 $m\le n$，可以减小状态宽度。

### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int P = 1e9 + 7;
int n, m, i, j, k, r, S, ans, pw[13], f[531441], g[531441], q[12][12], e[12][12][4][4], z[12][12][4], x[4];
char a[12][12][5], b[12][12][5];
string s;
void add(int &x, long long y) {
	x = (x + y) % P;
}
void dfs(int c, int l, int x, int y, int v) {
	if (c == m) {
		if (f[x]) {
			add(g[y], 1LL * f[x]*v);
		}
		return;
	}
	for (int t = 0; t < q[i][c]; t++) {
		if (c && e[i][c][t][3] != l)continue;
		dfs(c + 1, e[i][c][t][1], x + e[i][c][t][0] * pw[c], y + e[i][c][t][2] * pw[c], 1LL * v * z[i][c][t] % P);
	}
}
int main() {
	cin >> n >> m;
	for (i = 0; i < n; i++) {
		for (j = 0; j < m; j++) {
			cin >> s;
			strcpy(a[i][j], s.c_str());
		}
	}
	if (m > n) {
		for (i = 0; i < n; i++) {
			for (j = 0; j < m; j++) {
				strcpy(b[j][n - 1 - i], a[i][j]);
			}
		}
		swap(n, m);
		memcpy(a, b, sizeof a);
	}
	for (i = 0; i < n; i++) {
		for (j = 0; j < m; j++) {
			for (r = 0; r < 4; r++) {
				for (k = 0; k < 4; k++) {
					x[k] = (a[i][j][(k - r + 4) % 4] == 'C' ? 0 : a[i][j][(k - r + 4) % 4] == 'R' ? 1 : 2);
				}
				for (k = 0; k < q[i][j]; k++) {
					if (!memcmp(e[i][j][k], x, sizeof x)) break;
				}
				if (k == q[i][j])memcpy(e[i][j][q[i][j]++], x, sizeof x);
				z[i][j][k]++;
			}
		}
	}
	pw[0] = 1;
	for (i = 1; i <= m; i++) {
		pw[i] = pw[i - 1] * 3;
	}
	S = pw[m];
	for (i = 0; i < S; i++) {
		f[i] = 1;
	}
	for (i = 0; i < n; i++) {
		memset(g, 0, sizeof g);
		dfs(0, 0, 0, 0, 1);
		memcpy(f, g, sizeof f);
	}
	for (i = 0; i < S; i++) {
		add(ans, f[i]);
	}
	cout << ans;
	return 0;
}
```
