---
layout: post
title: Imperial roads(最小生成树+树上倍增|树剖)
date: 2019-11-03 19:50:00
Author: Lee-Coderr
categories: 
- 数据结构|树链剖分
- 数据结构|最小生成树
tags: [最小生成树,树上倍增]
comments: true
toc: true
pinned: true
---



# 题目

[ICPC Latin American Regional – 2017-Problems-I](https://codeforces.com/group/xrTA2IaQje/contest/258354/attachments/download/7471/statements-2017-latam-regional.pdf) 



# 题意

n个点m条边，q个询问，每次固定一条遍，求剩余的最小生成树。

# 题解

先求出最小生成树，对每个询问加上一条边必定产生环，求环上的除指定边的最大权值，即求树上两点路径最大权值。若指定的边在最小生成树上求出的答案无影响。

可以用树上倍增LCA方式求最大值。也可以用树剖求。

# 树剖

```c++
#include <bits/stdc++.h>
#define ls rt<<1
#define rs rt<<1|1
#define LL long long
using namespace std;
typedef pair<int, int> P;
const int maxn = 5e5+5;

vector<P> E[maxn];
int a[maxn], f[maxn], top[maxn], id[maxn], rk[maxn], siz[maxn], d[maxn], son[maxn], cnt;
void dfs1(int u, int fa){
    d[u] = d[fa] + 1; f[u] = fa; siz[u] = 1; son[u] = 0;
    for(auto it : E[u]){
        int v = it.first, w = it.second;
        if(v == fa) continue;
        a[v] = w; dfs1(v, u);
        siz[u] += siz[v];
        if(siz[son[u]] < siz[v]) son[u] = v;
    }
}
void dfs2(int u, int rt){
    id[u] = ++cnt; top[u] = rt; rk[cnt] = a[u];
    if(son[u]) dfs2(son[u], rt);
    for(auto it : E[u]){
        int v = it.first, w = it.second;
        if(v == f[u] || v == son[u]) continue;
        dfs2(v, v);
    }
}

std::map<P, int> mp;
int n, m, Q, u, v, w;
int Max[maxn<<2];
void build(int rt, int l, int r){
    if(l == r){
        Max[rt] = rk[l];
        return ;
    } int mid = l+r >> 1;
    build(ls, l, mid); build(rs, mid+1, r);
    Max[rt] = max(Max[ls], Max[rs]);
}
int qry(int rt, int l, int r, int L, int R){
    if(L <= l && R >= r) return Max[rt];
    int mid = l+r >> 1, res = 0;
    if(L <= mid) res = max(res, qry(ls, l, mid, L, R));
    if(R > mid) res = max(res, qry(rs, mid+1, r, L, R));
    return res;
}
int Query(int u, int v){
    int fu = top[u], fv = top[v], res = -1;
    while(fu != fv){
        if(d[fu] < d[fv]) swap(fu, fv), swap(u, v);
        res = max(res, qry(1, 1, n, id[fu], id[u]));
        u = f[fu]; fu = top[u];
    }
    if(u == v) return res;
    if(d[u] > d[v]) swap(u, v);
    res = max(res, qry(1, 1, n, id[son[u]], id[v]));
    return res;
}
namespace Min_Tree{
    int head[maxn], num_edge;
    struct Ee{
        int u, v, w;
        bool operator < (const Ee & a)const{return w < a.w;}
    }edge[maxn];
    void addedge(int u, int v, int w){edge[++num_edge] = Ee{u, v, w};}
    int fa[maxn];
    int Find(int x){return x==fa[x] ? x : fa[x] = Find(fa[x]);}

    int Kru(){
        for(int i=1; i<maxn; i++) fa[i] = i;
        int sum = 0;
        sort(edge+1, edge+num_edge+1);
        for(int i=1; i<=num_edge; i++){
            int fx = Find(edge[i].u), fy = Find(edge[i].v);
            if(fx == fy) continue;
            fa[fy] = fx; sum += edge[i].w;
            E[edge[i].u].push_back( P(edge[i].v, edge[i].w) );
            E[edge[i].v].push_back( P(edge[i].u, edge[i].w) );
        }
        return sum;
    }
}
int main(){
    scanf("%d%d", &n, &m);
    for(int i=1; i<=m; i++){
        scanf("%d%d%d", &u, &v, &w);
        Min_Tree::addedge(u, v, w);

        if(u > v) swap(u, v); mp[P(u, v)] = w;
    }

    int sum = Min_Tree::Kru();
    dfs1(1, 0); dfs2(1, 1);
    build(1, 1, n);

    scanf("%d", &Q);
    while(Q--){
        scanf("%d%d", &u, &v);
        if(u > v) swap(u, v);
        printf("%d\n", sum - Query(u, v) + mp[P(u, v)]);
    }
}
```

# 树上倍增

```c++
int d[maxn], f[maxn][21], Max[maxn][21];
void dfs2(int u, int fa){
    d[u] = d[fa] + 1; f[u][0] = fa;
    Max[u][0] = max(v[u], v[fa]);
    for(int i=1; i<=20; i++){
        f[u][i] = f[f[u][i-1]][i-1];
        Max[u][i] = max(Max[u][i], max(Max[u][i-1], Max[f[u][i-1]][i-1]));
    }
    for(auto it : e[u]) if(it.first != fa) dfs2(it.first, u);
}
 
int lca(int u, int x){
    int mx = max(v[u],v[x]);
    if(d[u] > d[x]) swap(u, x);
    for(int i=20; i>=0; i--)
        if(d[u] <= d[x]-(1<<i))
            mx = max(mx, Max[x][i]), x = f[x][i];
    if(u == x) return mx; // u
    for(int i=20; i>=0; i--){
        if(f[u][i] != f[x][i])
            mx = max(mx, max(Max[u][i], Max[x][i])), u = f[u][i], x = f[x][i];
    }
    return max(mx, v[f[x][0]]); //f[u][0]
}
```

