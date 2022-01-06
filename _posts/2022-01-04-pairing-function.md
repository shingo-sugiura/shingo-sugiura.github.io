---
title: Pairing function, 짝짓기 함수
date: 2022-01-04 21:53:00 +/-TTTT
categories: [Note, Math]
tags: [math]  
math: true
---

## Paring function

<https://en.wikipedia.org/wiki/Pairing_function#Cantor_pairing_function>

자연수 두 개를 한 개로 맵핑하는 함수.  
자연수 두 개를 key로 하여 새로운 key를 만들고자 할 때 써먹을 수 있다.  

$$\pi: \mathbb{N} \times \mathbb{N} \to \mathbb{N} $$

$$ \pi(a, b) = \frac{(a + b)(a + b + 1)}{2} + b\,(where\,a,\,b \in \mathbb{N}) $$  

역변환도 가능하다.  
역변환시 인자 순서가 보장된다.  

$$ p = \pi(a,\,b) $$  

$$ w = [\frac{\sqrt{8p+1} - 1}{2}] $$  

$$ t = \frac{w^2 + w)}{2} $$  

$$ b = p - t $$  

$$ a = w - b $$  

## Code  
아래는 타입스크립트 코드.  
```typescript

// Author Sopiro(https://github.com/Sopiro)
export interface Pair<A, B>
{
    p1: A,
    p2: B,
}

// Cantor pairing function, ((N, N) -> N) mapping function
// https://en.wikipedia.org/wiki/Pairing_function#Cantor_pairing_function
export function make_pair_natural(a: number, b: number): number
{
    return (a + b) * (a + b + 1) / 2 + b;
}

// Reverse version of pairing function
// this guarantees initial pairing order
export function separate_pair(p: number): Pair<number, number>
{
    let w = Math.floor((Math.sqrt(8 * p + 1) - 1) / 2.0);
    let t = (w * w + w) / 2.0;

    let y = p - t;
    let x = w - y;

    return { p1: x, p2: y };
}
```

아래와 같은 형식으로 맵핑된다고 한다.

![image](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/Cantor%27s_Pairing_Function.svg/800px-Cantor%27s_Pairing_Function.svg.png){:style="max-width: 50%"}

