---
layout: post
title:  "CSS - Flexible Box, 가변 그리드 레이아웃"
date:   2019-06-20 02:02:59
author: Inhyuk
categories: Web
tags: web
cover:  "/assets/instacode.png"
name: 2019-06-22-web6.md
---

가변 그리드 레이아웃
======================

- grid system : 화면을 몇 개의 column으로 나누어 요소들을 배치하는 시스템
- 가변(fluid) 그리드 레이아웃 : 레이아웃을 백분율로 지정하여 브라우저에 따라서 요소의 값이 변하는 시스템

#### 전체를 감싸는 요소 확인

- 가변 그리드를 사용하기 위해서는 전체 웹 요소들을 감싸는 wrapper를 만들어야 한다
- wrapper의 width를 백분율 값으로 변환

```css
#wrapper{
  width:95%;
}
```

- 각 요소의 너비 = (요소의 너비/전체를 감싸는 요소의 너비)\*100

1) 가변 요소
-------------

- 브라우저에 따라 요소들의 너비가 자연스럽게 바뀌어야함

#### (1) 가변 마진 & 가변 패딩

- 마진의 크기도 %로 만들지 않으면 제대로 배치가 되지 않을 수도 있음.
- 패딩의 경우 width가 아닌 부모의 너비에서 %로 계산해야한다.

#### (2) 가변 글꼴

- px : 모니터(표시장치)에 따라 상대적인 값
- 텍스트 크기를 px로 지정하면 브라우저에 따라 너무 작아서 보이지 않을 수도 있다(브라우저에서 zoom 기능을 지원하기 때문에 요새는 그냥 px쓰기도 한다)
- em : 부모 요소에서 지정한 폰트의 대문자 *M* 의 너비를 1em(16px)으로 지정한다. 브라우저에 맞게
- 글자 크기(em) = 글자크기(px)/16
- rem : em과 비슷하지만 기준 문자가 html의 폰트사이즈이다.

#### (3) 가변 이미지

- 이미지를 삽입할 때 max-width를 percent로 지정하면 창에 따라 그 크기가 변한다. 보여주는 크기만 줄이기 때문에 다운받은 파일의 크기는 같다(과한 데이터 사용을 초래함)
- **srcset** 속성: pixel 밀도에 따라 다른 이미지를 보여줌

```html
<img src="a.jpg" srcset="a.jpg 2x" alt="연습용">
```

- - -


flexible box layout
===================

- 그리드 레이아웃을 기본으로 플렉스 박스를 위치시키는 것
- flex container(box) : 웹 문서에 요소들을 플렉서블하게 사용할 때 묶는 요소
- flex item : flex container에 담기는 요소들

1) 기본 속성
-----------

#### display

- flex box를 지정하기 위해서는 flex container의 display 속성을 flex 또는 inline-flex로 지정해야 한다
- flex : 박스 레벨 요소 / inline-flex : 인라인 레벨 요소

#### flex-direction

- flex item을 배치할 방향
- row, row-inverse, column, column-inverse

#### flex-wrap

- flex-direction만 사용시 한 줄로 배치한다
- flex-wrap을 이용하면 여러 줄로 배치 가능
- no-wrap : 한 줄로 표시. 기본 값
- wrap : 여러 줄에 표시
- wrap-reverse : 여러 줄로 표시 + 기존 방향과 반대

### flex-flow

- flex-direction과 flex-wrap 을 한 번에 지정

```css
flex-flow : flex-direction flex-wrap;
```

#### order

- flex item의 순서를 정한다
- order:순서(0~)

#### flex

- flex item의 크기 조절

```css
flex:[flex-grow flex-shrink flex-basis] | auto | initial
```

- initial : 항목의 width,height 갑세 의해 결정. flex container의 공간이 부족하면 최소 크기까지 줄어든다
- flex-basis : 기본 크기를 지정.

#### justify-content

- 주축(배치 방향)기준 배치 방법 지정
- flex-start : 시작점 기준(기본)
- flex-end : 끝점을 기준으로 배치
- center : 중앙
- space-between : 시점과 끝점에 하나씩 배치하고 중간에 같은 너비로 나머지를 배치
- space-around : 같은 간격으로 배치

#### align-items , align-self

- 교차축(주축의 수직 축)의 배치 방법 지정
- stretch : 꽉 교차축 기준 꽉 채움. 기본 값
- flex-start : 교차축의 시작점 기준으로 배치
- flex-end : 교차축 끝점을 기준
- center : 중앙
- baseline : 가장 큰 flex item을 기준으로 배치한다
