---
layout: post
title:  "Search"
date:   2019-01-02 02:00:00
author: Inhyuk
category: Algorithms
tags:	Algorithms
cover:  "/assets/instacode.png"
name: 2019-01-03-Search.md
---

Search(탐색)은 말그대로 어떤 자료에서 원하는 자료만을 찾아내는 작업이다.
그 자료형이 배열일 수 있고, linked list일 수 있고, stack일 수도 있는 것이다.
이번 포스트에서는 다양한 자료형에 쓰인 여러 탐색 방법들을 알아볼 것이다.

- - -

순차탐색(Sequential Search)
===========================

순차탐색은 처음부터 끝까지 순서대로 확인하며 원하는 데이터를 찾는 알고리즘이다.

효율이 안좋은 방법으로 유명하지만, 정렬되지 않은 자료구조에서 원하는 것을 찾는 유일한 방법이다. 또한, 구현이 간단하다는 장점도 있다. 주로 적은 양의 자료의 경우에 사용한다.

```cpp
class Node{
  public:
    int value;
    Node* next;
    Node(int value){
      this->value=value;
      this->next=nullptr;
    }

class LinkedList{
  private:
    Node* first;
  public:
    LInkedList(){this->first=nullptr;}
    void insert(int value){
      Node* A=new Node(value);
      Node* child = this->first;
      if(child==nullptr)this->first=A;
      else{
        for(;child->next!=nullptr;child->child->next);
        child->next=A;
      }
    }
    void search(int value){
      Node* child =this->first;
      for(;child!=nullptr;child=child->next){
        if(value==child->value){
          cout<<"Exist"<<endl;
          return;
        }
        cout<<"Not Exist"<<endl;
      }
    }
    void remove(int value){
      Node* parent = this->first;
      Node* child = this->fist;
      for(;child != nullptr ; child = child->next){
        if(value ==child ->value){
          parent -> next = child ->next;
          delete child;
          return;
        }
      }
    }
}
```

위의 코드는 [Linked list]({{site.baseurl}}/data_structure/2019/01/02/Tree.html)에서 구현된 **linked list** 이다.

함수 search부분에서 시행하는 것이 순차 탐색이다.

- - -

이진탐색(Binary search)
=======================

이진 탐색은 정렬된 데이터에서 사용하는 탐색방법이다. 탐색의 범위를 1/2씩 점점 줄여주는 방법으로 탐색을 하기 때문에 검색 속도가 매우 빠르다.( $ O(logN) $ )

```py
import time
import random
def search(list,x):
    for i,y in enumerate(list):
        if y==x:return i
    return -1


def bsearch(list,x,first=None,last=None):
    if first==None and last ==None:
        first = 0
        last = len(list)-1
    if first==last:
        if list[first]==list[x]:return first
        else: return -1
    mid = int((first+last)/2)
    if list[mid]==x:return mid
    elif list[mid]>x:return bsearch(list,x,first,mid)
    else:return bsearch(list,x,mid,last)

A = [i+1 for i in range(9000000)]

time1=0
time2=0
for _ in range(50):
    index = random.randint(0,9000000)

    start = time.time()
    b=bsearch(A,index)
    last = time.time()

    start2 = time.time()
    a=search(A,index)
    last2 = time.time()

    time1 += last-start
    time2 += last2-start2
    print(index,b,a,time1,time2)
print("%s %s"%(time1/50,time2/50))

```
파이썬으로 정렬되어 있는 list에서 순차 탐색과 이진 탐색을 이용하여 random하게 특정 데이터의 index를 출력하는 작업을 수행하고 평균을 내는 코드이다. 결과는 다음과 같다.

##### 1.9965171813964843e-05   0.20193270683288575

이진 탐색이 월등히 좋은 성능을 보여준다.

- - -

이진탐색트리(Binary search tree)
================================

이진탐색은 자료구조가 배열인 경우에만 사용할 수 있다. **Linked list** 같은 경우에는 중간 요소에 접근할 수 어렵기 때문에 이진탐색을 사용할 수 없다. 이런 경우에 사용할 수 있는 방법이 이진 탐색 트리이다.

이진 탐색 트리는 자식이 2개이고 왼쪽 후손은 자기보다 key값이 작은 것들, 오른쪽 자손들은 자기보다 key값이 큰 것들로 이루어진 트리이다.

이런 경우에 탐색은 이진탐색과 비슷하게 진행된다. 어떤 값을 찾으려 할 때 현재 node의 key값이 찾으려는 값보다 크면 왼쪽으로 작으면 오른쪽으로 이동하는 것이다.

삽입
----
이진탐색트리에서 삽입은 다음과 같이 진행된다.
1. 탐색을 진행하는 것처럼 비교하여 오른쪽 또는 왼쪽으로 내려간다.
2. 마지막 node에서 왼쪽이나 오른쪽으로 내려갈 때 그곳에 node가 없으면 그 자리에 안착한다.

삭제
----
이진탐색트리에서 삭제는 삽입과 다르게 복잡하다. 이진탐색트리의 구조를 유지하면서 삭제를 해야하기 때문이다.
세 가지 경우로 나누어서 생각한다.
1. 삭제하려는 node의 자식이 없는 경우
* 이 경우에는 그냥 node를 삭제하면 된다.
2. 삭제하려는 node의 자식이 한 개인 경우
* 삭제하려는 node의 자식을 삭제하려는 node의 부모의 자식으로 연결한다.
3. 양쪽에 자식이 존재하는 경우
* 삭제하려는 node의 오른쪽 후손(sub tree)중 가장 작은 값을 삭제하려는 node의 자리에 위치시킨다.


문제점
------
이진탐색트리를 만들 때 들어오는 수가 정렬되어있다면(예를 들어 1,2,3,5,7,8), 이진탐색트리는 일렬로 이어진 트리가 되어서 이진탐색의 효율을 얻을수 없다.(탐색의 경우  $ O(N) $ 의 시간복잡도를 필요로 하게 된다.)
이에 따라 만들어진 것이 아래의 Red black tree이다.

- - -

Red Black Tree
===============

레드블랙트리는 일종의 이진탐색트리이다. 정확히는 이진탐색트리에 색깔요소를 추가하여 트리의 depth를 일정하게 유지하고 이진탐색트리의 문제점을 해결하기 위하여 제안된 트리 구조이다.

추가된 색 배정 방법은 다음과 같다.
1. root node는 검정색이다.
2. leaf node는 검은색이다.(NIL이라는 검정색 leaf node를 추가한다.)
3. 빨간 node의 자식들은 검정색이다.
4. root node에서 모든 leaf node로의 경로는 전부 같은 검은 node를 포함한다.

#### 구조


![rbtree]({{site.baseurl}}/post_img/{{page.name}}/rbtree.png)

회전
----
레드블랙트리는 이진트리의 구조적 문제점을 해결하기 위해 제안되었다고 하였다. 그러므로 트리의 구조를 일정하게 만들기 위하여 **회전** 이라는 새로운 연산이 필요하다.

#### 우회전
우회전은 왼쪽자식과 부모를 교환하는 작업을 의미한다.
단순히 돌리면 되겠지라고 생각할 수 있지만 그렇지 않다.
위의 그림에서 왼쪽의 1과 8을 우회전 한다고 생각해보자. 단순히 회전만하면 1은 3개의 자식을 가지게 된다.
그렇게 하면 레드블랙트리의 구조를 잃게 되므로
* 우회전시 왼쪽 자식(X)의 오른쪽 자식(Z)을 부모(A)의 왼쪽 자식으로 만들고, 왼쪽 자식 노드(X)의 오른쪽 자식에 부모(A)를 연결하고 왼쪽 자식 노드(X)를 부모(A)의 원래 부모에 연결한다.

#### 자회전
자회전을 할 경우에도 우회전과 비슷하다.

* 오른쪽 자식(X)의 왼쪽 자식을 부모(A)의 오른쪽에 연결하고, 부모(A)를 오른쪽 자식(A)의 왼쪽 자식으로 연결한 후 부모(A)의 원래 부모에 X를 연결한다.

삽입
----

삽입은 이진탐색트리와 처음은 비슷하다. 맞는 위치를 찾아서 삽입한다. 그 후가 달라진다.
규칙에서 1번은 단순히 root를 검정색으로 칠하면 해결되고, 2번은 leaf node로 검정색 NIL을 추가하였기 때문에 문제 없으며, 4번의 경우에도 부모의 검정색 NIL을 제거하고 빨간색과 그에 연결된 두 개의 검정 NIL node들을 연결하였기 때문에 문제가 되지 않는다.

결국 문제가 되는 부분은 3번(빨간 노드의 자식은 검정색)이다.

문제가 되는 경우들을 생각해보자.
1. 부모(빨간색)의 형제도 빨간색일 때
2. 부모(빨간색)의 형제는 검은색, 새로 삽입한 노드가 부모의 왼쪽 자식
3. 부모(빨간색)의 형제는 검은색, 새로 삽입한 노드가 부모의 오른쪽 자식

#### 1번
부모의 부모는 검은색이기 때문에 부모의 형제를 검은색으로 칠하고 부모의 부모를 빨간색으로 칠한다.
이 경우에는 부모의 부모의 색 변화로 인해 문제가 발생할 수 있기 때문에 그 노드를 기준으로 반복해서 1 또는 2 또는 3 번을 진행한다.

#### 2번
부모를 검은색, 부모의 부모를 빨간색으로 칠한 후, 부모의 부모를 기준으로 우회전한다.

#### 3번
부모를 기준으로 좌회전 한다. 그 후에 2번을 진행한다.
![rbinsert]({{site.baseurl}}/post_img/{{page.name}}/rbinsert.png)

삭제
----

복잡한 이진탐색트리의 삭제연산보다도 더 복잡한 과정을 거쳐야 한다.
일단 이진탐색트리에서 삭제연산을 하는 것과 같이 연산을 진행하여 삭제를 한 후 재구성한다. 그 후에 red black tree의 조건을 맞춰주기 위하여 색을 새로 칠한다.
삭제하려는 노드가 빨간색이면 문제가 되지 않는다.
삭제하려는 노드가 검은색이면 root로부터 leaf node로의 경로의 검은색 node의 수가 달라질 수 있다. 그러므로 삭제를 한 후에 재구성을 하면서 색을 맞추어야 한다.
나머지는 [wiki](https://ko.wikipedia.org/wiki/%EB%A0%88%EB%93%9C-%EB%B8%94%EB%9E%99_%ED%8A%B8%EB%A6%AC)에서 확인하자. 너무 길다.
