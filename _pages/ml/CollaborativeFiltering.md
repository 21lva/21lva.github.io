---
layout: post
title:  "Collaborative Filtering"
date:   2019-01-15 02:00:00
author: Inhyuk
categories: Machine_Learning
tags:	ml
cover:  "/assets/instacode.png"
name: CollaborativeFiltering.md
---

**Collaborative filtering** (협업 필터링)은 **Recommendation System**(추천 시스템)의 주된 방식 중 하나이다. 사람들 혹은 contents(아이템)들의 특징을 분석하여 그 contents들을 사람들에게 추천해주는 것이 **Recommendation system** 이다. 그 중에서도 가장 많이 쓰이는 것은 **Collaborative system**(이하 CF) 이고, **Recommendation System** 이라고 하면 보통 **CF** 를 의미한다.

이번 포스트에서는 **Collaborative Filtering** 에 대해 다룬다.

- - -

CF vs Content-based
===================

**Collaborative filtering** 과 **Content-based** 을 소개하는 글들에서 가끔 **content-based filtering** 을 content based CF라고 소개하는 곳도 있고, 따로 소개하는 곳도 있다.(분류가 별로 중요한것 같지는 않지만)

**CF** 는 어떤 물건들을 추천 할 때, 비슷한 유저 혹은 비슷한 아이템을 살펴보고 비슷한 것들을 위주로 추천을 진행하는 것이다. 이와 달리 **Content based** 는 말 그대로 content를 사용한다. 예를 들어, 어떤 영화를 추천할 때, 그 영화가 어떤 성향의 영화인지, 폭력성 및 선정성 등의 정도가 어느 정도인지등을 이용하여 추천을 진행한다.

**CF** 의 경우에는 **cold start** 문제가 존재한다. **Cold start** 문제란 첫 유저나 아이템등이 평가의 부족으로 인하여 추천을 진행하기 어려운 것을 뜻한다. 또한, **CF** 의 경우에는 data의 양이 많아야 한다.

**Content-based** 의 경우에는 **Cold start** 문제는 적지만, 계속해서 비슷한 아이템들만을 추천하는 문제가 있다.

- - -

Memory Based CF
===============

Memory based CF는 유저가 아이템을 평가(rating)한 정보를 memory에 전부 저장하고, 비슷한 값들을 이용하여 사용하는 기법이다.

Similarity
----------

비슷한 정도(유사도)는 주로 **Cosinee similarity** 와 **Pearson Similarity** 를 주로 사용한다.

#### Cosine Similarity
유저가 어떤 아이템을 평가한 정보를 벡터화 시키고 가장 그 벡터들 사이의 각도에서 생성되는 cosine 값을 이용하여 비슷한 정도를 따진다.

$$
\begin{matrix}
v_1 ,v_2 : \text{vectors of two Users}\\
cos\theta = \frac{v_1\bullet v_2}{|v_1||v_2|}\\
-1 \leq cos\theta \leq 1
\end{matrix}
$$

예를 들어 보자.

|    |대부|스타워즈|
|----|----|-------|
|영희|1   |2      |
|철수|2|1|          
|미향|3|6|          

다음과 같은 경우에 영희와 가장 성향이 비슷한 사람은 누구일까?

$$
\begin{matrix}
cosine(\text{영희,철수})=\frac{2+2}{\sqrt{5}\sqrt{5} }=\frac{4}{5 }\\
cosine(\text{영희,미향})=\frac{3+12}{\sqrt{5}\sqrt{45} }=\frac{15}{\sqrt{225} }=1
\end{matrix}
$$

한 눈에 보기에는 영희와 철수가 같은 유사도를 보이는 것 같지만, **cosine similarity** 를 계산하면 그렇지 않다. 단순히 두 vector사이의 각도만을 비교 대상으로 보기 때문이다.

#### Pearson Similarity
cosine similarity에서 vector에 평균값을 빼고 계산한다.
평균을 고려하기 때문에 극단적인 값을 주는 유저에 의해서 발생하는 문제를 어느 정도 해결 할 수 있다.

$$
\begin{matrix}
v_1 ,v_2 : \text{vectors of two Users}\\
Pearson(v_1,v_2) = \frac{(v_1-mean(v_1))\bullet (v_2-mean(v_2))}{|v_1-mean(v_1)||v_2-mean(v_2)|}\\
-1 \leq pearson(v_1,v_2) \leq 1
\end{matrix}
$$

User based
----------

비슷한 선호도를 가지는 유저들을 비교하여 추천을 진행한다. 예를 들어, 철수와 영희가 비슷한 선호도를 가지는 경우, 철수가 '다크나이트' 라는 영화를 좋아하면 영희한테도 '다크나이트'를 추천하는 것이다.

|   |다크나이트| 레미제라블 | 군함도 | 집으로 |
|---|--------|----------|------|-------|
|철수|1       |-1         |1    |x      |
|영희|-1      |1        |-1    |-1     |
|동현|-1      |-1        |-1    |1      |

다음과 같은 상황(긍정적 평가는 1, 부정적 평가는 0으로 나타내었다)에서 철수의 '집으로'의 평가가 긍정적인지 부정적인지를 예측해본다고 해보자.
Cosine similarity를 이용하여 철수와 영희, 철수와 동현의 유사도를 계산하자.

$$
\begin{matrix}
cos(\text{철수,영희}) = \frac{-3}{3}\\
cos(\text{철수,동현}) = \frac{-2}{3}
\end{matrix}
$$

즉, 철수와 영희의 유사도가 철수와 동현의 유사도보다 작기 때문에 동현의 '집으로'의 긍정적 평가가 철수의 평가라고 예측할 수 있다.

Item based
----------

어떤 사람에게 무엇인가를 추천하려 할 때, 그 유저가 평가한 아이템과 추천하려 하는 아이템의 유사도를 측정하여 비슷한 유사도를 가지는 이미 평가한 아이템의 평가를 가지고 추천하는 것이다.

위의 표를 다시 이용해보자.

$$
\begin{matrix}
cos(\text{다크나이트,집으로})=\frac{0}{2} \\
cos(\text{레미제라블,집으로})=\frac{-2}{2} \\
cos(\text{군함도,집으로})=\frac{0}{2}
\end{matrix}
$$

가장 유사하지 않은 것은 다크나이트와 군합도에서의 철수의 긍정적 평가가 집으로에서도 이루어질 것이라고 예측한다.

이 외에도 KNN(K nearest neighbors)을 이용하여, K개의 가장 유사한 아이템의 평균을 내는 방식을 사용하기도 한다.

하지만, 상술한 cold start문제가 이 방법을 이용하는 경우에는 더욱 강하게 나타난다.

- - -

Model based
===========

Model based는 model을 만들어서 예측을 진행한다. 많은 방법들이 있지만, 가장 많이 사용하는 matrix factorization(SVD,PCA)를 주로 설명하고자 한다.

SVD
---

**Singular value decomposition** 은 matrix를 3부분으로 나눈다.

$$
\begin{matrix}
X = UDV^t\\
X : \text{an original matrix with } m \times n\\
U : \text{a matrix with } m \times m\\
D: \text{a diagonal matrix with } m \times n\\
V : \text{a matrix with } n \times n
\end{matrix}
$$

이 때, $U$와 $V$는 *orthogonal* 한 성질이 있다.($UU^t = I$,$VV^t=I$)(singular value는 $AA^t$의 eigenvalues의 양의 제곱근을 의미한다.)

X를 두 부분으로 나누면 $UD$과 $V^t$이 된다. $UD$를 유저의 latent vector, $V$를 아이템의 latent vector라고 할 수 있다. 이렇게 latent vector는 유저의 특성을, 아이템의 특성을 나타내므로 이를 이용하여 다른 추천을 진행할 수도 있다. 또한, SVD를 이용하는 이유는 차원을 축소하여 원하는 만큼 메모리를 줄여서 사용할 수 있기 때문이다.

$$
\begin{matrix}
X_k = U_k{D}_k {V^t}_k\\
X_k : \text{an original matrix with } m \times n\\
U_k : \text{a matrix with } m \times k\\
{D}_k: \text{a diagonal matrix with }k \times k\\
V_k : \text{a matrix with }n \times k
\end{matrix}
$$

latent vector의 크기를 $k$로 고정하면, 원래의 $X$에 근사한 $X_k$를 만들 수 있다.(적절하지 못한 k를 고르면 $X_k$와 $X$의 차이가 너무 커진다. 적당한 값을 구하는 것이 좋다.)

하지만 볼 수 있듯이 행렬의 모든 값들이 정해져 있어야 SVD를 진행할 수 있다. 비어 있는 공간이 있는 상태에서 SVD를 진행할 수 없다. 이렇게 비어있는 부분이 있는 경우에도 SVD를 이용하는 몇 가지 방법이 있다.

첫 번째로는 임의의 값(평균 등)을 채우고 SVD를 진행하는 것이다.
하지만 이 방법으로는 빈칸이 많은 경우에는 SVD를 진행하면 biased가 매우 크게 된다.(비어있는 부분으로 SVD를 진행하고 임의의 user에 대해 어떤 item을 추천하려 한다면 당연히 문제가 발생한다.)

다른 방법으로는 $UD$와 $V$를 직접 계산하는 것이다.

$$
\begin{matrix}
min \sum(r_{ui}-p_u\bullet q_i)^2\\
r_{ui}: \text{a rating of user u to item i}\\
p_u: \text{u th row of } UD\\
q_i: \text{i th row of }V
\end{matrix}
$$

위의 최소화를 r이 존재하는 모든 부분에서 진행한다. SVD와 PCA의 더 정확한 수학적 구조는 다음에 다루도록 하겠다.

참고자료
======

- <https://grouplens.org/blog/similarity-functions-for-user-user-collaborative-filtering/>
- <https://www.fun-coding.org/recommend_basic3.html>
- <http://nicolas-hug.com/blog/matrix_facto_3>
- <http://sanghyukchun.github.io/73/>
- <http://aircconline.com/ijcsea/V6N3/6316ijcsea01.pdf>
