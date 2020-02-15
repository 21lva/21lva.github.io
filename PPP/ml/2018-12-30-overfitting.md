---
layout: post
title:  "Overfitting 피하기 & Batch Normalization"
date:   2018-12-29 02:00:00
author: Inhyuk
categories: Machine_Learning
tags:	ml
cover:  "/assets/instacode.png"
name: 2018-12-30-overfitting.md
---


이번 포스트에서는 overfitting을 피하기 위한 Regularization들과 dropout을 다루고,
마지막으로 Batch normalization을 다룬다.

- - -

**Maximum likelihood estimation(MLE)** 의 특성상 가용 데이터의 수가 적고 parameter의 수가 상대적으로 많으면 **overfitting** 이 일어날 수 있다.
[overfitting](https://en.wikipedia.org/wiki/Overfitting "overfitting") 은 모델의 미래에 대한 예측 능력을 크게 저하시키므로 지양해야할 수단이다.

Bishop의 pattern recognition and machine learning이라는 책에서는 MLE의 이러한 문제점을 해결하기 위하여 Baye
sian 학습을 제시하지만, 현실적으로 잘 쓰이지는 않는 것 같다.(parameter의 prior distribution을 구하는 것은 꽤 어려워 보인다.)
결국, MLE를 할 수 밖에 없으므로 다른 방법들로 이런 overfitting문제를 해결하고자 노력한다.

- - -

Regularization
==============

Regularization은 많이 사용되고 선호되는 방식이다.
기존의 norm(1차든 2차든)을 사용하는 간단한 예측 모델에서 loss function은 다음과 같았다.

$$
L(x)=\sum_{k=1}^{N}Error(Target,Prediction)
$$

Regularization은 loss function에 regularization term이라는 항을 추가한다.

$$
L(x)=\sum_{k=1}^{N}Error(Target,Prediction) + \gamma L_p \\
L_p = (\sum_{j=1}{M}|w_j|^p)\\
 w = (w_1,w_2,...,w_M)
$$

$\gamma$는 regularization coefficient, $L_p$는 regularizer라고 부른다.

주로 사용하는 L2 regularization은 $p$=2인 상황으로 수식을 다시 적으면 다음과 같다.

$$
L(x)=\sum_{k=1}^{N}Error(Target,Prediction) + \gamma ||w||^2
$$

이 L2 regularization을 Ridge regulariztion이라고 부르기도 한다.

물론 Lasso라고 불리우는 L1 regularization도 존재한다.

이제 이 regularization의 기능에 대해 알아보자.
상술했다시피 위의 방법을 사용하는 이유는 overfitting의 방지에 있다.
L2를 중점적으로 설명하면, $w_j$가 커지면 모델이 복잡해지지만 overfitting의 위험도 커진다. regularization term은 이렇게 $w_j$가 커질 때, loss function에 penalty를 부과하여서 $w_j$가 커지는 것을 막고 overfitting도 막는다.
매우 간단한 것이다.

물론, $\gamma$는 hyper parameter이므로 대충 때려 맞추던지 validation을 이용하여 적절히 고르던지 해야한다.
$\gamma$가 너무 크면 regularization term의 영향력이 과해져서 모델이 적절한 복잡성을 가지지 못하고, 너무 작으면 overfitting을 방지하는 역할을 수행하지 못하기 때문에 적절한 값을 고르는 것도 중요하다.

- - -

Dropout
=======

[Bagging(bootstrap aggregating)](https://en.wikipedia.org/wiki/Bootstrap_aggregating)은 다양한 모델을 만들어 놓고 학습을 하여 투표 혹은 평균값을 취하는 방식으로 모델에 있을 수 있는 error을 줄이는 **ensemble** 방식이다.

이렇게 평균을 내거나 투표를 통하는 방법은 효과적이긴 하나, 모델을 여러 가지를 준비해야하고 학습을 진행해야하는 만큼 시간을 소모하게 되어서 특히 모델이 크고 학습시간이 오래 걸리는 경우에는 잘 사용하지는 않는다.

이 문제를 해결하면서 **Bagging** 의 장점을 사용하고자, [Dropout](http://www.cs.toronto.edu/~rsalakhu/papers/srivastava14a.pdf)이라는 기법이 제시되었다.

L1,L2와 같은 **regularization** 의 효과를 얻을 수 있으며, **Bagging** 에 비해 시간도 오래 걸리지 않는다.
![dropoutfig]({{site.baseurl}}/post_img/{{page.name}}/dropoutfig.png)


**Dropout** 은 간단히 설명하면 위의 그림처럼 모든 뉴런을 사용하는 것이 아니라 일정 확률로 몇 뉴런들을 비활성화 시키는 방식이다.
이런 방식으로 하면 가끔 input layer와 output layer가 연결이 되어있지 않거나 뉴련이 없거나 하는 경우도 존재하지만, 신경망의 크기가 크면 이런 확률은 극히 적게 되어 딱히 고려하지 않아도 된다.

**Dropout** 은 학습과 추론이 살짝 다르다.
학습시에는 뉴런이 $1-p$의 확률로 사라지지만, 추론시에 이 방법을 사용하는 것이 아니다.
추론을 할 때 뉴런은 이전 layer의 모든 input을 받게된다.(**ensemble** 을 하기 위하여 전부 받는다.) 그 상태로 출력을 하게 되면 평균을 내는 효과가 없으므로 평균 값인 $px+(1-p)0 = px$로 수정한 값을 출력한다.

![dropouttrain]({{site.baseurl}}/post_img/{{page.name}}/dropouttrain.png)

물론 p를 곱해주는 것이 시간이 오래 걸릴 때가 있다. 이런 경우를 위하여 **inverted dropout** 이라고 불리우는 방법을 사용한다.
이 방법은 설명하지 않겠다.

**Dropout** 은 계산이 많지 않고 모델에 관계없이 사용할 수 있다는 장점이 존재한다. (물론 항상 성능을 향상 시키는 것은 아니다. 만고불변의 ML에서의 진리)                                     

- - -

Batch Normalization
====================

이 장은 <http://sanghyukchun.github.io/88/>을 참고하였다.

근래에 대부분의 신경망에서는 [Batch Noramlization]("https://arxiv.org/pdf/1502.03167.pdf")을 사용하고 있다.

**Batch Noarmalization** 은 [vanishing/exploding gradient problem](http://www.cs.toronto.edu/~rgrosse/courses/csc321_2017/readings/L15%20Exploding%20and%20Vanishing%20Gradients.pdf)을 방지하고 학습속도를 가속화 시키기 위한 방법으로 제시되었다.

Gradient vanishing and Gradient exploding
------------------------------------------

*Gradient vanishing* 은 작은 미분값들이 지속적으로 곱해지면서 그 값이 컴퓨터가 저장할 수 없을 만큼 작아지는 현상을 일컬으며, 반대로 gradient가 곱의 연산에 따라 매우 커지게 되는 것을 *Gradient exploding* 이라고 한다.

이 문제를 해결하는 방법은 여러가지가 있다. ReLU를 쓴다던지 하는 방법도 있다.
이전 포스트에서 ReLU를 쓰면 gradient vanishing이 발생할 수 있다고 하였는 데, 그것은 이 것과는 살짝 다른 얘기이다.
여기서 설명하는 gradient vanishing은 back propagation에서 곱해지는 값들이 -1과 1사이의 값들이므로 그 값이 아래로 내려오면 올수록 작아진다는 얘기이고, ReLU가 겪는 gradient vanishing은 ReLU가 입력이 음수이면 출력이 0인 함수이기 때문에 backpropagation에서 gradient가 0이 되버리는 현상이다.



Internal Covariance Shift
--------------------------

각각의 layer들의 input distribution이 재각각으로 되는 현상을 일컫는다. 이 논문에서는 각 layer의 parameter들이 다른 layer의 parameter에 영향을 받기 때문에 아래로 내려갈수록 gradient가 사라진다고 하였다.

Whitening
----------

**Internal covariance shift** 문제를 해결하기 위하여, input의 distribution을 평균이 0이고 분산이 1인 Gaussian distribution으로 Noramlization을 하는 방법.
이 방법은 multivariate의 경우에는 계산량이 크고, mean과 variance를 train마다 계산하기에는 시간이 걸리기 때문에 문제가 있다.

Batch normalization
--------------------

그러므로 이 논문에서는 다음과 같은 방법을 사용한다.

* Input의 각 feature들은 correlated가 아니다. -> 각각에 대해 각각 mean과 variance를 구한다.
* mean을 0으로 과 variance를 1로 하는 방법은 non-linearlity를 없애는 경우가 생길 수 있으므로, normalization시에 약간의 상수를 더해준다.
* train단위로 하는 계산은 그 양이 막대하므로 mini-batch단위로 mean과 variance를 계산한다.

![batch_normalization_algorithm]({{site.baseurl}}/post_img/{{page.name}}/batch_normalization_algorithm.png)

학습할 때와는 다르게 추론시에는 mini-batch가 아니라 이제까지 얻어온 mean과 variance를 이용하여 unbiased mean과 variance를 계산하고 이를 이용하여 전체 data를 transform한다.

이 방법의 장점은
1. learning rate를 너무 높게 잡으면 gradient vanishing 혹은 exploding이 존재했던 기존의 방법과는 다르게 BN을 사용하면 learning rate를 높게 잡을 수 있으므로 학습의 속도를 증진시키녀, parameter들의 scale를 같게 만들어 주기 때문에 sacle에 의해 생기는 문제도 해결할 수 있다.
2. 자체적인 regularization효과가 존재하므로 기존의 regularization을 사용하지 않아도 된다.
3. Dropout과 비슷한 효과를 내기 때문에 학습 속도가 느린 Dropout를 사용하지 않으므로 속도적인 측면에서 효과를 얻을 수 있다.


효과
----

BN은 많은 딥러닝 라이브러리에 구현이 되어 있다.
실제 tensorflow에서 제공하는 BN을 이용해 보자.
![Tensorflow]({{site.baseurl}}/post_img/{{page.name}}/Tensorflow.png)

https://www.tensorflow.org/tutorials/keras/basic_text_classification 의 코드를 **Batch Normalization** 을 넣고 안넣고의 차이를 두고 결과를 보았다.
그래프에서도 알 수 있듯이 **Batch Normalization** 이 있을 때 더 빨리 수렴하는 것을 알 수 있다.

- - -

참고자료
========
<http://cs231n.github.io/neural-networks-2/#reg>
<http://sanghyukchun.github.io/88/>
<https://towardsdatascience.com/batch-normalization-in-neural-networks-1ac91516821c>
<https://brunch.co.kr/@chris-song/39>
