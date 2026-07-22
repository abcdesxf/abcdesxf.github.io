---
title: "数位DP笔记"
date: 2026-07-22 11:00:00
updated: 2026-07-22 11:00:00
tags:
  - "C++"
  - "算法"
categories:
  - "算法笔记"
description: "此文为作者学习数位DP一课的笔记。"
cover: "/img/posts/20260722-111050-课程学习笔记---数位DP/SecondClassHeadPicture.png"
top_img: "/img/posts/20260722-111050-课程学习笔记---数位DP/SecondClassHeadPicture.png"
---

# 课程学习笔记 - 数位DP

## 数位DP是什么？

数位DP，顾名思义，即在数位上进行DP，常用来解决求一个区间中满足某条件的数的个数一类问题。

数位DP也有状态与转移，个人比较偏向记搜写法，这里仅介绍记搜写法。

## 数位DP的思想与实现

先来看一道例题：

> 给出 $N$，求 $1$ 到 $N$ 中出现了几个数字 $6$。
>
> $N \le 10^{18}$。

首先考虑到直接枚举，$10^{18}$ 次枚举显然无法接受，考虑更简单的做法。

我们发现，对于 $10$ 进制下的每一位，本质上只有两种情况：是 $6$ 与不是 $6$。例如下面几种状态我们都可以视为一种：
$$
231\large6\normalsize48\\
3\large6\normalsize9012\\
4431\large6\normalsize8\\
$$
因为可视为同一种状态，所以可以使用记忆化搜索优化。

恭喜你，理解了数位DP的核心思想。

那么在真正的写代码中，我们的搜索还会有以下几个参数：

- $pos$：当前枚举到第几位。
- $isup$：之前的位是否为上界的位（例如上界为 $3547$，那么 $35\texttt{xx},3\texttt{xxx},354\texttt{x}$ 状态的 $isup$ 都为 $1$，这个参数是用来判断当前枚举下一位的上界的，比如上界还是 $3547$，当前状态为 $35\texttt{xx}$，那么由于 $isup = \texttt{true}$，第三位只能枚举到 $4$，这个是最难理解的）

- $others$：其他参数，可能有多个。（例如上面的例题就可以加一个 $isok$ 变量，当数位枚举到 $6$ 之后就变为 $\texttt{true}$，到最后判断 $isok$）

以及用到几个数组：

- $mem$：记忆化数组，通常根据参数的大小定义。
- $dig$：数位，通常存储上界的数位信息。

以下是一份基本的代码模板：

```cpp
void dfs(int pos, int isup, 其他参数...) {
    if (pos == 上界数位长度) {
        return (题目给出的条件);//例如数位和是否等于30即为sum == 30
    }
    int& ans = mem[参数...];//通常使用引用美观代码
    if (ans != -1) {//这里的默认值可以自己调
        return ans;
    }
    ans = 0;//ans在这里一定为-1，所以调成0
    for (int i = 0; i <= isup ? dig[pos] : 9/*如果顶着上界就是上界的当前位数字，否则就是9*/; i++) {
        ans += dfs(pos + 1, isup && (i == dig[pos])/*用与运算，如果有一个不顶着上界后面就都不顶着了*/, 其他参数调整...);
    }
    return ans;
}
```

## 例题1 - 包含49的数

在前面，我们已经了解了包含6的数如何操作，那么判断49只需要加上一个 $pre$ 参数表示前面选择了哪个数，再进行判断即可。

代码：

```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
int n, sum[20][2][11][2], dign[20], len;
int dfs(int pos, bool isup, int pre, bool isok) {
    if (pos == len) {
        return isok;
    }
    if (sum[pos][isup][pre][isok] != -1) {
        return sum[pos][isup][pre][isok];
    }
    sum[pos][isup][pre][isok] = 0;
    for (int i = 0; i <= (isup ? dign[pos] : 9); i++) {
        sum[pos][isup][pre][isok] += dfs(pos + 1, isup && i == dign[pos], i, isok || (pre == 4 && i == 9));
    }
    return sum[pos][isup][pre][isok];
}
signed main() {
    memset(sum, -1, sizeof(sum));
    cin >> n;
    string s = to_string(n);
    for (auto ch : s) dign[len++] = ch - '0';
    cout << dfs(0, true, 10, false);
    return 0;
}
```

## 例题2 - 书面零计数

要求统计所有数字中0出现的次数，可以用一个 num 变量来存储 0 的个数，但是我们发现前导零也会被算进去导致错误，所以我们需要再加一个 isst 变量表示是否开始数字部分，但是如果 isst 一直是 false，那么就是 0 的情况，num 需要+1。

代码：

```cpp
#include<bits/stdc++.h>
#define int long long
using namespace std;
int n, m, dign[11], digm[11], ln, lm, mem[11][2][2][11];
int dfs(bool norm, int pos, bool isup, bool isst, int num) {
    if (pos == (norm ? lm : ln)) {
        if (isst == false) {
            num++;
        }
        return num;
    }
    int& ans = mem[pos][isup][isst][num];
    if (ans != -1) {
        return ans;
    }
    ans = 0;
    for (int i = 0; i <= (isup ? (norm ? digm[pos] : dign[pos]) : 9); i++) {
        ans += dfs(norm, pos + 1, isup && (i == (norm ? digm[pos] : dign[pos])), isst || (i != 0), num + (i == 0 && isst));
    }
    return ans;
}
int calc(int x) {
    memset(mem, -1, sizeof(mem));
    return dfs((x == n ? false : true), 0, true, false, 0);
}
int check(int x) {
    string s = to_string(x);
    int cnt = 0;
    for (auto ch : s) cnt += ch == '0';
    return cnt;
}
signed main() {
    cin >> n >> m;
    string s = to_string(n), t = to_string(m);
    for (auto ch : s) dign[ln++] = ch - '0';
    for (auto ch : t) digm[lm++] = ch - '0';
    cout << calc(m) - calc(n) + check(n);
    return 0;
}
```

## 例题3 - 平衡数统计

这道题我们可以用一个三进制状态来表示每个数位的奇偶次数，在选择一个数位后：
$$
0\to1\\
1\to2\\
2\to1
$$
可以发现，三进制状态中 $1$ 表示出现奇数次，$2$ 表示出现偶数次，$0$ 表示未出现（注意题目中不算未出现的数字）。

最后枚举状态的每一位判断即可。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
using ull = unsigned long long;
using ll = long long;
ull n, m, mem[25][60000];
int dig[25], len, pw3[11];
bool vis[25][60000];
int change(int stt, int i) {
    if (stt / pw3[i] % 3 == 0) {
        stt += pw3[i];
    } else if (stt / pw3[i] % 3 == 1) {
        stt += pw3[i];
    } else {
        stt -= pw3[i];
    }
    return stt;
}
bool check(int stt) {
    bool flg = false;
    for (int i = 0; i <= 9; i++) {
        if (stt / pw3[i] % 3 == 0) continue;
        flg = true;
        if (i & 1) {
            if (stt / pw3[i] % 3 != 2) return false;
        } else {
            if (stt / pw3[i] % 3 != 1) return false;
        }
    }
    return flg;
}
ull dfs(int pos, int stt, bool isup, bool isst) {
    if (pos == len) {
        return isst && check(stt);
    }
    if (!isup && isst && vis[pos][stt]) {
        return mem[pos][stt];
    }
    ull ans = 0;
    for (int i = 0; i <= (isup ? dig[pos] : 9); i++) {
        if (!isst && i == 0) {
            ans += dfs(pos + 1, stt, isup && (i == dig[pos]), false);
        } else {
            ans += dfs(pos + 1, change(stt, i), isup && (i == dig[pos]), true);
        }
    }
    if (!isup && isst) {
        vis[pos][stt] = true;
        mem[pos][stt] = ans;
    }
    return ans;
}
ull calc(ull x) {
    string s = to_string(x);
    len = s.size();
    for (int i = 0; i < len; i++) {
        dig[i] = s[i] - '0';
    }
    memset(vis, 0, sizeof(vis));
    return dfs(0, 0, true, false);
}
int main() {
    pw3[0] = 1;
    for (int i = 1; i <= 10; i++) {
        pw3[i] = pw3[i - 1] * 3;
    }
    cin >> n >> m;
    cout << calc(m) - (n == 0 ? 0 : calc(n - 1)) << '\n';
    return 0;
}
```

## 例题4 - 镜像数统计

这题不能直接存储选择情况，可以用一个小巧思：只枚举回文的一半，保证枚举出来的都是回文。

还需要注意一下枚举新数位的范围。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
string s, hf;
ll n, h, mem[50][2];
bool vis[50][2];
ll dfs(int pos, bool isup) {
    if (pos == h) {
        string t = hf;
        for (int i = n / 2 - 1; i >= 0; i--) t += hf[i];
        return t <= s;
    }
    if (!isup && vis[pos][0]) return mem[pos][0];
    ll ans = 0;
    for (char c : string(pos == 0 ? "18" : "018")) {
        if (isup && c > s[pos]) continue;
        hf.push_back(c);
        ans += dfs(pos + 1, isup && c == s[pos]);
        hf.pop_back();
    }
    if (!isup) {
        vis[pos][0] = true;
        mem[pos][0] = ans;
    }
    return ans;
}
ll calc(string x) {
    s = x;
    n = s.size();
    ll ans = 1, pw = 1;
    for (int i = 1; i < n; i++) {
        if ((i + 1) / 2 > 1) pw *= 3;
        ans += 2 * pw;
    }
    h = (n + 1) / 2;
    hf = "";
    memset(vis, 0, sizeof(vis));
    return ans + dfs(0, true);
}
bool check(string x) {
    for (char c : x) {
        if (c != '0' && c != '1' && c != '8') return false;
    }
    string t = x;
    reverse(t.begin(), t.end());
    return t == x;
}
int main() {
    string N, m;
    cin >> N >> m;
    cout << calc(m) - calc(N) + check(N) << '\n';
    return 0;
}
```
