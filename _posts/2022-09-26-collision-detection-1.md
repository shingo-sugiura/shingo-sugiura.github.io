---
title: 충돌 감지 알고리즘 1
date: 2022-09-26 03:50:00 +/-TTTT
categories: [Algorithm, Programming]
tags: [Physics, C++, Collision detection]  
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

## Convex polygon이 점을 포함하는지 판별하기

Convex polygon(볼록 폴리곤)은 도형을 가로지르는 선분이 도형의 Edge를 딱 2번만 만나는 도형을 말한다. (Edge의 tangent나 꼭지점을 지나는 경우 말고) 반대로는 Concave polygon이 있다. 앞으로 다룰 모든 충돌 관련 알고리즘은 Convex polygon에 대해 다룰것이다. 충돌 검출에 있어서 Convex polygon의 특성은 여러 경우에서 간단한 형태로 계산할 수 있도록 해준다. ( 도형 내부 임의의 선분이 polygon의 edge와 충돌하지 않는 다는 특성 등등)

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

우선 코드는 아래와 같다.

```c++
bool TestPoint(const Polygon& polygon, const Vec2& p)
{
    const std::vector<Vec2>& vertices = polygon.vertices;
    Vec2 localP = MulT(polygon.transform, p); // ----------------------------------------- a

    size_t count = vertices.size();
    size_t i0 = count - 1;
    for (size_t i1 = 0; i1 < count; i1++)
    {
        Vec2 normal = Cross(vertices[i1] - vertices[i0], 1.0f); // ----------------------- b
        // normal.Normalize(); <- 정규화 안해도 된다.

        if (Dot(normal, localP - vertices[i0]) > 0.0f) // -------------------------------- c
        {
            return false;
        }

        i0 = i1;
    }

    return true;
}
```

우선 a 부분에서 query point p를 폴리곤의 local space로 가져온다. (MulT 함수는 vector를 inverse transform 해주는 함수이다.)  
b 부분에서는 폴리곤의 면의 normal을 계산한다. Polygon 구조체에 Normal을 미리 계산해 두었다면 인덱스로 직접 가져와도 된다.
마지막으로 c 부분에서 폴리곤의 모든 edge에 대해 signed distance가 음수임을 확인해서 점이 폴리곤 내부에 있다고 판별한다. 점이 edge normal을 기준으로 내부인지, 외부인지 기준점(vertices[i0]) 을 이용해서 Dot 연산으로 판별하는 것이다.

![polygon](/assets/img/collision/tpsd.png)  

## 정리

이 포스트에서는 Convex shape이 점을 포함하는지 판별하는 알고리즘을 알아봤다. 다음 포스트에서는 Convex shape들 간의 Overlap을 판별하는 알고리즘을 알아본다.  