---
layout: post
title:  "Bit mask"
date:   2019-01-29 02:00:00
author: Inhyuk
category: Algorithms
tags:	Algorithms
cover:  "/assets/instacode.png"
name: 2019-01-29-bit_mask.md
---

이 포스트는 bitmask에 대해 아주 잘 정리되고 문제 풀이도 어느 정도 있는 이 [블로그](https://m.blog.naver.com/jh20s/221150706030)를 많이 참고하였다.

Bitmask
========

bitmask는 bit를 이용하는 기법이다. 주로 알고리즘 문제에서 존재여부, 사용여부 같은 것들을 확일할 때 쓰인다.

C++ bit 연산자
-------------

```cpp

#include <iostream>
#include <bitset>
using namespace std;
int main() {
	int a = 7, b = 5;
	cout << bitset<3>(a | b) << endl; // a와 b의 OR연산
	cout << bitset<3>(a&b) << endl; //a와 b의 AND연산
	cout << bitset<3>(a^b) << endl; //a와 b의 XOR연산(같으면 0 다르면 1)
	cout<<bitset<4>(1<<3)<<endl;//왼쪽 쉬프트,뒷부분은 0으로 채운다.
	cout << bitset<3>(~b) << endl; //b의 1인 부분은 0으로 0인 부분은 1로 바꾼다.)
	system("pause");
}
```

#### 결과
111

101

010

1000

010



집합으로 사용하기
----------------

### 공집합, 전체 집합

```cpp
#include <iostream>
int empty =0;
int full_set = (1<<20)-1; // 1을 20만큼 밀고 1을 빼므로 1이 19개 있는 숫자가 된다.
```

### 원소 다루기

```cpp
#include <iostream>
set |= (1<<x);//위치 x에 원소 추가
set&(1<<x) ;//위치 x에 1이 없으면(원소가 없으면) 0이 된다.
set&=~(1<<x);//위치 x에 원소가 존재하면 삭제
```

### 집합 연산

```cpp
(a|b);// a U b
(a&b); // a 교집합 b
(a&~b); //  a\b(차집합)

//원소의 갯수 구하기
int cnt=0;
for(int i=set;i!=0;set=(set>>1))if(set&1)cnt++;

//가장 작은 값 찾기
set&(-set);//보수를 사용

//부분 집합
for(int subset = set ;subset ; subset = (subset-1)&set)
```

이용하기
-------

1. 알고스팟 문제

[알고스팟 문제](https://algospot.com/judge/problem/read/GRADUATION)에 나와 있는 문제를 bitmask를 이용하여 풀었다.

bitmask를 이제껏 배운 수업과, 열리는 수업, 선수강 과목을 표현하는 데에 사용하였


```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <time.h>
#include <queue>
#include <stack>

#define MAXNUM 9999

using namespace std;

int C,N, K, M, L;
int cache[20][1<<20-1];
int open_subject[20];
int prior[20];


int f(int semester,int already) {
	if (cache[semester][already] != -1)return cache[semester][already];
	int num_taken=0;
	for (int i = 0; i < N; ++i)if(already&(1 << i))num_taken++;

	if (num_taken >= K)return cache[semester][already] = 0;
	if (semester == M) return cache[semester][already] = MAXNUM;

	int not_taken = 0;
	int ret = MAXNUM;
	for (int i = 0; i < N; i++) {
		if (already&(1 << i))continue;//the class is already taken.
		if ((prior[i] & already) != prior[i])continue;//the class needs prior classes.
		if (open_subject[semester] & (1 << i))not_taken |= (1 << i);
	}

	for (int subset = not_taken; subset; subset=not_taken&(subset - 1)) {
		int number = 0;

		for (int i = 0; i < 13; i++) {
			if (subset&(1 << i))number++;
			if (number > L)break;
		}
		if (number > L)continue;
		ret = ret < f(semester + 1, already | subset)+1? ret:f(semester + 1, already|subset)+1;
	}
	ret = ret < f(semester + 1, already) ? ret : f(semester + 1, already );
	return cache[semester][already] = ret;
}

int main() {
	cin>>C;

	for (int idx_case=0; idx_case < C; ++idx_case) {
		cin >> N >> K >> M >> L;
		for (int i = 0; i < 14; ++i) {
			for (int j = 0; j < (1 << 14) - 1; ++j)cache[i][j] = -1;
		}
		for (int i = 0; i < N; i++) {
			int num_prior;
			cin >> num_prior;
			prior[i] = 0;
			for (int j = 0; j < num_prior; j++) {
				int idx_prior;
				cin >> idx_prior;
				prior[i] |= (1 << idx_prior);
			}
		}
		for (int i = 0; i < M; i++) {
			int num_open;
			open_subject[i] = 0;
			cin >> num_open;
			for (int j = 0; j < num_open; j++) {
				int idx_open;
				cin >> idx_open;
				open_subject[i] |= (1 << idx_open);
			}
		}
		int ret = f(0, 0);
		if (ret <= K)cout << ret << endl;
		else cout << "IMPOSSIBLE" << endl;
	}

	return 0;
}
```

2. 외판원 순회 문제(TSP)

[백준 외판원 문제](https://www.acmicpc.net/problem/2098)를 bitmask를 이용하여 해결하였다.

```cpp
#include <iostream>

#define MAXNUM 9999999
using namespace std;

int cache[17][1 << 18];
int cost[17][17];
int can_go[17];
int already = 0;
int num_cities;

int f(int now,int already) {
	if (cache[now][already] != -1)return cache[now][already];

	if (already == ((1 << num_cities )- 1)) {
		if (can_go[now]&1)return cost[now][0];
		else return MAXNUM;
	}

	int go_now = can_go[now] & (~already);
	int ret=MAXNUM;
	for (int i = 0; i < num_cities; ++i) {
		if (go_now&(1 << i)) {
			int tmp = f(i, already | (1 << i)) + cost[now][i];
			ret = (ret < tmp) ? ret :tmp ;
		}
	}
	return cache[now][already] = ret;

}
int main() {
	cin >> num_cities;

	for (int i = 0; i < num_cities; ++i) {
		can_go[i] = 0;
		for (int j = 0; j < num_cities; ++j) {
			int tmp;
			cin >> tmp;
			cost[i][j]=tmp;
			if (cost[i][j] != 0)can_go[i] |= (1 << j);
		}
	}
	for (int i = 0; i < num_cities; ++i)for (int j = 0; j < (1 << 18); ++j)cache[i][j] = -1;
	cout << f(0, 1) << endl;
	return 0;
}
```
