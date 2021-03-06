---
layout: post
title:  "FTRL"
date:   2019-01-15 02:00:00
author: Inhyuk
category: Machine_Learning
tags:	ml
cover:  "/assets/instacode.png"
name: FTRL.md
---

이번 포스트에서는 [Ad Click Prediction](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/41159.pdf)의 **FTRL-proximal** 부분을 요약 및 번역을 할 것이다.

Abstract
========

광고의 **CTR(click-through rates)** 을 예측하는 것은 광고업계에서 중요한 문제이다.

이 문제를 해결하기 위하여 **FTRL-proximal** 이라는 online learning algorithm을 소개하려 한다. (그 외는 FTRL과 관련 없어서 생략)

Introduction
=============

Search advertising, contextual advertising, display advertising과 같은 광고 산업은 정말 많은 돈을 불러 모은다. 이러한 산업은 CTR의 성능에 의존하고 있다.

Brief System Overview
======================

User가 search q(query)를 시행하면, 학습되어있는 모델에서 광고들이 얼마나 적합한지, 어떤 순서로 보여주어야 하는지등을 결정한다. 그러기 위해서 가장 중요한 것은 $P(click|q,a)$(어떤 광고 $a$가 query $q$에서 주어졌을 때 그 광고가 $click$될 확률)를 estimate하는 것이다.

이 estimation을 위하여 다양한 방법들(including regularized logistic regression)이 사용된다. (여기서는 FTRL에 대한 부분은 적다.)

Online Learning And Sparsity
============================

Feature vector의 dimension이 매우 크더라도 실제로 각 instance에서 사용되는 non-zero features의 수는 비교적 적다. 모델이 sparse해야 하는 이유이다.

정확한 논의를 위하여 몇 가지 notataion을 정의할 것이다.

$\mathbf{g_t}$는 $t$번째 training instance의 loss의 gradient를 뜻하고 vector형식이다. 이 vector의 i번째 component(entry)는 $g_{t,i}$라고 표현할 것이다. 또한, $\mathbf{g_{1:t}}$는 $\sum_{t}^{s=1}\mathbf{g_s}$를 뜻한다.

logisitic regression을 이용할 경우, sigmoid를 output layer의 activation function으로 사용을 할 것이고, loss function으로 **negative log-likelihood** 를 사용할 것이다. 즉, time $t$에서의 weight(parameter) $\mathbf{w_t}$의 loss function과 gradient는 다음과 같다.

$$
l_t(\mathbf{w_t}) = -y_t logp_t - (1-y_t)log(1-p_t)

\bigtriangledown l_t(\mathbf{w}) = (\sigma(\mathbf{w}\bullet \mathbf{x_t})-y_t)\mathbf{x_t} = (p_t-y_t)\mathbf{x_t}
$$

weight update를 위하여 **SGD** (stochastic gradient descent)와 같은 다양한 방식들이 **OGD** (online gradient descent)를 사용해 왔다. 데이터양의 증가로 인해 batch 방식의 update의 계산량을 감당하기 힘들기 때문이다. (**SGD** 와 **OGD** 는 사실 같은 말이다. 여기서는 Batch 방식의 **SGD** 와 구분하기 위하여 OGD라는 말을 썼다. 즉, online stochastic gradient descent가 **OGD** 인 것이다.)

**OGD** 에서 weight을 update하는 식은 다음과 같았다.

$$
\mathbf{w_{t+1} } = \mathbf{w_t}-{\eta}_t\mathbf{\hat{g_t}}

\hat{g_t} \in subgradient[l_t(\mathbf{w_t}) + {\lambda}_1 {||\mathbf{w_t}||}_1]
$$

(${\eta}_t$ : a non-increasing learning rate schedule, e.g. ${\eta}_t = \frac{1}{\sqrt{t}}$)

이 식은 다음의 식과 같다.

$$
\mathbf{w_{t+1} } = argmin_{\mathbf{w} }(\mathbf{\hat{g_{1:t}} }\bullet \mathbf{w}+\frac{1}{2{\eta}_t}{ {||\mathbf{w}-\mathbf{w_t}||}_2}^2)
$$

이 식을 **Composite-Objective Online Gradient Descent(CO-OGD)** 이라고 부른다.

**OGD** 의 경우 sparse model을 생성하는 것에 좋은 방식이 아니기 때문에 **FOBOS** 나 **RDA** 와 같은 방법들이 제안되었다. **RDA** 의 경우 **FOBOS** 보다 accuracy vs sparsity tradeoffs에서는 좋은 성능을 보였지만, **SGD** 에 비해서 정확도에서는 좋은 성능을 보여주지는 못했다.

이렇게 **OGD** 의 accuracy에서의 성능과 **RDA** 의 sparsity를 제공하는 부분을 모두 얻기 위하여 논문에서는 **FTRL-proximal** 이라는 방법을 제안한다.

(사실, 다른 [논문](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/37013.pdf)에서 **FOBOS** 와 **RDA** 도 FTRL이라고 한다.)

**FTRL-proximal** 에서는 다음과 같은 식을 이용하여 update를 진행한다.

$$
\mathbf{w_{t+1} } = argmin_{\mathbf{w} }(\mathbf{g_{1:t} }\bullet \mathbf{w}+\frac{1}{2}\sum^{t}_{s=1}{\sigma}_s { {||\mathbf{w}-\mathbf{w_s}||}_2}^2+{\lambda}_1 {||\mathbf{w}||}_1)

{\sigma}_{1:t} = \frac{1}{ {\eta}_t}
$$

(어떤 식에서는 $\lambda$앞에 $t$를 붙이기도 한다. 붙이는 것이 맞는 것 같지만 이 논문에 써있는 것을 기준으로 하겠다. 왜 그러는지 알게 되면 수정을 해야겠다.)

이 경우 ${\lambda}_1$가 0이면 위의 **OGD** 에서의 update식과 아래의 **FTRL-proximal** 의 식은 같아진다.

<!--$\hat{g_t}$를 $g_t+{\phi}_t$, $${\phi}_t \in subgradient( {\lambda}_1 {||\mathbf{w_t}||}_1)$$이라고 정의하면, OGD의 update식은 다음과 같이 쓸 수 있다.-->
L1 regularization을 사용하는 CO-OGD를 다르게 쓰면 다음과 같다.

$$
\mathbf{w_{t+1} } = argmin_{\mathbf{w} }(\mathbf{g_{1:t} }\bullet \mathbf{w}+{\phi}_{1:t-1}\mathbf{w}+{\lambda}_1 {||\mathbf{w}||}_1 +\frac{1}{2{\eta}_t}{ {||\mathbf{w}-\mathbf{w_t}||}_2}^2)

\hat{g_t} = g_t+{\phi}_t

{\phi}_t \in subgradient[{\lambda}_1{||\mathbf{w_t}||}_1]
$$

(첫 항은 loss의 approximation, 두 번째+세 번째 항은 L1의 approximation, 마지막 항은 convexity를 제공하는 term이다)

두 식의 차이는 $L1$ penalty를 다루는 방법의 차이이다. ${\phi}_{1:t-1}\mathbf{w}+{\lambda}_1 {||\mathbf{w}||}_1$가 FTRL의 $t{\lambda}_1 {||\mathbf{w}||}_1$를 approximate한 것이다.

이 차이로 인해 FTRL의 더 sparse model을 만든다(FTRL은 approximation를 진행하지 않기 때문이다.)

**FTRL-proximal** 의 식을 update하기 위해 $\mathbf{g_{t}}$와 $\mathbf{w_s}$들을 저장에 필요한 다량의 메모리가 필요해 보이지만, 다음과 같이 식을 바꾸면 그렇지 않게 된다.

$$
(\mathbf{g_{1:t} }- \sum^{t}_{s=1}{\sigma}_s \mathbf{w_s})\bullet \mathbf{w} + \frac{1}{ 2{\eta}_t}{ {||\mathbf{w}||}_2}^2+{\lambda}_1{||\mathbf{w}||}_1 + constant \:\: \cdot \cdot \cdot \:\: (1)
$$

(논문에서는
$$\frac{1}{ {\eta}_t}{ {||\mathbf{w}||}_2}^2
$$
라고 나와있지만, 계산을 해보면 위의 식이 맞는 것 같다.) <!--확실하지는 않다.)-->

그리고
$$
\mathbf{z_{t-1} } = \mathbf{g_{1:t-1} }-\sum^{t-1}_{s=1}{\sigma}_s \mathbf{w_s}
$$
라고 정의하고,
$$\mathbf{z_t} = \mathbf{z_{t-1} } + \mathbf{g_t} + (\frac{1}{ {\eta}_t} - \frac{1}{ {\eta}_{t-1} })\mathbf{w_t}
$$
를 통해서 update하면 $\mathbf{g_t}$와 $\mathbf{w_s}$들을 전부 저장할 필요 없이 $\mathbf{z_t}$만 저장하면 된다.

이를 *closed form* 으로 구하면 다음과 같다.

$$
\mathbf{w_{t+1,i} } = \begin{cases}
0 & \text{ if } |z_{t,i}|\leq {\lambda}_1

-{\eta}_t(z_{t,i}-sgn(z_{t,i}){\lambda}_1) & \text{ otherwise}  
\end{cases}
$$

#### Closed form 계산

$(1)$식을 $\mathbf{w_{t+1,i} }$에 대하여 편미분하면 다음과 같다.

$$
\mathbf{z_t,i}+\frac{1}{ {\eta}_t}w_{t+1,i}+{\lambda}_1 sgn(w_{t+1,i})
$$

**FTRL** 에서 $argmin F(\mathbf{x})+\phi \bullet \mathbf{x}$는 $\bigtriangledown F(\mathbf{x})+\phi = 0$과 같다. ([논문](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/37013.pdf)의 3rd page 참고)
그러므로 위의 식을 0으로 하고 정리하면 *closed form* 과 비슷한 식이 나온다.

$$
w_{t+1,i} = -{\eta}_t(z_{t,i}+sgn(w_{t+1,i}){\lambda}_1)
$$

$$
|z_{t,i}| > {\lambda}_1
$$
에서
$$
sgn(w_{t+1,i})* sgn(z_{t,i}) <0
$$이다. \\
또한,
$$
|z_{t,i}| \leq {\lambda}_1
$$
의 경우에는 좌변이 0 이상이면 우변은 0 이하이고, 좌변이 0 이하이면 우변은 0 이상이기 때문에 모든 경우에 대하여 0이 해당된다.

결국, 위의 이유에 따라 *closed form* 이 계산된 것이다.

Per-Coordinate Learning Rates
------------------------------

**SGD** 에서는 leanring rate를 decay를 하던지 말던지 global한 leanring rate를 사용한다. 여기서 global이라는 것은 모든 feature에 대하여 learning rate를 같게 만드는 것이다.(물론 [Adagrad](https://en.wikipedia.org/wiki/Stochastic_gradient_descent#AdaGrad)등의 경우에는 paramter의 많이 변하지 않은 부분의 learning rate를 크게 한다.)

이 논문에 나온 예를 살펴보자.

10개의 coin을 던지고, logisitic regression을 이용하여 10개의 coin들에 대하여 $P(\text{heads}|{\text{coin}}_i)$를 estimate한다고 해보자. 각 round마다 한 개의 coin이 던져지고 확인하는 형식을 할 것이다. 이 경우에 round마다 coin이 한 개만 던져지므로 10개의 독립적인 regression을 행해야 하지만, 한 개의 model을 사용하여 이를 해결하려고 하는 것이 문제이다.

만약에 각 feature(이 경우에는 각 coin들)을 학습할 때 같은 learning rate를 학습한다면, 많이 변한 feature들(많이 던져진 coin들)때문에 적게 변한 feature들(적게 던져진 coin들)은 학습이 제대로 이루어지지 않을 것이다.

그러므로 이 논문에서는 다음과 같은 learning rate를 정의한다.(각 feature들 마다 각각의 learning rate들을 정의하였다.)

$$
{\eta}_{t,i} = \frac{\alpha}{\beta + \sqrt{\sum^{t}_{s=1}{ {g_{s,i} }^2} } }
$$

$g_{s,i}$는 $l_s(\mathbf{w_s})$의 $i$번째 coordinate이므로, 위의 learning rate는 각 coordinate(component, feature와 혼용 중이다)의 변한만큼에 따라 달라지게 된다. $\alpha$와 $\beta$는 [progressive validation](http://hunch.net/~jl/projects/prediction_bounds/thesis/mathml/thesisse44.xml)을 통하여 구한다.

아래의 algorithm은 이제까지 설명한 내용을 종합한 것이다.

![algorithm]({{site.baseurl}}/post_img/{{page.name}}/algorithm.png)

위의 알고리즘을 보면 $L2$ regularization을 추가한 것이 보인다.

$(1)$의 식에 $L2$ regularization term ($\frac{ {\lambda}_2}{2}{ {||\mathbf{w}||}_2}^2$)을 추가하여 똑같은 방식으로 gradient을 구해주면 된다.
(${\lambda}_1$과 ${\lambda}_2$를 0으로 하고, learning rate를 고정시키면 우리가 익히 알고 있는 **SGD**  알고리즘이 완성된다.)

실험 결과
=======

![result]({{site.baseurl}}/post_img/{{page.name}}/result.png)

위의 표는 **FTRL-Proximal** 대비 non-zero parameter의 수(sparsity가 얼마나 잘되어 있는지를 판단하는 기준)과 **AucLoss** (1-[AUC](https://en.wikipedia.org/wiki/Receiver_operating_characteristic#Area_under_the_curve))를 계산한 결과를 보여준다.

- - -

Regret과 per-coordinate learning rate
=====================================

별도로 [논문](https://arxiv.org/pdf/1002.4862.pdf)의 앞부분을(뒷부분은 더 자세한 내용이긴하다. 어떻게 하면 **per-coordinate** 를 더 좋게 만들 수 있는 지 등이 나온다)간략하게(자세한 증명없이) 살펴보고, **Regret** 의 관점에서 **per-coordinate** 를 사용하는 이유에 대해 알아 볼 것이다.

Data set의 크기가 과거에 비해 비대해지면서, 사용자들은 한 번에 모든 data들을 학습하는 **batch** 방식보다는 **OGD** 같이 한 번의 update에 하나의 training data만 사용하거나, **mini-batch** 처럼 한 번에 다룰 수 있는 양의 data들만 이용하는 방식을 선호하게 되었다.

하지만,OGD에서의 학습속도는 batch 방식에서의 학습 속도보다 느리다. 그 이유로는 batch 방식에서는 loss를 적절히 조절하여 모든 coordinate 방향으로의 이동 단위가 비슷하게 preprocessing 할 수 있지만, 언제 무슨 데이터가 어떤 형태를 가지고 들어올지 알지 못하는 OGD의 경우에는 이 방식을 쓰기 쉽지 않다. 그렇기 때문에 후에 서술하는 regret을 사용하지 않는 OGD의 경우에는 특정 feature(coordinate)로 overfitting하고 나머지에 대해서는 underfitting하는 경향이 있다.

저자는 이 문제를 해결하기 위하여 각각의 coordinate마다 다르게 변하는 learning rate를 이용할 것이다. 이 방식을 사용하면 regret bound(regret이 값을 가질 수 있는 범위)가 전보다 훨씬 작아진다고 한다.

Regret
------

convex optimization에서는 각 round $t$마다 $x_t$(여기서는 weight또는 parameter를 뜻한다)를 뽑고, loss $f_t(x_t)$를 계산한다.

이 때 regret은 아래의 식과 같이 time $T$까지의 loss들의 합에서 $T$까지 얻을 수 있는 loss들의 합의 최소값(을 만드는 최적의 $x$를 골랐을 때)을 뺀 것이다.

$$
\text{Regret} : \sum^{T}_{t=1}f_t(x_t) - {\text{min}}_x [\sum^{T}_{t=1}f_t(x)]
$$

즉, OGD에서 $f_t(x)$는 한 sample의 prediction과 실제 target에서 나오는 loss이고, $\text{Regret}$은 이제까지 loss합과 가능한 loss들의 합의 최소값의 차이다.

적절한 learning rate를 선택할 시 regret은 $O(GD\sqrt{T})$이다. ($D$는 weight이 가질 수 있는 범위의 크기, $G$는 gradient의 norm의 최대값이다.)

One-Dimension Example
---------------------

Weight의 차원이 하나인 train을 생각해 보자. learning rate가 너무 크면 제대로 학습하지 못하고 진동(osciliate)하거나 발산할 것이다. 반대로 너무 작은 경우에는 학습이 너무 더디게 진행될 수도 있다. 두 경우를 고려하면 모든 learning rate에 대하여 regret은 다음과 같은 범위를 갖는다.(계산은 논문에 잘 나와있다.)

$$
max[\frac{D^2}{4\eta},G^2\eta \frac{T}{2}] \leq \text{Regret} \leq \frac{D^2}{2\eta}+G^2\eta \frac{T}{2}
$$

이 경우에 learning rate $\eta$가 $\frac{D}{G}$의 비례하는 값을 사용하는 것이 가장 좋다는 것을 알 수 있다.

각 coordinate마다 $D$와 $G$가 다를 수 있기 때문에 learning rate를 각 coordinate마다 다르게 고르는 것의 중요함을 보여주고 있다. 또한, 특히 $G$의 크기는 처음에는 알 수 없기 때문에 시간에 맞게 고쳐주는 것도 좋은 방법이 될 것이다.

Bad example for global learning
-------------------------------

논문의 Theorem 1에서는 다음과 같이 말한다.

*There exists a family of online convex optimization problems, parameterized by their lengths(number of rounds T),where gradient descent with a non-increasing global learning rate incurs regret at least $\Omega(T^{\frac{2}{3}})$, whereas gradient descent with an appropriate per-coordinate learning rate has regret $O(\sqrt{T})$*

즉, 어떤 문제들에 대해서는 global한 learning rate(decay를 하는 경우와 constant인 경우 모두)를 사용하는 경우에의 regret의 최소값이 per-coordinate learning rate를 사용하는 경우의 regret의 최대값보다 커진다는 얘기다. (줄여 얘기하면 per-coordinate learning rate 쓰자.)


참고 자료
========
- [H.Brendan McMahan. Follow-the-Regularized-Leader and Mirror Descent: Equivalence Theorems and L1 Regularization ](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/37013.pdf)
- [H.Brenden McMahan, et al. Ad Click Prediction: a View from the Trenches](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/41159.pdf)
- [Matthew Streeter, H. Brenden McMahan. Less Regret via Online Conditioning](https://arxiv.org/pdf/1002.4862.pdf)
- <https://courses.cs.washington.edu/courses/cse599s/12sp/scribes/Lecture8.pdf>
