---
title: Trie 树学习笔记
date: 2022-05-16 20:33:40
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

![实际中每个节点并不会保存自己的前缀，此处为了方便理解](https://upload.wikimedia.org/wikipedia/commons/b/be/Trie_example.svg)

Trie 树通常被用于查找字符串，相比较字符串哈希，Trie 树可以动态的查找，也可以对每一个前缀节点维护信息。当然，字符串哈希的执行效率更高。

通常，我们会在每个插入的末尾节点打标记以表明这里有一个字符串。

AC 自动机便是对 Trie 树上的每一个节点维护{% label @失配指针%}[^1]来进行高效模式匹配。

## 实现

对于{% label info@普通的%} Trie，我们通常有 $\texttt{std::unordered_map} + \texttt{std::shared_ptr}$[^2] 与**数组版**两种实现。

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

首先考虑到维护后缀比较困难，我们可以**反转单词**来转化成维护前缀。于是我们可以用 Trie 来维护。

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

[^1]: 就是 KMP 的失配指针
[^2]: 炫技成分偏多（
[^3]: [洛谷 P3294 [SCOI2016]背单词](https://www.luogu.com.cn/problem/P3294)