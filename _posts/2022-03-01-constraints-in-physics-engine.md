---
title: Constraints in physics engine
date: 2022-03-01 08:50:00 +/-TTTT
categories: [Programming, Physics]
tags: [Physics, Physics Engine]
math: true
---

이 글은 메모글 입니다. 처음 읽으시는 분들은 이해하기 힘듭니다..  

## Constraint in physics engine

물리엔진에서는 모든 것들을 constraint(제약)로 정의한다.  
추상적인 위 문장을 좀 더 자세히 풀어서 설명하자면 다음과 같다.  

우리는 어떤 시스템을 만드는데, 이 시스템을 구성하는 요소들을 constraint로 정의하고 이 시스템의 모든 constraint가 만족(satisfied)된 상태라면 이 시스템은 문제가 없는 상태라고 본다. 반면 어떤 constraint가 위반(violated)되었다면 그것을 해결(solve)하여 문제가 없는 상태로 만든다.  

물리엔진은 이 시스템을 우리가 사는 세계처럼 물리적으로 옳게 보이는 constraint들을 하나 하나 정의하여 만든다. 예를 들어 어떤 강체(Rigid body)가 움직이다가 다른 강체와 겹쳐지는 상황을 생각해 보자. 아무런 constraint도 없는 시스템이라면 이런 상황은 문제될게 없다. 반면 non-penetration constraint가 정의된 시스템(물리엔진)이라면 이 상황은 문제되는 상황이고 이를 적절하게 해결해야 한다.

## Modeling and solving constraint  

어떤 오브젝트의 position 정보 $q$를 이용하여 position constraint, $C(q)=0$를 정의한다.  
$C$를 미분하여 velocity constraint, $\dot{C} = Jv$을 구한다.  
여기서 $J$는 jacobian matrix다. $( \dot{C} = \frac{\partial{C}}{\partial{q}} \dot{q}) $

$\bar v_2 = v_1 + M^{-1} \cdot F_{ext} \cdot \Delta t$  

위 식에서 $\bar v_2$ 는 이전 프레임에서의 속도 $v_1$에 외력 $F_{ext}$를 적용하여 잠재적으로 velocity constraint를 위반한 상태인 속도 벡터이다. 즉 $v_1$에 외력을 적용한 tentative velocity $\bar v_2$를 solve하여 velocity constraint를 만족하게 되는 $v_2$를 계산한다.

$v_2 = \bar v_2 + M^{-1} \cdot P_c = \bar v_2 + \Delta V_c$  

위 식에서 $P_c$는 corrective impulse이다. 해석하자면 tentative velocity 에 적절한 impulse를 가해서 constraint가 만족하도록 만든다는 것이다.

$P_c = J^T \lambda\;(\because P_c \parallel J^t)\;$ 여기서 $\lambda$ 는 Lagrangian multiplier

이 부분이 constraint solve 과정에서 가장 이해하기 힘든 부분이다. 어째서 corrective impulse의 방향이 $J^T$인 것인가..  
아주 간단하고 직관적으로 설명하자면, 시스템에서 constraint는 passive하기 때문에 constraint force(impulse)는 일을 하지 않아야 하기 때문에 $v$에 수직인 $J^T$와 평행한 방향이어야 한다는 것이다.

자 이제 corrective impulse의 방향은 알았으니 그 크기 $\lambda$ 를 구하면 된다.

$Jv_2 + b = 0\;(\because \dot C = Jv+b = 0)$  
$J(\bar v_2 + M^{-1} \cdot P_c) + b = 0$  
$J(\bar v_2 + M^{-1} \cdot J^T \lambda) + b = 0$  
$J\bar v_2 + J \cdot M^{-1} \cdot J^T \cdot \lambda + b = 0$  
$\lambda = (J M^{-1} J^T)^{-1} \cdot -(J\bar v_2 + b)$  
$P_c = J^T \cdot (J M^{-1} J^T)^{-1} \cdot -(J\bar v_2 + b)$  
$\therefore v_2 = \bar v_2 + M^{-1} \cdot J^T \cdot (J M^{-1} J^T)^{-1} \cdot -(J\bar v_2 + b)$  

## Conclusion

물리엔진은 정말 모든것을 constraint로 정의한다. 위에서 언급한 non-penetration constraint뿐만 아니라 다양한 관절(joint)들 심지어 모터, 도르래 같은 것들도 모두 다 constraint로 정의한다. 또한 constraint마다 jacobian $J$ 만 다르기 때문에 solve 과정이 모든 constraint에 대해 일관적(unified way)이다.  

이 글에서 정리한 내용은 물리엔진 constraint에 대한 내용 속에서 핵심적인 내용이긴 하지만 아주 일부이다. 언급 안한 bias $b$ 에도 많은 내용이 숨겨져 있다.. positional error correction, soft constraint 등등.. 또한 왜 acceleration 레벨이 아닌 velocity 레벨에서 impulse로 constraint를 해결하는지 등등 이러한 내용들도 다 정확히 이해해야 한다.

기회가 된다면 물리엔진 튜토리얼 글을 쓰고 싶다.
