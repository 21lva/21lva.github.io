---
layout: post
title:  "Graph part 2"
date:   2019-01-25 02:00:00
author: Inhyuk
category: Algorithms
tags:	Algorithms
cover:  "/assets/instacode.png"
name: 2019-01-25-Graph2.md
---

최단 경로 탐색
=============

집에서 도서관까지 간다고 가정해보자. 택시를 타는 경로와 버스를 타는 경로는 같은 길을 따라가지 않는다. 그 길을 서로 길이가 다를 뿐만 아니라 도로의 상태도 다를 것이고, 혼잡도도 다를 것이다. 이 모든 것들은 집에서 도서관을 가는 데에 걸리는 시간에 영향을 미친다.

이번 포스트에서는 출발지점으로부터 모든지점으로 까지 가는 데에 걸리는 d최소의 경로의 거리를 찾는 **dijkstra algorithm** 을 다룰 것이다.

- - -

Dijkstra algorithm
==================

알고리즘의 기본적인 생각은 간단하다. minimum spanning tree를 구하는 algorithm 중 **Prim's algorithm** 과 매우 유사하다.

**Prim** 에서는 edge를 연결할 때에 그 edge의 값만을 비교하여 가장 작은 edge를 추가하는 방식이고, **Dijkstra** 는 그 edge까지의 거리(edge까지의 edge들의 weight들의 합)을 비교하여 가장 작은 edge들을 추가한다.

구조
----

1. 모든 node의 key들을  $ \inf $ 로, 집합  $ A $ 를 공집합으로 초기화한다.
2. 출발 지점의 key값을 0으로 초기화 하고, (node,key)를  $ A $ 에 넣는다.
3.  $ A $ 에서 key값을 비교하여 가장 작은 node  $ x $ 를 뽑는다.
4.  $ x $ 와 연결되어 있고,  $ B $ 에 들어있지 않은 node  $ y $ 들의 key를 다음과 같이 update한다.  $ y.key = \begin{cases}
y.key & \text{ if } y.key < x.key+w(x,y)  \\
x.key+w(x,y) & \text{otherwise }
\end{cases} $ (BFS를 실행한다고 보면 된다.)
5.  $ A $ 가 공집합일때까지 반복한다.

![di]({{site.baseurl}}/post_img/{{page.name}}/di.gif)

비용
---

위의  $ A $ 를 priority queue를 이용하여 구현하면,  $ O(ElogV) $ 의 시간복잡도를 가진다.

최대로 queue에 들어갈 수 있는 edge의 수는  $ E $ 이고, 최소값을 구하는 방법은  $ O(logE)=O(logV^2) $ 이다. (그러므로 반복문은 총  $ O(E) $ 개)

고려 사항
---------

중요하게 생각해야 할 부분은 weight들이 모두 양수이어야 한다는 것이다.
음의 weight를 가지는 edge가 존재한다고 가정해보자.

어떤 특정한 시간에  $ x $ 가 뽑혔고,  $ w(x,y) $ 가 음수이면,  $ x.key>y.key $ 이다. 그렇다면,  $ y $ 는  $ x $ 보다 먼저 뽑혔어야 한다.

 $ x $ 보다  $ y $ 가 먼저 뽑혔어야 한다는 얘기는  $ x $ 의 지금의 key가 최소값을 보장하지 않는다는 말이다.
즉, 알고리즘이 제대로 작동을 하고 있지 않다는 뜻이다.
또는 이 문제를 해결하기 위하여 *이미 검사한 node는 건너뛴다.* 라는 조건이 없어지면, 계속해서  $ x $ 와  $ y $ 를 왔다갔다 할 것이다.(반복할 수록 key값이 작아진다)

해결 방법으로는 최소값으로 다 더해서 양수를 만들어 주거나, 아래에 나올 알고리즘을 사용하는 것이다.

구현
----

```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <set>

#define INF 99999
#define numNode 6

using namespace std;

typedef struct _comp {
	bool operator()(pair<int, int> x, pair<int, int> y) {
		return x.second > y.second;
	}
}comp;

typedef struct _node{
	struct _node * parent;
	int level;
}node;

int G[numNode][numNode] = { {0,2,3,0,0,0},{2,0,5,3,4,0},{3,5,0,0,4,0},{0,3,0,0,2,3},{0,4,4,2,0,5},{0,0,0,3,5,0} };
vector<int> key(numNode);
bool visited[numNode] = { false, };
priority_queue<pair<int,int>, vector<pair<int,int> >, comp> Q;

int main() {
	int start;
	cin >> start;
	for_each(key.begin(), key.end(), [](int &x) {x = INF; });
	key.at(start) = 0;
	Q.push(pair<int, int>(start, key.at(start)));
	while (!Q.empty()) {
		pair<int,int> x = Q.top();
		Q.pop();
		for (int next = 0; next < numNode; ++next) {
			if (G[x.first][next] == 0||visited[next])continue;
			key.at(next) = (key.at(next) <= x.second + G[x.first][next]) ? key.at(next) : x.second + G[x.first][next];
			Q.push(pair<int, int>(next, key.at(next)));
		}
		visited[x.first] = true;
	}
	for_each(key.begin(), key.end(), [](int &x) {cout<<x<<endl; });

	return 0;
}
```

#### 결과
0
2
3
5
6
8


- - -

Bellamn-Ford Algorithm
======================

다익스트라 알고리즘은 양의 weight의 그래프에서 특정 지점으로부터 다른 모든 지점까지의 거리를 찾는 알고리즘이었다. 벨만포드 알고리즘은 음수를 다룰 수 있다.

![bell]({{site.baseurl}}/post_img/{{page.name}}/bell.png)

눈에 띄는 점은 *Relax* 이다. Relax는 위의 다익스트라에서도 나온 것 처럼 아래의 update를 말한다.

$$
y.key = \begin{cases}
y.key & \text{ if } y.key < x.key+w(x,y)

x.key+w(x,y) & \text{otherwise }
\end{cases}
$$


각 node에 대해서 다익스트라와 같은 방식으로 진행을 한다.(중요한 것은 이미 update한 것도 다시 update한다. 이는 다익스트라에서 이루어지지 않던 과정이다). 이렇게 하고 마지막 반복문에서 음의 순환을 찾는다(다익스트라에서 문제가 된 부분. 다익스트라는 그냥 문제만 만들지만, 벨만포드에서는 있는지 없는지를 찾는다.)

시간 복잡도
-------------

각  node에 대해서 반복을 진행한다.( $ O(V) $ ), relax는  $ O(E) $ 가 걸리므로, 시간 복잡도는  $ O(VE) $ 이다.
