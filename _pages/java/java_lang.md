---
layout: post
title:  "lang 패키지 & 유용한 class"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_lang.md
category: java
---

java.lang 패키지
===============

- 가장 기본이 되는 클래스
- import 없이 사용가능
- **Object, String, System** 클래스 등을 포함

- - -

Object 클래스
=============
- 모든 클래스의 조상 클래스

#### public boolean equals(Object obj)

- 매개변수로 객체의 참조변수를 받아서 비교 후 같은지를 확인.
- 참조변수로만 확인을 하기 때문에 실제 값이 같더라도 주소가 다름 false를 반환
- 값을 비교하기 원하면 다음과 같이 오버라이딩 한다

```java
public boolean equals(Object obj){
  return this.val == obj.val;
}
```

#### public int hashCode()

- 객체의 주소값을 이용해서 해시 코드를 반환
- 객체의 주소값이 아닌 특정 멤버 변수로 해쉬 코드를 반환하기 원하면 오버라이딩하면 된다

#### public String toString()

- 정보를 문자열로 제공
- 보통은 인스턴스 변수에 저장된 값들을 문자열로 표현

```java
//기본적인 toString()
public String toString(){
  return getClass().getName()+"@"+Integer.toHexString(hashCode());
}
```

- 위의 기본적으로 제공되는 메서드를 오버라이딩 하면 원하는 값을 얻을 수 있다

```java
public class iii {
    public static void main(String[] args) {
        A a = new A();
        System.out.println(a.toString());
    }
}
class A{
    private final int s=12;
    public String toString(){
       return Integer.toString(s);
    }
}
```

#### protected Object clone()

- 자신을 복사해서 새로운 객체를 생성한다
- 원래 객체는 보존하고 복사본에 특정 작업을 처리하는 데에 사용
- 변수의 갑만을 복사하기 때문에 얕은 복사(참조 변수는 복사되지 않음)된다
- 깊은 복사를 원하면 오버라이딩 하자

#### public Class getClass()

- 자신이 속한 클래스의 Class 객체 반환
- Class 객체에는 클래스의 멤버의 이름, 개수 등 클래스에 대한 정보를 가지고 있음. 각 클래스 당 1개만 존재
- 클래스 파일이 클래스 로더에 의해 메모리에 올라갈 때 자동으로 Class 객체 생성

```java
//Class 객체를 얻는 방법
Class cObj = new Card().getClass();// 생성된 객체로부터 얻는다
Class cObj = Card.class;//클래스 리터럴로부터 얻음
Class cObj = Class.forName("Card");//클래스 이름으로부터 얻음
```

- - -

String 클래스
============

- c언어에서는 char배열로 문자열을 다뤘지만, 자바에서는 객체로 다룸
- immutable 클래스. 객체에 값을 바꾸면 새로운 객체를 생성

```java
//String 클래스 객체를 만드는 방법
//1.
String str1 = "ab";
String str2 = "ab";

//2.
String str1 = new String("ab");
String str2 = new String("ab");
```

1번의 경우에는 "ab"라는 **리터럴** 을 str1과 str2이 동시에 가리킨다. 2번은 새로운 객체가 각각 생성된다.

- String 클래스의 equals 메서드는 문자열을 비교할 수 있다(오버라이딩 되어 있음). **==** 연산자는 단순히 참조값(주소값)이 같은지만 확인

```java
String str1 = "ab";
String str2 = "ab";
String str3 = new String("ab");
String str4 = new String("ab");

str1==str2;//result : true
str1.equals(str2);//result : true
str3==str4;//result : false
str3.equals(str4);//result : true
```

문자열 리터럴
------------

- 자바 소스파일에 사용하는 모든 문자열은 컴파일시에 클래스 파일에 한번에 올라옴.
- 각 문자열 리터럴도 String 객체.
- String이 immutable이기 때문에 한 번 생성된 문자열 리터럴은 그냥 공유하면 됨

join(), StringJoiner
-------------------

- join() : 여러 문자열 사이에 구분자를 넣어서 결합. split()과 정반대
- java.util.StringJoiner 클래스 : 문자열 결합.

```java
StringJoiner sj = new StringJoiner(",","[","]");
String [] strArr = {"a","b","c"};

for(String s: strArr)
  sj.add(s);

System.out.println(sj.toString());//result : [a,b,c]
```

format()
-----

- 형식화된 문자열을 만드는 바업

```java
String str =String.format("%d 나누기 %d 는?",12,3);
System.out.println(str);
```

- 다음과 같은 방식으로 숫자를 문자로 변환가능하다

```java
//숫자를 String으로
String str1 = 10.4+"";
String str2 = String.valueOf(10.4);

//String을 숫자로
Float.valueOf(str1);
```

- - -

StringBuffer 클래스
==================

- 내부적으로 buffer를 가지고 있어서 mutable String 클래스이다

```java
StringBuffer sb = new StringBuffer("ab");
System.out.println(sb);//result : ab
sb.append(34);
System.out.println(sb);//result : ab34
```

- String 클래스와 달리 equals 메서드가 오버라이딩 되어 있지 않아서 ==와 같은 결과를 보여줌

```java
//대등비교
StringBuffer sb1 = new StringBuffer("ab");
StringBuffer sb2 = new StringBuffer("ab");
String s1 =sb1.toString();
String s2 =sb2.toString();
System.out.println(s1.equals(s2));//result : true
```

- - -

java.util.StringTokenizer 클래스
=========

- 긴 문자열을 지정된 구분자(delimiter)를 기준으로 토큰이라는 여러개의 문자열로 잘라냄
- String의 split(정규표현식)이랑 비슷하지만, StringTokenizer는 정규표현식을 쓰지않고도 사용할 수 있다
- 구분자로 하나의 문자열만 사요

```java
String src ="12,345,341";
StringTokenizer st = new StringTokenizer(src,",");
while(st.hasMoreTokens()){
  System.out.println(st.nextToken());
}
```

- - -

java.math.BigInteger 클래스
================

- 큰 정수를 다룬다
- immutable

```java
//생성
BigInteger val1 = new BigInteger("2312314141414141421");
BigInteger val2 = BigInteger.valueOf(2312314141414141421);

//연산
BigInteger val3 = va1.add(va2);
BigInteger val3 = va1.subtract(va2);
BigInteger val3 = va1.multiply(va2);
BigInteger val3 = va1.divide(va2);
BigInteger val3 = va1.remainder(va2);
```
