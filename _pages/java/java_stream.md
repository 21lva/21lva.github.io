---
layout: post
title:  "Stream"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_stream.md
category: java
---

스트림
=======

- 기존의 for와 foreach는 로직이 섞이고 재활용성이 떨어짐
- 데이터 소스를 추상화하고, 데이터를 다루는 데 자수 사용되는 메서드를 정의한 것
- 스트림은 데이터 소스를 변경하지 않음
- 일회용
- 작업을 내부 반복으로 처리
- 병렬처리 가능
- [참고자료](https://futurecreator.github.io/2018/08/26/java-8-streams-advanced/)

- - -

스트림 생성
===========

- 보통 배열과 collection을 이용하여 만듬
- Stream.builder() : 원하는 값을 직접 넣어서 스트림 생성
- Stream.generate() : 인자가 없는 람다식 사용가능. 무한하게 생성해서 개수를 제한해야함
- Stream.iterate() : 초기 값과 해당 값을 다루는 람다식 사용. 결과가 다음 input이 됨. 무한하게 생성
- []Stream.range() : 범위의 값을 가지는 스트림 생성
- []Stream.rangeClosed() : 우측 값을 포함하는 스트림 생성

- 기본 타입형 스트림 : 제너릭을 사용하지 않음. 오토 박싱이 되지 않음
  - IntStream, LongStream, DoubleStream

```java
//배열 스트림
String[] arr = { "abc","def","fwfs"};
Stream<String> stream = Arrays.stream(arr);
Stream<String> stream2 = Arrays.stream(arr,1,3);//arr[1],arr[2]포함

//콜렉션 스트림
List<String> list = Arrays.asList(arr);
Stream<String> stream = list.stream();
Stream<String> stream2 = list.paralleStream();//병렬처리 스트림


//스트림 빌더 사용
Stream <String> bs = Stream.<String>builder().add("a23").add("123123").build();

//generate 메서드 사용
Stream<String> gs = Stream.generate(()->"ab").limit(5)//generate로 만들면 개수가 무한

//stream.iterate()
Stream<String> is = Stream.iterate(20,i->i+1).limit(5)

//range,rangeClosed
IntStream is = IntStream.range(2,7);//[2,3,4,5,6]
IntStream is = IntStream.rangeClosed(2,7);//[2,3,4,5,6,7]
```

- - -

연결
====

```java
Stream<String> s1 = Stream.of('abc','sdfs','sdfwf');
Stream<String> s2 = Stream.of('123','33571','164');
Stream<Strinc> s12 = Stream.concat(s1,s2);
```

- - -
중간 가공
==========

- 작업은 여러 작업을 이어붙임(chaining)

- 연산
  - filter : 스트림 내의 요소들을 걸러내는 작업. Predicate<T>를 받음
  - map : 특정 값으로 변환. Function<T,R>를 매개변수로 받음
  - sorted : Comparator를 이용하여 정렬
  - peek : 확인해보는 정도(System.out::println). 결과에 영향 X


```java
//filter
Stream<String> stream = list.stream().filter(name->name.contains("32"));

//map
Stream<String> stream = list.stream().map(String::toUpperCase);

//sort
stream.sorted((a,b)->a.compareTo(b));

//peek
int x = IntStream.of(1,2,3,4,5).peek(System.out::println).sum();
```

- - -

결과
====

- 마지막으로 값을 만드는 단계
- 모든 연산은 결과 작업이 있어야만 수행된다

계산
------

- count, sum, min, max

```java
//count, sum, min, max
int c = IntStream.of(1,2,3,4,5).count();//빈 경우 0
int s = IntStream.of(1,2,3,4,5).sum();//빈 경우 0
OptionalInt m = IntStream.of(1,2,3,4,5).min();
OptionalInt M = IntStream.of(1,2,3,4,5).max();
```

reduce
--------

- 메서드를 이용하여 결과를 만들어냄. 최대 3개의 매개변수 받음
- 매개변수
  - accumulator : 계산 로직. 요소가 올 때마 중간 결과를 생산하는 로직
  - identity : 계산을 위한 초기값. 스트림이 비어있어서 계산할 내용이 없으면 이 값 반환
  - combiner : 병렬 스트림에서 나워 계산한 결과를 하나로 합치는 동작

```java
int r = IntStream.range(1,10).reduce(3//idnetity
,(input,element)->{
  System.out.println(element);
  return input+element;
}//combier 생략
);
```

collect
-------

- **Collectors** 객체에서 제공하는 Collector 타입의 인자를 받아서 처리 or supplier, accumulator, combiner를 받아서 처리
- Collectors의 메서드
  - Collectors.toList() : 리스트로 반환
  - Collectors.joining(delimiter,prefix,sufix) : 스트림에서 작업한 결과를 하나의 스트링으로 붙임. delimiter= 중간에 들어갈 문자.
  - Collectors.averageingInt() : 평균
  - Collectors.summingInt() : 합
  - Collectors.summarizingInt() : 합, 평균, 최대값, 최소값, count등을 담은 IntSummaryStatistics 객체 반환
  - Collectors.groupingBy() : 특정 조건으로 그룹화. 조건으로 함수형 인터페이스 Function<T,R> 를 사용
  - Collectors.partitioningBy() : Predicate를 받아서 boolean을 기준으로 나눈다
  - Collectors.collectingAndThen() : 특정 타입으로 collect한 후에 추가 작업을 위함

```java
//List 객체 생성
List<String> list = Arrays.asList("James","John","Lisa","Kevin");

//toList()
List<String> ll = list.stream().sorted().collect(Collectors.toList());

//joining()
System.out.println(list.stream().sorted().collect(Collectors.joining()));//JamesJohnKevinLisa
System.out.println(list.stream().sorted().collect(Collectors.joining("/","[","]")));//[James/John/Kevin/Lisa]

//groupingBy()
System.out.println(list.stream().collect(Collectors.groupingBy(String::length)));//{4=[John, Lisa], 5=[James, Kevin]}

//partitioningBy()
System.out.println(list.stream().collect(Collectors.partitioningBy(s->s.length()<5)));//{false=[James, Kevin], true=[John, Lisa]}
```

- collect(Supplier supplier, BiConsumer accumulator, BiConsumer combiner)
  - supplier : 요소들이 수집될 컨테이너 객체(R) 생성. 각 스레드마다 한 번씩 실행
  - accumulator : 컨테이너 객체(R)에 요소(T)를 수집하는 역할
  - combiner : 컨테이너 객체(R)을 결합하는 역할. 병렬처리 스트림에서 호출

```java
List<String> list = Arrays.asList("James","John","Lisa","Kevin");
ArrayList<String> ll = list.stream().collect(
        ArrayList::new,
        (r,t)-> r.add(t+"k"),
        List::addAll
);
System.out.println(ll);//[Jamesk, Johnk, Lisak, Kevink]
```

Matching
--------

- 조건 람다식 Predicate<T> 를 받아서 해당 조건을 만족하는 요소가 있는지 체크

- 메서드
  - anyMatch : 만족하는 요소가 하나라도 존재하는지
  - allMatch : 모두가 만족
  - noneMatch : 만족하는 요소가 하나도 없는지

```java
List<String> list = Arrays.asList("James","John","Lisa","Kevin");
boolean x = list.stream().anyMatch(s->s.length()<5);
boolean y = list.stream().allMatch(s->s.length()<5);
boolean z = list.stream().noneMatch(s->s.length()<5);
System.out.println(x);//true
System.out.println(y);//false
System.out.println(z);//false
```

foreach
--------

- 요소를 돌면서 해당 작업을 시행. 보통 출력을 하는데에 사용
- peek과의 차이 : 중간 작업(peek)이 아닌 최종작업(foreach)이다

- - -

동작순서
======

```java
List<String> list = Arrays.asList("James","John","Lisa","Kevin");
list.stream().map(
        element->{
            System.out.println(element);
            return element+"a";
        }
).map(
        element->{
            System.out.println(element);
            return element+"a";
        }
).forEach(System.out::println);
```

- 위의 코드의 실행 결과는 다음과 같다

```java
//결과
James
Jamesa
Jamesaa
John
Johna
Johnaa
Lisa
Lisaa
Lisaaa
Kevin
Kevina
Kevinaa
```

- 모든 요소들이 한 작업을 한 후에 다음 작업으로 넘어가는 것이 아니라 한 요소가 모든 작업을 끝맞춰야 다음 요소가 작업을 수행한다
