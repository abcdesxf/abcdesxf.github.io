---
title: '课程学习笔记：字符串'
date: 2026-07-28 22:11:51
updated: 2026-07-28 22:11:51
tags:
  - 'C++'
  - '算法'
categories:
  - '课程学习笔记'
description: ''
---

# 课程学习笔记+算法介绍：字符串基础算法

## A. T恤分配

见 [另一篇文章]([课程学习笔记 - 二分图匹配 | gjr's Blog](https://abcdesxf.github.io/2026/07/26/课程学习笔记 - 二分图匹配/)) 。

## B. 单词博弈

将所有单词插入 Trie，游戏过程就是从根节点不断走向子节点。设 `w[u]` 表示当前玩家从节点 `u` 出发能否获胜，`l[u]` 表示当前玩家能否让自己最终失败。叶子节点无法继续操作，因此 `w=false，l=true`；其余节点只要存在一个儿子满足 `w=false` 就能获胜，只要存在一个儿子满足 `l=false` 就能让自己失败。

若根节点 `w=false`，先手第一局必败，最终也是后手获胜；若 `w=true、l=true`，先手可以控制胜负，因此一定获胜；否则每局胜负固定交替，答案取决于 `k` 的奇偶性。

Trie 的节点数不超过所有字符串的长度之和，时间复杂度为 $O(\sum |S|)$，空间复杂度为 $O(\sum |S|)$。

代码：
```cpp
#include <bits/stdc++.h>
using namespace std;
struct Node {
    int nxt[26];
    bool win, lose;
    Node() : win(false), lose(false) {
        fill(nxt, nxt + 26, -1);
    }
};
int n, k;
vector<Node> trie(1);
int main() {
    cin >> n >> k;
    while (n--) {
        string s;
        cin >> s;
        int u = 0;
        for (char ch : s) {
            int c = ch - 'a';
            if (trie[u].nxt[c] == -1) {
                trie[u].nxt[c] = trie.size();
                trie.emplace_back();
            }
            u = trie[u].nxt[c];
        }
    }
    for (int u = trie.size() - 1; u >= 0; u--) {
        bool flg = true;
        for (int v : trie[u].nxt) {
            if (v == -1) continue;
            flg = false;
            if (!trie[v].win) trie[u].win = true;
            if (!trie[v].lose) trie[u].lose = true;
        }
        if (flg) trie[u].lose = true;
    }
    if (!trie[0].win) cout << "Second\n";
    else if (trie[0].lose) cout << "First\n";
    else cout << (k % 2 ? "First\n" : "Second\n");
    return 0;
}
```

## C. 相似字符串计数

两个字符串相似，当且仅当将同一位置的字符忽略后，剩余字符完全相同。由于题目保证不存在两个相同字符串，因此不需要额外判断被忽略的字符是否不同。

先计算每个字符串的双模哈希。枚举被忽略的位置，从完整哈希中减去该字符的贡献，再用哈希表统计相同结果。新字符串对答案的贡献就是该哈希此前的出现次数。代码使用开放寻址哈希表，并通过时间戳避免每次枚举位置时清空整个数组。

需要处理 $N\times L$ 个哈希值，平均时间复杂度为 $O(NL)$，空间复杂度为 $O(N+L)$。

代码：
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
using ull = unsigned long long;
const int N = 30010, L = 210, H = 131072;
const int P = 1000000007, Q = 1000000009, B = 911382323;
string s[N];
ll a[N], b[N], p[L], q[L], ans;
ull ky[H];
int n, m, ct[H], vs[H], tim;
int add(ull x) {
    int u = (x ^ x >> 23 ^ x >> 47) & (H - 1);
    while (vs[u] == tim && ky[u] != x) u = (u + 1) & (H - 1);
    if (vs[u] != tim) vs[u] = tim, ky[u] = x, ct[u] = 0;
    return ct[u]++;
}
int main() {
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);
    cin >> n >> m;
    p[0] = q[0] = 1;
    for (int i = 1; i < m; i++) p[i] = p[i - 1] * B % P, q[i] = q[i - 1] * B % Q;
    for (int i = 0; i < n; i++) {
        cin >> s[i];
        for (auto c : s[i]) {
            int x = (unsigned char)c + 1;
            a[i] = (a[i] * B + x) % P;
            b[i] = (b[i] * B + x) % Q;
        }
    }
    for (int j = 0; j < m; j++) {
        tim++;
        int e = m - j - 1;
        for (int i = 0; i < n; i++) {
            int x = (unsigned char)s[i][j] + 1;
            ll u = (a[i] - x * p[e]) % P, v = (b[i] - x * q[e]) % Q;
            if (u < 0) u += P;
            if (v < 0) v += Q;
            ans += add((ull)u << 32 | v);
        }
    }
    cout << ans << '\n';
    return 0;
}
```

## D. 最长公共子串

先用第一个字符串建立后缀自动机。再扫描第二个字符串，维护当前状态 `p` 和当前匹配长度 `l`：若存在对应转移就继续匹配；否则沿后缀链接回退，直到找到可转移的状态。扫描过程中 `l` 的最大值就是最长公共子串长度。

后缀自动机的状态数不超过第一个字符串长度的两倍，时间复杂度为 $O(|S|+|T|)$，空间复杂度为 $O(|S|)$。

代码：
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 100010;
struct Node {
    int nx[26], lk, ln;
} tr[N * 2];
int sz = 1, ls = 1;
string s, t;
void add(int c) {
    int p = ls, u = ++sz;
    tr[u].ln = tr[p].ln + 1;
    while (p && !tr[p].nx[c]) tr[p].nx[c] = u, p = tr[p].lk;
    if (!p) tr[u].lk = 1;
    else {
        int q = tr[p].nx[c];
        if (tr[p].ln + 1 == tr[q].ln) tr[u].lk = q;
        else {
            int v = ++sz;
            tr[v] = tr[q], tr[v].ln = tr[p].ln + 1;
            while (p && tr[p].nx[c] == q) tr[p].nx[c] = v, p = tr[p].lk;
            tr[q].lk = tr[u].lk = v;
        }
    }
    ls = u;
}
int main() {
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);
    cin >> s >> t;
    for (auto c : s) add(c - 'a');
    int p = 1, l = 0, ans = 0;
    for (auto c : t) {
        int x = c - 'a';
        while (p != 1 && !tr[p].nx[x]) p = tr[p].lk, l = tr[p].ln;
        if (tr[p].nx[x]) p = tr[p].nx[x], l++;
        else p = 1, l = 0;
        ans = max(ans, l);
    }
    cout << ans << '\n';
    return 0;
}
```

## E. 字符串的幂

设字符串长度为 $n$，其 KMP 前缀函数最后一项为 `ne[n-1]`，那么最小循环节的候选长度为 $p=n-ne[n-1]$。若 $n$ 能被 $p$ 整除，字符串就是长度为 $p$ 的前缀重复 $n/p$ 次；否则字符串只能看作自身重复一次。

每组数据只需线性计算前缀函数，时间复杂度为 $O(n)$，空间复杂度为 $O(n)$。

代码：
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1000010;
int ne[N];
string s;
int main() {
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);
    while (cin >> s && s != ".") {
        int n = s.size();
        ne[0] = 0;
        for (int i = 1, j = 0; i < n; i++) {
            while (j && s[i] != s[j]) j = ne[j - 1];
            if (s[i] == s[j]) j++;
            ne[i] = j;
        }
        int p = n - ne[n - 1];
        cout << (n % p ? 1 : n / p) << '\n';
    }
    return 0;
}
```

## F. 三个好朋友

若 `U` 的长度为偶数，删除一个字符后无法分成两个等长字符串，答案一定不存在。否则设 $|U|=2m+1$，被插入的字符只可能位于前 $m+1$ 位或后 $m+1$ 位。

若插入位置在后半部分，原串 `S` 必为 `U` 的前 $m$ 个字符，只需判断后 $m+1$ 个字符能否删除一个字符后与其相同；若插入位置在前半部分，原串 `S` 必为 `U` 的后 $m$ 个字符，同理判断即可。因此候选答案至多有两个，若两个候选都合法且内容不同则答案不唯一。

两次判断均为线性扫描，时间复杂度为 $O(N)$，空间复杂度为 $O(N)$。

代码：
```cpp
#include <bits/stdc++.h>
using namespace std;
string s;
int m;
bool ck(int l, int p) {
    int i = 0, j = 0;
    bool f = 0;
    while (i <= m && j < m) {
        if (s[l + i] == s[p + j]) i++, j++;
        else {
            if (f) return 0;
            f = 1, i++;
        }
    }
    return 1;
}
int main() {
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);
    cin >> s;
    int n = s.size();
    if (n % 2 == 0) {
        cout << "NOT POSSIBLE\n";
        return 0;
    }
    m = n / 2;
    bool a = ck(m, 0), b = ck(0, m + 1);
    string x = s.substr(0, m), y = s.substr(m + 1);
    if (!a && !b) cout << "NOT POSSIBLE\n";
    else if (a && b && x != y) cout << "NOT UNIQUE\n";
    else cout << (a ? x : y) << '\n';
    return 0;
}
```

## G. 最长回文

使用 Manacher 算法。`a[i]` 表示以 `i` 为中心的最长奇回文半径，`b[i]` 表示以 `i-1` 和 `i` 之间为中心的最长偶回文半径。计算每个位置时，利用当前最靠右回文区间内的对称位置作为初值，再继续向两侧扩展。

奇回文长度为 $2a_i-1$，偶回文长度为 $2b_i$。每个字符只会使右边界向右移动常数次，因此每组数据的时间复杂度为 $O(n)$，空间复杂度为 $O(n)$。

代码：
```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 110010;
int a[N], b[N];
string s;
int main() {
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);
    while (cin >> s) {
        int n = s.size(), ans = 1;
        for (int i = 0, l = 0, r = -1; i < n; i++) {
            int k = i > r ? 1 : min(a[l + r - i], r - i + 1);
            while (i - k >= 0 && i + k < n && s[i - k] == s[i + k]) k++;
            a[i] = k--;
            if (i + k > r) l = i - k, r = i + k;
            ans = max(ans, a[i] * 2 - 1);
        }
        for (int i = 0, l = 0, r = -1; i < n; i++) {
            int k = i > r ? 0 : min(b[l + r - i + 1], r - i + 1);
            while (i - k - 1 >= 0 && i + k < n && s[i - k - 1] == s[i + k]) k++;
            b[i] = k--;
            if (i + k > r) l = i - k - 1, r = i + k;
            ans = max(ans, b[i] * 2);
        }
        cout << ans << '\n';
    }
    return 0;
}
```
