---
layout: post
title:  "Hash"
date:   2019-01-12 02:00:00
author: Inhyuk
category: Algorithms
tags:	Data Structure
cover:  "/assets/instacode.png"
name: 2019-01-12-Hash.md
---


Hash table은 데이터의 key(hash 값)을 테이블의 주소로 이용하는 탐색 알고리즘.\\
Random access를 하기 때문에 속도가 매우 빠르다.

- - -

Hash Table
==========

배열을 만들면 배열의 원소에 접근하기 위해 그 index를 이용하였다.

이렇게 순서대로가 아니라 어떤 한 원소에 직접 접근하는 방법을 **Random access** 라고 부른다.
**Hash Table** 은 이 index를  key또는 hash 값이라고 부른다.
이런 key 값을 만드는 함수를 **Hash function** 이라고 부른다.

즉, 어떤 값을 저장하고 싶으면 그 값을 **Hash function** 에 넣어 key값을 구한 후, table의 해당하는 장소에 저장을 한다.
보통 Table의 크기는 prime number(소수)로 정한다.

- - -

Hash Function
=============

Division method
---------------

$$
address = value mod N
$$

주소로 위하여 입력 값을 N(보통 테이블의 크기)로 나눈 나머지를 사용한다. 최소 0에서 최대 N-1의 값을 주소로 사용할 수 있다.

이 방법은 매우 간단하지만, 충돌문제(다른 값이 같은 hash값을 가지는 현상)과 cluster(대부분의 입력값이 특정 주소집단에 뭉치는 현상)이 발생할 확률이 높다.

이 문제들은 **Hash function** 이라면 당연히 겪는 문제이다. 그러나 **Division Method** 는 더 겪는다.

Digits Folding
--------------

위의 문제들을 살짝이라도 줄이기 위해서 만들어진 방법이다.
자릿수 K를 정하고, 입력값은 K의 크기를 가지는 자릿수로 나눈다. 그 후에 그 값을 전부 더해서 hash 값으로 사용한다.

예를 들어, K=1인 경우에는  $ 71393 $  의 hash 값은  $ 7+1+3+9+3 = 23 $  이 된다.

또한, K=2인 경우에 같은  $ 71393 $  의 hash 값은  $ 71+39+3 = 113 $  이 된다.

K가 증가하면 hash 값이 가질 수 있는 수의 범위도 늘어난다.
물론 이 문제도 문제가 존재한다. K를 어떻게 정하냐에 따라서 오버플로우가 생길 수도 있고, 메모리의 낭비가 발생할 수도 있다.

- - -

How to solve collision problems
================================

충돌을 해결하는 방법으로는 **Open Hashing** 과 **Closed Hashing** 이 존재한다.

Chaining
--------
**Open Hashing** 의 한 방법이다.(**Closed Addressing** 의 일종이기도 하다.)
**Hash table** 을 단순한 값을 가지는 배열 구조가 아닌, **linked list** 를 가지는 배열의 구조로 만드는 방법이다.

![chaining]({{site.baseurl}}/post_img/{{page.name}}/chaining.png)

#### 탐색

1. **Hash function** 을 이용하여 hash 값을 구한다.
2. hash 값을 이용하여 해당하는 **linked list** 를 찾는다.
3. 원하는 값이 나올 때 까지 끝까지 가면서 찾는다.

#### 삽입


1. **Hash function** 을 이용하여 hash 값을 구한다.
2. hash 값을 이용하여 해당하는 **linked list** 를 찾는다.
3. 비어 있는지에 상관없이 linked list 맨 앞에 넣는다.

#### 문제점
**Hash table** 방법의 장점은 Random access를 통해서 시간을 단축시킬 수 있다는 것이다.

그러나 만약에 collision이 지속해서 발생하면 table의 한 list에만 지속적으로 저장을 할 것이고, list의 search 시간만큼 시간이 소모되므로 **Hash table** 의 장점을 잃어버리게 된다.
이 문제를 해결하기 위하여 linked list 대신에 **Red Black tree** 같이 search 시간 복잡도가 작은 구조를 이용하기도 한다.


Open Addressing
---------------

주소 값으로 **Hash function** 에서 얻은 key외의 값을 사용하기도 하는 경우가 존재하는 방법을 **Open addressing** 이라고 한다. 반대는 **Closed addressing** 이라고 한다. 이 전에 살펴본 **chaining** 이 **Closed addressing** 의 한 예이다.

주로, 충돌이 발생하였을 때 어떤 기준에 의해 다른 곳으로 이동하면서 빈 곳을 찾고, 빈 곳이 존재하면 그 곳에 저장하는 방법이다.

#### Linear probing

충돌이 발생할 때 마다 한칸식 옆으로 옮기는 방식. **Collision** 이 많이 발생할 수록 **Cluster** 를 형성할 확률이 높아진다.

#### Quadratic probing

충돌이 발생하면 거리를 1, 4, 9, 16 순으로 띄어가면서 찾는 방법이다.
이 방법으로 **cluster** 문제를 해결하는 것은 아니다.
겉보기에는 떨어져 있는 것 같지만, 결국에는 같은 순서로 움직이므로 **cluster** 를 형성한다.

#### Double hashing

**Quadratic probing** 의 문제는 같은 수 만큼 이동한다는 것에서 야기되었다.

그래서 **Double hasing** 에서는 이동 폭을 새롭게 정의한 **hashing function** 을 이용한다.

즉, 첫 **hashing function** 으로 첫 hash 값을 찾고, 그 값의 주소에서 충돌이 발생하면 두 번째 **hasing function** 으로 이동 폭을 정의하여 그 폭만큼 이동하면서 빈 곳을 찾아나가는 것이다.
보통 두 번째로 나누는 값은 첫 번째와 서로소(relative prime)이면 좋다. 이렇게 해야 겹치는 부분이 줄어든다. (테이블의 크기를 소수로 잡고 step size를 그것보다 살짝 작은 소수로 정하자.)

```py
Empty = -1
TableSize = 11
Hash = [Empty for i in range(TableSize)]
ChainHash = [[] for i in range(TableSize)]
DoubleHash = [Empty for i in range(TableSize)]

#Division method
def div_hash(value,IsFirstFunc=True,stepsize=7):
    if IsFirstFunc:
        return value%TableSize
    else:
        return value%stepsize

#Search and add for division method and chaining
def search(value,hash_func=div_hash,IsChainHash=False):
    if not IsChainHash:
        return Hash[hash_func(value)]!=Empty and Hash[hash_func(value)]==value
    else:
        return (value in ChainHash[hash_func(value)])

def add(value,hash_func = div_hash,IsChainHash=False):
    key = hash_func(value)
    if not IsChainHash:
        Hash[key]= value if Hash[key]==Empty else Hash[key]
    else:
        if value not in ChainHash[key]:
            ChainHash[key].append(value)

#Double hashing
def double_search(value,hash_func=div_hash):
    for i in range(TableSize):
        key = hash_func(value)+i*hash_func(value,IsFirstFunc=False)
        return DoubleHash[key]==Empty

def double_add(value,hash_func=div_hash):
    for i in range(TableSize):
        key = (hash_func(value)+i*hash_func(value,IsFirstFunc=False))%TableSize
        if DoubleHash[key]==Empty:
            DoubleHash[key]=value
            break

add(1)
add(1,IsChainHash=True)
double_add(1)
add(12)
add(12,IsChainHash=True)
double_add(12)
print(Hash)
print(ChainHash)
print(DoubleHash)
print(search(12))
print(search(12,IsChainHash=True))
print(double_search(12))
```

[-1, 1, -1, -1, -1, -1, -1, -1, -1, -1, -1]

[[], [1, 12], [], [], [], [], [], [], [], [], []]

[-1, 1, -1, -1, -1, -1, 12, -1, -1, -1, -1]

False

True

False

참고 자료
========
<https://www.geeksforgeeks.org/hashing-set-2-separate-chaining/>
