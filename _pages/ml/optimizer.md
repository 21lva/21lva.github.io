---
layout: post
title:  "여러가지 optimizer"
date:   2018-12-26 01:02:59
author: Inhyuk
categories: Machine_Learning
tags:	ml
cover:  "/assets/instacode.png"
name: optimizer.md
---

**Neural network** 을 학습할 때 많이 쓰이는 **optimizer** 는 그 종류가 여러가지이다.

대부분의 **optimizer** 가 gradient descent의 변형이지만,

데이터의 양이 많아지고 있기 때문에 **on-line learning** 에 사용되는 gradient descent방법들이 더욱 중요해지고 그 종류도 많아졌다.

아래에서 살펴볼 것들은 전부 loss function의 first order만 고려한다.

second-order와 **Hessian Matrix** 를 사용하는 방식은 계산이 많이 필요하기 때문에 잘 사용하지 않고, 사용을 하더라도 Hessian Matrix를 근사하는 방식으로 진행한다.(e.g. BFGS)

여기서는 그 것들을 알아볼 것이다.

SGD
=======

**Stochastic gradient descent(SGD)** 는 하나의 데이터에 한번의 연산을 처리하는 방식이다.

정의는 그러하지만 대부분이 mini-batch방식을 사용한다.

반대로 모든 데이터를 한번에 학습하는 방식을 batch gradient descent라고 한다.

loss를 정의하고 그 loss를 가장 작게하는 방향(gradient)로 weight를 update하는 방식이다.

$$
w\leftarrow w-\alpha  {\bigtriangledown}_{w}L
$$

위의 식에서 $w$는 update하고자하는 weight, $\alpha$는 learning rate이다.

learning rate가 매우 크면 update가 제대로 되지 않아서 loss 값이 더욱 커지게 되고, 반대로 너무 작으면 최적의 값을 찾는 데에 너무 많은 시간을 필요로 하게 된다.

${\bigtriangledown}_{w}L$는 L의 w에 대한 기울기를 구하는 식이다. gradient라고 한다.

gradient는 weight의 차원과 같은 vector인데, 그 vector의 방향이 loss가 가장 커지는 방향, 반대는 loss가 가장 작아지는 방향이다.

위의 식은 loss가 가장 작아지는 방향으로 weight를 learning rate만큼 변화시키겠다는 의미이다.

SGD가 모든 것을 해결해주면 얼마나 좋을까.

그렇지 못하기 때문에 다양한 사람들이 많은 방법들을 찾아낸 것이다.

Adagrad
=========

**Adaptive gradient(adagrad)** 는 weight의 update를 모두 같은 비율로 하는 것이 아니라, 현재까지 변화를 많이한 변수들은 적게 변화시키고 적게 변화한 변수들은 많이 변화시키는 방식이다. 자주 변화한 weight들은 적게 변화한 weight들보다 최적의 값에 가까이 있을 것이라는 가정을 한 것이다.

수식을 보도록 하자.

$$
G_{t}=G_{t-1}+({\bigtriangledown}_{\theta }J({\theta}_{t}))^2

{\theta}_{t+1}={\theta}_t-\frac{\eta}{\sqrt{G_t+\epsilon }}{\bigtriangledown}_{\theta}J({\theta}_{t})
$$

$G_t$는 time step t까지 변화한 양을 변화한 방향을 고려하지 않고 저장한다. 이를 이용하여 weight의 update는 $\frac{1}{\sqrt{G_t+\epsilon}}$만큼 변화한 gradient만큼 된다.

**SGD** 에서 learning rate를 고정시키기도 하지만 learning rate를 시간에 따라 점점 줄어들게 하기도 한다. 이는 최적의 값에 가까워지면 더 작은 값으로 미세하게 조정하기 위함이다. 그러나 **Adagrad** 에서는 그럴 필요 없다. 0.01정도의 learning rate를 초기화 한후에 진행을 하면 학습이 진행됨에 따라 learning rate가 weight 각각에 맞게 점점 줄어들게 된다. 하지만 이 방식의 문제는 원하는 만큼 학습이 되지 않았는 데 learning rate가 너무 작아지는 문제가 발생할 수 있다.

RMSProp
=======

**Adagrad** 의 문제점으로 충분히 학습이 되지 않은 상태에서 learning rate가 너무 줄어드는 현상이 있다고 하였다. **RMSPro** 은 이런 문제점을 해결하기 위하여 제시되었다.

weight을 update하는 부분은 **Adagrad** 와 같지만, $G_t$를 update하는 부분이 살짝 다르다. 수식은 다음과 같다.

$$
G_{t}=\gamma G_{t-1}+(1-\gamma)({\bigtriangledown}_{\theta }J({\theta}_{t}))^2 

{\theta}_{t+1}={\theta}_t-\frac{\eta}{\sqrt{G_t+\epsilon }}{\bigtriangledown}_{\theta}J({\theta}_{t})
$$

이렇게 $G$를 update하면 $G$가 무한하게 커지지는 않기 때문에 **Adagrad** 에서 서술한 문제는 피할수 있다.

AdaDelta
========

**RMSProp** 처럼 **Adagrad** 방식의 문제점을 해결하기 위하여 제시되었다.

$$
G_{t}=\gamma G_{t-1}+(1-\gamma)({\bigtriangledown}_{\theta }J({\theta}_{t}))^2 

s=\gamma s+(1-\gamma)(\frac{\sqrt{s+\epsilon}}{\sqrt{G+\epsilon}}{\bigtriangledown}_{\theta}J({\theta}_t))^2

{\theta}_{t+1}={\theta}_t-\frac{\sqrt{s+\epsilon}}{\sqrt{G+\epsilon}}{\bigtriangledown}_{\theta}J({\theta}_t)
$$

이러한 생각이 나온 이유는 [논문](https://arxiv.org/pdf/1212.5701.pdf "논문")을 읽으면 자세한 설명이 나와았다. 간단하게 설명하면, second-order 방식인 **Newton method** 를 이용한 것이다.

Momentum
========

만약에 loss function이 convex일 경우에는 SGD로도 충분하다.

적절한 learning rate와 시간만 있으면 최적의 닶에 근사하도록 weight를 만들 수 있다는 말이다.

하지만 loss function이 항상 convex인 것은 아니다.

대표적인 non linear한 model인 **neaurl network** 도 convex가 아닐 수 있다.

convex가 아닌 loss function에 단순히 **SGD** 를 사용하면 global minima(loss function이 가장 작아지는 곳들)이 아닌 local minima에 빠질 수가 있다.

이는 지양해야 하는 현상이므로, 사람들은 **momentum** 이라는 것을 생각해 냈다.

momentum은 물리학을 배운 사람이라면 친숙한 용어이다(관성 모멘트 등등). 물체가 높은 곳에서 떨어질 때, 어떤 웅덩이(local minimum)에 빠지더라고 관성에 의해 그 웅덩이에서 튕겨나와서 더 아래로 내려갈 수도 있다. gradient descent에서도 momentum을 고려하여, 현재 이동하려는 방향과 상관없이 이전에 이동했던 방향으로 조금 더 움직이려는 효과를 집어 넣은 것이다.

$$
{v}_{t}\leftarrow \gamma {v}_{t-1}+\alpha \bigtriangledown

w\leftarrow w-{v}_{t}
$$

${v}_{t}$는 time step t에서의 이동 vector이고, $\gamma$는 momentum 계수로 moementum을 얼마나 고려할 것인지에 대한 값이다.

![sigmoid2]({{site.baseurl}}/post_img/{{page.name}}/avoiding_minimum.png)

식과 그림을 함꼐 보자. 그림의 가장 오른쪽은 local minimum에 빠졌다. 만약에 이 공이 오른쪽에서 온것이라면, ${v}_{t-1}$은 왼쪽으로 이동하도록 값을 부여할 것이다. 이 값에 의하여 weight의 update가 비록 그 당시에는 loss를 크게하더라고 momentum에 의하여 global minimum를 찾아갈 수 있게 해줄 수도 있다.

하지만 이 방식은 weight뿐만 아니라 기존의 $v$를 저장하는 변수를 따로 지정을 해놔야 하기 때문에 메모리가 더 필요하게 된다.

NAG
===

**Nesterov momentum gradient** 의 약자이다.

**Momentum** 의 방식을 이용하지만 gradient descent와는 살짝 다르다.

**NAG** 는 이동 벡터를 계산할때 현재 위치의 gradient와  momentum을 따로 계산하는 **Momentum** 방식과는 달리 NAG는 momentum을 고려하여 이동하였다고 가정을 한 후에 그 자리에서 gradient를 구하는 방식을 사용한다.

$$
v_t=\gamma v_{t-1}+\eta {\bigtriangledown}_{\theta}J(\theta-\gamma v_{t-1})

\theta = \theta - v_t
$$

**NAG** 는 **Momentum** 에 비해 더 효과적이다. 그 이유는 **Momentum** 과 달리 먼저 움직여보고 이동을 어떤 식으로 해야하나를 결정하기 때문이다.

convex한 경우에 수렴이 더 잘된다(이러면 무슨 의미가...)


Adam
====

**tensorflow** 나 **pytorch** 같은 딥러닝 라이브러리로 구현한 것들의 대부분은 이 **Adam** 을 쓰고 있다.

이 방식은 **RMSProp** + **Momentum** 이다.

$$
m_t = {\beta}_1 m_{t-1}+(1-{\beta}_1){\bigtriangledown}_{\theta}J(\theta) 

v_t = {\beta}_2 v_{t-1}+(1-{\beta}_2)({\bigtriangledown}_{\theta}J(\theta))^2 

\hat{m_t} = \frac{m_t}{1-{({\beta}_1)^t}} 

\hat{v_t} = \frac{v_t}{1-{({\beta}_2)^t}} 

\theta = \theta - \frac{\eta}{\sqrt{\hat{v_t}+\epsilon}}\hat{m_t}
$$

${\beta}_1=0.9$, ${\beta}_2=0.999$, $\epsilon=10^{-8}$를 주로 사용한다.

$m$과 $v$가 처음에 0으로 만들고 첫 두 줄에서 biased로 update하기 때문에 그 다음 두 줄에서 unbiased하게 변화시킨다.

gradient가 커지더라도 step size(weight update 시키는 정도)가 bound되어 있기 때문에 다양한 objective function을 사용하더라도 안정적으로 update할 수 있는 장점이 있다.

- - -

위의 다양한 방식들 중 **Adam** 을 주로 사용하긴 하지만 무엇이 좋다고 하기는 힘들다.

말은 이렇게 장황하고 복잡하게 설명해놓았지만 실제로 사용할 때는 코드 한 두 줄이면 사용할 수 있다.

**tensorflow** 를 예로 들자면,

```py
train = tf.train.AdamOptimizer(learning_rate=0.01)
sess.run(train,feed_dict={X:Input})
```

second order optimization은 hyper parameter($\epsilon,\beta$ 같은 것들)을 고려하지 않아도 되는 장점이 있지만, first order optimization보다 계산량이 많기 때문에 위의 optimizer들에 validation을 수행하여 hyper parameter를 적절히 채택하는 방식을 주로 쓰는 것 같다.

참고자로
=======

- <https://www.researchgate.net/figure/Minimizing-a-non-convex-objective-function-Gradient-descent-techniques-may-become_fig20_2459030>//
- <http://dalpo0814.tistory.com/29>
