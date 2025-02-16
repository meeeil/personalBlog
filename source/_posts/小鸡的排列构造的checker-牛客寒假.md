---
title: 小鸡的排列构造的checker-牛客寒假
date: 2025-02-16 17:08:42
categories:
  - 算法竞赛
tags: 
  - 离线处理
  - 树状数组
excerpt: 题解
math: true
---
### 题目大意

给定 $T$ 组数据，每组数据包含一个长度为 $n$ 的排列和 $m$ 个查询。每个查询给定区间 $[l, r]$ 和位置 $c$，要求将排列在区间 $[l, r]$ 的元素升序排列后，原位置 $c$ 的元素的新位置。

### 问题转化

我们需要找到区间排序后，原位置 $c$ 的元素在有序区间中的相对位置。设原位置 $c$ 的值为 $v$，则新位置可转化为：

$$
\text{新位置} = (l-1) + \text{区间}[l,r]中 \leq v 的元素个数
$$

因此问题转化为**快速统计每个查询区间内比 $v$ 小的元素数量**。

---

## 解决

### Part1: 离线处理思想

我们采用**离线处理**策略：

具体步骤：
- 预处理每个数值对应的原始位置
- 将所有查询按目标值 $v$（即查询位置 $c$ 的值）排序
- 随着数值从小到大依次插入树状数组，遇到对应 $v$ 的查询时立即处理

### Part2: 树状数组优化

使用树状数组维护插入元素后的区间和信息：

```cpp
int lowbit(int x) { return x & -x; }

void add(int x, int p){// 在位置x插入元素
	while(x<=n){
		q[x]+=p;
		x=lowbit(x)+x;
	}
}

int getsum(int x){// 查询前缀和
	int ans=0;
	while(x){
		ans+=q[x];
		x=x-lowbit(x);
	}
	return ans;
}
```

---

### 参考代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN=3e5+10;

int T,n,m,cnt,idxx[MAXN],p[MAXN],q[MAXN];
int lowbit(int a){
	return a&(-a);
}
int getsum(int x){
	int ans=0;
	while(x){
		ans+=q[x];
		x=x-lowbit(x);
	}
	return ans;
}
void add(int x, int p){
	while(x<=n){
		q[x]+=p;
		x=lowbit(x)+x;
	}
}
struct Node{
	int l,r,c,idx,ans;
} msg[MAXN];
int cmp(Node a, Node b){
	if(p[a.c]!=p[b.c]){
		return p[a.c]<p[b.c];
	}
	else if(a.l!=b.l){
		return a.l<b.l;
	}
	else{
		return a.r<b.r;
	}
}
int cmp1(Node a, Node b){
	return a.idx<b.idx;
}

int main(void){
	// ios::sync_with_stdio(false);
	// cin.tie(0);cout.tie(0);
	cin>>T;
	while(T--){
		cnt=1;
		memset(q, 0, sizeof(q));
		cin>>n>>m;
		for(int i=1;i<=n;i++){
			cin>>p[i];
			idxx[p[i]]=i;
		}
		for(int i=1;i<=m;i++){
			msg[i].idx=i;
			cin>>msg[i].l>>msg[i].r>>msg[i].c;
		}		
		sort(msg+1,msg+m+1,cmp);
		for(int i=1;i<=n;i++){
			add(idxx[i],1);
			while(cnt<=m&&i==p[msg[cnt].c]){
				msg[cnt].ans=getsum(msg[cnt].r)-getsum(msg[cnt].l-1)+msg[cnt].l-1;
				cnt++;
			}
		}
		sort(msg+1,msg+1+m,cmp1);
		for(int i=1;i<=m;i++){
			cout<<msg[i].ans<<"\n";
		}
	}
}
```

### 复杂度分析

- **时间复杂度**：$O(T*(n + m \log m))$  
  每个数值插入树状数组 $O(\log n)$，处理查询 $O(m \log m)$