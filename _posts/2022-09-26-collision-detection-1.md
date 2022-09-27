---
title: 충돌 감지 알고리즘 1
date: 2022-09-26 03:50:00 +/-TTTT
categories: [Physics, Programming]
tags: [Algorithm, Physics, C++]  
---

~~컴퓨터가 고장나서 노트북으로 쓰기 시작하는 시리즈..~~

## 충돌 감지 알고리즘

누구나 프로그래밍을 하다 보면 언젠가는 마주하는 충돌 감지 알고리즘!  
게임 속 플레이어가 다른 오브젝트와 충돌했는지 판별하는 경우는 전형적으로 충돌 감지 알고리즘이 필요한 경우이고, 마우스가 어떤 UI 오브젝트에 올라갔는지 판별하고자 하는 경우도 일종의 충돌 감지 알고리즘이 필요한 경우이다. 이처럼 생각보다 쉽게 마주치는 녀석 치고는 실제로 처음으로 구현하고자 할 때 꽤나 당황스러울수 있는 주제이다.  

이 시리즈에서는 어떠한 도형이 점을 포함하는지 판별하는 알고리즘부터 임의의 폴리곤 간의 충돌 감지 알고리즘까지 모두 다룰 것이다.  

지금 이 포스트에서는 어떤 볼록 도형(Convex shape)이 점을 포함하는지에 대한 내용이고 다음 포스트에서는 Convex shape들 간의 충돌에 대해 다룬다. 2D를 기준으로 설명하겠지만 알고리즘들은 쉽게 3D로 확장 가능하다.  

## 원이 점을 포함하는지 판별하기 

원은 위치(중심)와 반지름으로 정의할 수 있다.  

![circle](/assets/img/collision/circle.png)  

```c++
struct Circle
{
    Vec2 position;
    float radius;
};
```

원이 점을 포함한다는 것은 점과 원의 중심사이의 거리가 반지름보다 작은지 확인하면 되겠다.  
어려움 없이 바로 코드로 옮기면 아래와 같다.  

```c++

bool TestPoint(const Circle& c, const Vec2& p)
{
    float d2 = Dist2(c.position, p);

    return d2 < c.radius * c.radius;
}

```

Dist2 함수는 두 점 사이 거리의 제곱을 리턴하는 함수다. 두 점 사이 거리는 피타고라스 정리고 계산 가능하고 과정에서 제곱근 연산이 필요하다. 가능한 한 무거운 연산을 피하고자 반지름의 제곱과 거리의 제곱을 비교하는 점이 포인트이다.  

## AABB(Axis Aligned Bounding Box)가 점을 포함하는지 판별하기

AABB(Axis Aligned Bounding Box)란 Box이긴 한데 Edge들이 축에 정렬되어 있는 Box다. AABB는 원 만큼 간단한 형태이고 회전이 없는 형태이기에 충돌 감지에 있어서 다양한 이점이 있다.  

AABB는 하한(lower bound)와 상한(upper bound)로 정의 할 수 있다. 

![aabb](/assets/img/collision/aabb.png)  

```c++
struct AABB
{
    Vec2 min; // lower bound
    Vec2 max; // upper bound
};
```

AABB가 점을 포함한다는 것은 점이 하한보단 위에 있고 상한보다는 아래에 있는지 확인하면 된다. Edge들이 축에 정렬되어 있기 때문에 간단하게 판별이 가능하다.  
코드로 옮기면 아래와 같다.  

```c++

bool TestPoint(const AABB& aabb, const Vec2& p)
{
    if (aabb.min.x > p.x || aabb.max.x < p.x) return false;
    if (aabb.min.y > p.y || aabb.max.y < p.y) return false;

    return true;
}

```

점이 포함되는 경우를 기준으로 코드를 작성하게 된다면 위 코드에서 모든 등호가 반대로 되고 and 로 엮여야 하기 때문에 4번 모두 확인해야 하지만 점이 포함되지 않는 경우라면 위 4가지 판별중에 하나만 false라면 early return 할 수 있다. 사족이 길지만 나름의 최적화라고 할 수 있겠다.  

## Convex polygon이 점을 포함하는지 판별하기

Convex polygon(볼록 폴리곤)은 모든 내각이 180도 아래로 구성된 도형을 말한다. 반대로는 Concave polygon이 있다. 앞으로 다룰 모든 충돌 관련 알고리즘은 Convex polygon에 대해 다룰것이다. Convex polygon의 특성이 여러 경우에서 간단한 형태로 계산할 수 있도록 해준다. 대표적으로 도형 내부 임의의 선분이 polygon의 edge와 충돌하지 않는 다는 특성이 있다.  

![convex-concave](/assets/img/collision/convex-concave.png)  

Convex polygon은 원점을 기준으로 정점들이 위치해 있고 이 점들을 world space로 변환시키는 transform으로 정의한다. 정점들의 winding order는 시계방향 또는 반시계 방향 하나로 정해져 있어야 한다.

```c++
struct Polygon
{
    std::vector<Vec2> vertices;
    Transform t;
};
```

local space에 vertices를 두고 transform을 이용해 오브젝트를 정의하는 방식은 그래픽스/게임 프로그래밍에서 흔하게 사용되는 방법이다.  

![polygon](/assets/img/collision/polygon.png)  

어떤 점이 폴리곤 내부에 포함되어 있는지 확인하기 위해 우리는 정점들의 winding order가 일관적이라는 점을 이용한다. 코드는 다음과 같다.  

```c++
bool TestPoint(const Polygon& polygon, const Vec2& p)
{
    const std::vector<Vec2>& vertices = polygon.vertices;
    Vec2 localP = MulT(polygon.transform, p); // ----------------------------------------- a

    float dir = Cross(vertices[0] - localP, vertices[1] - localP); // -------------------- b
    size_t count = vertices.size();

    for (size_t i = 1; i < count; i++)
    {
        float nDir = Cross(vertices[i] - localP, vertices[(i + 1) % count] - localP); // - c

        if (dir * nDir < 0) // ----------------------------------------------------------- d
        {
            return false;
        }
    }

    return true;
}
```

우선 a 부분에서 query point p를 폴리곤의 local space로 가져온다. (MulT 함수는 vector를 inverse transform 해주는 함수이다.)  

점 p가 폴리곤 내부에 있다면 점 p 에서 n번째 정점으로 향하는 벡터와 n+1번째 정점으로 향하는 벡터, 이 두 벡터간의 외적 결과 벡터는 항상 같은 방향일 것이다. (2d에서는 외적 결과가 (0, 0, scalar) 형태로 x, y 값은 항상 0이므로 z값 scalar만 리턴하도록 Cross 함수를 작성한다.)  

b 부분에서 p에서 0번 정점으로 향하는 벡터와 p에서 1번 정점으로 향하는 벡터를 외적하고 그 값(방향)을 저장한다.  

다음으로 for문을 돌면서 Cross(p to v_n, p to v_n+1) 을 계산하면서(c 라인) b에서 처음 구한 방향과 같은 방향인지 확인한다(d 라인) 모두 같은 방향이면 폴리곤은 점을 포함한다. 정점 winding order가 일정하기 때문에 확인 할 수 있다.

## 정리

이 포스트에서는 Convex shape이 점을 포함하는지 판별하는 알고리즘을 알아봤다. 다음 포스트에서는 Convex shape들 간의 Overlap을 판별하는 알고리즘을 알아본다.  