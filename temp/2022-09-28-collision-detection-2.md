---
title: 충돌 감지 알고리즘 2
date: 2022-09-28 02:00:00 +/-TTTT
categories: [Algorithm, Programming]
tags: [Physics, C++, Collision detection]  
---

이전 포스트에서는 Convex shape이 point를 포함하는지 판별하는 알고리즘을 알아봤다. 이번 포스트에서는 Convex shape간의 충돌(Overlap)을 다룬다.  

## 원과 원의 충돌

원과 원이 충돌(Overlap) 판별은 간단하다. 두 원의 중심 사이 거리가 두 원의 반지름의 합보다 작으면 충돌이다. 코드로 적으면 아래와 같다.  

```c++

bool TestOverlap(const Circle& c1, const Circle& c2)
{
    float d2 = Dist2(c.position, c2.position);
    float r = c1.radius + c2.radius;

    return d2 < r * r;
}

```

Circle 구조체는 이전 포스트 구조체와 같다. 이전 포스트처럼 거리의 제곱을 비교하는 점이 포인트이다.  

## AABB간의 충돌

원과 원의 충돌과 마찬가지로 AABB간의 충돌도 간단하게 판별할 수 있다. 바로 코드로 옮기면 아래와 같다.  

```c++

bool TestOverlap(const AABB& a, const AABB& b)
{
    if (a.min.x > b.max.x || a.max.x < b.min.x) return false;
    if (a.min.y > b.max.y || a.max.y < b.min.y) return false;

    return true;
}

```

하한이 상한보다 크거나 상한이 하한보다 작으면 비충돌이다.  

원 vs. 원, AABB vs. AABB 충돌은 어떻게 보면 특수한(?) 경우이고 간단하게 충돌 판별이 가능하다. 하지만 많은 경우 충돌을 검출하고자 하는 오브젝트는 복잡한 모양이다. 적어도 단순한 박스 형태가 아닌 Convex 폴리곤 형태이고 회전도 같이 고려되어야 한다.  

원이나 AABB는 더 정교하고 비용이 높은 충돌 알고리즘을 이용하기 전에 충돌 가능성을 확인하기 위해 사용된다. 두 오브젝트를 감싸는 AABB의 충돌이 없다면 더 정교한 충돌 감지 스텝을 패스하고 충돌이 있다면 정교한 알고리즘을 이용한 충돌을 확인한다. AABB의 BB, Bounding Box가 여기서 나온 의미다.  


## Convex polygon간의 충돌 (SAT)

이제 Convex polygon간의 충돌을 판별하는 알고리즘을 알아보자. 이 포스트에서는 SAT(Separating Axis Theorem)를 설명할 것이고 다음 포스트에서는 SAT보다 효율적이지만 조금 더 복잡한 GJK(Gilbert Johnson Keerthi) distance 알고리즘을 이용해 충돌을 검출하는 방법을 설명한다.  

SAT(Separating Axis Theorem), 분리축 정리는 "두 도형이 충돌하지 않는다면 도형의 정사영이 겹치지 않는 분리축이 존재한다" 라는 내용이다. 아래 그림을 보자.  

![sat](/assets/img/collision/sat.png)_https://en.wikipedia.org/wiki/Hyperplane_separation_theorem 잘 안보이면 다크모드 해제!_  

사진속 두 Convex shape의 초록 선(분리축) 위 정사영은 분리되어 있다. 직관적으로 정리가 납득되기도 하지만 수학적 증명이 필요하다면 위 사진출처를 참고하자.  

자 그렇다면 이 정리를 이용해서 Convex polygon간의 충돌을 검출하려면 적절히 후보 분리축을 찾고, 폴리곤을 분리축에 투영(Projection)해봐서 영역이 분리되는지 판별해 봐야 한다. 그렇다면 분리축을 어떻게 찾을까? 이 알고리즘을 구현하는데 있어서 핵심은 후보 분리축을 임의로 정하는것이 아니라는 것이다.  

결론부터 말하면 분리축의 후보는 두 폴리곤 edge의 법선(Normal)이고 두 폴리곤의 모든 법선에 대해서 투영결과가 하나라도 겹치지 않는다면 충돌이 아닌것이다. 

왜 분리축의 후보가 edge의 노말일까? 분리축은 무수히 많을 수 있다.  
두 Convex polygon이 충돌하는 경우를 잘 생각해 본다면, 모든 충돌은 한 폴리곤의 edge와 다른 폴리곤의 버텍스가 충돌하는 경우로 볼 수 있다. 그래서 테스트해 보려는 분리축 후보로 edge의 노말을 고른다면 분리축 투영 테스트는 자연스레 한 폴리곤의 edge와 다른 폴리곤의 버텍스 간의 테스트가 된다. (edge의 버텍스는 정사영의 끝점이 된다.) 모든 edge들의 노말을 분리축 후보로 테스트하면 충돌이 발생 가능한 모든 시나리오를 검사하는 것과 같다.  

코드는 다음과 같다.  

```c++

// a에 대한 b의 signed distance를 계산함
static float ComputeSeparation(const std::vector<Vec2>& va, const std::vector<Vec2>& vb)
{
    float maxSeparation = -FLT_MAX;

    for (uint32 i = 0; i < va.size(); ++i)
    {
        const Vec2& va0 = va[i];
        const Vec2& va1 = va[(i + 1) % va.size()];

        // 오른손 좌표계, 반시계방향 버텍스 winding order에서 edge normal을 구함.
        Vec2 normal = Cross((va1 - va0).Normalized(), 1.0f); // == Cross(edge, {0.0f, 0.0f, 1.0f})
        float separation = FLT_MAX;

        for (uint32 j = 0; j < vb.size(); ++j)
        {
            const Vec2& vb0 = vb[j];

            // 현재 edge에 대해서 b 버텍스의 signed distance를 계산하고 최소값만 keep함.
            // signed distance가 양수라면 노말의 방향으로 떨어진, 즉 edge 바깥의 점,
            // 음수라면 edge 내부의 점이라고 판별.
            separation = Min(separation, Dot(normal, vb0 - va0));
        }

        // signed distance의 최대값, 즉 max separation를 저장한다.
        maxSeparation = Max(separation, maxSeparation);
    }

    return maxSeparation;
}

bool SAT(Polygon* a, Polygon* b)
{
    const std::vector<Vec2>& va = a->GetVertices();
    const std::vector<Vec2>& vb = b->GetVertices();

    std::vector<Vec2> wva(va.size());
    std::vector<Vec2> wvb(vb.size());

    // local 버텍스들을 world space 버텍스로 변환
    std::transform(va.begin(), va.end(), wva.begin(), [&](const Vec2& v) { return a->GetTransform() * v; });
    std::transform(vb.begin(), vb.end(), wvb.begin(), [&](const Vec2& v) { return b->GetTransform() * v; });

    // a의 edge에 대해 b의 버텍스를, b의 edge에 대해 a 버텍스를 모두 테스트 한다.
    // 두 테스트 결과가 다 음수라면, 즉 Separating Axis가 없다면 충돌이다.
    return ComputeSeparation(wva, wvb) < 0.0f && ComputeSeparation(wvb, wva) < 0.0f;
}

```

위 코드는 signed distance라는 개념을 이용해서 SAT를 단순화한다.  

![sd](/assets/img/collision/sd.png)_edge의 노말을 기준으로 바깥쪽은 + 거리, 안쪽으로는 - 거리_

테스트하려는 edge에 대해 상대 버텍스들의 signed distance 구하고 최솟값만 비교해서 max separation(overlap)를 구한다. max separation이 음수라면 분리축을 찾지 못한 것이다.  
a의 edge에 대해서 b를, 반대로 b의 edge에 대해서 a를 다 테스트해서 분리축을 찾지 못하면 충돌했다고 판별한다. (코드에 달아놓은 주석을 참고해 주세요. 그림을 그려보면 이해가 쉽습니다.)  

시간 복잡도는 코드를 보면 알 수 있듯이 O(n*m)이 된다.  

## Concave polygon의 충돌

Concave polygon간의 충돌은 어떻게 판별할까?  

단순하게 Concave polygon을 Convex polygon으로 쪼개서 검사한다. 이 과정을 Convex decomposition 이라고 한다.

![cd](/assets/img/collision/cd.png)  