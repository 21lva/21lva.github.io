---
layout: post
title:  "Divide and Conquer && Dynamic Programming"
date:   2019-01-02 02:00:00
author: Inhyuk
category: Algorithms
tags:	Algorithms
cover:  "/assets/instacode.png"
name: DPDC.md
---

분할 정복
========

분할 정복은 어떤 문제를 해결하기 위하여 문제를 작은 문제로 나누어 해결한다.
큰 문제에서 작은 문제로 나누어서 해결한다.(Top-down)
**Quick sort** 같은 알고리즘이 **Divide and Conquer** 의 일종이다.

- - -

동적 계획법(Dynamic programming)
================================

동적 계획법은 분할정복의 방법과 유사하다.
가장 큰 차이점은 분할정복과 달리 작은 문제를 해결하고 이를 이용해서 큰 문제를 해결한다는 것이다.
분할정복은 작은 문제들이 아무 관련이 없기 때문에 그냥 작은 문제로 나누고 합치면 되는 반면에, 동적 계획법은 작은 문제들이 서로 관련이 있어서 이전 단계에서 구한 답을 저장해 두었다가 사용하는 방법으로 문제를 해결한다. 이렇게 하기 위하여 최적부분구조(optimal substructure)를 가져야 한다. 이는 전체 문제의 최적의 답이 부분문제의 답들을 포함하고 있을 때 이다.

많은 곳에서 DP와 분할정복를 Top-Down과 Bottom-up으로 구분하기도 하기도 하고 어떤 부분에서는 겹치는 sub problem을 어떻게 다루는가로 구분하기도 한다. 중요한 부분은 overlapping sub-problem이 존재하느냐의 차이이다. 아래의 예시들은 전부 overlapping sub problem이 존재하는 문제들로 DP에서 메모이제이션에 따른 성능의 차이를 볼 것이다.

파스칼의 법칙
--------------

파스칼의 법칙은 다음의 식을 의미하는 것이다.

$$
{n\choose k }= {n-1\choose k-1} + {n-1\choose k}
$$

이 문제를 해결하기 동적계획법(메모이제이션)을 이용한 것과 아닌 것의 성능 차이를 비교해보자.

#### 코드
```
#include <iostream>
#include <list>
#include <ctime>
using namespace std;

unsigned long long int cache[1000][1000];

unsigned long long int comb1(int n,int k){
        if(cache[n][k]!=-1)return cache[n][k];
        if(k<0)return 0;
        if(n==k||n==1)return 1;
        return cache[n][k]=comb1(n-1,k-1)+comb1(n-1,k);
}
unsigned long long int comb2(int n,int k){
        if(k<0)return 0;
        if(n==k||n==1)return 1;
        return comb2(n-1,k-1)+comb2(n-1,k);
}

int main(){
        for(int i=0;i<1000;++i)for(int j=0;j<1000;++j)cache[i][j]=-1;
        clock_t start = clock();
        cout<<comb1(30,15)<<endl;
        float time1 = (float)(clock()-start);

        start = clock();
        cout<<comb2(30,15)<<endl;
        float time2 = (float)(clock()-start);
        cout<<"DP : "<<time1<<endl;
        cout<<"not DP : "<<time2<<endl;

        return 0;

}
```

### 결과
155117520

155117520

DP : 0

not DP : 1.45312e+06


시간이 엄청 절약 되었다. DP에서는 sub problem중에 겹치는 부분을 전부 계산을 하는 것이 아니라 한 번 계산해놓은 값을 cache 저장해 놓고 단순히 저장된 값을 가져다 쓰는 것이기 때문에 더 시간을 적게 쓰는 것이다.

최장 공통 부분 순서(LCS)
-----------------------

최장 공통 부분 순서는 두개의 문자열에서 공통으로 존재하는 문자들을 말한다. 이 때 이 문자열의 순서는 두 문자열에서 같아야 한다.

예를 들어, *appartment* 와 *apple* 의 LCS는 *appe* 이다.

**LCS알고리즘** 은 이렇게 두 문자열에서 longets common sequence를 찾는 것이다.

임의의 두 문자 $A=[a_1,a_2,...,a_n]$와 $B=[b_1,...,b_m]$ 이 있다고 하자. 그렇다면 $A$와 $B$의 sub string $A_i=[a_1,a_2,...,a_i]$,$B_j=[b_1,...,b_j]$의 최장 수열을 $LCS(i,j)$라고 정의한다.

$LCS(i+1,j+1)$의 경우 두 가지를 고려해야 한다.
1. $a_{i+1}$와 $b_{j+1}$이 같은 경우: $LCS(i+1,j+1)$는 $LCS(i,j)+1$이 된다.
2. $a_{i+1}$와 $b_{j+1}$이 같지 않은 경우: 이럴 때는 $a_{i}$와 $b_{j+1}$이 같은 경우와 $a_{i+1}$와 $b_{j}$이 같은 경우를 고려해야 한다.

두 가지를 모두 고려하면 다음과 같음을 알 수 있다.

$$
LCS(i+1,j+1)=\begin{cases}
0 & \text{if} i=0 \text{or}j=0\\
LCS(i,j)+1 & \text{if} a_{i+1}=b_{j+1}\\
max\{LCS(i,j+1),LCS(i+1,j)\}&\text{if}a_{i+1}\neq b_{j+1}
\end{cases}
$$

이를 분할정복으로 구현해보자.

```
#include <iostream>
#include <string>
#include <time.h>
using namespace std;

string a = "fasdfasdffsadfwqf";
string b = "afsddasdfqfgqdsfq";

int lcs(int i, int j) {
	if (i < 0 || j < 0)return 0;
	if (a.at(i) == b.at(j))return lcs(i-1, j-1) + 1;
	int left = lcs(i - 1, j),right = lcs(i,j-1);
	return (left > right) ? left : right;
}

int main() {
	clock_t s, e;
	s = clock();
	cout<<lcs(a.length()-1, b.length()-1)<<endl;
	e = clock();
	cout << "time : " << (float)(e-s) << endl;
	return 0;
}
```

#### 결과
11
time : 1087

그렇다면 이 것을 **Dynamic Programming** 을 이용하는 경우를 생각해보자.

LCS(10,10)을 계산하기 위하여 LCS(9,10),LCS(10,9)를 계산해야 한다.
LCS(9,10)은 LCS(9,9),LCS(8,10)을, LCS(9,9),LCS(8,10)을 계산해야 한다.

여기서도 LCS(9,9)가 겹친다.

깊이가 깊어지면 질수록 겹치는 부분은 늘어나게 된다. 겹치는 부분을 저장해놓으면 당연히 좋은 효과를 볼 것이다.

아래의 식은 **DP** 를 이용하여 구현한 LCS이다.

```
#include <iostream>
#include <string>
#include <time.h>
using namespace std;

string a = "fasdfasdffsadfwqf";
string b = "afsddasdfqfgqdsfq";
int cache[100][100];

int lcs(int i, int j) {
	if (i < 0 || j < 0)return 0;
	if (cache[i][j] != -1)return cache[i][j];
	if (a.at(i) == b.at(j))return cache[i][j]=lcs(i-1, j-1) + 1;
	int left = lcs(i - 1, j),right = lcs(i,j-1);
	return cache[i][j]=(left > right) ? left : right;
}

int main() {
	for (int i = 0; i < 100; ++i)for (int j = 0; j < 100; j++)cache[i][j] = -1;
	clock_t s, e;
	s = clock();
	cout<<lcs(a.length()-1, b.length()-1)<<endl;
	e = clock();
	cout << "time : " << (float)(e-s) << endl;

	return 0;
}
```

#### 결과

11
time : 0

많은 차이를 보였다.

시간 복잡도
-----------

분할정복에서 시간 복잡도는 분기하는 횟수(깊이)에 각 깊이에서 이루어지는 연산의 횟수를 곱해주면 된다. 예를 들어, **merge sort** 에서 깊이는 $logN$이다. 각 깊이에서 길이가 $a_n$인 두 배열을 합치는 데에 드는 연산은 $a_n$이다. 또한, 깊에 $h$에서 배열의 갯수는 $2^h$이며 그 때의 배열들 각각의 길이는 $\frac{N}{2^h}$이다.

총 계산은 다음과 같다.

$$
\sum^{logN}_{k=1}\frac{N}{2^h}2^{h-1}\\
=\frac{N}{2}logN
$$

결국 시간복잡도는 $O(NlogN)$이 된다.

DP에서의 시간복잡도의 계산은 살짝 다르다. 분할정복에서는 모든 계산이 실제로 이루어져서 깊이와 각 깊이에서의 계산량을 알면 쉽게 계산을 할 수 있지만, DP에서는 계산이 이루어지지 않는 부분이 있기 때문에 그럴 수 없다. 특히, 메모이제이션을 고려해야 한다.

LCS 알고리즘에서 동적 할당의 경우에 두 문장의 길이를 $N,M$이라고 해보자.

$LCS(N,M)$은 최대로 $LCS(N-1,M),LCS(N,M-1)$로 나뉘어진다. 총 깊이가 $h$이면 전체의 연산의 갯수는 $2^{h+1}$가 된다.$h$는 깊이이므로 $max{N,M}$이 된다. 결국 총 시간복잡도는 2의 지수배이다. DP에 메모이제이션을 사용하는 경우에는 이미 계산된 부분은 계산이 이루어지지 않으므로, 총 연산은 $O(NM)$이다.
