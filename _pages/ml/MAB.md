---
layout: post
title:  "Multi armed bandit"
date:   2018-12-26 01:02:59
author: Inhyuk
category: Machine_Learning
tags:	ml
cover:  "/assets/instacode.png"
name: MAB.md
---

Bandit?
=======

**Multiarmed bandit** 은 Reinforcement Learning을 공부하다보면 가장 먼저 접하게 되는 것 중 하나이다.

**Bandit**(슬롯머신) 이라고도 불리우는 이 문제는 다음과 같이 정의 된다.

*Reward $r$을 출력하는 K개의 probability distribution이 존재한다고 하자. 모든 round $t$(제한된 횟수만 시행 가능)에서 하나의 probability distribution에서 reward(payment, 0또는 1로 간소화 한다)가 출력된다면, 전체 reward의 합을 최대로 하는 policy(어떤 상황에서 어떤 probability distribution에서 reward를 출력할 것인지)를 찾는 것이 목표이다.*

얼핏 보기에는 reinforcement learning(이하 RL)과 같은 것 같지만 차이점이 존재한다.(사실 강화학습이 아니다. Sutton의 책에는 소개되어 있지만)

* RL에서는 agent(player)의 action이 state를 변화시킨다. 하지만, Bandit에서는 agent의 action이 state를 변화시키지 않는다. (기본 MAB에서는 state가 존재하지도 않는다.)
* RL에서는 보통 reward가 연속적인 action후에 delay되어서 주어지지만, MAB에서는 바로 주어진다.

Reward를 최대화 한다고 하였지만, 그와 비슷한 regret을 정의할 것이다. Reward를 최대화 하는 문제에서 regret을 최소화하는 문제로 변형하는 것이다.

$$
Regret = E[\sum^{H}_{t=1}({r_t}^{optimal}-r_t)]

{r_t}^{optimal} : \text{round}\:t\text{에서 optimal policy 로 얻을 수 있는 최대 reward}
$$

다양한 방식으로 문제를 정의하고, regret을 정의하기도 한다.

- - -

문제 해결
========

$\epsilon$-greedy
-----------------

Bandit에서는 [Exploration-Exploitation]() 문제가 중요하다. 처음에 좋은 결과를 낸 슬롯머신에서 계속 게임을 진행하는 것은 바보같은 짓이다. 그 슬롯머신의 첫 게임이 우연치 않게 좋은 결과를 내었고, 옆에 있는 슬롯머신이 총합으로 보았을 때 더 좋은 결과를 내는 슬롯머신이라면 처음의 슬롯머신을 계속해서 이용하는 것은 좋은 policy라고 할 수 없다.

$\epsilon$ greedy는 간단하다.
임의의 1보다 작은 양수 $\epsilon$을 정의하고, $1-\epsilon$의 확률로 이제까지 최고의 reward를 낸 슬롯머신을 이용하고(Exploitation), $\epsilon$의 확률로 나머지 슬롯머신 중 하나를 uniform하게 선택하여 게임을 진행한다.(Exploration)

또 다른 비슷한 방법으로는 $\epsilon$을 양의 상수가 아니라 시간에 따라 감소하도록 만들어서 사용하기도 한다. 이는 시간이 지나면 충분히 Exploration(탐색)을 진행하였다고 판단하고, 더 이상의 탐색은 오히려 reward의 총합을 떨어뜨리는 것이라는 생각을 바탕으로 한다.


UCB
---

$\epsilon$ greedy의 알고리즘의 문제점은 탐색을 진행할 때 최고가 아닌 나머지를 같은 확률로 고려한다는 것이다. 탐색을 진행하다보면 최고의 성능을 보이지 않는 슬롯머신에서도 좋은 성능을 보이는 것도 있고 아무리 봐도 아닌 것 같은 슬롯머신도 존재할 것이다. **UCB** 에서는 이런 잠재성을 upper confidence라고 정의한다. 각 슬롯머신이 reward의 범위를 제한한다. 그러므로 어떤 슬롯머신에서 round $t$에서 가질 수 있는 reward는 다음과 같다.

$$
{R}(a) \leq {\hat{R}}_t(a) + {\hat{U}}_t(a)

{R}(a): \text{expected reward from action a}

{\hat{R}}_t(a): \text{estimated reward from action a at round t}

{\hat{U}}_t(a): \text{upper bound at round t and action a}
$$

${\hat{U}}_t(a)$는 보통 $N_t(a)$(action을 시도한 횟수)가 증가하면 감소하도록 정의한다.

Upper confidence bound(UCB)에서의 action의 선택은 다음과 같다.

$$
{Action}_t = argmax_{a}[{\hat{R}}_t(a) + {\hat{U}}_t(a)]
$$

임의의 bound한 probability distribution은 다음과 같은 성질을 가진다.

$$
\text{Let} { {\{X_i\} }_{i=1} }^N\: \text{be a set of i.i.d random variables which are bounded.}

\text{For every} \:u>0,
P[E[X]>\bar{X_t}+u] \leq e^{-2tu^2}

(\bar{X_t} : \frac{1}{t} \sum^{t}_{s=1} X_s)
$$

이를 변형하면 다음과 같은 식을 얻을 수 있다.

$$
P[R(a) > \hat{R}(a)+\hat{U_t}(a)] \leq e^{-2t{\hat{U_t}(a)}^2}
$$

실제 얻은 reward가 우리가 평가한 bound 보다 작은 경우는 매우 적어야 하므로 부등호의 우측은 매우 작은 값을 주어져야 한다.
$p$를 $e^{-2t{\hat{U_t}(a)}^2}$라고 하면,
$$
\hat{U_t}(a) = \sqrt{\frac{-log p}{2N_t(a)} }
$$
이다.

가장 많이 쓰이는 $p$는 $\frac{1}{t^{2k^2}}$를 사용하는 것이다.(UCB1은 $\frac{1}{t^{4}}$를 사용한다) 그러면 action을 선택하는 수식이 다음과 같아진다.

$$
action = {argmax}_{a} \{ \hat{R}(a) + k\sqrt{\frac{logt}{N_t(a)} } \}
$$

여기서 우변의 2번째 term $k\sqrt{\frac{logt}{N_t(a)} }$이 잠재성을 판단하는 부분이다. $k$는 얼마나 잠재성을 신뢰할 것인지를 정해주는 parameter이다.

Thompson Sampling
-----------------

우리의 목표는 reward의 합(cumulative reward)를 최대로 하는 것이다.

Reward $r$이 어떤 action과 알지 못하는 parameter에 의한 probability distribution이기 때문에 원하는 것은 $E[r|a,{\theta}^{optimal}]$을 최대화 하는 ${\theta}^{\text{optimal} }$을 찾는 것이다.

이를 위하여 Thompson sampling도 위의 2 방법처럼 Exploration과 Exploitation 둘을 적절히 이용한다.

#### Exploitation만 사용

$$
E[r|a] = \int E[r|a,\theta]P[\theta|D]d\theta

D = \{(a_t,r_t)\}
$$

위의 식을 최대화 하는 action을 선택한다.

#### Exploration도 함께 사용.

Exploration과 Exploitation을 함께 적용하기 위해 Probability matching heuristic을 사용하였다. probability matching에서는 어떤 action하에서 reward의 확률 분포(reward는 $\theta$와 action의 확률 분포이다)에 맞게 action이 선택될 확률을 고려 한다. 예를 들어, action 1이 0.4의 확률로 reward를 1로 만들고, action 2가 0.6 확률로 reward를 1로 만들면 action 1을 0.4의 확률로 고르고, action 2를 0.6의 확률로 선택하는 것이다.

**$\epsilon$-greedy** 처럼 무작위하게 진행을 하는 것이 아니라, UCB처럼 해당 action이 최고의 reward를 뽑아낼 확률을 바탕으로 Exploration을 진행한다.

즉, 아래의 식의 확률을 바탕으로 action을 선택한다.

$$
\int I[E[r|a,\theta] = {max}_{\hat{a}}E[r|\hat{a},\theta]]P(\theta|D)d\theta

I[] :\text{indicate function}
$$

![ThompsonSampling]({{site.baseurl}}/post_img/{{page.name}}/ThompsonSampling.png)
(context는 MAB에서는 사용하지 않는다.)

알고리즘의 pesudo code를 보자.

4, 5번째 줄이 위의 식을 반영하고 있다. 확률에 따라 $\theta$를 구하고 그 $\theta$하에서 expected reward를 최대로 만드는 action을 선택한다.

MAB에서 어떤 arm(reward를 보내는 슬롯머신)이 Bernouli distribution을 따른다면, Beta 분포가 그 Bernouli distribution(여기서는 likelihood이다)의 conjugate posterior가 된다.

![TS2]({{site.baseurl}}/post_img/{{page.name}}/TS2.png)

- - -

간략한 Contextual Bandit 설명
=================

위의 MAB에서는 context라는 것을 고려하지 않았다.

Context는 쉽게 생각하면 우리가 게임을 진행하는 환경에서의 state라고 할 수 있다. MAB에서는 state라는 것이 존재하지 않았다. 슬롯머신에서 사과,사과,사과를 확인하고 어떤 action을 수행하였을 때 *777* 을 확인하였다고 하여도 그것을 사과,사과,사과를 state라고 정의하고 같은 action을 수행해도 같은 결과가 나오지는 않는다. 즉, state를 정의할 수 없다는 것이다. RL에서 다음 state는 이전 state와 action의 확률 분포이지만, MAB에서 state(라고 불릴만한 것)과 action의 확률분포로 다음 state가 결정되지 않는다.

Contextual bandit에서는 context를 관찰하고 action을 선택한 후에 reward를 확인한다. 얼핏 보기에는 reinforcement learning과 같아 보이지만, state의 변화여부와 delayed reward의 측면에서 살짝 다르다.

광고 산업을 예로 들어보자.

어떤 유저에게 어떤 광고를 추천하는 시스템을 만들고자 할 때 무작정 MAB를 진행할 수 없다. 사람마다 원하는 것, 필요한 것, 싫어하는 것 등(context)이 다르기 때문에 그에 맞게 적절한 광고 추천(action)을 진행하는 것이 더 나은 선택이 될 것이다.

참고 자료
=========

- <https://en.wikipedia.org/wiki/Multi-armed_bandit>
- <http://sanghyukchun.github.io/96/>
- <http://pavel.surmenok.com/2017/08/26/contextual-bandits-and-reinforcement-learning/>
- <http://banditalgs.com/2016/09/18/the-upper-confidence-bound-algorithm/>
- <https://lilianweng.github.io/lil-log/2018/01/23/the-multi-armed-bandit-problem-and-its-solutions.html>
- <https://en.wikipedia.org/wiki/Thompson_sampling>
- <https://web.stanford.edu/~bvr/pubs/TS_Tutorial.pdf>
- <http://papers.nips.cc/paper/4321-an-empirical-evaluation-of-thompson-sampling.pdf>
