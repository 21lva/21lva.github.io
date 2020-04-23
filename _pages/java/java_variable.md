---
layout: post
title:  "기본"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_variable.md
category: java
---

이 파트는 [JAVA의 정석](http://www.yes24.com/Product/Goods/24259565)와 여러 다른 자료들을 정리한 것이다.

기초적인 내용들이 포함되어있지만, C언어와 겹치는 여러 부분은 스킵하였다

- - -

변수
====

- 변수의 선언 : 값을 저장할 공간을 마련
- 변수타입 변수이름;
- 기본 타입(Primitive type) : 실제 값 저장
1. 문자 : char(2 byte)
2. 숫자 : byte(1 byte), short(잘 안쓰임), int(4), long(8)
3. 실수 : float(4),double(8)
4. 논리 : boolean(1 byte)

- 참조형(reference type) : 기본형을 제외한 나머지. 메모리 주소를 저장

- 리터럴 : 값 그 자체

```java
final int score = 100;
//100 : 리터럴
//score : 변수
//final : 변수를 상수로 만듬
```

- 접미사 : float에는 f가 long에는 L(생략 가능)이 붙음.

```java
float pi = 3.14f;
long longNum1 = 100L
long longNum2 = 100;
```

![자동 형변환]({{site.baseurl}}/post_img/{{page.name}}/conv.png)

- 수동으로 형변환을 원하면 **(원하는타입)변수**



Scanner
-----------

- 화면으로부터 데이터를 입력받는 기능을 제공하는 클래스
- 적용방법
  1. import
  2. 객체 생성
  3. 입력 받기

```java
import java.util.*;

public class iii {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int a = Integer.parseInt(scanner.nextLine());
        float input = scanner.nextFloat();
        System.out.printf("%.2f",input);
        System.out.printf("%d",a);
    }
}
```

- - -

배열
=====

```java
//배열이 생성 되기 위한 참조 변수 선언
int[] score;
int score[];

//배열 생성
score = new int[5];

//한 번에
int[] score = new int[5];

//배열 길이
score.length

//초기화도 동시에
int[] score = {10,20,30};
int[] score = new int[]{10,20,30};

//한 번에 복사
System.arraycopy(origin,0,newArray,0,origin.length);

//다차원 배열
int[][] arr = new int[3][4];
int[][] arr = {
  {1,2,3},{4,5,6}
};
```

- 한 번 만든 배열의 길이는 변경 불가능

- 자바에서는 배열의 배열을 만들때 내부 배열의 길이를 다르게 지정할 수 있다.

```java
int[][] arr = new int[5][];
arr[0]=new int[4];
arr[1]=new int[3];
arr[2]=new int[2];
arr[3]=new int[1];
arr[4]=new int[5];
```
