---
title: "线段树详解"
date: 2026-07-17 10:40:00
updated: 2026-06-21 10:40:00
tags:
  - "C++"
  - "算法"
categories:
  - "算法笔记"
description: "此文详细地介绍了线段树的核心思想与用途。"
cover: "/img/posts/20260719-224502-Segment-Tree/SegTreeHeadPicture.png"
top_img: "/img/posts/20260719-224502-Segment-Tree/SegTreeHeadPicture.png"
---

# 线段树

## 线段树的结构

线段树，顾名思义，即为线段构成的树结构，主要解决区间查询与修改以及类似的问题，可借助图片理解：

<img src="C:\Users\Administrator\Documents\Hexo-Butterfly-智能管理工具-v4\Hexo智能管理工具-v4\SegTreeHeadPicture.png" style="zoom:80%;" />

## 线段树的核心思想

线段树的核心思想是将一个连续区间递归地划分为若干个子区间，并以树形结构维护这些区间的信息。

整棵树的根节点表示完整区间，每个非叶子节点表示一个区间 `[L, R]`，其左右儿子分别维护 `[L, mid]` 与 `[mid + 1, R]` 两个子区间，其中 `mid = (L + R) / 2`。当区间长度为 `1` 时，该节点为叶子节点，对应原数组中的一个元素。

可借助上图理解。

## 主要操作

### 建树

建树是线段树的初始化过程，用于根据原数组构造整棵树。建树时从根节点开始递归处理区间 `[1, n]`。若当前区间左右端点相同，即 `L == R`，说明该节点是叶子节点，直接存储原数组中对应位置的值。否则，将当前区间从中点 `mid` 分成左右两个子区间，分别递归建立左子树与右子树。

当左右子树建立完成后，需要根据维护的信息进行合并，即用左右儿子的值更新当前节点。例如维护区间最小值时，有：

```cpp
mn[x] = min(mn[Lson], mn[Rson]);
```

维护区间和时，则通常写作：

```cpp
sum[x] = sum[Lson] + sum[Rson];
```

由于每个节点只会被访问一次，而线段树的节点数量为 `O(n)`，因此建树的时间复杂度为 `O(n)`，空间复杂度一般开到 `4n`，以保证数组式存储时不会越界。

### 修改

修改操作分为单点修改和区间修改。

单点修改指只修改数组中的某一个位置。操作时从根节点开始，根据目标位置 `P` 与当前区间中点 `mid` 的关系，递归进入左子树或右子树，直到到达对应的叶子节点。修改叶子节点后，在递归回溯的过程中重新合并父节点信息，从而保证整棵树的信息仍然正确。单点修改的时间复杂度为 `O(log n)`。

区间修改指对一个连续区间 `[L, R]` 内的所有元素进行统一修改，例如整体加上某个值。若当前节点维护的区间与修改区间没有交集，直接返回；若当前区间被修改区间完全覆盖，则直接更新当前节点信息，并给该节点打上懒标记；若二者部分相交，则需要先下传已有标记，再递归修改左右子树，最后重新合并当前节点信息。

区间修改配合懒标记后，不需要访问区间内的每一个元素，而是以整段区间为单位进行更新，因此时间复杂度通常为 `O(log n)`。

### 查询

查询操作用于获取某个区间 `[L, R]` 的统计信息，例如区间和、区间最小值、区间最大值等。

查询时从根节点开始递归判断当前节点区间与目标查询区间的关系。若当前区间与查询区间没有交集，则返回一个对答案无影响的单位值。例如查询最小值时返回正无穷，查询最大值时返回负无穷，查询区间和时返回 `0`。若当前区间被查询区间完全包含，则直接返回当前节点存储的信息。若当前区间与查询区间部分相交，则递归查询左右子树，并将左右子树的查询结果进行合并。

在线段树中，一次区间查询只会访问与目标区间相关的少量节点，因此时间复杂度通常为 `O(log n)`。在含有懒标记的线段树中，查询过程中若需要继续访问子节点，应先执行 `PushDown`，确保子节点信息正确。

### PushDown

`PushDown`，即标记下传，是懒标记线段树中的关键操作。它的作用是将当前节点上尚未传递给子节点的修改信息下放到左右儿子，从而保证在后续访问子区间时，子节点维护的信息是正确的。

以区间加法修改和区间最小值查询为例，若当前节点 `x` 上存在懒标记 `tag[x]`，表示该节点所维护的整个区间都已经加上了 `tag[x]`，但该修改还没有传递给左右子节点。当需要继续递归访问左右儿子时，就要将该标记分别加到左右儿子的 `tag` 上，同时更新左右儿子的维护值：

```cpp
tag[Lson] += tag[x];
tag[Rson] += tag[x];
mn[Lson] += tag[x];
mn[Rson] += tag[x];
tag[x] = 0;
```

下传完成后，需要清空当前节点的懒标记，表示该节点的延迟修改已经被传递。通过这种方式，线段树避免了大量无意义的递归修改，使区间修改能够保持高效。

需要注意的是，`PushDown` 的具体写法与维护的信息和修改方式有关。若维护的是区间和，区间加法修改时更新子节点的值时还需要乘上子区间长度；若维护的是区间最小值或最大值，则区间整体加法只需直接加上修改值即可。

## 例题

### 单点更新

**题意：**

给出一个数列，有两种操作：

1. `1 id v`：将数列中`id`位置的数加`v`。
2. `2 l r` ：求数列中`[l, r]`区间的最小值。

回答所有询问。

**思路：**

板子题。

**代码：**

```cpp
#include<bits/stdc++.h>
using namespace std;
#define Lson (2 * x)
#define Rson (2 * x + 1)
struct SegmentTree {
    vector<int> sum;
    int* a;
    SegmentTree(int NN, int* A) {
        sum.resize(4 * NN);
        fill(sum.begin(), sum.end(), 0x3f3f3f3f);
        a = A;
    }
    void build(int x, int L, int R) {
        if (L == R) {
            sum[x] = a[L];
            return;
        }
        int mid = (L + R) / 2;
        build(Lson, L, mid);
        build(Rson, mid + 1, R);
        sum[x] = min(sum[Lson], sum[Rson]);
    }
    void modify(int x, int L, int R, int P, int V) {
        if (L == R) {
            sum[x] += V;
            return;
        }
        int mid = (L + R) / 2;
        if (P <= mid) {
            modify(Lson, L, mid, P, V);
        } else {
            modify(Rson, mid + 1, R, P, V);
        }
        sum[x] = min(sum[Lson], sum[Rson]);
    }
    int query(int x, int L, int R, int QL, int QR) {
        if (QL <= L && R <= QR) return sum[x];
        int Result = 0x3f3f3f3f;
        int mid = (L + R) / 2;
        if (QL <= mid) Result = min(Result, query(Lson, L, mid, QL, QR));
        if (QR > mid)  Result = min(Result, query(Rson, mid + 1, R, QL, QR));
        return Result;
    }
};
int a[1000010];
int main() {
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    SegmentTree Seg(n, a);
    Seg.build(1, 1, n);
    int m;
    cin >> m;
    while (m--) {
        int op, x, y;
        cin >> op >> x >> y;
        if (op == 1) {
            x++;
            Seg.modify(1, 1, n, x, y);
        } else {
            x++, y++;
            cout << Seg.query(1, 1, n, x, y) << "\n";
        }
    }
    return 0;
}
```

### 区间更新

**题意：**

给出一个数列，有两种操作：

1. `1 l r v`：将数列中`[l, r]`区间的所有数加`v`。
2. `2 l r` ：求数列中`[l, r]`区间的最小值。

回答所有询问。

**思路：**

板子，注意是区间查询。

**代码：**

```cpp
#include<bits/stdc++.h>
#define Lson 2 * x
#define Rson 2 * x + 1
#define int long long
using namespace std;
const int INF = 0x3f3f3f3f3f3f3f3f;
int n, m, a[100010], mn[1000010], tag[1000010];
void build(int x, int l, int r) {
    if (l == r) {
        mn[x] = a[l];
        return ;
    }
    int mid = (l + r) / 2;
    build(Lson, l, mid);
    build(Rson, mid + 1, r);
    mn[x] = min(mn[Lson], mn[Rson]);
}
void pushdown(int x, int l, int r) {
    if (tag[x] == 0) return;
    int mid = (l + r) / 2;
    tag[Lson] += tag[x];
    tag[Rson] += tag[x]; 
    mn[Lson] += tag[x];
    mn[Rson] += tag[x];
    tag[x] = 0;
}
void modify(int L, int R, int num, int x, int l, int r) {
    if (R < l || L > r) return ;
    if (L <= l && r <= R) {
        mn[x] += num;
        tag[x] += num;
        return ;
    }
    pushdown(x, l, r);
    int mid = (l + r) / 2;
    modify(L, R, num, Lson, l, mid);
    modify(L, R, num, Rson, mid + 1, r);
    mn[x] = min(mn[Lson], mn[Rson]);
}
int query(int L, int R, int x, int l, int r) {
    if (r < L || l > R) return INF;
    pushdown(x, l, r);
    if (L <= l && r <= R) return mn[x];
    int mid = (l + r) / 2;
    return min(query(L, R, Lson, l, mid), query(L, R, Rson, mid + 1, r));
}
signed main() {
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> a[i];
    build(1, 1, n);
    cin >> m;
    for (int i = 1; i <= m; i++) {
        int op, x, y, k;
        cin >> op >> x >> y;
        x++, y++;
        if (op == 2) {
            cin >> k;
            modify(x, y, k, 1, 1, n);
        } else {
            cout << query(x, y, 1, 1, n) << "\n";
        }
    }
    return 0;
}
```

### 区间求和

**题意：**

给出一个数列，有两种操作：

1. `1 l r v`：将数列中`[l, r]`区间的所有数加`v`。
2. `2 l r` ：求数列中`[l, r]`区间的和。

回答所有询问。

**思路：**

把求最小值换成了求和，注意要改掉所有的求最值部分以及INF部分，INF是求最值时用的，求和时会输出巨大的正/负数。

**代码：**

```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
int n, m, a[100010], sum[1000010], tag[1000010];
void build(int x, int l, int r) {
    if (l == r) {
        sum[x] = a[l];
        return ;
    }
    int mid = (l + r) / 2;
    build(2 * x, l, mid);
    build(2 * x + 1, mid + 1, r);
    sum[x] = sum[2 * x] + sum[2 * x + 1];
}
void pushdown(int x, int l, int r) {
    int mid = (l + r) / 2;
    tag[2 * x] += tag[x];
    tag[2 * x + 1] += tag[x]; 
    sum[2 * x] += tag[x] * (mid - l + 1);
    sum[2 * x + 1] += tag[x] * (r - mid);
    tag[x] = 0;
}
void modify(int L, int R, int num, int x, int l, int r) {
    if (R < l || L > r) return ;
    if (L <= l && r <= R) {
        sum[x] += num * (r - l + 1);
        tag[x] += num;
        return ;
    }
    pushdown(x, l, r);
    int mid = (l + r) / 2;
    modify(L, R, num, 2 * x, l, mid);
    modify(L, R, num, 2 * x + 1, mid + 1, r);
    sum[x] = sum[2 * x] + sum[2 * x + 1];
}
int query(int L, int R, int x, int l, int r) {
    if (r < L || l > R) return 0;
    pushdown(x, l, r);
    if (L <= l && r <= R) return sum[x];
    int mid = (l + r) / 2;
    return query(L, R, 2 * x, l, mid) + query(L, R, 2 * x + 1, mid + 1, r);
}
signed main() {
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> a[i];
    build(1, 1, n);
    cin >> m;
    for (int i = 1; i <= m; i++) {
        char op;
        int x, y, k;
        cin >> op >> x >> y;
        if (op == 'C') {
            cin >> k;
            modify(x, y, k, 1, 1, n);
        } else {
            cout << query(x, y, 1, 1, n) << "\n";
        }
    }
    return 0;
}
```

### 苹果树

**题意：**

有一棵苹果树，苹果会长在某些节点上，有两种操作：

1. 将某个节点的苹果状态翻转；
2. 询问某个节点的子树内的苹果数量。

回答所有询问。

**思路：**

求出dfs序作为区间然后做单点修改区间查询即可。

也可以用树状数组。

**代码：**

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, m, tin[1000010], tout[1000010], tim, pg[1000010];
vector<int> g[1000010];
struct SegmentTree {
    int tree[1000010];
    void build(int x, int l, int r) {
        if (l == r) {
            tree[x] = 1;
            return;
        }
        int mid = (l + r) / 2;
        build(x * 2, l, mid);
        build(x * 2 + 1, mid + 1, r);
        tree[x] = tree[x * 2] + tree[x * 2 + 1];
    }
    void modify(int x, int l, int r, int pos, int val) {
        if (l == r) {
            tree[x] = val;
            return;
        }
        int mid = (l + r) / 2;
        if (pos <= mid) {
            modify(x * 2, l, mid, pos, val);
        } else {
            modify(x * 2 + 1, mid + 1, r, pos, val);
        }
        tree[x] = tree[x * 2] + tree[x * 2 + 1];
    }
    int query(int x, int l, int r, int ql, int qr) {
        if (ql <= l && r <= qr) {
            return tree[x];
        }
        int mid = (l + r) / 2;
        int ans = 0;
        if (ql <= mid) {
            ans += query(x * 2, l, mid, ql, qr);
        }
        if (qr > mid) {
            ans += query(x * 2 + 1, mid + 1, r, ql, qr);
        }
        return ans;
    }
} seg;
void dfs(int u, int fa) {
    tin[u] = ++tim;
    for (int v : g[u]) {
        if (v == fa) continue;
        dfs(v, u);
    }
    tout[u] = tim;
}
int main() {
    cin >> n;
    for (int i = 1; i <= n - 1; i++) {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }
    dfs(1, 0);
    for (int i = 1; i <= n; i++) {
        pg[i] = 1;
    }
    seg.build(1, 1, n);
    cin >> m;
    while (m--) {
        char op;
        int x;
        cin >> op >> x;
        if (op == 'C') {
            pg[x] ^= 1;
            seg.modify(1, 1, n, tin[x], pg[x]);
        } else if (op == 'Q') {
            cout << seg.query(1, 1, n, tin[x], tout[x]) << '\n';
        }
    }
    return 0;
}
```

### 最大子段和

**题意：**

给定一个数列，给出 $M$ 次查询，每次查询`x y`询问数列中 $[x,y]$ 区间的最大子段和，回答所有询问。

**思路：**

首先这道题没有修改。

其次发现答案只能是左区间的答案、右区间的答案以及跨中点区间的答案，跨中点的区间可以视为左区间的后缀与右区间的前缀，于是再预处理这两个值即可。

**代码：**

```cpp
#include <bits/stdc++.h>
using namespace std;
int n, m, a[1000010];
struct Node {
    int sum, lmx, rmx, ans;
};
Node tree[1000010];
Node merge(Node left, Node right) {
    Node res;
    res.sum = left.sum + right.sum;
    res.lmx = max(left.lmx, left.sum + right.lmx);
    res.rmx = max(right.rmx, right.sum + left.rmx);
    res.ans = max({left.ans, right.ans, left.rmx + right.lmx});
    return res;
}
void build(int x, int l, int r) {
    if (l == r) {
        tree[x].sum = a[l];
        tree[x].lmx = a[l];
        tree[x].rmx = a[l];
        tree[x].ans = a[l];
        return;
    }
    int mid = (l + r) / 2;
    build(x * 2, l, mid);
    build(x * 2 + 1, mid + 1, r);
    tree[x] = merge(tree[x * 2], tree[x * 2 + 1]);
}
Node query(int x, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) {
        return tree[x];
    }
    int mid = (l + r) / 2;
    if (qr <= mid) {
        return query(x * 2, l, mid, ql, qr);
    }
    if (ql > mid) {
        return query(x * 2 + 1, mid + 1, r, ql, qr);
    }
    return merge(query(x * 2, l, mid, ql, qr), query(x * 2 + 1, mid + 1, r, ql, qr));
}
int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    build(1, 1, n);
    cin >> m;
    while (m--) {
        int x, y;
        cin >> x >> y;
        cout << query(1, 1, n, x, y).ans << '\n';
    }
    return 0;
}
```

### 开根号求和

**题意：**

给定一个长度为 `N` 的正整数序列，需要支持 `M` 次操作：

1. `0 x y`：将区间 `[x, y]` 中的每个数都变成其平方根向下取整，即 `A[i] = floor(sqrt(A[i]))`。
2. `1 x y`：查询区间 `[x, y]` 的元素和。

对于每次查询操作，输出对应区间和。

**思路：**

开根号打不了懒标记，发现一个数开根10次以内就是1了，所以记录一个`sum`和`mx`，暴力合并即可（`mx`为1时直接返回）。

**代码：**

```cpp
#include <bits/stdc++.h>
#define Lson 2 * x
#define Rson 2 * x + 1
#define int long long
using namespace std;
int n, m, a[1000010], sum[1000010], mx[1000010];
int isqrt(int v) {
    int r = sqrtl((long double)v);
    while ((__int128)(r + 1) * (r + 1) <= v) r++;
    while ((__int128)r * r > v) r--;
    return r;
}
void pushup(int x) {
    sum[x] = sum[Lson] + sum[Rson];
    mx[x] = max(mx[Lson], mx[Rson]);
}
void build(int x, int l, int r) {
    if (l == r) {
        sum[x] = a[l];
        mx[x] = a[l];
        return;
    }
    int mid = (l + r) / 2;
    build(Lson, l, mid);
    build(Rson, mid + 1, r);
    pushup(x);
}
void modify(int L, int R, int x, int l, int r) {
    if (R < l || L > r) return;
    if (mx[x] == 1) return;
    if (l == r) {
        int v = isqrt(sum[x]);
        sum[x] = v;
        mx[x] = v;
        return;
    }
    int mid = (l + r) / 2;
    if (L <= mid) modify(L, R, Lson, l, mid);
    if (R > mid) modify(L, R, Rson, mid + 1, r);
    pushup(x);
}
int query(int L, int R, int x, int l, int r) {
    if (R < l || L > r) return 0;
    if (L <= l && r <= R) {
        return sum[x];
    }
    int mid = (l + r) / 2;
    int ans = 0;
    if (L <= mid) ans += query(L, R, Lson, l, mid);
    if (R > mid) ans += query(L, R, Rson, mid + 1, r);
    return ans;
}
signed main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    build(1, 1, n);
    cin >> m;
    while (m--) {
        int op, x, y;
        cin >> op >> x >> y;
        if (op == 0) {
            modify(x, y, 1, 1, n);
        } else {
            cout << query(x, y, 1, 1, n) << '\n';
        }
    }
    return 0;
}
```

### 队列查询

**题意：**

维护一个队列，支持三种操作：

1. `in x`：将整数 `x` 加入队尾；
2. `out`：弹出队首元素；
3. `query`：查询当前队列中的中位数。

若当前队列中有 `m` 个数，将它们升序排序后，中位数定义为第 `floor(m / 2) + 1` 个数。

输入保证所有加入过的 `x` 都互不相同，且队列为空时不会执行 `out` 或 `query`。

**思路：**

这道题既有队列的先进先出，又有动态查询中位数。我们可以用队列负责维护元素进入和弹出的顺序，线段树负责维护当前队列中有哪些数，并查询第 `k` 小（第`floor(m / 2) + 1`小即为中位数）。

由于 `x` 的范围很大，达到 `10^9`，所以要离散化。

离散化之后，每个数都有一个排名。在线段树中维护每个排名当前是否存在于队列中：

* `in x`：将 `x` 的排名位置加 `1`，并把它放入普通队列中；
* `out`：从普通队列中取出队首元素，将其在线段树中的位置减 `1`；
* `query`：设当前队列大小为 `m`，需要查询第 `floor(m / 2) + 1` 小的数。

线段树每个节点维护该区间中当前存在的数字个数。查询第 `k` 小时，从根节点开始：

* 如果左子树数量大于等于 `k`，说明答案在左子树；
* 否则答案在右子树，并令 `k` 减去左子树数量。

最后到达叶子节点，该叶子对应的值就是中位数。

**代码：**

```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
struct node {
	int l, r;
	int cnt;
} tr[4000010];
int a[1000010];
string s[1000010];
int b[1000010], len, t[1000010];
void pushup(int i) {
	tr[i].cnt = tr[i * 2].cnt + tr[i * 2 + 1].cnt;
}
void build(int i, int l, int r) {
	tr[i] = {l, r, 0};
	if (l == r) return;
	build(i * 2, l, (l + r) >> 1);
	build(i * 2 + 1, ((l + r) >> 1) + 1, r);
}
void modify(int i, int l, int k) {
	if (tr[i].l == tr[i].r) {
		tr[i].cnt += k;
		return;
	}
	int mid = tr[i].l + tr[i].r >> 1;
	if (l <= mid) modify(i * 2, l, k);
	else modify(i * 2 + 1, l, k);
	pushup(i);
}
int query(int i, int k) {
	if (tr[i].l == tr[i].r) return tr[i].l;
	if (tr[i * 2].cnt >= k) return query(i * 2, k);
	return query(i * 2 + 1, k - tr[i * 2].cnt);
}
signed main() {
	int n;
    cin >> n;
	for (int i = 1; i <= n; i++) {
		cin >> s[i];
		if (s[i] == "in") {
            cin >> b[++len];
			t[len] = b[len];
		}
	}
	sort(t + 1, t + 1 + len);
	for (int i = 1; i <= len; i++) {
		b[i] = lower_bound(t + 1, t + 1 + len, b[i]) - t;
	}
	build(1, 1, len);
	len = 0;
	queue<int>q;
	for (int i = 1; i <= n; i++) {
		if (s[i] == "in") {
			modify(1, b[++len], 1);
			q.push(b[len]);
		} else if (s[i] == "out") {
			modify(1, q.front(), -1);
			q.pop();
		} else {
			cout << t[query(1, q.size() / 2 + 1)] << "\n";
		}
	}
	return 0;
}
```

### 局部排序

**题意：**

给定一个 `1` 到 `n` 的全排列，需要进行 `m` 次局部排序操作：

1. `0 l r`：将区间 `[l, r]` 升序排序；
2. `1 l r`：将区间 `[l, r]` 降序排序。

所有操作完成后，询问第 `q` 个位置上的数字。

**思路：**

暴力排序复杂度爆炸。由于只询问一个位置的值，考虑二分答案。

假设需要判断第 `q` 个位置上的数字是否大于等于 `mid`，那么可以把原数组转化成一个 `01` 数组：

$$
b_i=[a_i\ge \operatorname{mid}]
$$
这样就转化成了对 `01` 数组的局部排序。然而，对于一个 `01` 区间来说，升序排序后一定是前面全是 `0`，后面全是 `1`；降序排序后一定是前面全是 `1`，后面全是 `0`。因此，对于每次排序操作，只需要在线段树中查询区间 `[l, r]` 中有多少个 `1`，记为 `cnt`，这个`cnt`可以用线段树的区间求和做，然后排序就相当于把`cnt`个1放前面/后面，直接用线段树的区间修改即可。

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;
int n, m, a[100010], q, l = 1, r, cnt[400010], lan[400010], ans;
struct Data {
  int opt, l, r;
  void In() {cin >> opt >> l >> r;}
} dat[100010];
void pd(int l, int r, int p) {
  int mid = (l + r) / 2;
  if (~lan[p] && l != r) {
		cnt[2 * p] = (mid - l + 1) * lan[p], lan[2 * p] = lan[p],
		cnt[2 * p + 1] = (r - mid) * lan[p], lan[2 * p + 1] = lan[p], lan[p] = -1;
  }
}
void bd(int l, int r, int p, int v) {
  lan[p] = -1;
  if (l == r) {
		cnt[p] = (a[l] >= v);
		return;
  }
  int mid = (l + r) / 2;
  bd(l, mid, 2 * p, v), bd(mid + 1, r, 2 * p + 1, v);
  cnt[p] = cnt[2 * p] + cnt[2 * p + 1];
}
void upd(int l, int r, int p, int ql, int qr, int v) {
  if (ql > qr) {
		return;
  }
  if (l >= ql && r <= qr) {
		cnt[p] = (r - l + 1) * v, lan[p] = v;
		return;
  }
  pd(l, r, p);
  int mid = (l + r) / 2;
  if (ql <= mid) {
		upd(l, mid, 2 * p, ql, qr, v);
  }
  if (qr > mid) {
		upd(mid + 1, r, 2 * p + 1, ql, qr, v);
  }
  cnt[p] = cnt[2 * p] + cnt[2 * p + 1];
}
int Q(int l, int r, int p, int ql, int qr) {
  if (l >= ql && r <= qr) {
		return cnt[p];
  }
  pd(l, r, p);
  int mid = (l + r) / 2, ans = 0;
  if (ql <= mid) {
		ans += Q(l, mid, 2 * p, ql, qr);
  }
  if (qr > mid) {
		ans += Q(mid + 1, r, 2 * p + 1, ql, qr);
  }
  return ans;
}
signed main() {
  cin >> n >> m;
  for (int i = 1; i <= n; i++) cin >> a[i];
  for (int i = 1; i <= m; i++) dat[i].In();
  cin >> q;
  r = n;
  while (l <= r) {
		int mid = (l + r) / 2;
		bd(1, n, 1, mid);
		for (int i = 1; i <= m; i++) {
			int cntt = Q(1, n, 1, dat[i].l, dat[i].r);
			if (!dat[i].opt) {
			upd(1, n, 1, dat[i].r - cntt + 1, dat[i].r, 1),
				upd(1, n, 1, dat[i].l, dat[i].r - cntt, 0);
			} else {
			upd(1, n, 1, dat[i].l, dat[i].l + cntt - 1, 1);
			upd(1, n, 1, dat[i].l + cntt, dat[i].r, 0);
			}
		}
		if (Q(1, n, 1, q, q)) l = mid + 1, ans = mid;
		else r = mid - 1;
  }
  cout << ans;
  return 0;
}
```

### DFS序线段树

**题意：**

给定一棵以 `1` 为根的树，每个节点有一个点权。需要支持三种操作：

1. 将节点 `x` 的点权增加 `a`；
2. 将以节点 `x` 为根的子树中所有节点的点权都增加 `a`；
3. 查询节点 `x` 到根节点路径上所有点的点权和。

**思路：**

这道题的查询是“根到某个点的路径和”，修改却分为单点修改和子树修改。可以先用 DFS 序把子树转化为连续区间。

设：

* `dfn[x]` 表示节点 `x` 的 DFS 序；
* `siz[x]` 表示节点 `x` 的子树大小；
* 那么节点 `x` 的子树对应区间为 `[dfn[x], dfn[x] + siz[x] - 1]`；
* `dep[x]` 表示节点深度。

先考虑第 `3` 种询问。对于一个节点 `u`，它到根的路径和可以记为 `ans[u]`。如果能维护所有节点当前的 `ans[u]`，那么查询节点 `x` 时，只需要查询 `dfn[x]` 位置的值。

接下来分析修改对 `ans[u]` 的影响。

对于操作 `1 x a`，只修改节点 `x` 自身的点权。只有 `x` 的子树中的节点，它们到根的路径才会经过 `x`。因此，所有 `u` 属于 `x` 的子树时，`ans[u]` 都会增加 `a`。

所以操作 `1` 可以转化为：对子树区间 `[dfn[x], dfn[x] + siz[x] - 1]` 进行区间加 `a`。

对于操作 `2 x a`，会把 `x` 子树中每个点的权值都增加 `a`。对于 `x` 的某个子孙节点 `u`，它从根到 `u` 的路径上，属于 `x` 子树的点是从 `x` 到 `u` 的这一段，共有`dep[u] - dep[x] + 1`个点。

所以 `ans[u]` 会增加`a * (dep[u] - dep[x] + 1)`，可以变形为：`a * dep[u] + a * (1 - dep[x])`

也就是说，对于 `x` 子树中的每个节点 `u`，要加上的值是一个关于 `dep[u]` 的一次式。

因此，线段树可以维护区间加的两类懒标记：

* 一个系数标记 `k`，表示要加上 `k * dep[u]`；
* 一个常数标记 `b`，表示要加上 `b`。

对于操作 `2 x a`，就在 `x` 的子树区间上增加：

* 系数 `a`；
* 常数 `a * (1 - dep[x])`。

对于操作 `1 x a`，可以看成只加常数：

* 系数 `0`；
* 常数 `a`。

**代码：**

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 200010;
struct node {
	long long l, r;
	long long sum, tag;
	int zheng, fu;
} tr[N << 2];
vector<int>e[N];
long long chu[N], in[N], out[N], len, zheng[N];
long long a[200010], sum[200010];
void pushup(int i) {
	tr[i].sum = tr[i * 2].sum + tr[i * 2 + 1].sum;
}
void build(int i, int l, int r) {
	tr[i] = {l, r, 0, 0, 0, 0};
	if (l == r) {
		tr[i].sum = a[l];
		return;
	}
	int mid = l + r >> 1;
	build(i * 2, l, mid);
	build(i * 2 + 1, mid + 1, r);
	pushup(i);
}
void pushdown(int i) {
	node &fa = tr[i], &ls = tr[i * 2], &rs = tr[i * 2 + 1];
	if (fa.tag == 0) return;
	ls.sum += 1ll * (sum[ls.r] - sum[ls.l - 1]) * fa.tag;
	ls.tag += fa.tag;
	rs.sum += 1ll * (sum[rs.r] - sum[rs.l - 1]) * fa.tag;
	rs.tag += 1ll * fa.tag;
	fa.tag = 0;
}
void modify(int i, int l, int r, int k) {
	if (tr[i].l >= l && tr[i].r <= r) {
		tr[i].sum += (sum[tr[i].r] - sum[tr[i].l - 1]) * k;
		tr[i].tag += k;
		return;
	}
	pushdown(i);
	int l1 = tr[i].l, r2 = tr[i].r;
	int mid = l1 + r2 >> 1;
	if (l <= mid) modify(i * 2, l, r, k);
	if (r > mid) modify(i * 2 + 1, l, r, k);
	pushup(i);
}
long long qurey(int i, int l, int r) {
	if (tr[i].l >= l && tr[i].r <= r) return tr[i].sum;
	pushdown(i);
	long long ans = 0;
	int mid = tr[i].l + tr[i].r >> 1;
	if (l <= mid) ans += qurey(i * 2, l, r);
	if (r > mid) ans += qurey(i * 2 + 1, l, r);
	return ans;
}

void dfs(int u, int fa) {
	in[u] = ++len;
	a[len] = chu[u];
	sum[len] = sum[len - 1] + 1;
	for (auto v : e[u]) {
		if (v != fa) dfs(v, u);
	}
	out[u] = ++len;
	a[len] = -chu[u];
	sum[len] = sum[len - 1] - 1;
}
signed main() {
	int n;
	cin >> n;
	int m;
	cin >> m;
	for (int i = 1; i <= n; i++) cin >> chu[i];
	for (int i = 1; i <= n - 1; i++) {
		int a, b;
		cin >> a >> b;
		e[a].push_back(b);
		e[b].push_back(a);
	}
	dfs(1, 0);
	build(1, 1, len);
	while (m--) {
		int op;
		cin >> op;
		if (op == 1) {
			int x, a;
			cin >> x >> a;
			modify(1, in[x], in[x], a);
			modify(1, out[x], out[x], a);
		} else if (op == 2) {
			int x, a;
			cin >> x >> a;
			modify(1, in[x], out[x], a);
		} else {
			int x;
			cin >> x;
			cout << qurey(1, 1, in[x]) << '\n';
		}
	}
	return 0;
}
```

### 下标过滤和

**题意：**

维护一个集合 `S`，初始为空，支持三种操作：

1. `add x`：向集合中加入数字 `x`；
2. `del x`：从集合中删除数字 `x`；
3. `sum`：将当前集合中的数字升序排序为 `a1 < a2 < ... < ak`，求所有满足 `i mod 5 = 3` 的 `a[i]` 之和。

对于每次 `sum` 操作，输出答案。

**思路：**

由于 `x` 的范围很大，先离散化。

然后在线段树上维护这些值是否存在。

关键在于线段树节点需要维护的不只是区间内有多少个数，还要维护这些数按照升序排列后，不同下标模 `5` 的元素和。

对于每个节点，维护：

* `cnt`：该区间内当前存在的数字个数；
* `sum[0]` 到 `sum[4]`：该区间内的数按升序排列后，下标模 `5` 分别为 `0,1,2,3,4` 的元素和。

设左儿子有 `cnt` 个数，那么右儿子中的每个数在合并后的排名都会整体向后移动 `cnt` 位。

因此，合并时：

* 左儿子的各类 `sum` 可以直接加入父节点；
* 右儿子的下标模数需要整体平移 `cnt mod 5`。

对于 `add x`，在线段树中把对应叶子改为存在，此时该叶子 `cnt = 1`，并且它作为区间内第 `1` 个数，应加入下标模 `5` 为 `1` 的那一类。

对于 `del x`，把对应叶子清空即可。

对于 `sum`，答案就是整棵线段树根节点中 `sum[3]` 的值。

**代码：**

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 200010;
struct node {
	long long l, r;
	long long sum, tag;
	int zheng, fu;
} tr[N << 2];
vector<int>e[N];
long long chu[N], in[N], out[N], len, zheng[N];
long long a[200010], sum[200010];
void pushup(int i) {
	tr[i].sum = tr[i * 2].sum + tr[i * 2 + 1].sum;
}
void build(int i, int l, int r) {
	tr[i] = {l, r, 0, 0, 0, 0};
	if (l == r) {
		tr[i].sum = a[l];
		return;
	}
	int mid = l + r >> 1;
	build(i * 2, l, mid);
	build(i * 2 + 1, mid + 1, r);
	pushup(i);
}
void pushdown(int i) {
	node &fa = tr[i], &ls = tr[i * 2], &rs = tr[i * 2 + 1];
	if (fa.tag == 0) return;
	ls.sum += 1ll * (sum[ls.r] - sum[ls.l - 1]) * fa.tag;
	ls.tag += fa.tag;
	rs.sum += 1ll * (sum[rs.r] - sum[rs.l - 1]) * fa.tag;
	rs.tag += 1ll * fa.tag;
	fa.tag = 0;
}
void modify(int i, int l, int r, int k) {
	if (tr[i].l >= l && tr[i].r <= r) {
		tr[i].sum += (sum[tr[i].r] - sum[tr[i].l - 1]) * k;
		tr[i].tag += k;
		return;
	}
	pushdown(i);
	int l1 = tr[i].l, r2 = tr[i].r;
	int mid = l1 + r2 >> 1;
	if (l <= mid) modify(i * 2, l, r, k);
	if (r > mid) modify(i * 2 + 1, l, r, k);
	pushup(i);
}
long long qurey(int i, int l, int r) {
	if (tr[i].l >= l && tr[i].r <= r) return tr[i].sum;
	pushdown(i);
	long long ans = 0;
	int mid = tr[i].l + tr[i].r >> 1;
	if (l <= mid) ans += qurey(i * 2, l, r);
	if (r > mid) ans += qurey(i * 2 + 1, l, r);
	return ans;
}

void dfs(int u, int fa) {
	in[u] = ++len;
	a[len] = chu[u];
	sum[len] = sum[len - 1] + 1;
	for (auto v : e[u]) {
		if (v != fa) dfs(v, u);
	}
	out[u] = ++len;
	a[len] = -chu[u];
	sum[len] = sum[len - 1] - 1;
}
signed main() {
	int n;
	cin >> n;
	int m;
	cin >> m;
	for (int i = 1; i <= n; i++) cin >> chu[i];
	for (int i = 1; i <= n - 1; i++) {
		int a, b;
		cin >> a >> b;
		e[a].push_back(b);
		e[b].push_back(a);
	}
	dfs(1, 0);
	build(1, 1, len);
	while (m--) {
		int op;
		cin >> op;
		if (op == 1) {
			int x, a;
			cin >> x >> a;
			modify(1, in[x], in[x], a);
			modify(1, out[x], out[x], a);
		} else if (op == 2) {
			int x, a;
			cin >> x >> a;
			modify(1, in[x], out[x], a);
		} else {
			int x;
			cin >> x;
			cout << qurey(1, 1, in[x]) << '\n';
		}
	}
	return 0;
}
```