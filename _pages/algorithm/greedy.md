---
layout: post
title:  "Greedy 알고리즘"
date:   2019-01-29 02:00:00
author: Inhyuk
category: Algorithms
tags:	Algorithms
cover:  "/assets/instacode.png"
name: 2019-01-29-greedy.md
---

탐욕 알고리즘은 부분문제의 해답을 최적의 해답의 부분으로 생각하고 문제를 해결해 나간다. 그러므로 탐욕알고리즘이 적용되는지 알려면 정말로 최적의 해답이 부분문제의 최적의 답으로 이루어져 있는지를 알아야 한다. 또한, dynamice programming에서 처럼 부분문제의 답을 이용(최적이긴 하지만)하므로 최적 부분 구조를 갖고 있어야 한다.

탐욕 알고리즘은 3단계를 통하여 진행된다.
1. 부분문제의 최적의 답을 구한다.
2. 부분문제의 해답이 문제의 제약조건을 위반하지 않는지를 검사한다. 위반시 1 로 돌아간다.
3. 전체 문제의 해답이 되는지를 확인한다. 아닐 시 1 로 돌아간다.

- - -

예시
====

크루스칼 알고리즘
----------------

크루스칼 알고리즘은 minimum spanning tree를 만드는 알고리즘이었다.

Graph의 모든 edge들을 오름차순으로 정리하고, 최소의 값이 사이클을 형성하지 않으면 그 edge를 tree에 추가하였다. 모든 node들이 포함되면 반복을 그만두었다.(edge가 사이클을 만드는지는 edge에 연결되어있는 두 node의 조상이 같은지 아닌지를 확인하여 알아내었다)

탐욕 알고리즘의 관점에서 보자.
1. 부분 문제의 해답(최소의 값)을 얻어낸다.
2. 최적의 edge가 사이클을 형성하는지 확인한다.
3. 최적의 edge을 추가할 시 tree를 형성하는지 확인한다.

다익스트라 알고리즘
------------------

다익스트라 알고리즘은 양의 weight를 가지는 graph에서 특정 node로부터 다른 모든 node로의 최소의 경로를 구하는 알고리즘이다. 크루스칼과 비슷하게 탐욕알고리즘의 관점으로 보자.

1. 현재 priority queue에서 key 값이 가장 작은 node를 선택한다.
2. 선택한 node가 이미 탐색을 마친 것인지를 확인한다.
3. 모든 node가 탐색을 마쳤다면 그만한다. 아닐 시 key값을 비교+업데이트하고 1로 돌아간다.

허프만 코딩
----------

많은 포맷(JPEG)에서 사용되는 데이터 압축기술이다.

### Prefix code

접두어 코드(prefix code)는 글자에 상관없이 사용하는 데이터의 길이가 항상 일정한 고정길이 코드(e.g. ASCII)와 다르게 가변적인 데이터 길이를 사용한다. 구별을 위해서 접두어 코드에서는 모든 코드(글자)들은 다른 코드의 접두어가 되지 않는다.

예를 들어, 집합 $$A=\{00,010,100,101\}$$의 모든 요소들은 다른 코드의 접두어(앞부분)이 되지 않으므로 집합 $A$는 접두어 코드로 사용할 수 있다. 반면에 집합 $$B=\{01,011,010\}$$은 $$01$$이 $$011$$의 접두어로 사용할 수 없다.

가변적인 길이를 사용하므로 고정길이 코드보다 같은 정보를 저장하는 데에 더 적은 양을 저장한다. 또다시 예를 들어 살펴보면, 32bit를 사용하는 경우에 *abc* 를 저장하기 위해서는 32*3=96bit가 필요하다. 하지만, *a* 를 00에 *b* 를 010에 *c* 를 100에 배치하면 단 8bit만을 사용하여 데이터를 표현할 수 있다.

### 허프만 Tree

허프만 코딩을 하기 위해서는(압축,해제) 허프만 트리를 이용하면 된다.
허프만 트리는 이진 트리로 각 leaf를 데이터로 사용하면 된다. 그렇게 되면 모든 데이터는 다른 데이터의 접두어가 되지 않는다.

하지만 아무렇게 배치를 하는 경우에는 딱히 좋은 방법이라고 할 수 없다. 많이 나오는 글자를 가장 높은 위치의 leaf에 할당을 해야 적은 양의 메모리를 사용하여 데이터를 표현할 수 있다.

### 탐욕 알고리즘과 무슨 관련?

허프만 코딩을 하기 위해서 허프만 트리를 만들어야하고, 사용빈도가 높을 수록 높은 위치에 배당해야 한다.
이를 선택하는 과정은 다음과 같이 이루어진다.

1.  가장 적게 사용하는 데이터 두개를 선택하여 key를 빈도수로 하고, 둘을 하나의 부모노드의 자식으로 만든다. 이 때 부모노드의 key값은 자식의 key값의 합이다.
2. 부모 노드를 원래의 데이터 집합에 넣고 1을 모든 노드가 다 쓰일 때까지 반복한다.

이를 위의 탐욕알고리즘의 3단계의 관점에서 보면,

1. 부분해를 구한다: 빈도수가 가장 작은 두 데이터를 선택한다.
2. 제약조건을 검사: 데이터 node가 leaf노드인지 확인한다.(언제나 통과)
3. 해결 검사: 전체 데이터를 사용하였는지 검사

### 압축

압축 방법은 간단하다.

1. 빈도수를 계산하고 허프만 트리를 형성하여 각 데이터에 접두어 코드를 할당한다.
2. 허프만 코드를 바탕으로 데이터를 쓴다.

### 해제

압축에 사용된 허프만 트리를 이용하여 압축된 데이터 열을 쭉 읽으면서 해제한다.

### 구현

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <time.h>
#include <queue>
#include <stack>

using namespace std;

string Target= "abcdsfqgqgwewggwqwgtkio";
string table[26];
int number_used[26] = { 0 , };

class node {
public:
	bool is_leaf;
	char value;
	int num_used;
	node *left;
	node *right;
	node(bool Is_leaf=true):is_leaf(Is_leaf),value(' '),num_used(0),left(nullptr),right(nullptr) {	}
};


struct cmp {
	bool operator()(node *x, node *y) {
		return x->num_used > y->num_used;
	}
};

void num_used(const string &Target) {
	for (int i = 0; i < 26; i++)number_used[i] = 0;
	for_each(Target.begin(), Target.end(), [](const char &x) {number_used[x - 97]++; });

}

void huffman_table(node *root,deque<char> &dq) {
	if (root->is_leaf) {
		table[root->value - 97] = "";
		for (deque<char>::iterator iter = dq.begin(); iter != dq.end(); ++iter) { table[root->value - 97] += (*iter); }
	}
	else {
		dq.push_back('0');
		huffman_table(root->left, dq);
		dq.pop_back();
		dq.push_front('1');
		huffman_table(root->right, dq);
		dq.pop_back();
	}
}

void make_tree(const string &Target) {
	num_used(Target);
	priority_queue< node*, vector<node*>, cmp> pq;
	for (int i = 0; i < 26; i++) {
		if (number_used[i] == 0)continue;
		node *x=new node;
		x->num_used = number_used[i];
		x->value = i + 97;
		pq.push(x);
	}

	while (pq.size()!=1) {
		node *x=pq.top();
		pq.pop();
		node *y=pq.top();
		pq.pop();
		node *mother=new node(false);
		mother->left = x;
		mother->right = y;
		mother->num_used = x->num_used + y->num_used;
		pq.push(mother);
	}

	deque<char> dq;
	dq.push_back('0');

	huffman_table(pq.top(), dq);
	string encoded_string="";
	for (int i = 0; i < Target.length(); ++i)encoded_string+=table[Target.at(i) - 97];
	cout << encoded_string << endl;
	cout << Target.length()<<"->"<<encoded_string.length() << endl;

}

int main() {
	make_tree(Target);
	system("pause");
	return 0;
}
```

#### 결과
11100011110011110011111011100011110011111001111100000111110000100100000111100010011111011111111000111111
23->104

원래였으면 23*32bit를 사용해야 하지만 허프만 코딩을 사용하면 104bit로 표현 가능하다.

교실 할당 문제
---------------

하나의 교실을 여러개의 수업에 배정을 하는 문제이다. 이 때, 교실을 사용하는 수업의 수(수업의 길이와 상관 없다)를 가장 크게 하는 경우를 고르는 문제이다.

모든 수업은 2가지의 값(시작 시각, 종료 시각)을 가지고 있다. 탐욕알고리즘을 사용하기 위해서는 3가지의 고려사항이 생기는 것이다.

1. 시작 시각이 가장 먼저 인 것을 고른다.
2. 사용시간(종료시각 - 시작시각)이 가장 긴 것을 먼저 선택한다.
3. 종료 시각이 가장 먼저인 것을 선택한다.

첫 번째의 경우에는 반례가 존재한다.

예를 들어, $x_0=(0,3),x_1=(1,2),x_2=(2,3)$인 경우에 $x_1,x_2$이 선택되는 것이 더 좋은 선택이지만 시작 시간이 먼저인 것을 기준으로 하면 $x_0$이 선택될 것이다.

두 번째의 경우에도 문제가 발생한다.

예를 들어, $x_0=(0,2),x_1=(1,9),x_2=(4,11)$이 있는 경우에 $x_0,x_2$를 선택하는 것이 $x_1$를 선택하는 것보다 좋지만 정작 선택 기준에 의해 $x_1$이 선택이 된다.

마지막의 경우에는 당연히 최적의 선택을 가져온다.
class 들의 집합을 $A=\{x_0,x_1,...,x_n\}$라고 하자. $i$번째 선택(가장 먼저 종료되는 것)에 의해 만들어진 집합을 $A_i(\subsetA)$라고 하자.
이 때, $i$번째 선택되는 $a_i$가 아닌 다른 $b_i$를 선택한다고 가정하자. 만약에 새로 만들어진 $\hat{A_i}$가 $A_i$보다 좋은 선택이면 마지막 선택 기준도 좋지 못한 것이다.

$a_i$가 선택되었다는 말은 $b_i$의 종료시각이 $a_i$의 종료시각보다 느리다는 의미이다. 그러므로 $b_i$이 후에 선택 가능한 $A$의 부분집합은 당연히 $a_i$가 선택되고 그 다음부터 선택 가능한 $A$의 부분집합이므로 $i$번째 이후의 선택에서는 $a_i$를 선택하는 것이 더 좋다.(가능성이 넓어진다)

$i$번째 선택 이전에 선택된 수업들은 같았고, $i$번째 선택에서 똑같이 하나씩 선택하였으므로 $a_i$를 선택하는 것이 $b_i$를 선택하는 것보다 나쁠 수는 없다.

여기서 중요한 것은 "나쁠 수는 없다"이다. 탐욕 알고리즘에서 항상 하나의 해만을 고려하기 때문에 여러 해답이 있는 경우에 그 모든 것들을 찾을 수는 없다.

적용을 해보자.

|index|0|1|2|3|4|5|6|7|8|
|------|-|-|-|-|-|-|-|-|-|
|시작시각|0|2|4|1|5|6|10|7|
|종료시각|4|3|9|2|6|15|12|13|

위의 수업들을 종료시각이 가장 작은 순으로 나열하면 다음과 같다.

|index|3|1|0|4|5|2|7|8|6|
|------|-|-|-|-|-|-|-|-|-|
|시작시각|1|2|0|5|4|10|7|6|
|종료시각|2|3|4|6|9|12|13|15|

탐욕 알고리즘을 적용하자.

1. 부분 답을 구한다(현재 선택할 수 있는 것 중 가장 작은 것을 고른다)
2. 제약조건을 확인한다(이미 선택된 것들과 겹치는지 확인한다)
3. 종료 조건을 확인한다(모두 고려대상으로 확인하였는지 확인한다)

결국, *3,1,4,7* 이 선택될 것이다. 하지만 이 답만 최적의 답은 아니다.

*3,1,5,6* 을 고르는 것도 최적의 해이다.

- - -

탐욕알고리즘의 한계
==================

탐욕 알고리즘이 항상 옳은 것은 아니다. 탐욕 알고리즘을 사용할 수 있는 경우에는 위에서 설명하였듯이 부분해의 최적의 해가 항상 전체 해의 최적의 해에 포함될 때 이다.
다음은 그렇지 않은 경우의 예시로 많이 사용되는 **0-1 knapsack problem** 이다.

0-1 knapsack problem
--------------------

**0-1 knapsack problem** 은 무게와 가치를 가지는 여러 물건 중에 선택하는 문제로 고른 물건들의 무게의 총합이 주어진 한계를 넘어서지 않아야 하며 와중에 선택한 물건들의 가치의 합은 최대로 만들어야 한다. **0-1** 인 이유는 이 물건들을 쪼갤 수는 없다는 뜻이다.

탐욕 알고리즘을 적용하고 싶으면 각 부분 문제에서의 선택 기준이 있어야 한다. 여기서도 선택할 수 있는 것은 세 가지이다.

1. 무게가 덜 나가는 것을 먼저 선택한다.
2. 가치가 높은 것을 먼저 고른다.
3. 무게 대비 가치($\frac{\text{가치}}{\text{무게}}$)가 높은 것을 선택한다.

첫 번째 경우에는 한 눈에 보기에도 고른 물건들이 최고의 가치가 아닐 수도 있다는 것이다. 100g짜리 초콜릿을 100kg담아봐야 다이아몬드 1kg담는 것이 더 가치가 있다. 반대로 무게가 많이 나가는 것도 의미 없다. 물 아무리 담아봐야 다이아 1kg이 더 가치 있다.(귀찮아서 이렇게 설명하는 것은 아니다.)

두 번째 경우에는 다음의 예시를 보면 알 수 있다.

100kg을 담을 수 있는 가방이 있을 때, 무게 70kg, 가치 80의 TV와 무게 50 가치 50의 플스, 무게 50 가치 50의 닌텐도가 있다고 하면 당연히 플스+닌텐도를 가져오는 것이 좋은 선택이다(가치100). 하지만 이 기준을 사용하면 그보 덜한 TV(가치 70)을 가져오게 된다.

마지막 경우에도 위의 예시에 적용하면 똑같은 결과를 얻게 된다.(TV의 가치/무게 는 1보다 크지만 나머지는 전부 1이다)

**0-1 knapsack problem** 은 탐욕알고리즘을 적용할 수 없는 대표적인 문제이다. 이 문제를 해결하는 방법은 DP를 사용하는 것이다.

우리가 얻을 수 있는 가치는 남은 가용 weight(넣을 수 있는 공간,무게)와 현재 까지 고려한 요소들로 구성된다.

$$
maxvalue(W,i) = \begin{cases}
0 & \text{ if } i>N\text{ or }W==0\\
maxvalue & \text{ if } w_i>W\\
max\{maxvalue(W-w_i,i+1)+w_i,maxvalue(W,i+1)\}\\
(N\text{:the number of items})
$$
