---
layout: post
title:  "Graph part 1"
date:   2019-01-02 02:00:00
author: Inhyuk
category: Algorithms
tags:	Algorithms
cover:  "/assets/instacode.png"
name: 2019-01-12-Graph1.md
---

Graph
=====

정의
----
1.  $ V $ : node(정점,vertex)의 집합
2.  $ E $ : node와 node사이의 edge의 집합( $ (v_i,v_j) $ 꼴, directed graph의 경우 왼쪽에서 오른쪽의 node로 방향성을 가지는 edge 존재)

그래프는  $ G=(V,E) $ 의 tuple을 의미한다. 즉, node들과 그 node들을 잇는 edge들의 집합을 그래프라고 한다.

3. Path from  $ v_i $  to  $ v_j $ : 그래프내에서  $ v_i $ 로부터  $ v_j $ 로 가는 경로이다. 수학적으로 보면,


$$
\text{경로 p: E의 부분집합} {e_1,..,e_n} \text{의 나열한 것 중 하나로,}

e_1=(v_i,v_{k_1}),e_2=(v_{k_1},v_{k_2}),...,e_n=(v_{k_{n-1}},v_j) \text{을 만족하고,}

\text{모든 v들(시작과 끝은 같아도 상관 없음)은 서로 다르며,}

\text{path에 속한 모든 edge들이 서로 다를 때, p를}

v_i \text{에서 } v_j \text{로의 경로라고 부른다.}
$$

4. Cycle : path 중 시작과 끝이 같은 경우를 의미한다.
5. Trail : path보다 큰 집합으로, path의 정의에서 node들이 서로 다르다는 것을 빼면 된다.
6. Walk : trail보다 큰 집합으로, edge들의 중복배제 조건도 삭제하면 된다.
7. Circuit : Cycle의 node 중복배제 조건을 삭제한 경우.


표현 방법
---------

수학에서는 그냥 그래프를 그리고 가져다 쓰면 되지만, 컴퓨터에서는 그렇게 추상적으로 정의를 할 수 없기 때문에 그래프의  $ V,E $ 를 어떤 방법으로 구체적으로 정의해야 한다.

#### Adjacency Matrix(인접행렬)

정점끼리의 연결관계를 행렬로 표현하는 방법.  $ v_i $ 로부터  $ v_j $ 로의 edge가 존재하면  $ i $ 행  $ j $ 열의 요소의 수가 증가한다. Undirected graph의 경우에는 symmetric matrix가 된다.

#### Adjacency List(인접 리스트)

연결관계를 **linked list** 로 표현하는 방법.  $ N $ 개의 node들이 존재하는 graph의 경우, N개의 **linked list** 를 만들고,  $ v_i $ 로부터  $ v_j $ 로의 edge가 존재하면  $ i $ 번째 **linked list** 에  $ j $ 를 추가한다.

![Adjacency]({{site.baseurl}}/post_img/{{page.name}}/adjacency.png)

#### 비교

* 인접 행렬은 어떤 node와 다른 node사이에 edge가 존재하는 지를 시간 복잡도  $ O(1) $안에 확인할 수 있는 장점이 있지만, graph가 **sparse** (graph의 node의 수에 비해 edge의 수가 많이 적은 경우,반대의 경우는 **dense** 라고 부른다)하면 메모리 낭비가 심해진다.
* 인접리스트는 **sparse** 한 경우에 적은 메모리로 저장을 할 수 있지만, 두 노드 사이의 edge가 존재하는 지를 확인하기 위하여 최악의 경우 시간이  $ O(N) , N:the number of nodes $ 만큼 걸릴 수가 있다.

따라서 상황에 맞게 표현 방법을 선택해야 한다.

Search
======

DFS
---

**Depth first search** 의 약자. 말 그대로 search를 진행할 때 막다른 곳에 닿을 때 까지 앞으로 계속 진행하는 방식이다. **stack** 을 이용하여 구현되며, **recursive** 한 방법을 많이 쓴다.
![dfs]({{site.baseurl}}/post_img/{{page.name}}/dfs.png)

```cpp
#include <iostream>
#include <stack>
#define numNode 5

using namespace std;

void dfs(int, int);

int G[numNode][numNode] = { {0,1,0,0,1},{1,0,1,1,1},{0,1,0,1,0},{0,1,1,0,1},{1,1,0,1,0} };
bool visited[numNode] = { false, };

int main() {
	//DFS from 1 with stack;
	int node = 0;
	stack<int> st;
	st.push(0);
	iter:
		node = st.top();
		st.pop();
		if (visited[node]) { goto iter; }
		cout << node+1 << " ";
		visited[node] = true;

		for (int i = 0; i < numNode ; i++) {
			if (!visited[i] && G[node][i]) {
				st.push(i);
			}
		}
		for (int i = 0; i < numNode; i++)if (!visited[i]) { goto iter; }
	cout << endl;

	//DFS from 1 with recursive function
	for (int i = 0; i < numNode; i++)visited[i] = false;
	dfs(0, 0);

	return 0;
}

void dfs(int node,int numUsed) {
	if (numUsed >= numNode)return;
	cout << node+1 << " ";
	visited[node] = true;
	for (int i = 0; i < numNode; i++) {
		if (!visited[i] && G[node][i]) {
			dfs(i, numUsed + 1);
		}
	}
}
```

| 1 5 4 3 2 |
| 1 2 3 4 5 |

BFS
---

**Breath frist search** 의 약자. 지금 있는 노드에 연결된 노드들을 먼저 탐색한 후 그 탐색한 노드들 중 하나로 움직여서 탐색을 계속하는 방식으로 진행한다. **queue** 를 이용하여 구현한다.

```cpp
#include <iostream>
#include <queue>
#define numNode 5

using namespace std;

void dfs(int, int);

int G[numNode][numNode] = { {0,1,0,0,1},{1,0,1,1,1},{0,1,0,1,0},{0,1,1,0,1},{1,1,0,1,0} };
bool visited[numNode] = { false, };

int main() {
	//DFS from 1 with stack;
	int node = 0;
	queue<int> q;
	q.push(0);

	iter:
		node = q.front();
		q.pop();
		if (visited[node]) { goto iter; }
		cout << node+1 << " ";
		visited[node] = true;

		for (int i = 0; i < numNode ; i++) {
			if (!visited[i] && G[node][i]) {
				q.push(i);
			}
		}
		for (int i = 0; i < numNode; i++)if (!visited[i]) { goto iter; }

	return 0;
}
```

| 1 2 5 3 4 |


Minimum spanning tree
=====================

이제 까지의 그래프들은 edge에 *weight* 이 곱해지지 않은 경우였다. 이제 다룰 내용은 edge에 *weight* 이라는 값이 가해진 **weighted graph** 이다.

예를 들어, 서울에서 인천으로 이동한다고 해보자. 길을 한 가지만 있는 것이 아니기 때문에 적절한 길을 선택해 나가야 최소한의 시간으로 움직일 수 있다. 이 길 마다 도로의 상태, 교통상황 등이 다르기 때문에 모든 길을 같게 취급해서 다룰 수는 없다.

이 때 길의 교통 상황 등을 고려하여 *weight* 라는 것을 추가할 수도 있다.

**Minimum spanning tree** 는 graph에서 만들 수 있는 tree중 edge들의 weight들의 합이 가장 작은 tree를 의미한다. 이 때, **Spanning tree** 는 그래프의 모든 node들을 포함하는 tree를 의미한다.

Prim's Algorithm
-----------------

**Minimum spanning tree** 를 찾는 방법 중 하나이다. 방법은 간단하다.
1.  $ M $ 를 공집합으로 만든다.
2.  $ M $ 에 어떤 node를 넣는다. 이 것이 root가 된다.
3.  $ M $ 에 들어있는 node들과  $ M $ 에 속하지 않는 node들 사이의 edge중 weight가 가장 작은 edge를 선택하고, 그 edge의 두 node 중  $ M $ 에 속하지 않은 node를 M에 추가한다.
4.  $ M $ 이  $ V $ 와 같아질 때까지 3을 반복한다.

이 방법을 다르게 표현하는 방식도 많이 사용한다.
1.  $ M $ 을  $ Q $  공집합으로 만든다.
2. 임의의 한 node(root)의 key값을 0으로 한다. 그 node를  $ Q $ 에 넣는다.
3.  $ Q $ 에서 key값이 가장 작은 node를 빼낸다. 해당 node를  $ M $ 에 넣는다.
4. 인접한 node들과의 edge값을 key로 하는 값을  $ Q $ 에 넣는다.
6.  $ M $ 이  $ V $ 와 같아질 때 까지 반복한다.

#### 시간복잡도
이 방법은  $ O(|V|)T_{extract min}+O(|E|)T_{decrease key} $ 의 시간 복잡도를 보인다.

|자료구조|시간복잡도|
|-------|---------|
|인접행렬| $ V^2 $ |
|priority queue| $ ElogV $ |
|Fibonacci heap| $ E+VlogV $ |

#### 증명

#### 구현
특정 edge 중 최소 값을 구하기 위해 **priority queue** 를 사용할 것이다.

```cpp
#include <iostream>
#include <queue>
#include <vector>
#define numNode 6

using namespace std;

struct comp {
	bool operator()(pair<int, int> x, pair<int, int> y) { return x.second > y.second; }
};

int G[numNode][numNode] = { {0,2,3,0,0,0},{2,0,5,3,4,0},{3,5,0,0,4,0},{0,3,0,0,2,3},{0,4,4,2,0,5},{0,0,0,3,5,0} };
bool visited[6] = { false, };
priority_queue<pair<int,int>, vector<pair<int, int> >, comp> Q;

int main() {
	int node = 0;
	int sum = 0;
	for (int i = 0; i < numNode; i++)Q.push(pair<int, int>(node, 0));
	iter:
		pair<int,int> node_key = Q.top();
		Q.pop();
		if (visited[node_key.first]) { goto iter; }
		visited[node_key.first] = true;
		sum += node_key.second;
		for (int i = 0; i < numNode; i++) {
			if (G[node_key.first][i] == 0 || visited[i])continue;
			Q.push(pair<int, int>(i, G[node_key.first][i]));
		}
		for (int i = 0; i < numNode; i++)if (!visited[i]) { goto iter; }
		cout << sum;
	return 0;
}
```
![prim]({{site.baseurl}}/post_img/{{page.name}}/prim.jpg)

|13|

Kruskal's Algorithm
--------------------

edge들을 오름차순으로 정렬하고, 정렬한 것들을 차례로 보면서 cycle을 만들지 않도록 tree를 만든다.

연결을 할 때 각 집합으로 생각한다. 연결을 할 때마다 합집합으로 생각을 하고 같은 집합내에서는 연결을 하지 않도록 한다. 연결을 하면 cycle을 형성하기 때문이다.

```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <set>
#define numNode 6

using namespace std;

typedef struct _comp {
	bool operator()(pair<pair<int, int>,int> x, pair<pair<int, int>,int> y) {
		return x.second > y.second;
	}
}comp;

typedef struct _node{
	struct _node * parent;
}node;

int G[numNode][numNode] = { {0,2,3,0,0,0},{2,0,5,3,4,0},{3,5,0,0,4,0},{0,3,0,0,2,3},{0,4,4,2,0,5},{0,0,0,3,5,0} };
bool visited[6] = { false, };
node* nodes[6];
priority_queue<pair<pair<int,int>,int>, vector<pair<pair<int, int>,int> >, comp> Q;

node* find_root(int x) {
	node* p1 = nodes[x];
	while (p1->parent != nullptr) { p1 = p1->parent; }
	return p1;
}

bool Is_same_set(int x, int y) {
	if (nodes[x]->parent == nullptr || nodes[y]->parent == nullptr)return false;
	return find_root(x) == find_root(y);
}

void union_set(int x, int y) {
	node* p1 = find_root(x);
	node* p2 = find_root(y);
  if (p1->level > p2->level) {
  p2->parent = p1;
  p1->level = (p1->level > p2->level + 1) ? p1->level : p2->level + 1;
}
else {
  p1->parent = p2;
  p2->level = (p2->level > p1->level + 1) ? p2->level : p1->level + 1;
}
}

int main() {
	int node = 0;
	int sum = 0;
	int ct = 0;
	for (int i = 0; i < numNode; i++) {
		nodes[i] = new struct _node;
		nodes[i]->parent = nullptr;
	}

	for (int i = 0; i < numNode; i++)
		for (int j = 0; j < i; j++) {
			if (G[i][j] == 0)continue;
			pair<pair<int, int>, int> tmp;
			tmp.first.first = i;
			tmp.first.second = j;
			tmp.second = G[i][j];
			Q.push(tmp);
			ct++;
		}

	while (ct > 0 && !Q.empty()) {
		pair<pair<int, int>, int> vvkey = Q.top();
		Q.pop();
		int v1 = vvkey.first.first;
		int v2 = vvkey.first.second;
		int w = vvkey.second;
		if (Is_same_set(v1, v2))continue;
		sum += w;
		ct--;
		union_set(v1, v2);
	}
	cout << sum << endl;

	return 0;
}
```

서로 같은 set인지 확인하기 위하여 [Union-find]({{site.baseurl}}/_posts/)이용하였다.

#### 시간 복잡도

시간 복잡도를 계산해보자. 가장 먼저 정렬 알고리즘이 $O(ElogE) $ 만큼 걸린다. 또한, while 반복문은  $ E $ 만큼 행해진다. 각 반복에서 union-find 연산이  $ logV $ 만큼 걸리므로 전체적으로 보았을 때 **Kruskal algorithm** 의 시간 복잡도는  $ O(ElogV)=O(ElogE) $ 이다.
