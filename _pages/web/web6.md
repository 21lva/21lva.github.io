---
layout: post
title:  "웹 - 반응형 웹 && 미디어 쿼리"
date:   2019-06-20 02:02:59
author: Inhyuk
category: Web
tags: web
cover:  "/assets/instacode.png"
name: 2019-06-22-web6.md
---

반응형 웹
=======

- 접속하는 브라우저, 기기, 환경에 맞춰서 레이아웃을 자연스럽게 보여주는 디자인
- 장점 : 유연한 레이아웃 변화, 다양한 스마트 기기에 대응 가능.
- 단점 : 모바일 사이트에 비해 무겁다. 모바일과 데스크탑 사이트의 기능을 다르게 할 수 없다.
- 반응형 웹사이트에 쓰이는 기술
1. 가변 그리드(flexible grid, flexible layout) : 화면의 크기에 맞게 늘어나도록 픽셀(px)대신 %를 사용하는 기술.
2. 미디어 쿼리 : 미디어의 종류에 맞게 웹사이트를 변경하는 기술
3. 뷰포트 : 화면에 보이는 영역을 제어하는 기술. 스마트폰의 경우 미디어쿼리만으로 제대로 작동안할 때가 있음.
4. 플렉서블 박스 : 가변적인 박스를 만드는 기술.


- - -

미디어 쿼리
==========

- css3 모듈 중 하나로 사이트에 접속하는 장칭 따라 특정한 css 스타일을 사용하도록 함
- **@media** 속성을 script 태그 안에 사용하여 특정 미디어에서 어떤 css를 적용할 것 인지를 지정

```css
미디어 쿼리 : @media [only|not] 미디어 유형 [and 조건] [and 조건]

ex)
@media screen and (min-width:200px) and (max-width:400px){
}

### 외부 파일 적용시
<link rel="stylesheet" media=미디어 쿼리 href="css파일 경로"> //주로 사용하는 방식
@import url(css파일 경로) 미디어 쿼리 조건
```

- 미디어 유형은 다음과 같다
1. **all** : 모든 미디어
2. **print** : 인쇄 장치
3. **screen** : 컴퓨터, 스마트폰 스크린
4. **tv** : TV
5. etc

1) 미디어 쿼리의 조건
---------------

#### (1) 웹 문서의 가로, 세로

- 뷰포트의 높이와 너비를 조건으로 사용
- width, height, min-width, min-height, max-width, max-height

#### (2) 단말기의 가로, 세로

- 브라우저가 아닌 단말기에 따른 가로, 세로를 사용
- device-width, device-height, min-device-width, min-device-height, max-device-width, max-device-height

#### (3) 회전

- orientation:portrait, orientation:landscape

#### (4) 화면 비율

- 브라우저 : aspect-ratio(화면 비율, width값/height값), min-aspect-ratio, max-aspect-ratio
- 단말기 : device-aspect-ratio(화면 비율, width값/height값), min-device-aspect-ratio, max-device-aspect-ratio

2) 중단점
--------

- **break point(중단점)** : 미디어 쿼리를 이용하여 서로 다른 css를 적용할 화면의 크기
- 모든 기기를 적용할 수 없기에 스마트폰, 태블릿, 데스크톱을 위주로 작성
- 스마트폰 : 모바일 페이지는 미디어 쿼리를 사용하지 않고 기본으로 css 사용. 방향을 고려한다면 세로(portrait)일 때 min-width:320px, landscape는 min-width:480px
- 테블릿 : 화면 크기가 portrait : 768px이상 landscape : 1024px이상
- 데스크톱 : 화면 크기가 1024px이상

- - -

뷰 포트
==========

- 렌더링 영역 : 실제 웹페이지가 그려지는 화면의 크기
- 뷰 포트 : 화면에서 실제 내용이 표시되는 영역
- 작은 화면에 웹페이지를 모두 표시하려다보니(풀브라우징) 전체적인 페이지의 배율이 조정된다. 이 화면에 맞게 적용하는 기술이 필요하다.

- *meta* 태그를 사용하여 *head* 태그 안에 작성한다.

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
  <th>width</th>
  <td>뷰포트 가로</td>
  <td colspan="4">
  <span style="font-weight:bold">device-width</span><br>
  <span style="font-weight:bold">크기</span><br>
  </td>
</tr>
<tr>
  <th>height</th>
  <td>뷰포트 세로</td>
  <td colspan="4">
  <span style="font-weight:bold">device-height</span> <br>
  <span style="font-weight:bold">크기</span>
  </td>
</tr>
<tr>
  <th>user-scalable</th>
  <td>사용자가 확대/축소 가능 여부</td>
  <td colspan="4">
  <span style="font-weight:bold">yes</span> : 기본값<br>
  <span style="font-weight:bold">no</span>
  </td>
</tr>
<tr>
  <th>initial-scale</th>
  <td>초기 확대/축소 값</td>
  <td colspan="4">
  <span style="font-weight:bold">1~10</span>, 기본값 : 1
  </td>
</tr>
<tr>
  <th>minimum-scale</th>
  <td>최소 확대/축소 값</td>
  <td colspan="4">
  <span style="font-weight:bold">0~10</span>, 기본값 :0.25
  </td>
</tr>
<tr>
  <th>maximum-scale</th>
  <td>최대 확대/축소 값</td>
  <td colspan="4">
  <span style="font-weight:bold">0~10</span>, 기본값 : 1.6
  </td>
</tr>
</tbody>
</table>

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```
