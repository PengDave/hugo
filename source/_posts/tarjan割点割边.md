---
title: tarjan割点割边
date: 2024-09-11 22:03:56
tags:
mathjax: true
---

# 割点

## 基础概念

割点是怎么回事呢？是这样的，如果图中的某一个点被删掉后图的最大连通子图数量变少了，也就是说那个点所在的最大连通子图被分成了多个最大连通子图那么这个点就是图的割点。

## 算法思想

知道概念后，又该怎么求呢。

首先，我们求出节点的 $dfn$ 序，这用 $dfs$ 就可以做到，同时，在 $dfs$ 时，维护 $low$ 表示一个点在不经过在自己在 $dfs$ 树中的父节点能到达的 $dfn$ 序最小的节点的 $dfn$，如果节点 $u$ 存在一个它在 $dfs$ 树中的子节点 $v$ 使得 $low_v\ge dfn_u$，那么节点 $u$ 就是一个割点，为什么呢？其实很简单，毕竟如果 $v$ 点不经过 $u$ 点就到不了前面 $dfn$ 序更小的 $u$ 的祖先，那就说明去掉 $u$ 节点后 $v$ 就与前面的不连通了。同时，还有一种情况需要讨论，那就是当 $u$ 是他所在 $dfs$ 树的根时，只要有两个及以上的去掉 $u$ 就无法互相连通的子节点，拿他也是割点。

接下来讲求 $low$ 的方法，先给方法。若 $v$ 是 $u$ 的 $dfs$ 树子节点，那 $low_u=\min(low_u,low_v)$，否则 $low_u=\min(low_u,dfn_v)$。证明？很可惜我现在还不会。

## 代码

```cpp
#include<iostream>
using namespace std;
int ans[20010];
struct edge{
    int to;
    int nxt;
}g[200010];
int head[20010],tot=0;
int dfn[20010],low[20010],cnt=0;
int res=0;
void addedge(int x,int y){
    tot++;
    g[tot].to=y;
    g[tot].nxt=head[x];
    head[x]=tot;
    return;
}
void tarjan(int u,int fa){
    dfn[u]=low[u]=++cnt;
    int t=0;
    for(int i=head[u];i;i=g[i].nxt){
        int v=g[i].to;
        if(!dfn[v]){
            t++;
            tarjan(v,u);
            low[u]=min(low[u],low[v]);
            if(fa&&!ans[u]&&low[v]>=dfn[u]){
                res++;
                ans[u]=1;
            }
        }else if(v!=fa){
            low[u]=min(low[u],dfn[v]);
        }
    }
    if(!fa&&t>1&&!ans[u]){
        res++;
        ans[u]=1;
    }
    return;
}
int main(){
    cin.tie(0);
    ios::sync_with_stdio(0);
    int n,m;
    cin>>n>>m;
    while(m--){
        int x,y;
        cin>>x>>y;
        addedge(x,y);
        addedge(y,x);
    }
    for(int i=1;i<=n;i++){
        if(!dfn[i]){
            tarjan(i,0);
        }
    }
    cout<<res<<endl;
    for(int i=1;i<=n;i++){
        if(ans[i]){
            cout<<i<<" ";
        }
    }
    cout<<flush;
    return 0;
}
```

# 割边

## 基础概念

割边和割点差不多，不过点变成了边。

## 算法思想

总体区别与割点并不多，首先便是判断时变成了 $low_v> dfn_u$，为什么呢？因为 $u$ 到 $v$ 的边是唯一的，所以从 $v$ 出发不经过 $u$ 结点就相当于不走 $(u,v)$ 这条边，所以如果去掉该边 $v$ 连 $u$ 都连不上，那就不行了。其次便是不用考虑是否是根。

## 代码

```cpp
#include<iostream>
#include<algorithm>
using namespace std;
struct edge{
    int to;
    int nxt;
}g[200010];
int head[20010],tot=0;
int dfn[20010],low[20010],cnt=0,fa;
int res=0;
struct node{
    int x,y;
}ans[200010];
bool operator<(node a,node b){
    if(a.x!=b.x)return a.x<b.x;
    else return a.y<b.y;
}
void addedge(int x,int y){
    tot++;
    g[tot].to=y;
    g[tot].nxt=head[x];
    head[x]=tot;
    return;
}
void tarjan(int u,int fa){
    dfn[u]=low[u]=++cnt;
    int t=0;
    for(int i=head[u];i;i=g[i].nxt){
        int v=g[i].to;
        if(!dfn[v]){
            t++;
            tarjan(v,u);
            low[u]=min(low[u],low[v]);
            if(low[v]>dfn[u]){
                res++;
                ans[res].x=u;
                ans[res].y=v;
            }
        }else if(v!=fa){
            low[u]=min(low[u],dfn[v]);
        }
    }
    return;
}
int main(){
    cin.tie(0);
    ios::sync_with_stdio(0);
    int n,m;
    cin>>n>>m;
    while(m--){
        int x,y;
        cin>>x>>y;
        addedge(x,y);
        addedge(y,x);
    }
    for(int i=1;i<=n;i++){
        if(!dfn[i]){
            tarjan(i,0);
        }
    }
    sort(ans+1,ans+res+1);
    for(int i=1;i<=res;i++){
        cout<<ans[i].x<<" "<<ans[i].y<<"\n";
    }
    cout<<flush;
    return 0;
}
```

