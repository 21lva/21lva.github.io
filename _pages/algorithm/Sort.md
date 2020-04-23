---
layout: post
title:  "Sort"
date:   2019-01-02 02:00:00
author: Inhyuk
category: Algorithms
tags:	Algorithms
cover:  "/assets/instacode.png"
name: 2019-01-03-Sort.md
---

정렬에 쓰이는 알고리즘들을 알아보자.

Bubble sort
============

데이터를 순회하면서 마지막에 가장 큰 원소를 차곡차곡 쌓아하는 알고리즘이다.
당연히 계산량은 $ N^2 $ 로 매우 크기 때문에 잘 쓰이지는 않는다.

#### 예시
```py
A = [2,4,3,7,1,5]

def BubbleSort(B):
    A=B[:]
    l = len(A)
    for i in range(l):
        for j in range(l-i-1):
            if A[j]>A[j+1]:
                tmp = A[j]
                A[j] = A[j+1]
                A[j+1]=tmp
    return A

print(A)
print(BubbleSort(A))
```

#### 결과
[2, 4, 3, 7, 1, 5]

[1, 2, 3, 4, 5, 7]

- - -

삽입정렬(Insertion sort)
========================

n(초기값 1)을 증가시키면서 다음을 반복한다.
1. n번째 존재하는 원소를 왼쪽으로 넘기면서 비교한다.
2. 이 원소가 비교하는 다른 값도다 작은 경우에는 자리를 바꾼다. 그렇지 않으면 멈춘다.

#### 예시
```py
A = [2,4,3,7,1,5]

def InsertionSort(B):
    A=B[:]
    l = len(A)
    for i in range(1,l):
        x = A[i]
        for j in reversed(range(i)):
            if x<A[j]:
                A[j+1]=A[j]
                if j==0: A[0]=x
            else:
                A[j+1]=x
                break
    return A

print(A)
print(InsertionSort(A))
```

#### 결과
[2, 4, 3, 7, 1, 5]

[1, 2, 3, 4, 5, 7]

버블 정렬보다는 약간 빠르지만 그래도 $ O(N^2) $ 의 연산속도를 가진다.

- - -

Quick sort
==========

이름부터 *quick* 이다. **Divide and Conquer(분할정복)** 을 이용한 알고리즘이다.
임의의 pivot을 기준으로 오른쪽에는 pivot보다 큰 값을 왼쪽에는 작은 값으로 순서에 상관없이 위치 시킨다(Conquer)

pivot을 기준으로 양쪽으로 나눈다.(Divide)
반복한다.


#### 예시

```py
X = [2,4,3,7,1,5]

def QuickSort(B):
    A=B[:]
    if len(A) <=1:
        return A

    pivot = A[0]
    last_of_smaller = 0
    for i in range(1,len(A)):
        if A[i]<pivot :
            if last_of_smaller+1==i:
                last_of_smaller+=1
            else:
                tmp = A[i]
                A[i] = A[last_of_smaller+1]
                A[last_of_smaller+1]=tmp
                last_of_smaller+=1
    tmp = pivot
    A[0] = A[last_of_smaller]
    A[last_of_smaller]=tmp
    return QuickSort(A[:last_of_smaller])+[A[last_of_smaller]]+QuickSort(A[last_of_smaller+1:])


print(X)
X=QuickSort(X)
print(X)
```

#### 결과
[2, 4, 3, 7, 1, 5]

[1, 2, 3, 4, 5, 7]

가장 왼쪽의 값을 pivot으로 잡고 그보다 작은 값들은 왼쪽에, 큰 값은 오른쪽에 둔다. 이것을 pivot을 기준으로 나누고 두 개를 다시 똑같이 시행한다(분할정복)

Recursion의 깊이는 $log(N) $이고, 각 recursion은 $N $ 의 연산을 수행하므로 보통 $O(Nlog(N)) $ 의 속력을 가진다고 알려져 있으나, 정렬이 역순으로 되어있거나 정렬이 되어있는 경우에는 $O(N^2) $의 속력을 보인다.

이를 방지하기 위하여 pivot을 항상 맨뒤나 맨 앞에서 고르는 것이 아닌 Random으로 뽑기도 한다.

- - -

Merge sort
==========

유명한 수학자 폰 노이만이 제안한 정렬 알고리즘이다.

분할 정복 알고리즘이며 거의 **Quick sort** 와 비슷한 성능을 보인다.

2개의 배열로 나누고(Divide), 원하는 작은 크기로 나눠진 배열은 정렬한다.(Conquer).이 들을 합친다.(Combine)

![merge_sort]({{site.baseurl}}/post_img/{{page.name}}/merge_sort.png)

#### 예시

```py
X = [2,4,3,7,1,5,9,0]


def merge(A,B):
    ret =[]
    i=0
    j=0
    while i<len(A) and j<len(B):
        if A[i]<B[j]:
            ret.append(A[i])
            i+=1
        else:
            ret.append(B[j])
            j+=1
    while i<len(A):
        ret.append(A[i])
        i+=1
    while j<len(B):
        ret.append(B[j])
        j+=1
    return ret
def MergeSort(A):
    if len(A)<=1:return A
    mid = int(len(A)/2)

    left = MergeSort(A[:mid])
    right = MergeSort(A[mid:])
    return merge(left,right)


print("Input : ",X)
X=MergeSort(X)
print("Output : ",X)
```

#### 결과
Input :  [2, 4, 3, 7, 1, 5, 9, 0]

Output :  [0, 1, 2, 3, 4, 5, 7, 9]

**Quick sort** 에 비해 **Merge sort** 가 가지는 단점은 정렬하려는 배열 외에도 메모리가 필요하다는 것(not in palce sorting)이 있다.

간단히 생각하면 모든 depth에서 연산은 $N $ 번 이루어진다. depth가 $logN $이므로 시간복잡도는 $O(NlogN) $이다. 또한, **Quick sort** 와 달리 최악의 경우(worst case)에서도 $O(NlogN) $의 시간복잡도를 유지한다.

- - -

Heap sort
=========

[Tree]({{site.baseurl}}/data_structure/2019/01/02/Tree.html)장에서 소개한 heap을 이용하는 방법을 말한다.

내림차순을 하고자하면 Max heap을, 오름차순을 하고자하면 min heap을 이용한다.
간단하다.

1. 주어진 배열로 Heap을 만든다.
2. 반복해서 삭제(뽑기)연산을 수행한다.

Heap을 만드는 데에 $NlogN $이 걸리고 삭제하는 데에도 $NlogN $이 필요하므로 시간복잡도는 $O(NlogN) $이다.
다행스럽게도 최악의 경우에도 시간 복잡도는 $O(NlogN) $이다.
