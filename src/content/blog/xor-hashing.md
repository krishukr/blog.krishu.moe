---
author: Kris Hu
pubDatetime: 2022-11-09T09:34:18.000+08:00
modDatetime:
title: 异或哈希
featured: true
draft: false
tags:
  - Note
  - 科技
  - Xor
  - Hashing
description: 又是只有我不会的科技。
---

又是只有我不会的科技。

## Table of contents

## 随机函数的选择

`std::mt19937` 和 `std::mt19937_64` 。又快又好，C++标准都说好，选它！

使用例：

```cpp
#include <random>
using ull = unsigned long long;
std::mt19937_64 mrand(1145142007);

ull a = mrand();
```

## 何时用

关注区间内是否**出现**了某些数字。

例题：

- 给定数列 $A, B$；
- 每次询问 $A$ 的前 $x$ 项出现的数字是否与 $B$ 的前 $y$ 项出现的数字相同。[^1]

我们可以为出现的每一个数字分配一个**随机值**，然后做一个前缀**异或和**。对于去重，考虑异或的恒等律[^2]，可以为数列中再次出现的位置分配 $0$。我们可以使用 `std::map` 来做值的映射。

```cpp
#include <algorithm>
#include <cstdio>
#include <iostream>
#include <map>
#include <random>

using ull = unsigned long long;

constexpr int MAX_N = 200050;

std::mt19937_64 mrand(1145142007);

int a[MAX_N], b[MAX_N];
ull sa[MAX_N], sb[MAX_N];
std::map<int, bool> va, vb;
std::map<int, ull> mp;

int main() {
    std::ios::sync_with_stdio(false);

    int n;
    read(n);
    for (int i = 1; i <= n; i++) {
        read(a[i]);
    }
    for (int i = 1; i <= n; i++) {
        read(b[i]);
    }
    for (int i = 1; i <= n; i++) {
        if (va[a[i]]) {
            a[i] = 0;
        } else {
            va[a[i]] = true;
        }
    }
    for (int i = 1; i <= n; i++) {
        if (vb[b[i]]) {
            b[i] = 0;
        } else {
            vb[b[i]] = true;
        }
    }

    for (int i = 1; i <= n; i++) {
        if (a[i] and (mp.find(a[i]) == mp.end())) {
            mp[a[i]] = mrand();
        }
    }
    for (int i = 1; i <= n; i++) {
        if (b[i] and (mp.find(b[i]) == mp.end())) {
            mp[b[i]] = mrand();
        }
    }
    for (int i = 1; i <= n; i++) {
        sa[i] = sa[i - 1];
        if (a[i]) {
            sa[i] ^= mp[a[i]];
        }
    }
    for (int i = 1; i <= n; i++) {
        sb[i] = sb[i - 1];
        if (b[i]) {
            sb[i] ^= mp[b[i]];
        }
    }

    int q;
    read(q);
    while (q--) {
        int x, y;
        read(x, y);
        std::cout << (sa[x] == sb[y] ? "Yes" : "No") << '\n';
    }

    return 0;
}
```

## 碰撞？

对于随机的 $p, q \in [0,1]$，$p \oplus q$ 有一半的情况为 $0$，一半的情况为 $1$。

即对于 $1 \mathrm{bit}$ 的随机数，碰撞的概率为 $2^{-1}$。

又因为异或是按位操作，因此一个 $64 \mathrm{bit}$ 的随机数碰撞的概率即为 $2^{-64}$。可以认为在 OI 出现的数据范围内忽略不计。

## 例

### 例 1

- 给定数组 $a_1, a_2, \dots, a_n \ (1 \le a_i \le n)$；
- 若 $a_l, a_{l+1}, \dots, a_r$ 中 $1, 2, \dots, r-l+1$ 恰好各出现一次，称其为一个子排列；
- 求 $a$ 中有几个子排列。[^3]

首先将 $[1,n]$ 的哈希及其异或前缀和预处理出来。

在 $a$ 中枚举 $1$ 的位置，向左右枚举长度。为了方便处理我们可以改成向右枚举长度后反转序列再枚举一次。

求出 $a, a'$ 的哈希异或前缀和，即可快速判断是否出现了 $[1,r-l+1]$。最后记得算上一次 $1$ 的数量。

<details><summary>Show code</summary>

```cpp
#include <algorithm>
#include <cstdio>
#include <iostream>
#include <random>

using ull = unsigned long long;

constexpr int MAX_N = 300050;

std::mt19937_64 mrand(1145142007);

int a[MAX_N];
ull r[MAX_N], s[MAX_N], p[MAX_N];

int main() {
    std::ios::sync_with_stdio(false);

    int n;
    read(n);
    for (int i = 1; i <= n; i++) {
        r[i] = mrand();
        s[i] = s[i - 1] ^ r[i];
    }
    for (int i = 1; i <= n; i++) {
        read(a[i]);
    }
    for (int i = 1; i <= n; i++) {
        p[i] = p[i - 1] ^ r[a[i]];
    }

    int ans{};
    for (int i = 1, j = 1; i <= n; i++) {
        if (a[i] == 1) {
            j = 1;
            ans++;
        } else {
            j = std::max(j, a[i]);
            if (i < j) {
                continue;
            }
            if ((p[i] ^ p[i - j]) == s[j]) {
                ans++;
            }
        }
    }

    std::reverse(a + 1, a + n + 1);
    for (int i = 1; i <= n; i++) {
        p[i] = p[i - 1] ^ r[a[i]];
    }
    for (int i = 1, j = 1; i <= n; i++) {
        if (a[i] == 1) {
            j = 1;
        } else {
            j = std::max(j, a[i]);
            if (i < j) {
                continue;
            }
            if ((p[i] ^ p[i - j]) == s[j]) {
                ans++;
            }
        }
    }
    std::cout << ans << '\n';

    return 0;
}
```

</details>

### 例 2

题面太长了，[自己看](https://www.luogu.com.cn/problem/P4065)。

不妨令所有颜色相同的位置哈希异或和为 $0$，在做前缀异或和时，若发现当前前缀异或和出现过即说明满足条件。

<details><summary>Show code</summary>

```cpp
#include <cstdio>
#include <iostream>
#include <map>
#include <memory>
#include <random>
#include <vector>

using ull = unsigned long long;

constexpr int MAX_N = 300050;

std::mt19937_64 mrand(1145141999);

std::map<ull, int> mp;
std::vector<int> c[MAX_N];
ull v[MAX_N];

void solve() {
    int n;
    read(n);
    for (int i = 1; i <= n; i++) {
        int x;
        read(x);
        c[x].push_back(i);
    }
    for (int i = 1; i <= n; i++) {
        if (c[i].empty()) {
            continue;
        }
        ull s{};
        for (int j = 1; j < c[i].size(); j++) {
            s ^= (v[c[i][j]] = mrand());
        }
        v[c[i][0]] = s;
    }

    mp[0] = 1;
    ull s{}, a{};
    for (int i = 1; i <= n; i++) {
        s ^= v[i];
        a += mp[s];
        mp[s]++;
    }
    std::cout << a << '\n';
```

</details>

[^1]: [[ABC250E] Prefix Equality](https://atcoder.jp/contests/abc250/tasks/abc250_e)
[^2]: $x \oplus 0 = x$
[^3]: [[CF1175F] The Number of Subpermutations](https://codeforces.com/contest/1175/problem/F)
