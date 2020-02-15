---
layout: post
title:  "Activation function(활성함수)"
date:   2018-12-26 01:02:59
author: Inhyuk
categories: Machine_Learning
tags:	ml
cover:  "/assets/instacode.png"
name: 2018-12-27-activation.md
---

Activation function(활성 함수)의 종류와 그 쓰임에 대해 알아보자.

Sigmoid
=======

![sigmoid2]({{site.baseurl}}/post_img/{{page.name}}/sigmoid.png)

sigmoid function은 다음과 같은 식을 의미한다.

$$
\sigma \left(x \right) = \frac{1}{1+{e}^{-x}}
$$

Non-linear function으로 함수 값은 0과 1사이로 제한 시킨다.
요새에는 Neural network의 hidden layer에서 activation function으로는 잘 쓰이지 않고, binary classification의 분류에서 output layer에서만 주로 쓰인다.

Activation function으로 쓰이지 않는 것은 두 가지 이유에서다.
1. Gradient Vanishing
sigmoid을 잘보면 x에서의 gradient(접선의 기울기 혹은 미분값)이 x=0인 부분에서 가장 크다. 즉, x가 0에서 멀어지면 멀어질수록 gradient의 크기가 0으로 수렴하기 때문에 학습의 역전파(back propagation)시 gradient가 소멸하는 경우가 발생할 수 있다.

2. Not zero-centered
sigmoid를 입힌 함수를 생각해보자.
$$output = f\left({w}^{t}x+b \right)$$
($f$는 activation function, $x$는 input, $w$, $b$는 각각 weight와 bias이다.)
activation function으로 sigmoid를 쓰는 경우에 위의 $output$은 항상 양수이다.
이 때, 이 $output$이 다음 layer의 input으로 들어간다고 하자.
그러면, 입력값이 모두 양수이므로 loss function의 weight에 대한 미분값의 부호는 loss function의 activation function에 대한 미분값의 부호와 같게 되어서 지그재그로 움직이게 될 수 있다.


Rectified linear unit(ReLU)
===========================
가장 많이 쓰이는 activation function이다.
함수의 정의는 다음과 같다.

![relu]({{site.baseurl}}/post_img/{{page.name}}/relu.png)

$$f=max(0,x)$$

linear하기 때문에 stochastic gradient시에 수렴 속도가 빠르며 미분시에 따로 비용이 들지 않는다.
하지만, zero-centered가 아니라는 점과 x가 음수일 경우에 함수값이 항상 0이기 때문에 neuron들이 죽는 현상이 발생한다.

Leaky ReLU & PReLU & ELU
=========================
3가지 모두 ReLU에서 neuron들이 죽는 현상을 해결하기 위해 만들어진 함수이다.
Leaky ReLU는 다음과 같이 정의 된다.

$$
f=max(0.01x,x)
$$

PReLU는 다음과 같다.

$$
f=max(\alpha x,x)
$$

이 때 $\alpha$는 매우 작은 값이다. Leaky ReLU가 일반회된 것이라고 볼 수 있다.

ELU(exponential linear units)

$$
f=\begin{cases}
\alpha({e}^{x}-1) & \text{ if } x\leq 0  \\
x & \text{ otherwise }
\end{cases}
$$

![lrelu]({{site.baseurl}}/post_img/{{page.name}}/lrelu.png)
![elu]({{site.baseurl}}/post_img/{{page.name}}/elu.png)

이 3가지 방법은 기존의 dying ReLU, not zero-centered 문제를 해결하였다.
하지만, ELU의 경우에는 미분시 exp를 추가로 계산해줘야 한다.

TanH
=====

쌍곡함수의 일종인 hyperbolic tangent를 이용하는 방법이다.
수식은 다음과 같이 정의된다.

$$
\frac{e^{x}-e^{-x}}{e^x+e^{-x}}
$$

형태는 sigmoid와 비슷하지만 sigmoid와 달리 zero-centered가 아니라서 많이 쓰였다.(요새는 ReLU가 대세인것 같다.)
사실 수식을 보면 tanh는 sigmoid를 x축으로는 축소하고 y축으로 늘린 다음에 아래로 살짝 내린 형태이다.

$$
tanh = 2sigmoid(2x)-1
$$

![tanh]({{site.baseurl}}/post_img/{{page.name}}/tanh.png)

어떻게 쓸까?
===========
* sigmoid는 binary classification의 ouput layer에 쓰는 것이 아니라면 쓰지 않는다.
* ReLU가 가장 많이 쓰이므로, ReLU를 사용하거나 그 변형인 ELU, leaky ReLU, PReLU등을 사용한다.
* 별로 좋지는 못하지만 tanh도 사용해보자.
