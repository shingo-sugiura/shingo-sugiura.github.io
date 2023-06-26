---
title: Quaternion differentiation
date: 2023-02-27 22:00:00 +/-TTTT
categories: [Math]
tags: [Note]  
math: true
---

[!] 이 문서에서 사용되는 \vec 표기는 벡터가 아니라 pure quaternion입니다. 예를 들어 $\vec w$는 $ 0 + w_x i + w_y j + w_z k $ 형태의 사원수. 또한 \hat은 표기는 unit quaternion 입니다.  

[!] 아래 유도 과정을 이해 하기 위해선 사원수에 대한 이해가 필요합니다.

## Intro

$$\hat q_{new} = \hat q_{old} + \frac 1 2 \vec w \hat q_{old} \Delta t $$  

Angular velocity vector를 사원수 orientation에 적용(수치 적분) 하는 방법을 찾다가 위와 같은 식 발견했는데 많은 사람들이 유도 과정에 대한 이해는 생략하고 black box formula 처럼 사용하는듯 했다..  

## Quaternion differentiation

$\hat q(t)$가 $t$ 에서의 rotation quaternion이고, $\hat r$은 unit time에 대한 rotational motion 이라고 생각한다면,

$$ \hat q(0)=\hat q_0 $$ 

$$ \hat q(t)=\hat r^t \hat q_0 $$

위와 같이 쓸 수 있다.  

unit quaternion $\hat r$은 오일러 공식을 이용해 아래와 같이 지수 함수로 표현 할 수 있다.  

$$ \hat r=e^{\frac \theta 2 \vec u} $$  

여기서 $\vec u$는 실수 부분이 0이고 벡터 파트는 회전축 단위 벡터인 pure quaternion 이다.  

$$ \vec u = u_x i + u_y j + u_z k $$  

오일러 공식을 테일러 급수를 이용해 유도 할 때 $e^x$의 x 자리에 $i$를 넣어 유도하는 부분을 기억하자. 허수 단위 $i$가 제곱해서 -1이 되듯 pure quaternion $\vec u$도 제곱하면 -1이 된다. 즉 대수적 성질이 동일하다.  

$$ \hat r^t=exp(\frac \theta 2 \vec u t) $$

$$ \hat q(t)=exp(\frac \theta 2 \vec u t)\hat q_0 $$  

양변을 미분하면,  

$$ \hat q'=\frac \theta 2 \vec u exp(\frac \theta 2 \vec u t)\hat q_0 $$

$$ \hat q'=\frac \theta 2 \vec u \hat q $$

여기서 $ \theta \vec u $는 각속도 pure quaternion $\vec w$ 이기 때문에

$$ \hat q'=\frac 1 2 \vec w \hat q $$

위와 같은 미분방정식을 얻게 된다.  

## Integration

이제 각속도와 Orientation에 대한 미분 방정식이 주어졌으니, 일반적인 방법대로 수치적분을 하면 된다.  

$$ \frac {\Delta \hat {q}} {\Delta t} \approx \frac 1 2 \vec w \hat q_0 $$  

$$ {\Delta \hat {q}} \approx \frac 1 2 \vec w \hat q_0 {\Delta t} $$  

$$ \hat q_1 \approx \hat q_0 + \frac 1 2 \vec w \hat q_0 {\Delta t} $$

여느 수치 적분법이 그렇듯 $\Delta t$가 크면 오류도 커진다. 더군다나 orientation을 나타내는 quaternion은 unit quaternion이어야 하기 때문에, 오류를 고려하여 시뮬레이션 중에 orientation을 renormalization하는 작업이 필요하다.

## References

<https://en.wikipedia.org/wiki/Quaternion>  
<https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation>  
<https://gamedev.stackexchange.com/questions/108920/applying-angular-velocity-to-quaternion>  