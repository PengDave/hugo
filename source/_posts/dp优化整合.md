---
title: dp优化整合
author: PengDave
date: 2025-07-14 19:30:04
tags:
mathjax: true
---
# 斜率优化

## P3195 [HNOI2008] 玩具装箱

考虑 dp，令 $s_i=\sum^{i}_{k=1}c_k$，$dp_i$ 为 $1$ 到 $i$ 装箱的最小代价，则转移方程为 $dp_i=\min dp_j+(i-j-1+s_i-s_j-L)^2(1\le j<i)$。

此时，为了方便，我们将所有的 $s_i$ 加 $1$，将 $L$ 也加 $1$，那么式子简化为 $dp_j+(s_i-s_j-L)^2=dp_j+s_i^2-2s_is_j-2s_iL+(s_j+L)^2$。

此时，假设有 $j_0<j_1$ 且由 $j_1$ 转移更优，则 $dp_{j_0}+s_i^2-2s_is_{j_0}-2s_iL+(s_{j_0}+L)^2\ge  dp_{j_1}+s_i^2-2s_is_{j_1}-2s_iL+(s_{j_1}+L)^2$，即 $(s_{j_1}-s_{j_0})\times 2s_i\ge [dp_{j_1}+(s_{j_1}+L)^2]-[dp_{j_0}+(s_{j_0}+L)^2]$，由于显然 $s_{j_1}\ge s_{j_0}$，令 $Y_i=dp_i+(s_i+L)^2,X_i=s_i$，因此 $2s_i \ge \frac{Y_{j_1}-Y_{j_0}}{X_{j_1}-X_{j_0}}$。

此时，我们发现这个式子很像斜率，考虑当我们多了一个的决策点时，我们会希望这个决策点与上一个点形成的斜率越小越好（毕竟越小根据上面的式子会更优），也就是我们在维护一些，因此我们可以用单调队列来维护决策点。

```cpp
#include<iostream>
using namespace std;
typedef long long ll;
const long double eps=1e-11;
ll c[50010],dp[50010];
int n,L;
ll X(int j){return c[j];}
ll Y(int j){return dp[j]+(c[j]+L)*(c[j]+L);}
long double slope(int i,int j){return 1.0L*(Y(j)-Y(i))/(X(j)-X(i));}
int q[50010];
int main(){
    cin>>n>>L;
    L++;
    for(int i=1;i<=n;i++)cin>>c[i];
    for(int i=1;i<=n;++i)c[i]+=c[i-1]+1;
    int l,r;l=r=0;
    q[r++]=0;
    for(int i=1;i<=n;i++){
        while(l<r-1&&slope(q[l],q[l+1])-2.0L*c[i]<eps)l++;//不最优的就弹出
        dp[i]=dp[q[l]]+(c[i]-c[q[l]]-L)*(c[i]-c[q[l]]-L);
        while(l<r-1&&slope(q[r-2],q[r-1])-slope(q[r-2],i)>eps)r--;//使斜率尽量小
        q[r++]=i;
    }
    cout<<dp[n]<<endl;
    return 0;
}
```

# 决策单调性分治

决策单调性指决策非严格递增，这样的 dp 转移时的代价通常可写为 $cost(i,j)$ 的形式，且当 $a<b<c<d$ 时，$cost(a,d)+cost(b,c)$ 劣于 $cost(a,c)+cost(b,d)$（即四边形不等式，可记忆为包含劣于交叉）。

在对于形如 $dp_{i,j}$ 且第一维为转移层数且只能从上一层往下一层转移并具有决策单调性时，可使用决策单调性分治。决策单调性分治一般使用递归实现，对于状态 $[l,r]$ 以及其决策区间 $[vl,vr]$，我们取中点 $mid=\displaystyle\frac{l+r}{2}$，并暴力计算出其决策点 $m$，那么状态 $[l,mid)$ 的决策区间为 $[vl,m]$，状态 $(mid,r]$ 的决策区间为 $[m,vr]$，递归解决即可。递归层数显然为 $O(\log{n})$ 级别，每一层复杂度为 $O(n)$ 因此进行动态规划 $k$ 层的总复杂度为 $O(kn\log{n})$。

## P4360 [CEOI 2004] 锯木厂选址

决策单调性分治板子，证明一般打一下表就行了，当然也可以证四边形不等式。代码略。

## CF868F Yet Another Minimization Problem

这道题目暴力 dp 易想到状态 $f_{i,j}$ 表示序列长度为 $j$ 分 $i$ 段的最小费用，转移为 $dp_{i,j}=\displaystyle\min_{k=i-1}^{j-1}dp_{i-1,k}+cost(k+1,j)$。可以证明该 dp 有决策单调性，决策单调分治即可，注意这题 $cost$ 不好算，可采用莫队思想，维护左右指针 $l$ 和 $r$，将左右指针每次挪一格，这题挪一格的复杂度可做到 $O(1)$，因此总复杂度为 $O(kn\log{n})$。


# 数据结构优化

## 单调队列优化

一般用在决策在一段类似滑动窗口的区间内的线性 dp。其代价一般较为固定。

### P1725 琪露诺

单调队列优化板子，转移方程为 $dp_i=\displaystyle\max_{j=i-R}^{i-L}dp_j+A_i$，用单调队列维护最大的 $dp_j$ 即可。

```cpp
#include<iostream>
#include<algorithm>
#include<cstdlib>
#include<cmath>
#include<cstring>
using namespace std;
int st=0,ed=0,q[200010],dp[200010],a[200010];
int main(){
	cin.tie(0);
	ios::sync_with_stdio(false);
	int n,l,r;
	cin>>n>>l>>r;
	for(int i=0;i<=n;i++){
		cin>>a[i];
	} 
	for(int i=1;i<l;i++){
		dp[i]=-2147483648;
	}
	int ans=-2147483648;
	for(int i=l;i<=n;i++){
		while(ed>st&&dp[i-l]>dp[q[ed-1]]) ed--;
		q[ed++]=i-l;
		if(q[st]<i-r) st++;
		dp[i]=dp[q[st]]+a[i];
		if(i+r>n)ans=max(ans,dp[i]);
	}
	cout<<ans<<endl;
	return 0;
}
```

## 线段树优化

没什么好讲的，比较活。

# wqs 二分优化

wqs 二分一般用在形如有 $n$ 个物品选 $m$ 个求最大或最小消费的 dp 中。wqs 二分的条件是凸性，即当我们将选的个数作为横坐标，对应的消费作为纵坐标时，相邻点之间的斜率非严格递增（下凸）或递减（上凸），一般来说当要取最大时为上凸，反之为下凸，凸性一般比较难证，但当 $O(mn)$ 的斜率优化无法通过时，一般就得用 wqs 二分，当然，打表也是可以的。下图是一个下凸函数的示例：

![](https://cdn.luogu.com.cn/upload/image_hosting/92nw1dpt.png)

对于这样的问题，我们考虑用直线 $kx+b$ 来切凸包，设 $f(x)$ 为选 $x$ 个的消费，则切点 $i$ 满足 $f(i)-ki$ 最小或最大（看凸性，但也可以直接看我们是要最小化还是最大化代价）。因此对于每个斜率 $k$，我们可以先抛开个数限制进行 dp，每分一组消费要减 $k$，最优的组数便是切点。

由于斜率是单调的，因此我们可以考虑二分斜率，例如当上凸时，如果求得的切点小于 $k$ 则斜率大了，反之小了。最终的答案为 $dp_n+km$。

## P5308 [COCI 2018/2019 #4] Akvizna

考虑 wqs 二分，可发现该函数为上凸，再加上斜率优化即可。

```cpp
#include<iostream>
#include<cstdio>
#include<cmath>
#include<algorithm>
#include<cstring>
#include<iomanip>
using namespace std;
const long double eps=1e-18;
const int N=1e5+10;
int n,k;
long double dp[N];int h[N],q[N];
long double X(int j){
    return 1.0L/(long double)(n-j);
}
long double Y(int j){
    return dp[j]-(long double)j/(n-j);
}
bool check(long double mid){
    int l=1,r=0;
    q[++r]=0;
    for(int i=1;i<=n;i++){
        while(l<r&&Y(q[l+1])-Y(q[l])>=-i*(X(q[l+1])-X(q[l])))l++;
        int j=q[l];
        h[i]=h[j]+1;
        dp[i]=dp[j]+(long double)(i-j)/(n-j)-mid;
        while(l<r&&(Y(i)-Y(q[r-1]))*(X(q[r])-X(q[r-1]))>=(Y(q[r])-Y(q[r-1]))*(X(i)-X(q[r-1])))r--;
        q[++r]=i;
    }
    return h[n]<=k;
}
int main(){
    cin>>n>>k;
    long double l=0,r=1,mid;
    while(fabs(r-l)>eps){
        mid=(l+r)/2.0L;
        if(check(mid))r=mid;
        else l=mid;
    }
    cout<<fixed<<setprecision(9)<<dp[n]+mid*k<<'\n';
    return 0;
}
```
