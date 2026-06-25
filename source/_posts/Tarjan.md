---
title: "Tarjan算法详解"
date: 2026-06-21 12:00:00
updated: 2026-06-21 12:00:00
tags:
  - "C++"
  - "算法"
categories:
  - "算法笔记"
description: "此文详细地介绍了Tarjan算法的核心思想与用途。"
cover: "/img/posts/20260625-221628-Tarjan/cover.png"
top_img: "/img/posts/20260625-221628-Tarjan/cover.png"
---

# Tarjan 算法

Tarjan 算法是一类基于深度优先搜索（DFS）的图论算法，常用于解决图的连通性问题，例如：

- 有向图强连通分量（SCC）
- 无向图割点
- 无向图割边，也称桥
- 边双连通分量（e-DCC）
- 点双连通分量（v-DCC）
- 缩点建图与 DAG 上的动态规划
- 更多连通性问题

Tarjan 算法的核心思想是：
在 DFS 的过程中，通过记录每个点的访问时间以及它能够回溯到的最早祖先（即 $dfn$ 值与 $low$ 值），从而判断某些点或边是否具有“分割图结构”的作用（即割点/割边）。

## DFS 树

在学习 Tarjan 算法之前，需要先理解一个重要概念：DFS 树。

> DFS 树是对图执行一次深度优先搜索后得到的搜索树。
> 树边表示 DFS 真正走过的边，非树边表示图中存在但 DFS 没有将其作为搜索路径的边。

由于图分为无向图和有向图，DFS 树中的边类型也有所不同。

### 无向图中的 DFS 树

在无向图中，DFS 过程中只会出现两类边：

1. **树边（tree edge）**
   DFS 实际经过并用于扩展新节点的边。
2. **返祖边（back edge）**
   从某个节点指向其祖先节点的非树边，也称回边或后向边。

需要特别注意的是：
在无向图的 DFS 树中，不会出现连接两个不同 DFS 子树的横叉边。

原因是：
如果两个不同子树中的节点之间存在一条边，那么 DFS 在遍历其中一个节点时，就会沿着这条边访问到另一个节点，从而改变 DFS 树结构。因此，在固定 DFS 顺序下，这种边不会作为“跨子树边”保留下来。

也就是说，在无向图 DFS 树中，非树边只有返祖边。

### 有向图中的 DFS 树

在有向图中，由于边具有方向，DFS 树中的边类型更加丰富，通常可以分为四类：

1. **树边（tree edge）**
   DFS 过程中从当前节点访问一个未访问节点所经过的边。
2. **返祖边（back edge）**
   从当前节点指向 DFS 树中某个祖先节点的边。
   返祖边的存在通常意味着图中存在环。
3. **前向边（forward edge）**
   从当前节点指向其子树中某个后代节点的非树边。
4. **横叉边（cross edge）**
   指向非祖先、非后代，且已经访问过的节点的边。

## 前置知识

### 连通分量

连通分量，简称 CC，是无向图中的概念。

> 在无向图中，若一个极大子图内部任意两点都可以互相到达，则这个子图称为一个连通分量。

“极大”的含义是：
不能再加入其他节点而仍然保持连通。

### 强连通分量

强连通分量，简称 SCC，是有向图中的概念。

> 在有向图中，若一个极大子图内部任意两点都可以互相到达，则这个子图称为一个强连通分量。

换句话说，对于强连通分量中的任意两个点 `u` 和 `v`：

- `u` 可以到达 `v`
- `v` 也可以到达 `u`

强连通分量可以使用 Tarjan 算法在线性时间内求出。

### 弱连通分量

弱连通分量，简称 WCC，也是有向图中的概念。

> 将有向图中的所有边忽略方向后，若某个极大子图变成无向连通图，则称其为一个弱连通分量。

弱连通只要求忽略方向后连通，不要求点之间在原图中互相可达。

### 割点

割点是无向图中的概念。

> 在无向图中，如果删除某个点以及与它相连的所有边之后，图的连通分量数量增加，则这个点称为割点，也称关节点。

对于 DFS 树而言，割点的判定分为两类：

1. **根节点**
   
   如果 DFS 根节点有至少两个 DFS 子树，则根节点是割点。
2. **非根节点**
   如果存在一个子节点 `v`，满足 `low[v] >= dfn[u]`，则 `u` 是割点。

其中：

- `dfn[u]` 表示节点 `u` 的 DFS 访问时间。
- `low[v]` 表示 `v` 的子树能够通过树边和返祖边回到的最早祖先的时间戳。

### 割边

割边，又称桥，是无向图中的概念。

> 在无向图中，如果删除某条边之后，图的连通分量数量增加，则这条边称为割边。

对于 DFS 树中的一条树边 `(u, v)`，其中 `u` 是 `v` 的父节点：

如果满足：

```cpp
low[v] > dfn[u]
```

那么边 `(u, v)` 是割边。

含义是：
`v` 的子树无法通过返祖边回到 `u` 或 `u` 的祖先，因此一旦删除 `(u, v)`，`v` 的子树就会与图的其余部分断开。

### 边双连通分量

边双连通分量常记为 e-DCC。

> 在无向图中，如果一个极大连通子图中不存在割边，则它是一个边双连通分量。

等价地说，在同一个边双连通分量中，任意两点之间至少存在两条边不重复的路径。

求边双连通分量的一般方法是：

1. 先用 Tarjan 求出所有割边。
2. 删除所有割边。
3. 在剩余图中 DFS，每个连通块就是一个边双连通分量。

### 点双连通分量

点双连通分量常记为 v-DCC。

> 在无向图中，如果一个极大连通子图中不存在割点，则它是一个点双连通分量。

等价地说，在同一个点双连通分量中，任意两点之间至少存在两条点不重复的路径。

需要注意：

- 一个割点可能属于多个点双连通分量。
- 一条边只会属于一个点双连通分量。
- 点双连通分量通常使用边栈维护。

## 核心思想

Tarjan 算法主要依赖两个数组：

```cpp
dfn[u]
low[u]
```

### dfn 数组

`dfn[u]` 表示节点 `u` 第几个被 DFS 访问到。

例如：

```cpp
dfn[u] = ++timer;
```

`dfn` 越小，说明这个点越早被访问，在 DFS 树中的深度也通常更浅。

### low 数组

`low[u]` 表示：

> 从 `u` 出发，经过若干条 DFS 树边，再至多经过一条返祖边，能够到达的最早节点的 `dfn` 值。

在无向图中：

```cpp
low[u] = min(
    dfn[u],
    所有子节点 v 的 low[v],
    所有返祖边 (u, v) 中的 dfn[v]
);
```

在有向图求强连通分量时，`low[u]` 的含义略有不同：

> `low[u]` 表示从 `u` 出发，在当前 DFS 栈内能够到达的最早节点的时间戳。

因此，在有向图 SCC 中，只有当目标点仍在栈中时，才可以用它更新 `low[u]`。

## Tarjan 求强连通分量

### 核心思想

在有向图中，如果若干个点互相可达，则它们属于同一个强连通分量。

Tarjan 算法通过 DFS 维护一个栈：

- 当一个节点第一次被访问时，将其入栈。
- 如果某个节点 `u` 满足 `dfn[u] == low[u]`，说明 `u` 是某个强连通分量在 DFS 树中的最高点。
- 此时不断弹栈，直到弹出 `u`，弹出的所有节点构成一个强连通分量。


### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 10010;
int n, m, ts, scc, dfn[MAXN], low[MAXN], vis[MAXN];
vector<int> G[MAXN], ans[MAXN];
stack<int> stk;
void tarjan(int u) {
    dfn[u] = low[u] = ++ts;//更新dfn，low
    stk.push(u);//入栈当前点
    vis[u] = 1;//标记在栈中
    for (int v : G[u]) {//枚举子节点
        if (!dfn[v]) {//没有走过
            tarjan(v);
            low[u] = min(low[u], low[v]);//用儿子的更新自己
        } else if (vis[v]) {//在栈中
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (dfn[u] == low[u]) {//满足SCC条件
        scc++;
        //退栈
        while (true) {
            int x = stk.top();
            stk.pop();
            ans[scc].push_back(x);
            vis[x] = 0;//取消在栈中标记
            if (x == u) break;
        }
    }
}
int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
    }
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) tarjan(i);
    }
    for (int i = 1; i <= scc; i++) {
        for (auto x : ans[i]) {
            cout << x << " ";
        }
        cout << "\n";
    }
    return 0;
}
```


### 时间复杂度

Tarjan 求 SCC 的时间复杂度为：

```cpp
O(n + m)
```

其中：

- `n` 是点数。
- `m` 是边数。

每个点只会入栈、出栈一次，每条边也只会被遍历一次。


## 缩点

### 什么是缩点

缩点是指：

> 将每一个强连通分量看成一个新点，如果两个强连通分量之间存在边，则在新图中连一条边。

缩点后得到的图一定是一个有向无环图，即 DAG。

原因是：
如果缩点后的图中仍然存在环，那么环上的所有强连通分量应该可以互相到达，它们本应属于同一个强连通分量，矛盾。


### 缩点代码

题目：[P3387 【模板】缩点 / 强连通分量 - 洛谷](https://www.luogu.com.cn/problem/P3387)

> [!TIP]
>
> 此题还需要用一个dp求最长路。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 10010;
int n, m, ts, scc, ans, a[MAXN], dfn[MAXN], low[MAXN], vis[MAXN], num[MAXN], A[MAXN], dp[MAXN];
vector<int> G[MAXN], G2[MAXN];
stack<int> stk;
void tarjan(int u) {
    dfn[u] = low[u] = ++ts;
    stk.push(u);
    vis[u] = 1;
    for (int v : G[u]) {
        if (!dfn[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else if (vis[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (dfn[u] == low[u]) {
        scc++;
        while (true) {
            int x = stk.top();
            stk.pop();
            vis[x] = 0;
            num[x] = scc;
            A[scc] += a[x];
            if (x == u) break;
        }
    }
}
int dfs(int u) {
    if (dp[u] != -1) return dp[u];
    dp[u] = A[u];
    for (int v : G2[u]) {
        dp[u] = max(dp[u], A[u] + dfs(v));
    }
    return dp[u];
}
int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    for (int i = 1; i <= m; i++) {
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
    }
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) tarjan(i);
    }
    for (int u = 1; u <= n; u++) {
        for (int v : G[u]) {
            if (num[u] != num[v]) {
                G2[num[u]].push_back(num[v]);
            }
        }
    }
    memset(dp, -1, sizeof(dp));
    for (int i = 1; i <= scc; i++) {
        ans = max(ans, dfs(i));
    }
    cout << ans << "\n";
    return 0;
}
```


## Tarjan 求割点

### 判定条件

在无向图中，对于 DFS 树上的一条树边 `(u, v)`，其中 `u` 是 `v` 的父节点：

#### 非根节点

如果：

```cpp
low[v] >= dfn[u]
```

则 `u` 是割点。

含义是：
`v` 的子树无法通过返祖边回到 `u` 的祖先。删除 `u` 后，`v` 的子树会与图的其他部分断开。

#### 根节点

如果 DFS 根节点有至少两个 DFS 子树，则根节点是割点。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 10010;
const int MAXM = 200010;
int n, m, ts, dfn[MAXN], low[MAXN];
bool cut[MAXN];
vector<pair<int, int>> G[MAXN];
void tarjan(int u, int inEdge) {
    dfn[u] = low[u] = ++ts;
    int child = 0;
    for (auto [v, id] : G[u]) {
        if (!dfn[v]) {
            child++;
            tarjan(v, id);
            low[u] = min(low[u], low[v]);
            if (inEdge != 0 && low[v] >= dfn[u]) {
                cut[u] = true;
            }
        } else if (id != inEdge) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (inEdge == 0 && child >= 2) {
        cut[u] = true;
    }
}
int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        int u, v;
        cin >> u >> v;
        G[u].push_back({v, i});
        G[v].push_back({u, i});
    }
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) {
            tarjan(i, 0);
        }
    }
    for (int i = 1; i <= n; i++) {
        if (cut[i]) {
            cout << i << " ";
        }
    }
    return 0;
}
```


## Tarjan 求割边

### 判定条件

对于 DFS 树上的一条树边 `(u, v)`，其中 `u` 是 `v` 的父节点：

```cpp
low[v] > dfn[u]
```

则边 `(u, v)` 是割边。

注意与割点的区别：

```cpp
割点：low[v] >= dfn[u]
割边：low[v] >  dfn[u]
```

为什么割边是严格大于？

因为如果 `low[v] == dfn[u]`，说明 `v` 的子树仍然可以通过返祖边回到 `u`。
此时删除边 `(u, v)` 后，仍然可以通过其他路径连接到 `u`，所以 `(u, v)` 不是割边。

### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 10010;
const int MAXM = 200010;
int n, m, ts, dfn[MAXN], low[MAXN];
bool bridge[MAXM];
int eu[MAXM], ev[MAXM];
vector<pair<int, int>> G[MAXN];
void tarjan(int u, int inEdge) {
    dfn[u] = low[u] = ++ts;
    for (auto [v, id] : G[u]) {
        if (!dfn[v]) {
            tarjan(v, id);
            low[u] = min(low[u], low[v]);
            if (low[v] > dfn[u]) {
                bridge[id] = true;
            }
        } else if (id != inEdge) {
            low[u] = min(low[u], dfn[v]);
        }
    }
}
int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        int u, v;
        cin >> u >> v;
        eu[i] = u;
        ev[i] = v;
        G[u].push_back({v, i});
        G[v].push_back({u, i});
    }
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) {
            tarjan(i, 0);
        }
    }
    for (int i = 1; i <= m; i++) {
        if (bridge[i]) {
            cout << eu[i] << " " << ev[i] << "\n";
        }
    }
    return 0;
}
```


## 边双连通分量

### 求解思路

边双连通分量的关键性质是：
割边不属于任何环。

因此求边双连通分量只需求出所有割边，将割边删除后剩下的就是边双连通分量。


### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 10010;
const int MAXM = 200010;
int n, m, ts, bccCnt;
int dfn[MAXN], low[MAXN], color[MAXN];
bool isBridge[MAXM];
vector<pair<int, int>> G[MAXN];
vector<int> bcc[MAXN];
void tarjan(int u, int inEdge) {
    dfn[u] = low[u] = ++ts;
    for (auto [v, id] : G[u]) {
        if (!dfn[v]) {
            tarjan(v, id);
            low[u] = min(low[u], low[v]);
            if (low[v] > dfn[u]) {
                isBridge[id] = true;
            }
        } else if (id != inEdge) {
            low[u] = min(low[u], dfn[v]);
        }
    }
}
void dfs(int u, int c) {
    color[u] = c;
    bcc[c].push_back(u);
    for (auto [v, id] : G[u]) {
        if (color[v]) continue;
        if (isBridge[id]) continue;
        dfs(v, c);
    }
}
int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        int u, v;
        cin >> u >> v;
        G[u].push_back({v, i});
        G[v].push_back({u, i});
    }
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) {
            tarjan(i, 0);
        }
    }
    for (int i = 1; i <= n; i++) {
        if (!color[i]) {
            bccCnt++;
            dfs(i, bccCnt);
        }
    }
    for (int i = 1; i <= bccCnt; i++) {
        for (int x : bcc[i]) {
            cout << x << " ";
        }
        cout << "\n";
    }
    return 0;
}
```


## 点双连通分量

### 求解思路

点双连通分量与割点关系密切。

对于 DFS 树中的边 `(u, v)`：

如果满足：

```cpp
low[v] >= dfn[u]
```

说明 `v` 的子树无法越过 `u` 回到更早的祖先。
此时，以 `u` 为分界，可以弹出一组边形成一个点双连通分量。

点双连通分量通常使用“边栈”维护，而不是直接使用点栈。


### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 10010;
const int MAXM = 200010;
int n, m, ts, dfn[MAXN], low[MAXN];
vector<pair<int, int>> G[MAXN];
stack<pair<int, int>> stk;
vector<vector<int>> dcc;
void tarjan(int u, int inEdge) {
    dfn[u] = low[u] = ++ts;
    for (auto [v, id] : G[u]) {
        if (!dfn[v]) {
            stk.push({u, v});
            tarjan(v, id);
            low[u] = min(low[u], low[v]);
            if (low[v] >= dfn[u]) {
                vector<int> comp;
                while (true) {
                    auto e = stk.top();
                    stk.pop();
                    comp.push_back(e.first);
                    comp.push_back(e.second);
                    if (e.first == u && e.second == v) {
                        break;
                    }
                }
                sort(comp.begin(), comp.end());
                comp.erase(unique(comp.begin(), comp.end()), comp.end());
                dcc.push_back(comp);
            }
        } else if (id != inEdge && dfn[v] < dfn[u]) {
            stk.push({u, v});
            low[u] = min(low[u], dfn[v]);
        }
    }
}
int main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        int u, v;
        cin >> u >> v;
        G[u].push_back({v, i});
        G[v].push_back({u, i});
    }
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) {
            if (G[i].empty()) {
                dcc.push_back({i});
            } else {
                tarjan(i, 0);
            }
        }
    }
    for (int i = 0; i < dcc.size(); i++) {
        for (int x : dcc[i]) {
            cout << x << " ";
        }
        cout << "\n";
    }
    return 0;
}
```

## 整体框架

Tarjan 算法虽然应用很多，但核心框架基本一致：

```cpp
void tarjan(int u) {
    dfn[u] = low[u] = ++timer;
    for (auto v : g[u]) {
        if (!dfn[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        } else {
            low[u] = min(low[u], dfn[v]);
        }
    }
    // 根据 dfn[u] 和 low[u] 的关系处理答案
}
```

不同问题的区别在于：

| 问题         | 关键判定                      |
| ------------ | ----------------------------- |
| 强连通分量   | `dfn[u] == low[u]`            |
| 割点         | `low[v] >= dfn[u]`            |
| 割边         | `low[v] > dfn[u]`             |
| 边双连通分量 | 删除所有割边后 DFS 染色       |
| 点双连通分量 | `low[v] >= dfn[u]` 时弹出边栈 |
| 缩点         | 将每个 SCC 看作一个新点       |


## 复杂度分析

Tarjan 算法的时间复杂度通常为：

```cpp
O(n + m)
```

空间复杂度通常为：

```cpp
O(n + m)
```

其中：

- `n` 表示点数。
- `m` 表示边数。

原因是 DFS 过程中每个点只会被访问一次，每条边也只会被遍历常数次。
