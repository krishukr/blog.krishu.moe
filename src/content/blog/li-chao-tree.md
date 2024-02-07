---
author: Kris Hu
pubDatetime: 2022-11-04T09:08:27.000+08:00
modDatetime:
title: 李超树
featured: false
draft: false
tags:
  - Note
  - Data Structure
  - Segment Tree
description: 李超树是用于高效维护多个一次函数在某点极值的数据结构。
---

李超树是用于高效维护多个一次函数在某点极值的数据结构。

## Table of contents

## 引

我们可以以类似权值线段树的形式，将一次函数的定义域作为线段树的区间。

当我们加入一条一次函数时，会与另外一条一次函数在一个区间内产生以下三种情况：

- 完全在其上方
- 完全在其下方
- 存在一个交点

因为我们维护的仅有极值，只需要对旧极值一条一次函数进行判断。

对于前两种情况，直接覆盖区间即可。

而如果发现存在一个交点，则递归往交点所在的子区间下放标记。

举例来说，假设两条一次函数分别为 $f(x)$ 和 $g(x)$ 。

- 在区间中点处 $f(mid)$ 更优，则将 $f$ 作为区间的标记。
- 但发现在区间右端点处 $g(r)$ 更优，说明在 $mid - r$ 之间存在一个交点。因此向右区间递归下去。

查询则一路取标记中的极值即可。

```cpp
struct L {
    double k;
    double b;

    double operator()(const int& x) { return k * x + b; }
} tree[MAX_N << 2];

void SegTree::mod(int x, int l, int r, L mv) {
    const auto mid = (l + r) >> 1;
    if (tree[x](mid) <= mv(mid)) {
        std::swap(tree[x], mv);
    }
    if (l == r) {
        return;
    }
    if (tree[x](l) <= mv(l)) {
        mod(lc, l, mid, mv);
    } else if (tree[x](r) <= mv(r)) {
        mod(rc, mid + 1, r, mv);
    }
}

double SegTree::query(int x, int l, int r, const int& qk) {
    if (l == r) {
        return tree[x](qk);
    }
    const auto mid = (l + r) >> 1;
    if (qk <= mid) {
        return std::max(tree[x](qk), query(lc, l, mid, qk));
    } else {
        return std::max(tree[x](qk), query(rc, mid + 1, r, qk));
    }
}
```

## 例

### 例 1

板子题。也许主要的难度在于处理读入。[^1]

<details><summary>Show code</summary>

```cpp
#include <cmath>
#include <cstdio>
#include <iostream>
#include <memory>

constexpr int MAX_N = 1000050;
constexpr int N = 1000000;

class SegTree {
   public:
    struct L {
        double k;
        double b;

        // 请注意本题中天数从 1 开始
        double operator()(const int& x) { return k * (x - 1) + b; }
    } tree[MAX_N << 2];

    SegTree() = default;
    ~SegTree() = default;

    void mod(int x, int l, int r, L mv);

    double query(int x, int l, int r, const int& qk);
};

template <typename T>
T read();

template <typename T>
void read(T& t);

template <typename T, typename... Args>
void read(T& t, Args&... rest);

template <>
char read();

template <>
double read();

int main() {
    std::ios::sync_with_stdio(false);

    auto segTree = std::make_unique<SegTree>();
    int T;
    read(T);
    while (T--) {
        char op;
        read(op);
        if (op == 'P') {
            double b, k;
            read(b, k);
            segTree->mod(1, 1, N, {k, b});
        } else if (op == 'Q') {
            int x;
            read(x);
            std::cout << (int)(segTree->query(1, 1, N, x)) / 100 << '\n';
        }
    }

    return 0;
}

#define lc (x << 1)
#define rc ((x << 1) | 1)

void SegTree::mod(int x, int l, int r, L mv) {
    const auto mid = (l + r) >> 1;
    if (tree[x](mid) <= mv(mid)) {
        std::swap(tree[x], mv);
    }
    if (l == r) {
        return;
    }
    if (tree[x](l) <= mv(l)) {
        mod(lc, l, mid, mv);
    } else if (tree[x](r) <= mv(r)) {
        mod(rc, mid + 1, r, mv);
    }
}

double SegTree::query(int x, int l, int r, const int& qk) {
    if (l == r) {
        return tree[x](qk);
    }
    const auto mid = (l + r) >> 1;
    if (qk <= mid) {
        return std::max(tree[x](qk), query(lc, l, mid, qk));
    } else {
        return std::max(tree[x](qk), query(rc, mid + 1, r, qk));
    }
}

#undef lc
#undef rc

template <typename T>
T read() {
    T x = 0, f = 1;
    char ch = getchar_unlocked();
    while (!isdigit(ch)) {
        if (ch == '-') f = -1;
        ch = getchar_unlocked();
    }
    while (isdigit(ch)) {
        x = x * 10 + ch - 48;
        ch = getchar_unlocked();
    }
    return x * f;
}

template <typename T>
void read(T& t) {
    t = read<T>();
}

template <typename T, typename... Args>
void read(T& t, Args&... rest) {
    t = read<T>();
    read(rest...);
}

template <>
char read() {
    char ch = getchar_unlocked();
    while (ch != 'Q' and ch != 'P') {
        ch = getchar_unlocked();
    }
    return ch;
}

template <>
double read() {
    long long x = 0, f = 1, k = 0, b = 0;
    bool g = false;
    char ch = getchar_unlocked();
    while (!isdigit(ch)) {
        if (ch == '-') f = -1;
        ch = getchar_unlocked();
    }
    while (isdigit(ch) or ch == '.') {
        if (ch == '.') {
            g = true;
        } else if (!g) {
            x = x * 10 + ch - 48;
        } else {
            b = b * 10 + ch - 48;
            k++;
        }
        ch = getchar_unlocked();
    }
    return f * (std::pow(0.1, k) * b + x);
}
```

</details>

### 例 2

- 给定 $a_i, b_i$ ，其中 $i \in [1,n]$ 。
- 每次询问给出 $p, k$，求 $$ \max\_{l \le p \le r}{\left( \sum_l^r{a_i} - k(\sum_l^r{b_i}) \right)} $$

首先对于包含 $p$ 这个要求进行转化，可以转化为 $$\max_{1 \le l \le r \le p} + \max_{p+1 \le l \le r \le n}$$

我们以式子左半边为例，即 $$ \max*{1 \le l \le r \le p}{\left( \sum*{i=l}^r{a*i} - k(\sum*{i=l}^r{b_i}) \right)} $$

假设有 $$sa_x=\sum_{i=1}^{x}{a_i}, sb_x=\sum_{i=1}^{x}{b_i}$$

可以将原式转化成 $$\max_{1 \le l \le r \le p}{\left( \left( sa_r - sa_{l-1} \right) - k\left( sb_r - sb_{l-1} \right) \right)}$$

即是 $$ sa*r - k sb_r - \min*{0 \le l < r}{\left( k sb_l - sa_l \right)} $$

注意到 $sb_l k - sa_l$ 是一次函数的形式，因此用李超树维护即可。

<details><summary>Show code</summary>

```cpp
#include <algorithm>
#include <climits>
#include <cstdio>
#include <iostream>
#include <memory>

using ll = long long;

constexpr int MAX_N = 1000050;
constexpr int N = 1000000;
constexpr ll INF = LLONG_MAX >> 4;

int a[MAX_N], b[MAX_N];
struct Q {
    int p;
    int k;
    int i;
} q[MAX_N];

class SegTree {
   public:
    struct L {
        ll k;
        ll b = INF;
        L() = default;
        L(ll k, ll b) { this->k = k, this->b = b; }
        // fxxk C++11, we need C++14!

        ll operator()(const ll& x) { return k * x + b; }
    } tree[MAX_N << 3];

    SegTree() = default;
    ~SegTree() = default;

    void mod(int x, int l, int r, L mv);

    ll query(int x, int l, int r, const ll& qk);
};

ll ans[MAX_N];

template <typename T>
T read();

template <typename T>
void read(T& t);

template <typename T, typename... Args>
void read(T& t, Args&... rest);

int main() {
    std::ios::sync_with_stdio(false);

    int n, m;
    read(n, m);
    for (int i = 1; i <= n; i++) {
        read(a[i], b[i]);
    }
    for (int i = 1; i <= m; i++) {
        read(q[i].p, q[i].k);
        q[i].i = i;
    }
    std::sort(q + 1, q + m + 1,
              [](const Q& x, const Q& y) { return x.p < y.p; });

    {
        std::unique_ptr<SegTree> segTree(new SegTree());
        ll sa{}, sb{};
        int i{1};
        for (int j = 1; j <= m; j++) {
            while (i <= n and i <= q[j].p) {
                segTree->mod(1, -N, N, {-sb, sa});
                sa += a[i];
                sb += b[i];
                i++;
            }
            ans[q[j].i] += sa - sb * q[j].k - segTree->query(1, -N, N, q[j].k);
        }
    }
    {
        std::unique_ptr<SegTree> segTree(new SegTree());
        ll sa{}, sb{};
        int i{n};
        for (int j = m; j >= 1; j--) {
            while (i >= 1 and i > q[j].p) {
                segTree->mod(1, -N, N, {-sb, sa});
                sa += a[i];
                sb += b[i];
                i--;
            }
            ans[q[j].i] += std::max(
                0ll, sa - sb * q[j].k - segTree->query(1, -N, N, q[j].k));
        }
    }

    for (int i = 1; i <= m; i++) {
        std::cout << ans[i] << '\n';
    }

    return 0;
}

#define lc (x << 1)
#define rc ((x << 1) | 1)

void SegTree::mod(int x, int l, int r, L mv) {
    const auto mid = (l + r) >> 1;
    if (tree[x](mid) >= mv(mid)) {
        std::swap(tree[x], mv);
    }
    if (l == r) {
        return;
    }
    if (tree[x](l) >= mv(l)) {
        mod(lc, l, mid, mv);
    } else if (tree[x](r) >= mv(r)) {
        mod(rc, mid + 1, r, mv);
    }
}

ll SegTree::query(int x, int l, int r, const ll& qk) {
    if (l == r) {
        return tree[x](qk);
    }
    const auto mid = (l + r) >> 1;
    if (qk <= mid) {
        return std::min(tree[x](qk), query(lc, l, mid, qk));
    } else {
        return std::min(tree[x](qk), query(rc, mid + 1, r, qk));
    }
}

#undef lc
#undef rc

template <typename T>
T read() {
    T x = 0, f = 1;
    char ch = getchar_unlocked();
    while (!isdigit(ch)) {
        if (ch == '-') f = -1;
        ch = getchar_unlocked();
    }
    while (isdigit(ch)) {
        x = x * 10 + ch - 48;
        ch = getchar_unlocked();
    }
    return x * f;
}

template <typename T>
void read(T& t) {
    t = read<T>();
}

template <typename T, typename... Args>
void read(T& t, Args&... rest) {
    t = read<T>();
    read(rest...);
}
```

</details>

[^1]: [[JSOI2008]Blue Mary开公司](https://www.luogu.com.cn/problem/P4254)
