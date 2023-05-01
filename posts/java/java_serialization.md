---
layout: post
title:  "Serialization"
date:   2019-12-18 01:02:59
author: Inhyuk
name: java_serialization.md
category: java
---

직렬화(serialization)
=====================

- 객체를 데이터 스트림으로 만드는 것
- 객체에 저장된 데이터를 스트림에 쓰기 위해 연속적인 데이터로 변환하는 것
- 역직렬화 : 스트림으로부터 데이터를 읽어서 객체를 만드는 것

ObjectInputStream, ObjectOutputStream
-------------------------------

- 직렬화와 역직렬화에 쓰이는 스트림
- InputStream, OutputStream을 상속. But 보조스트림(다른 스트림 필요)

```java
//간단한 방법
//쓰기
FileOutputStream fos = new FileOutputStream("objectfile.ser");
ObjectOutputStream oos = new ObjectOutputStream(fos);

oos.writeObject(new String());

//읽기. Object로 저장하기 때문에 형변환 필요
FileInputStream fis = new FileInputStream("objectfile.ser");
ObjectInputStream ois = new ObjectInputStream(fis);

String tmp = (String)ois.readObject();
```

직렬화 가능한 클래스 만들기
------------------------

- 직렬화가 가능한 클래스를 만들려면 **java.io.Serializable** interface를 구현해야 한다
- 이 interface는 아무런 내용이 없는 빈 interface
- **Serializable** inteface를 구현한 클래스를 상속 받으면 다시 구현할 필요 없음

- 클래스안에 직렬화할 수 없는 멤버변수(Object등)을 포함하면 직렬화할 수 없다(변수의 참조타입이 아닌 실제 변수의 타입)
- 이 경우에는 해당 변수에 **transient** 키워드를 붙여서 직렬화에서 제외할 수 있음. 역직렬화시에는 기본값(null등)으로 값이 정해진다

```java
public class iii {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        FileOutputStream fos = new FileOutputStream("objectfile.ser");
        ObjectOutputStream oos = new ObjectOutputStream(fos);

        MyObject c = new MyObject(1);
        MyObject d = new MyObject(2);

        System.out.println(c.x);//result : java.lang.Object@6e8cf4c6
        oos.writeObject(c);
        oos.writeObject(d);
        oos.close();

//읽기. Object로 저장하기 때문에 형변환 필요
        FileInputStream fis = new FileInputStream("objectfile.ser");
        ObjectInputStream ois = new ObjectInputStream(fis);
        MyObject tmp3 = (MyObject)ois.readObject();
        MyObject tmp4 = (MyObject)ois.readObject();

        System.out.println(tmp3.val);//result : 1
        System.out.println(tmp4.val);//result : 2
        System.out.println(tmp4.x);//result : null

        ois.close();
    }
}

class MyObject implements java.io.Serializable{
    public final int val;
    transient Object x=new Object();
    MyObject(int val){
        this.val=val;
    }
}
```

클래스의 버전관리
-------------

- 직렬화된 객체를 직렬화한 후에 역직렬화 하더라도 그 사이에 클래스가 변했으면 역직렬화가 실패하며 예외가 발생(InvalidClassException)
- static변수, 상수, transient가 붙은 인스턴스 변수가 추가되면 상관 없음
- 수동으로 클래스의 버전을 관리하고 싶으면 **serialVersionUID** 를 정의한다
- serialVersionUID는 중첩을 막기위해 **serialver 클래스** 를 콘솔에서 실행해서 생성된 값을 사용하는 것이 일반적


```java
class MyObject implements Serializable{
  static final long serialVersionUID = 351873167521231L;
  //코드
}
```
