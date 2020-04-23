---
layout: post
title:  "Markdown 수식"
date:   2018-12-26 01:01:59
author: Inhyuk
categories: Markdown
tags:	markdown
cover:  "/assets/instacode.png"
---


포스트에 자주 쓰일 수식과 그래프를 그리는 방법을 알아보자.
보통 사용하는 테마는 [Mathjax](https://www.mathjax.org/ "mathjax")를 제공하지 않는다.
그러므로 **_includes** 폴더의 **post.html**의 **<article>** 태그 바로 밑에 다음들을 적어 놓자.

```
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      processEscapes: true
    }
  });
</script>
```
```
<script type="text/javascript" async
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
```
위의 부분은 inline 코드를 쓸때 $를 사용할 수 있게 해주는 부분이고,
아래 부분이 mathjax를 사용하겠다는 코드이다.
위->아래 순으로 적으면 된다.

기본 사용법
-----------
1. 인라인 수식
```
이것을 $a^2 + b^2 = c^2$ 이렇게
$a^2 + b^2 = c^2$ 적을 수 있습니다.
```
이것을
$a^2 + b^2 = c^2$ 이렇게
$a^2 + b^2 = c^2$ 적을 수 있습니다.

2. 블록 수식
```
$$a^2 + b^2 = c^2$$
$$\int f(x)~dx$$
```

$$a^2 + b^2 = c^2$$

$$\int f(x)~dx$$

Latex로 바꾸는 것은 여러 사이트에 있다
<http://eclass.dongguk.edu/jsp/common/equation_editor.jsp>
와 같은 사이트에서 그냥 바꿔서 사용하자.

참고 사이트
----------
<http://juliahwang.kr/%EC%A7%80%ED%82%AC%20%EB%B8%94%EB%A1%9C%EA%B7%B8/2017/09/30/mathjaxjekyllblog.html>

<http://wani.kr/posts/2015/12/17/add-mathjax-to-jekyll/>

[Mathjax 문법](http://www.onemathematicalcat.org/MathJaxDocumentation/TeXSyntax.htm)
