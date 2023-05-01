---
layout: post
title:  "Tree"
date:   2019-01-02 02:00:00
author: Inhyuk
category: Algorithms
cover:  "/assets/instacode.png"
name: 2019-01-03-Tree.md
---

**Tree** 는 수학적으로 Graph의 일종이다.
Graph는 node들의 집합과 그 node들을 연결해주는 edge들의 집합으로 이루어져있다.
**Tree** 는 Graph 중에서도 cycle(어느 한 점에서 그 점으로 돌아오는 경로가 존재)과 loop(edge의 양끝이 같은 node에 이어지는 경우)가 존재하지 않는 Graph를 부른다.

많은 사람들이 **Rooted Tree** 와 **Tree** 를 헷갈려 한다.
**Rooted Tree** 는 tree의 임의의 한 점을 root라고 정해 놓고 그것들의 후손들과 subtree로 이루어진 **Tree** 의 일종이다.
그러므로 모든 **Tree** 를 설명할 때 root가 존재한다고 하면 틀린 설명이다.

관련 단어
========
1. leaf node : 자식이 없는 node
2. internal node : leaf도 아니고 root도 아닌 node
3. sibling : 같은 부모를 가지는 node
4. descendent : 자기의 자식과 그 자식들과 또 그 자식들로 leaf node까지 이어지는 모든 node들.
5. size : 자신 + 자신의 descendent의 수
6. depth : root부터 자기 자신까지의 경로의 node의 수 +1
7. level : 특정 depth를 가지는 node들의 집합
8. height: 최대 depth



이진트리
=======

Binary tree라고도 한다. 이는 모든 node들이 최대 2개의 node를 자식으로 가지는 tree를 의미한다.
여기서 자식은 **Rooted tree** 에서 root로 부터 내려왔을 때, 자기와 직접 직접 연결된 node들 중 자기 아래에 있는 node들을 의미한다.
모든 node들이 2개 혹은 0개의 node를 자식으로 가지면 **Full binary tree** 라고 부르고, 가장 왼쪽의 node들 부터 채워지는 tree를 **complete binary tree** 라고 부른다.
![full_complete]({{site.baseurl}}/post_img/{{page.name}}/full_complete.png)


이진 트리 순회
=============

전위 순회(pre-order)
--------------------

![pre]({{site.baseurl}}/post_img/{{page.name}}/pre.jpg)
Root 에서 왼쪽 subtree -> 오른쪽 subtree 순으로 순회하는 방법이다.

중위 순회(In-order)
-------------------

![inorder]({{site.baseurl}}/post_img/{{page.name}}/inorder.jpg)
왼쪽 subtree -> root -> 오른쪽 subtree 순으로 순회하는 방법.

후위 순회(post-order)
----------------------

![post]({{site.baseurl}}/post_img/{{page.name}}/post.jpg)
왼쪽 -> 오른쪽 -> root 순으로 순회.

Heap
====

Complete binary tree의 일종.
주로 priority queue에 의해 만들어진다.

### Priority queue
보통 queue는 처음에 들어온 것을 먼저 배출하는 FIFO구조이다. 우선 순위 queue는 배출 기준이 처음 들어온 순서가 아닌 다른 어떤 기준(key 값)이 있는 것이다.

많은 사람들이 heap과 priority queue를 혼동해 쓰기도 한다. Priority queue는 heap을 이용하는 것이 효율적이고 보편적이기 때문이다.

구조와 종류
----------
Heap에는 크게 2가지가 있다.
1. Max heap: 부모 node의 키 값이 자식 node의 키 값보다 큰 경우
2. Min heap : 부모 node의 키 값이 자식 node의 키 값보다 작은 경우

보통 배열을 통해 만든다. 부모와 자식의 index들의 관계는 다음과 같다.
$$
left child = parent*2+1

right child = parent*2+2

parent = (child-1)/2

(index of root =0)
$$

연산
---

heap에서 연산은 크게 삽입과 삭제(뽑기)로 이루어진다.

### 삽입
1. 배열의 가장 마지막에 위치시킨다.
2. 부모와 비교하여 부모보다 key값이 크면 부모와 자리를 바꾼다(Max heap의 경우)
3. 위로 올라가면서 부모보다 key가 작을 때 까지 반복한다.
 $ logN $ 의 시간복잡도가 필요하다.(complete binary tree이므로 높이가 거의  $ logN $ 이다.)

### 삭제
1. 최대값(배열의 0번째 원소)을 배열의 마지막과 바꾸고 마지막(이전의 배열[0])을 삭제한다.
2. 0번째 값을 자식들과 비교하여 자식 중 큰 값과 바꾼다.
3. 아래로 내려가면서 반복한다.
 $ logN $ 의 시간복잡도가 필요하다.

구현
----

```cpp
class MaxHeap:
    def __init__(self):
        self.A=[]
    def insert(self,x):
        self.A.append(x)
        idx = len(self.A)-1
        while idx>=0:
            parent_idx = int((idx-1)/2)
            if self.A[parent_idx]<self.A[idx]:
                self.A[parent_idx],self.A[idx] = self.A[idx],self.A[parent_idx]
                idx = parent_idx
            else:
                break
    def pop(self):
        if len(self.A)<=0:raise ValueError
        ret = self.A[0]
        self.A[0]=self.A[len(self.A)-1]
        self.A= self.A[:-1]
        idx = 0
        while idx*2+1<len(self.A):
            left = idx*2+1
            right = idx*2+2
            if len(self.A)>right:
                bigger = left if self.A[left]>self.A[right] else right
            else:
                bigger =left
            if self.A[bigger]>self.A[idx]:
                self.A[bigger],self.A[idx]=self.A[idx],self.A[bigger]
                idx=bigger
            else:
                break
        print(self.A)
        return ret
```



참고 자료
=========
- <http://3dmpengines.tistory.com/423>
- <https://gmlwjd9405.github.io/2018/05/10/data-structure-heap.html>
