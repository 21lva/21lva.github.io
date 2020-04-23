---
layout: post
title:  "Linked List"
date:   2019-01-02 02:00:00
author: Inhyuk
tags:	Data Structure
category: Algorithms
cover:  "/assets/instacode.png"
name: 2018-12-30-linkedlist.md
---

**Linked List** 는 *Node* 라고 불리우는 각 요소들을 이용하여 어떤 *Node* 와 다른 *Node* 를 앞*뒤로 일방향 혹은 양방향으로 연결해준다.
연결의 성질에 의해 리스트는 중간에 있는 *Node* 에 접근하기 위해서는 처음 혹은 마지막 *Node* 부터 순차적으로 검색을 시행해야 한다.

**Linked List** 의 주된 연산에는 **추가** , **삭제** , **탐색** ,**삽입** 이 있다.

먼저 *Node* 를 구현해보자.
```cpp
class Node{
  public:
    int value;
    Node* next;
    Node(int value){
      this->value=value;
      this->next=nullptr;
    }
```

구현한 *Node* 를 이용하여 추가, 삭제, 탐색을 하는 linked list class를 만들자.

```cpp
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

(탐색의 경우 간단하게 존재하는지 아닌지만 출력하도록 만들었다.)

위와 같은 코드는 가장 간단한 한방향 linked list이다. 이를 이용하여 노드의 갯수 등을 구할 수 있다.

유의해야 할 점은 탐색, 추가, 삭제, 갯수 구하기 등의 연산은 처음부터 시작하여 구하는 과정이 필요하기 때문에 연산이  $ O(N) $  ( $ N $ 은 노드의 수)만큼 걸린다. 배열에 비해 추가, 삭제, 삽입은 연산량이 적지만, 탐색은 오래 걸린다. 추가,삭제,삽입이 비교적 많은 연산에 적합한 자료형이다.

위는 단일 연결 리스트(한방향으로 연결된 리스트)이고, 다음에 설명할 내용은 이중 연결 리스트이다.

**이중연결리스트(double linked list)** 는 말그대로 두 개의  *Node* 가 앞뒤로 연결된 구조이다.

이를 *c++* 에서는 *STL* 로 구현되어 있다.

#### 주로 사용하는 함수
* list.back(): 마지막 원소를 참조
* p = list.begin() : list의 첫 원소를 가리키는 iterator
* p = list.end() : list의 마지막 원소의 iterator
* list.clear() : list를 비운다.
* list.empty() : list가 비어있는지 bool값을 출력
* p = list.insert(q,x) : q가 가리키는 위치에 x를 삽입하고 그 위치를 가리키는 iterator를 p로 출력
* list1.merge(list2): list1에 list2를 merge sort한다.
* list1.merge(list2,pred): pred(조건자)를 가지고 merge sort.
* list.pop_back(), list.pop_front() : 마지막, 첫 node 제거.
* list.push_back(x) : list의 마지막에 x값을 가지는 node 추가.
* list.push_front(x) : list 의 처음에 x값을 가지는 node 추가.
* list.remove(x) : x를 가지는 모든 node 제거.
* list.remove_if(pred) : pred에 맞는 node 모두 제거.
* list.size(): list의 크기.
* list.sort() : 오름차순 정렬
* list.sort(pred) : 조건을 기준으로 정렬
* list.reverse() : list의 순서를 뒤집는다.

```cpp
#include <iostream>
#include <list>
using namespace std;


int main(){
    list<int> linkedlist;
    linkedlist.push_back(1);
    linkedlist.push_back(3);
    linkedlist.push_back(2);
    linkedlist.push_back(4);
    for(list<int>::iterator iter = linkedlist.begin();iter!=linkedlist.end();++iter)
      cout<<*iter<<" ";
    cout<<endl;
    linkedlist.sort();
    for(list<int>::iterator iter = linkedlist.begin();iter!=linkedlist.end();++iter)
      cout<<*iter<<" ";
    cout<<endl;
}
```

#### 결과
1 3 2 4

1 2 3 4


참고 자료
--------
<http://hyeonstorage.tistory.com/326>
