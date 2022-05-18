---
title: Trie 树学习笔记
date: 2022-05-18 15:37:18
tags:
  - 字符串
  - 数据结构
  - Trie
categories: 笔记
---

# 起

## 简介

Trie，又称前缀树或字典树。一个字符串由其经过的所有边表示。一个节点的子孙拥有相同的前缀，即这个节点本身所表示的字符串。

<!-- more -->

![实际中每个节点并不会保存自己所表示的字符串，此处为了方便理解](https://upload.wikimedia.org/wikipedia/commons/b/be/Trie_example.svg)

Trie 通常被用于查找字符串，相比较字符串哈希，Trie 树可以动态的查找，也可以对每一个节点维护信息。当然，仅查找字符串而言哈希的执行效率更高。

通常，我们会在每次插入的末尾打上标记以表明这里有一个字符串。

AC 自动机便是对 Trie 上的每一个节点维护{% label @失配指针%}[^1]来进行高效模式匹配。

## 实现

对于{% label info@普通的%} Trie，我们通常有 $\texttt{std::unordered_map} + \texttt{std::shared_ptr}$[^2] 与**数组**两种实现。

```cpp
class Trie {
   private:
    struct Node {
        bool v;  // 是否为末尾节点
        std::unordered_map<char, std::shared_ptr<Node>> son;
    };

   public:
    std::shared_ptr<Node> root = std::make_shared<Node>();

    void insert(const std::string& s) {
        auto c = root;
        for (const auto& i : s) {
            c = ((c->son.find(i) != c->son.end())
                     ? c->son[i]
                     : (c->son[i] = std::make_shared<Node>()));
        }
        c->v = true;
    }

    bool query(const std::string& s) {
        auto c = root;
        for (const auto& i : s) {
            if (c->son.find(i) != c->son.end()) {
                // do something here
                c = c->son[i];
            } else {
                return false;
            }
        }
        return c->v;
    }
};
```

虽然看起来花里胡哨，但是考虑到 Trie 本身内存访问不甚连续，实际上额外开销并不大。

```cpp
struct Node {
    bool v;
    int son[32];
} trie[MAX_N];

int cnt = 1;

void insert(const std::string& s) {
    int c = 1;
    for (const auto& i : s) {
        const auto x = i - 'a';  // 这里还是要视字符集而定
        c = (trie[c].son[x] ? trie[c].son[x] : (trie[c].son[x] = ++cnt));
    }
    trie[c].v = true;
}

bool query(const std::string& s) {
    int c = 1;
    for (const auto& i : s) {
        const auto x = i - 'a';  // 同上
        if (trie[c].son[x]) {
            // 同
            c = trie[c].son[x];
        } else {
            return false;
        }
    }
    return trie[c].v;
}
```

看起来正常多了？但其实没快多少（

# 承

Trie 的一些{% label primary@高端操作%}。

## Trie + 重构

- 给定 $n$ 个单词 $s_i$，重排所有单词。[^3]
- 设离 $s_i$ 最近的后缀字符串为 $s_j$ ，
- 最小化 $\sum{i-j}$ 。

显然有：一个单词与其后缀，后缀先插入会更优。

考虑到维护后缀比较困难，我们可以**反转单词**来转化成维护前缀。于是我们可以用 Trie 来维护。

统计答案时，我们只需要关心所有所有单词，即 Trie 上有标记的关键点。

使用并查集保留所有关键点，然后 dfs 即可。

<a href="#skip-code-1" style="font-size: 0.75em; float: right; margin-right: 5px;">跳过代码</a>

```cpp
#include <algorithm>
#include <cstdio>
#include <iostream>
#include <vector>

constexpr int MAX_N = 510050;

struct Node {
    int w;
    int son[32];
} trie[MAX_N];

int cnt = 1;

void insert(const std::string&& s, const int idx);

int f[MAX_N];

inline int find(int x) { return f[x] == x ? x : (f[x] = find(f[x])); }

int dfn[MAX_N], siz[MAX_N];
int crt;
long long ans;
std::vector<int> G[MAX_N];

void build(int x);

void dfs1(int x);

void dfs2(int x);

int main() {
    std::ios::sync_with_stdio(false);

    int n;
    std::cin >> n;
    for (int i = 1; i <= n; i++) {
        std::string s;
        std::cin >> s;
        insert(std::move(s), i);
    }
    for (int i = 1; i <= cnt + 10; i++) {
        f[i] = i;
    }

    build(1);
    dfs1(0);
    dfs2(0);

    std::cout << ans << '\n';

    return 0;
}

void insert(const std::string&& s, const int idx) {
    int c = 1;
    for (auto it = s.rbegin(); it != s.rend(); it++) {
        const auto i = *it - 'a';
        c = (trie[c].son[i] ? trie[c].son[i] : (trie[c].son[i] = ++cnt));
    }
    trie[c].w = idx;
}

void build(int x) {
    for (int i = 0; i < 26; i++) {
        const int& v = trie[x].son[i];
        if (v) {
            if (trie[v].w) {
                G[trie[find(x)].w].push_back(trie[v].w);
            } else {
                f[v] = find(x);
            }
            build(v);
        }
    }
}

void dfs1(int x) {
    siz[x] = 1;
    for (const auto& v : G[x]) {
        dfs1(v);
        siz[x] += siz[v];
    }
    std::sort(G[x].begin(), G[x].end(),
              [](const int& i, const int& j) { return siz[i] < siz[j]; });
}

void dfs2(int x) {
    dfn[x] = crt++;
    for (const auto& v : G[x]) {
        ans += crt - dfn[x];
        dfs2(v);
    }
}
```
<a name="skip-code-1"></a>

## Trie + 判环

- 给定 $n$ 个单词。[^4]
- 你可以指定字母表顺序，然后按此字典序排序所有的单词。
- 问有多少单词可以字典序最小。

首先有一个显然的性质，如果一个单词有其前缀单词，那么前缀单词必然比这个单词小，即不可能满足题目要求。我们建 Trie 维护这一信息。

接着考虑，我们不妨假设某个单词为字典序最小，对其中的每个字母向 Trie 节点上的其他存在的{% label @后驱边%}[^5]建图。如果图中有环，则说明与我们的假设矛盾。

以 $\texttt{mm, mo, om}$ 为例子，假设 $\texttt{mo}$ 最小，那么我们有 $\texttt{m} < \texttt{o}$ 且 $\texttt{o} < \texttt{m}$ 。其构成一个环。

<a href="#skip-code-2" style="font-size: 0.75em; float: right; margin-right: 5px;">跳过代码</a>

```cpp
#include <bits/stdc++.h>

template <typename T>
using arr = std::array<T, 30>;

constexpr int MAX_N = 300050;

class Trie {
   protected:
    struct Node {
        mutable bool v;
        std::unordered_map<char, std::shared_ptr<Node>> son;
    };

    void topos(const arr<arr<bool>>&& e, arr<int>& d);

   public:
    Trie() = default;
    ~Trie() = default;

    std::shared_ptr<Node> root = std::make_shared<Node>();

    void insert(const std::string& s);

    bool query(const std::string& s);
};

std::string s[MAX_N];

int main() {
    std::ios::sync_with_stdio(false);

    int n;
    std::cin >> n;
    for (int i = 1; i <= n; i++) {
        std::cin >> s[i];
    }

    auto trie = std::make_unique<Trie>();
    for (int i = 1; i <= n; i++) {
        trie->insert(s[i]);
    }

    std::vector<std::string> ans;
    for (int i = 1; i <= n; i++) {
        if (trie->query(s[i])) {
            ans.push_back(s[i]);
        }
    }

    std::cout << ans.size() << '\n';
    for (const auto& i : ans) {
        std::cout << i << '\n';
    }

    return 0;
}

void Trie::insert(const std::string& s) {
    auto c = root;
    for (const auto& i : s) {
        c = ((c->son.find(i) != c->son.end())
                 ? c->son[i]
                 : (c->son[i] = std::make_shared<Node>()));
    }
    c->v = true;
}

void Trie::topos(const arr<arr<bool>>&& e, arr<int>& d) {
    std::queue<int> q;
    for (int i = 0; i < 26; i++) {
        if (!d[i]) {
            q.push(i);
        }
    }

    while (!q.empty()) {
        int p = q.front();
        q.pop();
        for (int i = 0; i < 26; i++) {
            if (e[p][i]) {
                d[i]--;
                if (!d[i]) {
                    q.push(i);
                }
            }
        }
    }
}

bool Trie::query(const std::string& s) {
    arr<arr<bool>> e{};
    arr<int> d{};

    auto c = root;
    for (const auto& i : s) {
        const auto x = i - 'a';
        if (c->v) {
            return false;
        }
        for (int j = 0; j < 26; j++) {
            if (x != j and c->son.find(j + 'a') != c->son.end() and !e[x][j]) {
                e[x][j] = true;
                d[j]++;
            }
        }
        c = c->son[i];
    }
    topos(std::move(e), d);

    for (int i = 0; i < 26; i++) {
        if (d[i]) {
            return false;
        }
    }
    return true;
}
```
<a name="skip-code-2"></a>

## Trie + 贪心

- 给定 $n$ 个单词。[^6]
- 通过以下 $3$ 种操作，输出所有单词。最小化操作数。
    1. 在缓冲区尾部**插入**一个字母
    2. 在缓冲区尾部**删除**一个字母
    3. **输出**缓冲区

首先注意到，操作 $1$ 和操作 $3$ 的次数是一定的，所以我们要尽可能降低回溯深度。

考虑建 Trie ，于是我们可以将字符串间相同的前缀合并，以此减少回溯。

因为最后一个字符串不用删除，我们不妨钦定最长的字符串最后输出。

Trie 上 dfs 输出即可。

<a href="#skip-code-3" style="font-size: 0.75em; float: right; margin-right: 5px;">跳过代码</a>

```cpp
#include <cstdio>
#include <iostream>

constexpr int MAX_N = 500050;

std::string s[MAX_N / 10];

struct Node {
    bool m;
    bool v;
    int son[32];
} trie[MAX_N];

int cnt = 1, ans, n;
std::string as;

void insert(const std::string& s);

void solve(int k);

int main() {
    std::ios::sync_with_stdio(false);

    int m{}, p{};
    std::cin >> n;
    for (int i = 1; i <= n; i++) {
        std::cin >> s[i];
        if (m < s[i].length()) {
            m = s[i].length();
            p = i;
        }
        insert(s[i]);
    }

    {
        int c = 1;
        for (const auto& i : s[p]) {
            const auto x = i - 'a';
            c = trie[c].son[x];
            trie[c].m = true;
        }
    }

    solve(1);

    return 0;
}

void insert(const std::string& s) {
    int c = 1;
    for (const auto& i : s) {
        const auto x = i - 'a';
        c = (trie[c].son[x] ? trie[c].son[x] : (trie[c].son[x] = ++cnt));
    }
    trie[c].v = true;
}

void solve(int k) {
    if (trie[k].v) {
        ans++;
        as.push_back('P');
    }
    if (ans == n) {
        std::cout << as.length() << '\n';
        for (const auto& i : as) {
            std::cout << i << '\n';
        }
        return;
    }

    for (int x = 0; x < 26; x++) {
        if (trie[k].son[x] and !trie[trie[k].son[x]].m) {
            as.push_back(x + 'a');
            solve(trie[k].son[x]);
            as.push_back('-');
        }
    }

    for (int x = 0; x < 26; x++) {
        if (trie[k].son[x] and trie[trie[k].son[x]].m) {
            as.push_back(x + 'a');
            solve(trie[k].son[x]);
            as.push_back('-');
        }
    }
}
```
<a name="skip-code-3"></a>

# 转

除了把 Trie 用于维护字符串相关问题，我们还可以将一个数的**二进制**看作一个字符串，其字符集只包括 $\texttt{0}$ 和 $\texttt{1}$。可以高效维护异或相关等问题。

## 异或极值

- 给定一颗带边权的树。[^7]
- 求树上的一条链，其上边权异或和最大。

假设树的根为 $r$，链 $\left( a, b \right)$ 的异或和为 $\operatorname{xor}\left( a, b \right)$ 我们有：[^8]

$$
\operatorname{xor}\left( a, b \right) = \operatorname{xor}\left( a, r \right) \oplus \operatorname{xor}\left( r, b \right)
$$

不妨维护一个 $s_i = \operatorname{xor}\left( i, r \right)$ ，此时问题转化为**任选两个数，使他们的异或和最大**。

考虑 Trie + 贪心，对于 Trie 上的一个节点，如果能往与当前位不同的后驱走，则往那边走。

<a href="#skip-code-4" style="font-size: 0.75em; float: right; margin-right: 5px;">跳过代码</a>

```cpp
#include <cstdio>
#include <iostream>
#include <memory>

constexpr int MAX_N = 100050;
constexpr int MAX_L = 30;

struct Node {
    int v;
    int nxt;
    int w;
} node[MAX_N << 1];

int head[MAX_N];
int cnt;

void create(int u, int v, int w);

int a[MAX_N];

void dfs(int x, int f);

class Trie {
   protected:
    struct Node {
        std::shared_ptr<Node> son[2]{nullptr, nullptr};
    };

   public:
    Trie() = default;
    ~Trie() = default;

    std::shared_ptr<Node> root = std::make_shared<Node>();

    void insert(int a);

    int query(int a);
};

template <typename T>
T read();

template <typename T>
void read(T& t);

template <typename T, typename... Args>
void read(T& t, Args&... rest);

int main() {
    std::ios::sync_with_stdio(false);

    int n;
    read(n);
    for (int i = 1; i < n; i++) {
        int u, v, w;
        read(u, v, w);
        create(u, v, w);
        create(v, u, w);
    }
    dfs(1, 0);

    auto trie = std::make_unique<Trie>();
    for (int i = 1; i <= n; i++) {
        trie->insert(a[i]);
    }

    int ans = 0;
    for (int i = 1; i <= n; i++) {
        ans = std::max(ans, trie->query(a[i]));
    }
    std::cout << ans << '\n';

    return 0;
}

void create(int u, int v, int w) {
    node[++cnt].v = v;
    node[cnt].nxt = head[u];
    node[cnt].w = w;
    head[u] = cnt;
}

void dfs(int x, int f) {
    for (int i = head[x]; i; i = node[i].nxt) {
        int v = node[i].v, w = node[i].w;
        if (v == f) {
            continue;
        }
        a[v] = a[x] ^ w;
        dfs(v, x);
    }
}

void Trie::insert(int a) {
    auto c = root;
    for (int i = MAX_L; i >= 0; i--) {
        const int x = (a >> i) & 1;
        c = (c->son[x] ? c->son[x] : (c->son[x] = std::make_shared<Node>()));
    }
}

int Trie::query(int a) {
    auto c = root;
    int ans{0};
    for (int i = MAX_L; i >= 0; i--) {
        const int x = (a >> i) & 1;
        if (c->son[x ^ 1]) {
            c = c->son[x ^ 1];
            ans = (ans << 1) | (x ^ 1);
        } else {
            c = c->son[x];
            ans = (ans << 1) | x;
        }
    }
    return ans ^ a;
}

template <typename T>
T read() {
    T x = 0, f = 1;
    char ch = getchar();
    while (!isdigit(ch)) {
        if (ch == '-') f = -1;
        ch = getchar();
    }
    while (isdigit(ch)) {
        x = x * 10 + ch - 48;
        ch = getchar();
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
<a name="skip-code-4"></a>

## <ruby><rb>Trie</rb><rp>[</rp><rt>线段树</rt><rp>]</rp></ruby>

> 我也很喜欢你，Trie！
> 
> 我喜欢 Trie，还有大家！

- 你有一个模式串池。
- 给定每个时刻模式串池的**添加或删除**变化，多次询问。[^9]
- 每次询问给定一个 01 字符串，问 $\left[ a,b \right]$ 时刻内所有模式串中最长前缀变了几次。

看到最长前缀匹配就应该想到 Trie 了。

显然答案是区间可加的，有 $\operatorname{ans}\left[ a,b \right] = \operatorname{ans}\left[ 1,b \right] - \operatorname{ans}\left[ 1,a \right)$ 。离线模式串池的变化。

假设存在两个模式串 $a,b$ ，其中 $a$ 是 $b$ 的前缀，对 $a$ 的修改影响的部分即为 {% label @$T_a - T_b$ %}[^10] 。我们从线段树那偷一个东西叫**懒标记**，与 Trie 杂交，这道题就做完了。

最后注意 Trie 是动态开点的。

<a href="#skip-code-5" style="font-size: 0.75em; float: right; margin-right: 5px;">跳过代码</a>

```cpp
#include <cstdio>
#include <iostream>
#include <memory>
#include <vector>

constexpr int MAX_N = 100050;

struct R {
    std::string ip;
    int op;
} r[MAX_N];

struct Q {
    std::string ip;
    int idx;
} qa[MAX_N];

std::vector<int> dq[MAX_N], aq[MAX_N];
int ans[MAX_N];

class Trie {
   private:
    struct Node {
        bool v;
        int son[2];
        int t;
        int a;
    } trie[MAX_N << 5];

    void pushdown(const int& x);

   public:
    Trie() = default;
    ~Trie() = default;

    int cnt = 1;

    void mod(const std::string& s, const int& mv);

    int query(const std::string& s);
};

int main() {
    std::ios::sync_with_stdio(false);

    int n, q;
    std::cin >> n >> q;
    for (int i = 1; i <= n; i++) {
        std::string op;
        std::cin >> op >> r[i].ip;
        r[i].op = (op[0] == 'A' ? 1 : -1);
    }
    for (int i = 1; i <= q; i++) {
        std::cin >> qa[i].ip;
        qa[i].idx = i;

        int l, r;
        std::cin >> l >> r;
        aq[r].push_back(i);
        dq[l].push_back(i);
    }

    auto trie = std::make_unique<Trie>();
    for (int i = 1; i <= n; i++) {
        trie->mod(r[i].ip, r[i].op);
        for (const auto& j : aq[i]) {
            ans[qa[j].idx] += trie->query(qa[j].ip);
        }
        for (const auto& j : dq[i]) {
            ans[qa[j].idx] -= trie->query(qa[j].ip);
        }
    }
    for (int i = 1; i <= q; i++) {
        std::cout << ans[i] << '\n';
    }

    return 0;
}

#define lc trie[x].son[0]
#define rc trie[x].son[1]

void Trie::pushdown(const int& x) {
    if (!lc) {
        lc = ++cnt;
    }
    if (!rc) {
        rc = ++cnt;
    }
    if (trie[x].t) {
        if (!trie[lc].v) {
            trie[lc].t += trie[x].t;
            trie[lc].a += trie[x].t;
        }
        if (!trie[rc].v) {
            trie[rc].t += trie[x].t;
            trie[rc].a += trie[x].t;
        }
        trie[x].t = 0;
    }
}

void Trie::mod(const std::string& s, const int& mv) {
    int c = 1;
    for (const auto& i : s) {
        const auto x = i - '0';
        pushdown(c);
        c = trie[c].son[x];
    }
    trie[c].v += mv;
    trie[c].a += 1;
    trie[c].t += 1;
}

int Trie::query(const std::string& s) {
    int c = 1;
    for (const auto& i : s) {
        const auto x = i - '0';
        pushdown(c);
        c = trie[c].son[x];
    }
    return trie[c].a;
}

#undef lc
#undef rc
```
<a name="skip-code-5"></a>

注意到我们成功给 Trie 嫁接了线段树的懒标记。

这不禁让我们思考 Trie 与线段树的关系。

# 合

线段树是二叉树；01 Trie 也是二叉树。

我们可以粗略的认为，01 Trie 是一颗动态开点的线段树。

线段树往左递归相当于选 $0$ ，往右递归相当于选 $1$ 。这样，每个节点所表示的 $01$ 串正好是其下标的**二进制**。

这也意味着线段树的高端操作同样可以运用在 {% label @01 Trie %}[^11] 上，如线段树合并、线段树分裂等。

最后，我们再看一道题吧。

- 给一个序列 $a$ ，进行如下操作。强制在线。[^12][^13]
  1. 对所有满足 $i \equiv y \left( \bmod 2^x \right)$ 的 $a_i$ 加上 $v$
  2. 查询所有满足 $i \equiv y \left( \bmod 2^x \right)$ 的 $a_i$ 之和

翻译一下，对于所有满足条件的下标 $i$ ，有长度为 $x$ 的公共后缀 $x$ 。

经典套路，翻转过来，我们就可以方便地用 Trie 维护前缀。

按上文所述方法，使用线段树的操作维护这颗二叉树。

<a href="#skip-code-6" style="font-size: 0.75em; float: right; margin-right: 5px;">跳过代码</a>

```cpp
#include <cstdio>
#include <iostream>
#include <memory>

#define int long long

constexpr int MAX_N = 2200050;
constexpr int MAX_L = 21;

class Trie {
   private:
    struct Node {
        int siz;
        int son[2];
        int a;
        int t;
    } trie[MAX_N << 1];

    int cnt;

    void seg_merge(const int& x);

    void pushdown(const int& x);

   public:
    Trie() = default;
    ~Trie() = default;

    int build(const int x, const int d);

    void mod(int x, int d, const int& mx, const int& my, const int& mv);

    int query(const int& r, const int& mx, const int& my);
};

int a[MAX_N];
int n, m;

template <typename T>
T read();

template <typename T>
void read(T& t);

template <typename T, typename... Args>
void read(T& t, Args&... rest);

signed main() {
    std::ios::sync_with_stdio(false);

    read(n, m);
    for (int i = 1; i <= n; i++) {
        read(a[i]);
    }

    auto trie = std::make_unique<Trie>();
    const int root = trie->build(0, 0);

    int la{};
    while (m--) {
        int op, x, y;
        read(op, x, y);
        op = (op + la) % 2 + 1;
        y &= (1 << x) - 1;

        if (op == 1) {
            trie->mod(root, 0, x, y, read<int>());
        } else {
            la = trie->query(root, x, y);
            std::cout << la << '\n';
        }
    }

    return 0;
}

#define lc trie[x].son[0]
#define rc trie[x].son[1]

void Trie::seg_merge(const int& x) { trie[x].a = trie[lc].a + trie[rc].a; }

void Trie::pushdown(const int& x) {
    if (trie[x].t) {
        trie[lc].a += trie[lc].siz * trie[x].t;
        trie[lc].t += trie[x].t;
        trie[rc].a += trie[rc].siz * trie[x].t;
        trie[rc].t += trie[x].t;
        trie[x].t = 0;
    }
}

int Trie::build(const int x, const int d) {
    int c = ++cnt;
    if (d == MAX_L) {
        if (x <= n) {
            trie[c].a = a[x];
            trie[c].siz = (int)((bool)(x));
        }
        return c;
    }
    trie[c].son[0] = build(x, d + 1);
    trie[c].son[1] = build(x | (1 << d), d + 1);
    trie[c].siz = trie[trie[c].son[0]].siz + trie[trie[c].son[1]].siz;
    seg_merge(c);
    return c;
}

void Trie::mod(int x, int d, const int& mx, const int& my, const int& mv) {
    if (d >= mx) {
        trie[x].a += trie[x].siz * mv;
        trie[x].t += mv;
        return;
    }
    pushdown(x);
    mod(trie[x].son[(int)((bool)(my & (1 << d)))], d + 1, mx, my, mv);
    seg_merge(x);
}

int Trie::query(const int& r, const int& mx, const int& my) {
    int x = r;
    for (int d = 0; d < mx; d++) {
        pushdown(x);
        x = trie[x].son[(int)((bool)(my & (1 << d)))];
    }
    return trie[x].a;
}

#undef lc
#undef rc

template <typename T>
T read() {
    T x = 0, f = 1;
    char ch = getchar();
    while (!isdigit(ch)) {
        if (ch == '-') f = -1;
        ch = getchar();
    }
    while (isdigit(ch)) {
        x = x * 10 + ch - 48;
        ch = getchar();
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
<a name="skip-code-6"></a>

---

至此，此文讲解了 Trie 与 01 Trie 的一些操作。希望对你有所启发。

碍于笔者能力所限，还有一些手法无法很好地阐述，还有**可持久化 Trie **也没有讲。

如果你对本文内容有指正或者疑问，欢迎[邮件](mailto:i@krishu.moe)联系我。

祝你学业进步。

[^1]: 就是 KMP 的失配指针
[^2]: 炫技成分偏多（
[^3]: [洛谷 P3294 [SCOI2016]背单词](https://www.luogu.com.cn/problem/P3294)
[^4]: [洛谷 P3065 [USACO12DEC]First! G](https://www.luogu.com.cn/problem/P3065)
[^5]: 即有相同前缀的其它字符串的同层级字母
[^6]: [洛谷 P4683 [IOI2008] Type Printer](https://www.luogu.com.cn/problem/P4683)
[^7]: [洛谷 P4551 最长异或路径](https://www.luogu.com.cn/problem/P4551)
[^8]: 因为 LCA 及以上的部分会经过两次，相互抵消
[^9]: [洛谷 P5460 [BJOI2016]IP地址](https://www.luogu.com.cn/problem/P5460)
[^10]: $T_i$ 表示字符串 $i$ 所在节点的子树
[^11]: 或许甚至推广至普通 Trie 上？我不知道
[^12]: [洛谷 P6587 超超的序列 加强](https://www.luogu.com.cn/problem/P6587)
[^13]: 这题挺粪的，理解即可
