---
layout: post
title:  "HTML - 기본 태그 & 표 & 이미지 & 링크 & Form & Semantic Tag"
date:   2019-06-17 02:02:59
author: Inhyuk
category: Web
tags: web
cover:  "/assets/instacode.png"
name: 2019-06-17-web1.md
---

**자바스크립트** 를 공부하다보니 웹에 대해 관심을 가지게 되었고, 이전에 공부했던 **HTML, CSS** 등을 다시 공부해지고 싶어졌다.

- - -

HTML이란
========

- HyperText Markup Language 의 약자
- Tag를 이용하는 Markup + 링크를 이용하는 hyper text
- 웹 표준 : 모든 브라우저들이 사용하기 쉽게 웹에 이용하는 언어들의 표준을 정한 것. 여기서는 HTML5와 CSS3을 사용한다.
- HTML5 + CSS3를 이용하면 **반응형** + **interactive**(웹이 사용자의 동작에 바로 반응하는 사이트) 한 사이트를 만들 수 있다.

- - -

HTML 태그
=========

1) 기본 태그
--------

##### <!doctype>

- 문서 유형을 지정

```html
<!doctype html>
```

##### <html> </html>

- 웹 문서의 시작과 끝을 알림
- *lang* 속성을 추가하여 문서에서 사용하는 언어를 지정하기도 함
- 지정된 언어를 이용하여 검색에 사용하기도 함

```html
<html lang="ko">
```

##### <head> </head>

- 화면에 보이지 않지만 웹브라우저가 알아야 할 정보를 저장하는 부분
- 문서에서 사용하는 외부 파일, 문서 제목(<title> 제목 </title>), 문서 인코딩 정보(meta) 등을 포함

##### <body> </body>

- 문서의 내용을 저장하는 부분

2) 문서 관련 태그
-------------

```html
<p></p>
```

- 단락을 만드는 태그
- 앞뒤에 줄바꿈이 있음

```html
<br>
```
- 줄 바꾸기 태그

```html
<hr>
```

- 수평 줄 넣기

```html
<blockquote> </blockquote>
```html

- 인용문을 넣는다
- 다른 텍스트보다 안에 들여 써짐
- cite 속성을 이용하여 인용한 사이트 주소를 표기할 수 있음

```html
\<strong> \</strong> / \<b> \</b>
```

- 굵게 표시
- *b* 태그는 단순히 굵게 표시하는 기능
- *strong* 태그는 중요한 부분을 표시하는 기능. 웹 접근성에 기여

```html
\<em> \</em> / \<i> \</i>
```

- 이탤릭체로 표시
- *i* 는 *b* 와,  *em* 는 *strong* 과 대응된다.

```html
\<q> \</q>
```
- 인용한 내용을 표시
- *blockquote* 와 달리 줄을 바꾸지 않고 인라인으로 만듬

```html
\<mark> \</mark>
```

- 형광표시

```html
\<span> \</span>
```

- 줄 바꿈 없이 텍스트를 묶어주는 역할
- 혼자가 아닌 여러 다른 기능과 섞어 씀


```html
안녕 <span style="color:red;">나는</span> 사람이야.
```

안녕 <span style="color:red;">나는</span> 사람이야.

```html
<div>
```

- 박스 형태(한 줄을 차지)로 내용을 묶어준다.
- div는 p 태그를 포함할 수 있다. 하지만 그 반대는 성립하지 않는다. p태그는 주로 문장을 담는데에 사용된다.

##### 기타

```html
<s>취소선</s>
<u>밑줄</u>
<br>
대상<sub>아래첨자</sub>
<br>
대상<sup>위첨자</sup>
```

<s>취소선</s>
<u>밑줄</u>
<br>
대상<sub>아래첨자</sub>
<br>
대상<sup>위첨자</sup>

3) 목록을 만드는 태그
------------------

```html
<ul>
  <li>이것은</li>
  <li>순서가 없는</li>
  <li>목록이다.</li>
</ul>
<ol>
  <li>이것은</li>
  <li>순서가 있는</li>
  <li>목록이다.</li>
</ol>
```

<ul>
  <li>이것은</li>
  <li>순서가 없는</li>
  <li>목록이다.</li>
</ul>
<ol>
  <li>이것은</li>
  <li>순서가 있는</li>
  <li>목록이다.</li>
</ol>


```html
<dl>
  <dt>제목1</dt>
  <dd>내용1</dd>
  <dd>내용2</dd>
  <dt>제목2</dt>
  <dd>내용1</dd>
  <dd>내용2</dd>
</dl>
```

<dl>
  <dt>제목1</dt>
  <dd>내용1</dd>
  <dd>내용2</dd>
  <dt>제목2</dt>
  <dd>내용1</dd>
  <dd>내용2</dd>
</dl>

4) 표 만들기
---------

- table : 표 시작
- 각 행은 tr로, 각 행에서 한 칸은 td로 만든다
- 각 행의 가장 왼쪽을 제목을 붙이고 싶으면 그 칸에서는 td 대신 th를 쓴다.
- colspan, rowspan을 써서 열을 합치거나 줄을 합친다.
- caption : table 태그 바로 밑에 사용. 표의 제목을 만들고 가장 위 중앙 정렬
- figcaption : 제목을 만들고 왼쪽 정렬 밑 그 자리에 만듬(위로 올리지 않음)
- thead, tbody, tfoot : 좀더 명확하게 보여주기 위해 사용하는 태그

```html
<table>
  <caption> 연습용 표 </caption>
  <thead>  
    <tr>
      <th> 제목 행</th>
      <td> 제목행 1열</td>
      <td> 제목행 2열</td>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <th> foot은</th>
      <td> 어디에 놔도</td>
      <td> 가장 아래에 보인다</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <th> 1행 제목</th>
      <td> 1행 1열</td>
      <td> 1행 2열</td>
    </tr>
    <tr>
      <th> 2행 제목</th>
      <td> 2행 1열</td>
      <td> 2행 2열</td>
    </tr>
    <tr>
      <th> 3행 제목</th>
      <td> 3행 1열</td>
      <td> 3행 2열</td>
    </tr>
    <tr>
      <th> 4행 제목</th>
      <td colspan="2" align="center"> 3행 1열</td>
    </tr>
  </tbody>
</table>
```

<table>
  <caption> 연습용 표 </caption>
  <thead>  
    <tr>
      <th> 제목 행</th>
      <td> 제목행 1열</td>
      <td> 제목행 2열</td>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <th> foot은</th>
      <td> 어디에 놔도</td>
      <td> 가장 아래에 보인다</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <th> 1행 제목</th>
      <td> 1행 1열</td>
      <td> 1행 2열</td>
    </tr>
    <tr>
      <th> 2행 제목</th>
      <td> 2행 1열</td>
      <td> 2행 2열</td>
    </tr>
    <tr>
      <th> 3행 제목</th>
      <td> 3행 1열</td>
      <td> 3행 2열</td>
    </tr>
    <tr>
      <th> 4행 제목</th>
      <td colspan="2" align="center"> 3행 1열</td>
    </tr>
  </tbody>
</table>

- - -

5) 이미지 삽입
-----------

- img : 이미지를 삽입하는 태그
- figure : 설명을 붙일 대상을 지정
- figurecaption : 설명 글 붙이기

```html
<figure>
  <img id="welsh" src="welsh.jpg" alt="Welsh Corgi" width="50%" height="50%">
  <figurecaption>웰시코기</figurecaption>
</figure>

//외부 이미지 삽입
<img src="https://commons.wikimedia.org/wiki/File:Pembroke_Welsh_Corgi.jpg" alt="웰시코기 / 출처: 나무위키">
```

<figure>
  <img  src="./welsh.jpg" alt="Welsh Corgi" width="50%" height="50%">
  <figurecaption>웰시코기</figurecaption>
</figure>

<img id="welsh" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/45/Pembroke_Welsh_Corgi.jpg/800px-Pembroke_Welsh_Corgi.jpg" alt="웰시코기  출처: 위키백과">

6) 링크
----

- href : 링크할 주소 입력. 같은 페이지의 태그로도 건너띄기 가능
- target : 링크한 대상이 열릴 위치(새 창, 현재 화면)
- download : 링크 내용을 보여주는 것이 아니라 다운로드 함
- rel : 현재 문서와의 관계를 표시
- type : 링크한 문서의 파일 유형을 알려줌
- iframe : 현재 페이지에 다른 웹문서를 넣는 것. 동영상 같은것을 넣을 수도 있음

```html
<a href="https://www.google.com/" target="_blank">구글 새 창</a>
<a href="https://www.google.com/" target="_self">구글 현재 창</a>
<a href="#welsh">웰시코기 사진으로 이동</a>
<iframe href="https://youtu.be/d9RqiaILIh0">동영상 삽입</iframe>
```

<a href="https://www.google.com/" target="_blank">구글 새 창_</a>
<br>
<a href="https://www.google.com/" target="_self">구글 현재 창_ </a>
<br>
<a href="#welsh">웰시코기 사진으로 이동</a>
<p align="middle">
<iframe src="https://www.youtube.com/embed/jNAK7QL5JjI" frameborder="0">동영상 삽입</iframe>
</p>

- - -

Form
====

1) Input
-----

- form : 웹사이트로 정보를 보낼 수 있는 요소들
- form 태그 : 정보를 넘겨주는 방식, 주소등을 정의하는 것.
- fieldset 태그: 태그 사이의 폼들을 하나의 영역으로 묶는 윤관석을 그림
- legend 태그: 그룹의 제목을 보여줌
- placeholder 속성: 힌트를 표시
- require 속성 : 필수적으로 입력해야 하는 태그 지정
- submit : 폼에 입력한 정보를 서버로 전송하는 버튼
- reset : 적어놓은 모든 것들을 지운다.
- type="button" : 스크립트 함수를 이용하여 버튼을 클릭했을 시 이루어지는 동작을 정의하여 이용한다
- type="file" : 파일을 첨부. 찾아보기 등의 기능이 표시됨

```html
<form action="insert.php" method="get" name="first form">
  <fieldset>
  <legend>폼 연습</legend>
  <label>이름 : <input type="text" name="name" autocomplete="on" placeholder="이름"></label>
  <input type="password" name="pwd" required>
  <p>
  <input type="checkbox" name="c1" value="1">체크박스 1번
  <input type="checkbox" name="c2" value="2">체크박스 2번
  </p>
  <p>
  <input type="checkbox" name="r1" value="1">라디오 1번
  <input type="checkbox" name="r2" value="2">라디오 2번
  </p>
  <input type="hidden" id="hid" name="hid" value="yes">
  <input type="button" value = "ALERT" onclick="alert('Hello!');">
  <input type="file" value ="파일 첨부">
  <input type="reset">
  <input type="submit" value ="제출">
  </fieldset>
</form>
```
<form action="insert.php" method="get" name="first form">
  <fieldset>
  <legend>폼 연습</legend>
  <label>이름 : <input type="text" name="name" autocomplete="on" placeholder="이름을 입력하세요"></label>
  <input type="password" name="pwd" required>
  <p>
  <input type="checkbox" name="c1" value="1">체크박스 1번
  <input type="checkbox" name="c2" value="2">체크박스 2번
  </p>
  <p>
  <input type="checkbox" name="r1" value="1">라디오 1번
  <input type="checkbox" name="r2" value="2">라디오 2번
  </p>
  <input type="hidden" id="hid" name="hid" value="yes">
  <input type="button" value = "ALERT" onclick="alert('Hello!');">
  <input type="file" value ="파일 첨부">
  <input type="reset">
  <input type="submit" value ="제출">
  </fieldset>
</form>

- 이 외에도 number, url, email, color, date, month, week, time 등의 input type 속성들이 있다.

2) 나열
----

- 입력이 아닌 여러 개 중에 선택을 하게 하는 기능

```html
<form action="select.php" method="get">
  <fieldset>
  <legend>나열 연습</legend>
  <label>음식</label>
  <select name="음식">
    <optgroup label="일식">
      <option value="ramen">라멘</option>
      <option value="sushi">스시</option>
      <option value="pork">돈까스</option>
    </optgroup>
    <optgroup label="중식">
      <option value="jj">자장면</option>
      <option value="JJ">짬뽕</option>
      <option value="tang">탕수육</option>
    </optgroup>
  </select>
  <input type="submit" value="제출">
  </fieldset>
</form>
```

<form action="select.php" method="get">
  <fieldset>
  <legend>나열 연습</legend>
  <label>음식</label>
  <select name="음식">
    <optgroup label="일식">
      <option value="ramen">라멘</option>
      <option value="sushi">스시</option>
      <option value="pork">돈까스</option>
    </optgroup>
    <optgroup label="중식">
      <option value="jj">자장면</option>
      <option value="JJ">짬뽕</option>
      <option value="tang">탕수육</option>
    </optgroup>
  </select>
  <input type="submit" value="제출">
  </fieldset>
</form>

3) Output
----

- 입력한 값을 이용하여 결과를 알려줌

```html
<form oninput="result.value=parseInt(num1.value)-parseInt(num2.value)">
  <input type="number" name="num1" value="0">
  -<input type="number" name="num2" value="0">
  = <output name="result"></output>
</form>
```


<form oninput="result.value=parseInt(num1.value)-parseInt(num2.value)">
  <input type="number" name="num1" value="0">
  -<input type="number" name="num2" value="0">
  = <output name="result"></output>
</form>

4) Progress
--------

- 진행 상태를 보여줌

```html
<progress value="20" max="100"></progress>
```

<progress value="20" max="100"></progress>

- - -

Semantic tags
=============

- HTML5 부터는 **시멘틱 태그(semantic tags)** 가 추가되었다.
- 시멘틱 태그를 이용하면 어디가 본문이고, 어디가 제목인지 쉽게 알 수 있다.

#### header

- 머릿말에 해당
- 주로 검색(*form* 이용), 목록(*nav* 이용) 등을 보여주는 부분

#### nav

- 사이트 안의 다른 사이트나 문서로 이동할 수 있도록 목록을 보여주는 태그

#### section

- 주제별로 내용을 감싸는 태그
- *section* 안에 또 다른 *section*을 넣기도 하고, *artical* 태그를 사용하기도 한다.

#### article

- *section* 안의 글이 많이 들어간 부분을 의미

#### aside

- 좌우 사이드바를 의미
- css를 이용하여 이 부분을 좌우에 배치한다.

#### footer

- 꼬릿말에 해당
- 주로 제작자 정보, 저작권, 날짜 등의 내용을 담는다

#### address

- *footer* 안에 주로 사용한다. 웹페이지의 제작자 정보등을 표시하는 데에 이용된다.


- - -

참고자료
========

- [iframe](https://aboooks.tistory.com/205)
