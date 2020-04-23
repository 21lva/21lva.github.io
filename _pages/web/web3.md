---
layout: post
title:  "CSS - 기본 : 스타일, 포지셔닝, 레이아웃, 배경, 표"
date:   2019-06-18 02:02:59
author: Inhyuk
categories: Web
tags: web
cover:  "/assets/instacode.png"
name: web3.md
---

스타일
=========

- 내부 스타일 시트는 head태그 안에 정의해야 한다.
- 외부 스타일 시트는 파일 경로를 추가한다.
- 인라인 스타일은 해당 태그 속성을 통해 정의한다.
- 전체에 적용하고 싶으면 선택자로 \* 를 사용한다.
- css는 여러 가지의 스타일이 존재하면 우선순위를 두고 적용을 한다. 다음은 그 순서이다.

1. 속성 값 뒤에 !important 를 붙인 속성
2. 인라인으로 style을 직접 지정한 속성
3. #id 로 지정한 속성
4. .클래스, :추상클래스 로 지정한 속성
5. 태그이름 으로 지정한 속성
6. 상위 객체에 의해 상속된 속성

1) 기본 선택자

```
1. 태그 : 태그이름{스타일 속성 : 속성값;}
2. class : .class{스타일 속성 : 속성값;}
3. id : #id{스타일 속성 : 속성값;}
4. 속성:
태그[속성]{} -> 해당 속성이 정의된 태그 선택
태그[속성=값]{} -> 정의된 속성과 속성값이 동일한 태그 선택
태그[속성~=값]{} -> 공백으로 구분되는 속성 값들 중 해당 값과 같은 것이 있는 태그
```

2) 가상 선택자

- 태그의 속성 중에는 눈에 보이지 않는 속성들이 있음
- 이러한 속성을 선택할 때에는 가상 선택자를 이용한다

```
태그:가상선택자{스타일 속성: 값}

ex)
a:link{} -> 사용자가 방문하지 않은 장소 표시
a:visited{} -> 사용자가 방문한 곳 표시
a:active{} -> 링크를 클릭하는 순간을 표시
a:hover{} -> 링크에 마우스 포인터를 올리는 순간
```

3) 조합 선택자

- 여러 선택자를 조합하여 선택하는 방법

```
1. 부모에 포함된 자식을 선택
부모 후손{}
2. 부모의 직계자식 선택자 선택
부모>자식{}
3. 그룹
선택자1, 선택자2, 선택자3{}
```


4) 스타일 지정 형식

```html
<!--내부 스타일 형식-->

<head>
<style>
p{color:red;}
.class{align:middle;}
#id{color:blue;}
#id1, #id2{
  color:red;
}//복수의 선택자에 적용
</style>
</head>

<!--외부 스타일 형식-->
<link href="경로" rel="stylesheet" type="text/css">

<!--inline-->
<p style="color:blue;">
  This is a paragraph
</p>
```

- - -

텍스트 관련 스타일
===============

1) 글꼴
---

<table>
<thead>
  <tr>
    <td>속성</td>
    <td>설명</td>
    <td colspan="4">속성 값</td>
  </tr>
</thead>
<tbody>
  <tr>
    <th>font-famly</th>
    <td> 웹에서 사용할 글꼴 정의</td>
    <td colspan="4"></td>
  </tr>
  <tr>
    <th>font-size</th>
    <td>글자 크기 조절</td>
    <td colspan="4">
    <span style="font-weight:bold">xx-small, x-small, small, medium, large, x-large, xx-large</span> : 브라우저에서 정한 크기 <br>
    <span style="font-weight:bold">99px</span> : 픽셀<br>
    <span style="font-weight:bold">27em</span>: 해당 글꼴의 M의 너비 기준으로 조절<br>
    <span style="font-weight:bold">larger,smaller</span> : 부모의 요소보다 작게, 크게 <br>
    <span style="font-weight:bold">백분율</span> : 부모의 요소 대비
    </td>
  </tr>
  <tr>
    <th>font-weight</th>
    <td>글자 굵기 조절</td>
    <td colspan="4">
    <span style="font-weight:bold">normal</span> : 일반적인 형태<br>
    <span style="font-weight:bold">lighter,bold,bolder</span> : 가늘게, 굵게, 더 굵게<br>
    <span style="font-weight:bold">숫자(100~900)</span> : 세밀하게 조절, normal=400
    </td>
  </tr>
  <tr>
    <th>font-style</th>
    <td>글자 형태 지정</td>
    <td colspan="4">
    <span style="font-weight:bold">normal</span> : 일반적인 형태<br>
    <span style="font-weight:bold">italic</span> : 이탤릭체
    </td>
  </tr>
</tbody>
</table>
<!--
<tr>
  <th></th>
  <td></td>
  <td colspan="4">
  <span style="font-weight:bold"></span> : <br>
  <span style="font-weight:bold"></span> : <br>
  <span style="font-weight:bold"></span> :
  </td>
</tr>
-->

- 여러 줄에 크기, 굵기 등을 조절하는 것이 귀찮으면 한줄에 사용할 수 있다.

```html
#id1{
  font-family:"굴림","고딕";
  font-size:12px;
  font-style:italic;
}
#id2{
  font:bold italic 12px;
}
```

2) 텍스트
-----

<table>
<thead>
  <tr>
    <td>속성</td>
    <td>설명</td>
    <td colspan="4">속성 값</td>
  </tr>
</thead>
<tbody>
<tr>
  <th>color</th>
  <td>텍스트 색상</td>
  <td colspan="4">
  <span style="font-weight:bold">rgb(x,y,z)</span> : rgb값으로 정의<br>
  <span style="font-weight:bold">red, green etc.</span> : 문자로 지정<br>
  <span style="font-weight:bold">#ff0000</span> : 16진수로 정의
  </td>
</tr>
<tr>
  <th>text-decoration</th>
  <td>텍스트의 줄 관리</td>
  <td colspan="4">
  <span style="font-weight:bold">none</span> : 선 제거<br>
  <span style="font-weight:bold">underline</span> : 밑줄<br>
  <span style="font-weight:bold">overline</span> : 위에 선 그림<br>
  <span style="font-weight:bold">line-through</span> : 취소선
  </td>
</tr>
<tr>
  <th>letter-spacing</th>
  <td>글자 사이 간격 조절</td>
  <td colspan="4">
  <span style="font-weight:bold">normal</span> : 보통크기<br>
  <span style="font-weight:bold">xx em</span> : xx만큼 크기 조절
  </td>
</tr>
<tr>
  <th>word-spacing</th>
  <td>단어 사이 간격 조절</td>
  <td colspan="4">
  <span style="font-weight:bold">normal</span> : 보통크기<br>
  <span style="font-weight:bold">xx em</span> : xx만큼 크기 조절
  </td>
</tr>
<tr>
  <th>text-align</th>
  <td>텍스트 정렬</td>
  <td colspan="4">
  <span style="font-weight:bold">left, right, center</span> : 왼쪽, 오른쪽, 가운데 정렬<br>
  <span style="font-weight:bold"></span> : xx만큼 크기 조절
  </td>
  sdfDFSF
</tr>
<tr>
  <th>text-align</th>
  <td>단어 사이 간격 조절</td>
  <td colspan="4">
  <span style="font-weight:bold">normal</span> : 보통크기<br>
  <span style="font-weight:bold">xx em</span> : xx만큼 크기 조절
  </td>
</tr>
<tr>
  <th>text-indent</th>
  <td>텍스트 들여 쓰기</td>
  <td colspan="4">
  <span style="font-weight:bold">크기</span> : px로 조절<br>
  <span style="font-weight:bold">백분률</span> : 부모 기준으로 정렬
  </td>
</tr>
<tr>
  <th>text-justify</th>
  <td>텍스트 정렬시 공백</td>
  <td colspan="4">
  <span style="font-weight:bold">auto</span> : 자동<br>
  <span style="font-weight:bold">inter-word</span> : 단어 사이의 공백을 조절<br>
  <span style="font-weight:bold">distribute</span> : 인접한 글자 사이의 공백을 맞춤<br>
  <span style="font-weight:bold">none</span> : 정렬 안함
  </td>
</tr>
<tr>
  <th>line-height</th>
  <td>줄 간격 조절</td>
  <td colspan="4">
  <span style="font-weight:bold">normal</span> : 기본<br>
  <span style="font-weight:bold">숫자</span> : 몇 배수<br>
  <span style="font-weight:bold">백분율</span> : 백분율<br>
  <span style="font-weight:bold">크기</span> : 실제 크기
  </td>
</tr>
</tbody>
</table>

3) 목록 스타일
----------

- 목록과 링크등을 조절할 수 있음

<table>
<thead>
  <tr>
    <td>속성</td>
    <td>설명</td>
    <td colspan="4">속성 값</td>
  </tr>
</thead>
<tbody>
<tr>
  <th>list-style-type</th>
  <td>목록의 불릿(순서 없는 목록의 앞 표시)와 번호 스타일 변경</td>
  <td colspan="4">
  <span style="font-weight:bold">순서 없음(disc, circle, square, none)</span> : 채운 원, 빈 원, 사각형, 없음 <br>
  <span style="font-weight:bold">decimal, decimal-leading-zero, lower-roman, upper-roman, lower-alpha, upper-alpha</span> : 순서 목록의 표기 방식<br>
  <span style="font-weight:bold">none</span> : 없애기
  </td>
</tr>
<tr>
  <th>list-style-image</th>
  <td>불릿 대신 이미지 넣기</td>
  <td colspan="4">
  <span style="font-weight:bold">(url(경로))</span> : 경로의 이미지 삽입<br>
  <span style="font-weight:bold">none</span> : list-style-type에서 지정한 형식으로 표기
  </td>
</tr>
<tr>
  <th>list-style-position</th>
  <td>목록에 들여 쓰는 효과</td>
  <td colspan="4">
  <span style="font-weight:bold">inside</span> : 들여쓰기<br>
  <span style="font-weight:bold">outside</span> : 기본 값. 불릿을 밖으로 내어 씀
  </td>
</tr>
<tr>
  <th>list-style</th>
  <td>위의 목록 속성을 한번에 표기</td>
  <td colspan="4">
  </td>
</tr>
</tbody>
</table>

- - -

배경
====

- 색상은 16진수, 색상 이름, rgb값으로 표기 가능(css3에서는 hsl도 이용. hsl = 색상, 채도, 명도)
<table>
<thead>
  <tr>
    <td>속성</td>
    <td>설명</td>
    <td colspan="4">속성 값</td>
  </tr>
</thead>
<tbody>
<tr>
  <th>background-color</th>
  <td>배경 색 지정</td>
  <td colspan="4">
  </td>
</tr>
<tr>
  <th>background-clip</th>
  <td>배경 적용 범위 조절</td>
  <td colspan="4">
  <span style="font-weight:bold">border-box</span> : 가장 외곽까지 적용. 기본 값<br>
  <span style="font-weight:bold">padding-box</span> : 테두리를 제외한 패딩까지 적용<br>
  <span style="font-weight:bold">content-box</span> : 내부만 적용
  </td>
</tr>
<tr>
  <th>background-image</th>
  <td>배경 이미지 넣기</td>
  <td colspan="4">
  <span style="font-weight:bold">url(경로)</span> : 경로의 이미지 사용
  </td>
</tr>
<tr>
  <th>background-repeat</th>
  <td>배경 이미지 반복 방법</td>
  <td colspan="4">
  <span style="font-weight:bold">repeat</span> : 가득 찰 때까지 가로와 세로로 반복. 기본 값<br>
  <span style="font-weight:bold">repeat-x</span> : 가득 찰 때까지 가로로 반복<br>
  <span style="font-weight:bold">repeat-y</span> : 가득 찰 때까지 세로로 반복<br>
  <span style="font-weight:bold">no-repeat</span> : 반복 없음
  </td>
</tr>
<tr>
  <th>background-size</th>
  <td>배경 이미지 크기 조절</td>
  <td colspan="4">
  <span style="font-weight:bold">auto</span> : 원 이미지 크기로 표시. 기본 값<br>
  <span style="font-weight:bold">contain</span> : 요소안에 다 들어오도록 확대 및 축소<br>
  <span style="font-weight:bold">cover</span> : 가득 찰 때까지 확대 또는 축소(여백x)<br>
  <span style="font-weight:bold">크기 값</span> : 너비와 높이값을 지정<br>
  <span style="font-weight:bold">백분률</span> : 요소를 기준으로 백분률값으로 확대 및 축소
  </td>
</tr>
<tr>
  <th>background-position</th>
  <td>배경 이미지 표시 위치 조절</td>
  <td colspan="4">
  <span style="font-weight:bold">수평 위치</span> : left, center, right, 백분률, 길이 값<br>
  <span style="font-weight:bold">수직 위치</span> : top, center, bottom, 백분률, 길이 값<br>
  <span style="font-weight:bold">cover</span> : 가득 찰 때까지 확대 또는 축소(여백x)<br>
  <span style="font-weight:bold">크기 값</span> : 너비와 높이값을 지정<br>
  <span style="font-weight:bold">백분률</span> : 요소를 기준으로 백분률값으로 확대 및 축소
  </td>
</tr>
<tr>
  <th>background-attachment</th>
  <td>배경 이미지 고정</td>
  <td colspan="4">
  <span style="font-weight:bold">scroll</span> : 고정 되지 않음. 기본 값<br>
  <span style="font-weight:bold">fixed</span> : 스크롤하더라도 배경이미지 변화 없음<br>
  <span style="font-weight:bold">cover</span> : 가득 찰 때까지 확대 또는 축소(여백x)<br>
  <span style="font-weight:bold">크기 값</span> : 너비와 높이값을 지정<br>
  <span style="font-weight:bold">백분률</span> : 요소를 기준으로 백분률값으로 확대 및 축소
  </td>
</tr>
<tr>
  <th>background</th>
  <td>배경관련 속성 지정</td>
  <td colspan="4">
  </td>
</tr>
</tbody>
</table>

- - -

레이아웃
========

- 블록 레벨 요소 : 태그를 사용해 요소를 삽입했을 때 혼자 한 줄을 차지하는 요소. ex) /div, p 태그
- 인라인 레벨 요소 : 줄을 차지 하지 않음.
- 박스 모델의 형태
![box]({{site.baseurl}}/post_img/{{page.name}}/box_model.png)
[이미지출처](http://tcpschool.com/css/css_boxmodel_boxmodel)

1) 콘텐츠 영역
----------

- 내용이 들어가 있는 영역

#### width, height

- 콘텐츠 영역의 크기를 지정.
- 크기, 백분율(부모의 상대적인 값), auto 사용.
- aside를 높이로 아래로 내리고 싶으면 부모인 html과 body의 height를 먼저 설정해야 한다.

#### display

- 블록레벨요소와 인라인 레벨 요소를 변경하는 기능
- block : 블록 레벨로 전환
- inline : 인라인 레벨로 전환
- inline-block : 한 줄로 배치하면서 너비와 높이, 마진등을 박스 모델값과 같게 적용
- none : 해당 요소를 표시하지 않음. 요소가 차지하고 있는 공간도 제거


2) Border
------

- 내용과 패딩 주변을 감싸는 테두리

#### border-style

- 테두리 스타일 지정
- none(기본값), hidden, dashed, dotted, double, groove, inset, outset, ridge, solid(실선)

#### border-width

- 테두리의 두께 지정
- border-top-width, border-right-width, border-left-width, border-bottom-width 도 존재
- thin, medium, thick

#### border-color

- 테두리 색상 지정
- border-top-color, border-right-color, border-left-color, border-bottom-color 도 존재

#### border

- 테두리 스타일 한번에 지정
- border-top, border-right, border-left, border-bottom 도 존재

#### border-radius

- 박스 모서리 둥글게 만들기
- border-top-left-radius, border-top-right-radius, border-bottom-left-radius, border-bottom-left-radius 등이 존재
- 반지름 크기, 백분률로 값을 지정
- 값을 두 가지 지정할 경우 타원형태로 만들기 가능

#### box-shadow

- 선택한 요소에 그림자 효과
- none : 기본 값
- 수평거리, 수직거리 : 반드시 지정해야 함.
- 흐림정도, 번짐 정도
- 색상
- inset : 지정 시 안쪽으로 그림자 생성

3) Margin
------

- 테두리와 이웃하는 요소 사이의 간격
- background-color 속성의 영향을 받지 않음

#### margin

- 요소 주변 여백 크기 설정
- margin-top, margin-left, margin-right, margin-bottom 존재
- 크기(px,pt,cm), 백분률(부모 요소 기준으로 백분율), auto(기본값) 가능

4) Padding
--------

- padding : 컨탠츠 영역과 border 사이의 여백
- padding-top, padding-left, padding-right, padding-bottom 존재
- 크기(px,pt,cm), 백분률(부모 요소 기준으로 백분율), auto(기본값) 가능

- - -

Positioning
============

- 포지셔닝 : 브라우저안에서 콘텐츠를 어떻게 배치할 것인지를 결정하는 것.

#### box-sizing

- 박스 너비 기준 결정
- content-box : width 값을 콘텐츠 영역 너비값으로 사용. 기본 값
- border-box : widht 값을 콘텐츠 영역에 테두리까지 포함한 값으로 사용

#### float

- 왼쪽이나 오른쪽으로 배치
- left, right, none(어느 쪽으로도 배치 하지 않는다)

#### clear

- float 사용 시 그 다음 요소들의 위치를 잡기 힘듬. -> clear를 통해서 속성을 없애주어야 함.

#### position

- 요소들을 배치하는 방법
- static : 흐름에 맞추어 배치. 기본 값
- relative : 이전 요소에 자연스럽게 연결해 배치하되 위치를 지정 가능. left, right, top, bottom 을 이용하여 상대적인 위치 설정
- absolute : 원하는 위치 지정. 기준으로 부모요소 혹은 조상 요소 중 relative 속성을 가진 요소를 사용
- fixed : 지정한 위치에 고정. 브라우저 창을 기준으로 좌표를 계산하여 위치 결정. absolute와 달리 스크롤을 하더라도 계속 유지된다.

```html
<style>
#relative1{
  background-color:red;
  position:static;
}
#relative2{
  background-color:blue;
  position:relative;
  left:30px;
}
</style>
<p id="relative1">P1</p>
<p id="relative2">P2</p>
```
<style>
#relative1{
  background-color:red;
  position:static;
}
#relative2{
  background-color:blue;
  position:relative;
  left:30px;
}
</style>
<p id="relative1">P1</p>
<p id="relative2">P2</p>

#### visibility

- 요소를 보이게 하거나 보이지 않게 함
- 보이지 않지만 공간은 차지한다.
- visible(기본값), hidden(요소를 감춤. 크기는 유지)

### z-index

- 어느 객체가 앞으로 나오고 뒤에 나올지를 배치하는 속성
- position이 relative, absolute, fixed인 경우만 사용
- auto : 설정하지 않음
- 숫자 : 높을수록 앞에 나옴
- initial : 기본값 사용

- - -

다단
====

- column(단)을 사용하여 페이지를 여러개로 분리 가능

#### column-width

- 단의 너비 고정하고 다단 구성
- 크기 : 단의 너비 지정, 너비가 고정되면 화면에 따라 단의 개수가 바뀐다
- auto : 단의 개수(column-count)가 고정되면 브라우저에 맞게 단의 너비가 정해짐

#### column-count

- 단의 개수 고정
- 숫자 : 단의 개수를 지정(>0)
- auto : 단의 너비에 따라 단의 개수가 자동으로 계산.

#### column-gap

- 단과 단 사이읭 여백 지정
- 크기, normal

#### column-rule

- 구분선의 스타일 지정
- column-rule-color : 색상
- column-rule-style : none, hidden, dotted, solid, double, groove, ridge, inset, outset
- column-rule-width : 크기, thin, medium, thick
- column-rule : 한 번에 지정

#### back-after

- 다단 위치 지정
- break-before, break-after, break-inside
- 특정 요소 앞부분에서 단을 나누려면 break-before:column사용
- 특정 요소 앞부분에서 단을 나누지 않게 하려면 break-before:avoid-column사용

- - -

표
====

#### caption-side

- 표 제목 위치 설정
- top, bottom

#### border

- margin에서 사용한것처럼 표의 테두리를 설정할 수 있다

#### border-collapse

- 표 바깥 테두리의 두 선을 합칠지 말지를 결정
- collapse, separate(기본 값)

#### border-spacing

- 인접한 셀들과의 거리를 정함

#### empty-cells

- 빈 셀 표시 여부 결정
- show(기본 값), hide

#### width, height

- 표 너비와 높이 지정

- - -

참고자료
=======

- [height 100% 적용](https://jos39.tistory.com/170)
