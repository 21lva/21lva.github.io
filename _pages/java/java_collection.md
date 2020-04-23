---
layout: post
title:  "collection framework"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_collection.md
---

- **collection framework** : 데이타 모음을 저장하는 클래스들을 표준화한 설계
- 3개의 타입을 3개의 인터페이스로 설계(List와 Set을 Collection 인터페이스로 묶음)
  - List : 순서가 있는 데이터의 집합. 중복 허용
  - Set : 순서가 없는 집합, 중복 불가
  - Map : 키와 값(value)로 이루어진 데이터. 키의 중복은 허용되지 않지만, 값의 중복은 허용됨

- - -

기본 interface
==============

collection interface
---------------------

- 컬렉션 클래스에 저장된 데이터를 읽고, 추가하고, 삭제하는 기본적인 메서드 정의
- 메서드
  - boolean add(Object o)  
  - boolean addAll(Collection c)
  - void clear() : 모든 객체 삭제
  - boolean contains(Object o) : 포함되어있는지 여부 확인
  - boolean containsAll(Collection c) : 포함되어있는지 여부 확인
  - boolean isEmpty()
  - Iterator Iteraator() : collection의 iterator 반환
  - boolean remove(Object o) : 지정된 객체 삭제
  - boolean removeAll(Collection c) : 지정된 객체 삭제
  - boolean retainAll(Collection c) : 지정된 객체 제외하고 전부 삭제
  - int size()
  - Object[] toArray() : 객체배열로 반환
  - Object[] toArray(Object a) : 지정된 배열에 Collection의 객체를 저장해서 반환



List interface
----------------

- 중복을 허용하면서 저장순서가 유지되는 collection
- **Vector,ArrayList,LinkedList,Stack** 가 구현함
- 메서드(Collection에 정의된 메서드 제외)
  - void add(int index,Obejct o)
  - boolean addAll(int index,Collection c)
  - Object get(int index)
  - int indexOf(Object o) : 해당하는 첫 요소 찾기
  - int lastIndexOf(Object o): 해당하는 마지막 요소 찾기
  - ListIterator listIterator()
  - ListIterator listIterator(int index)
  - Object remove(int index)
  - Object set(int index,Object element) : 지정된 위이체 객체 저장
  - void sort(Comparator c) : 비교자를 이용하여 정렬
  - List subList(int fromIndex,int toIndex) : 지정된 범위에 있는 객체 반환



Set interface
---------------

- 중복을 허용하지 않고, 저장순서가 유지되지 않는 collection
- **HashSet, SortedSet, TreeSet** 가 구현하였다
- [HashSet](https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html) : Set을 구현한 클래스, 중복된 요소를 추가하면 false 반환. 순서 X.
- [TreeSet](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html) : red-black tree로 구현됨. 기본적으로 오름차순으로 정렬
- 둘의 차이는 [여기](https://stackoverflow.com/questions/25602382/java-hashset-vs-treeset-when-should-i-use-over-other)


Map interface
------------

- key와 value로 이루어진 하나의 쌍을 저장하는 collection
- **Hashtable, HashMap, SortedMap, LinkedHashMap, TreeMap** 등이 구현
- 메서드
  - containsKey(Object key) : 지정된 key 객체와 일치하는 Map의 key 객체가 있는지 확인
  - containsValue(Object value)
  - Set entrySet() : key-value쌍을 Map.Entry 타입의 객체로 저장한 Set을 반환
  - Object get(Object key) : 지정된 key 객체에 대응하는 value객체를 반환
  - int hashCode()
  - Set keySet() : 모든 key 객체 반환
  - Object put(Object key, Object value) : key와 value를 연결하여 저장
  - void putAll(Map t)
  - Object remove(Object key)
  - int size()
  - Collection values() : 모든 value객체 반환

- [HashMap](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) : Map을 구현한 클래스. Hash를 사용하기 때문에 많은 양의 데이터를 검색하는 데에 뛰어남

Map.Entry interface
--------------------

- Map의 내부 인터페이스
- key-value 쌍을 다루기 위해 정의됨

- - -

구현 class
=========

- 자세한 사항은 [API문서](https://docs.oracle.com/javase/8/docs/api/)를 참조하자

ArrayList
------------


- List interface를 구현
- Vector와 구현원리와 기능적인 측면에서 비슷
- 크기가 가변적.
- 용량을 넘어서서 저장을 하면 새로운 객체를 만들면서 사용하므로 비효율적으로 바뀐다
- 내부의 객체를 하나 제거하면 뒤의 객체들이 앞으로 당겨진다
- [API문서](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html)

LinkedList
-----------

- ArrayList는 크기를 변경하기 힘들고, 비순차적인 데이터의 추가 또는 삭제에 시간이 많이 걸린다(모든 데이터를 옮겨야 하기 때문에)
- 위 단점을 보안하는 것이 **LinkedList** 이다
- 자세한 메서드는 역시 [API문서](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)를 이용하자
- queue 자료 구조를 구현할 때 사용된다

Stack & Queue
------

- 스택은 LIFO(last in first out)의 자료구조이다
- [Queue](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html)는 FIFO으로 LinkedList를 이용하면 구현할 수 있다. 물론, 이를 위한 인터페이스나 그것을 이용한 클래스가 존재한다
- Stack을 구현한 [클래스](https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html)가 존재한다

PriorityQueue
--------------

- Queue interface를 구현한 클래스
- 들어온 순서가 아닌 특정 기준을 가지고 저장을 해서 순서를 갖게한다
- heap이라는 자료구조를 이용한다
- [API](https://docs.oracle.com/javase/8/docs/api/java/util/PriorityQueue.html)

Deque(double-ended queue)
-----

- Queue의 자식 interface
- 한 쪽으로만 삭제/추가만 가능했던 Queue와 달리 양쪽으로 가능
- [API](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html)

- - -

Iterator
=========

- collection에 저장된 요소들을 읽어오는 표준화된 방법
- 각 collection은 Iterator 객체를 반환하는 **iterator()** 메서드가 존재
- 메서드
  - boolean hasNext() : 읽어올 요소가 남아있즌지 확인
  - Object next() : 다음 요소을 가져옴
  - void remove() : next()로 읽어온 요소를 삭제. next() 다음에 호출해야 한다

```java
ArrayList list1 = new ArrayList(20);
list1.add(12);
list1.add(3.4);
list1.add("hi");
Iterator itr = list1.iterator();
while(itr.hasNext()) {
    System.out.println(itr.next());
    itr.remove();
}
System.out.print(list1);
```

##### 결과
12

3.4

hi

[]

ListIterator, Enumeration
-------------

- Enmueration : Iterator의 구버전
  - boolean hasMoreElements() : 읽어올 요소가 있는지 확인. Iterator의 hashNext()와 같은 기능
  - Object nextElement() : Iterator의 next()와 같은 기능
- ListIterator : Iterator에 양방향 조회 기능을 추가한 것
  - void add(Object o) : 새로운 객체 추가
  - boolean hasNext()
  - boolean hasPrevious()
  - Object next()
  - Object previous()
  - int nextIndex()
  - int previousIndex()
  - void remove()
  - void set(Object o) : next() 또는 previous()로 읽어온 요소를 지정된 객체로 변경. 꼭, next또는 previous 이후에 호출되어야 한다

- - -

Comparator, Comparable
========================

- [참고자료](https://gmlwjd9405.github.io/2018/09/06/java-comparable-and-comparator.html)

Comparable
----------

- 기본 정렬 기준을 구현하는 데에 사용하는 interface
- 정렬이 가능한 클래스는 모두 이 interface를 구현하고 있음
- **compareTo()** 메서드를 오버라이딩하여 구현
- Array.sort(array) or Collections.sort(list) 사용
- 기준
  - this < other : 음수
  - this == other : 0
  - this > other : 양수

```java
class AbsVal implements Comparable<AbsVal> {
    int x;

    @Override
    public int compareTo(Point p) {
      int thisX = this.x>0 ? this.x : (-1)*this.x;
      int pX = p.x>0 ? p.x : (-1)*p.x;
      if(thisX<pX)return-1;
      else if(thisX>pX)return 1;
      else 0;
    }
}

// main에서 사용법
List<AbsVal> pList = new ArrayList<>();
pList.add(new AbsVal(x));
Collections.sort(pList);
```

Comparator
----------

- 정렬 가능한 클래스(Comparable interface를 구현한 클래스)를 기존 기준이 아닌 다를 기준으로 정렬하고자 할때
- java.util 패키지에 정의되어 있음
- 새로운 클래스 작성 : Comparator interface를 implements하고 **compare()** 메서드를 오버라이딩한다
- Arrays.sort(), Collections.sort() 의 두 번째 인자로 새로 정의한 Comparator 클래스를 넣어준다
- 기준
  - this < other : 음수
  - this == other : 0
  - this > other : 양수

```java
// x좌표가 증가하는 순, x좌표가 같으면 y좌표가 감소하는 순으로 정렬하라.
class MyComparator implements Comparator<AbsVal> {
  @Override
  public int compare(AbsVal p1, AbsVal p2) {
      int thisX = this.x>0 ? this.x : (-1)*this.x;
      int pX = p.x>0 ? p.x : (-1)*p.x;
      if(thisX>pX)return-1;
      else if(thisX<pX)return 1;
      else 0;
  }
}

// main에서 사용법
List<AbsVal> pList = new ArrayList<>();
pList.add(new AbsVal(y));
MyComparator myComparator = new MyComparator();
Collections.sort(pList, myComparator);
```

```java
//익명 클래스 이용
Comparator<AbsVal> myComparator = new Comparator<AbsVal>(){
  @Override
  public int compare(AbsVal p1, AbsVal p2) {
      int thisX = this.x>0 ? this.x : (-1)*this.x;
      int pX = p.x>0 ? p.x : (-1)*p.x;
      if(thisX>pX)return-1;
      else if(thisX<pX)return 1;
      else 0;
  }
}
```
