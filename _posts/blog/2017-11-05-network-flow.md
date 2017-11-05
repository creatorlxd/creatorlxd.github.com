---
layout: post
title: 网络流模板
categories: [OI,图论,网络流]
tags: 模板
---

# 网络流模板

&emsp;&emsp;既然是模板，就没什么好说的。
```c++
/**
 * 网络流模板
 * @author:creatorlxd
*/
#include <iostream>
#include <vector>
#include <queue>
#include <utility>
#include <algorithm>
using namespace std;
#define INF (1<<30)

struct edge
{
	int next=-1;
	int to=-1;
	int cap=0,flow=0;
};
vector<int> head;
vector<edge> edges;
vector<int> level;
int edge_cot=0;
int n,m,s,t;

void add_edge(int from,int to,int c)
{
	edges[edge_cot].next=head[from];
	edges[edge_cot].to=to;
	edges[edge_cot].cap=c;
	edges[edge_cot].flow=0;
	head[from]=edge_cot++;
	edges[edge_cot].next=head[to];
	edges[edge_cot].to=from;
	edges[edge_cot].cap=0;
	edges[edge_cot].flow=0;
	head[to]=edge_cot++;
}

bool bfs()
{
	level.clear();
	level.resize(n,0);
	level[s]=1;
	queue<int> que;
	que.push(s);
	while(!que.empty())
	{
		int point=que.front();
		que.pop();
		for(int i=head[point];i!=-1;i=edges[i].next)
		{
			int to=edges[i].to;
			if(!level[to]&&edges[i].cap>edges[i].flow)
			{
				level[to]=level[point]+1;
				que.push(to);
			}
		}
	}
	return level[t]!=0;
}

int dfs(int now,int max_flow)		//return ans		max_flow mean
{
	if(now==t)
		return max_flow;
	else
	{
		int addiction=0;
		for(int i=head[now];i!=-1&&addiction<max_flow;i=edges[i].next)
		{
			int to=edges[i].to;
			if(level[to]==level[now]+1&&edges[i].cap>edges[i].flow)
			{
				int tmp=dfs(to,min(max_flow-addiction,edges[i].cap-edges[i].flow));
				edges[i].flow+=tmp;
				edges[i^1].flow-=tmp;
				addiction+=tmp;
			}
		}
		return addiction;
	}
}

int main()
{
	cin>>n>>m>>s>>t;
	s-=1;
	t-=1;
	int ans=0;
	head.resize(n,-1);
	edges.resize(2*m);
	for(int i=0;i<m;i++)
	{
		int start,end,k;
		cin>>start>>end>>k;
		start-=1;
		end-=1;
		add_edge(start,end,k);
	}
	while(bfs())
	{
		ans+=dfs(s,INF);
	}
	cout<<ans<<endl;
	return 0;
}
```