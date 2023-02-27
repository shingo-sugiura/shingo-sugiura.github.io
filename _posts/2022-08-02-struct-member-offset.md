---
title: 멤버 변수 오프셋과 정렬
date: 2022-08-02 20:10:00 +/-TTTT
categories: [Note, Programming]
tags: [Programming, C++]
# math: true
---

Offset and alignment of member variables, 멤버 변수 오프셋과 정렬

## 멤버 변수의 offset

다음과 같은 구조체가 있을때 각 멤버 변수의 offset(메모리 상에서 해당 변수가 시작하는 지점)을 컴파일 타임에 구하고자 한다.

```c++
struct VertexLayout
{
  float position[3];
  float normal[3];
  float uv[2];
};
```

위와 같은 구조체에서 각 멤버 변수의 offset을 생각해 본다면 직관적으로 position은 0, normal은 position 12, uv는 24 일것이다. 구조체 전체 사이즈는 32바이트. float은 4바이트고 메모리상에 변수들이 연속적으로 위치해 있으므로 이 생각이 실제로도 맞다.

다음의 코드로 멤버 변수의 offset 값을 코드로 컴파일 타임에 계산할 수 있다.

```c++
size_t offset = (size_t) &((VertexLayout*)nullptr)->normal;
```

해석해 보자면

1. nullptr, 즉 (void*)0 을 offset을 구하고자 하는 구조체 포인터로 형변환 한다
2. 1번에서 구한 포인터에 -> 로 특정 멤버값을 얻어 오고 & 로 그 멤버의 주소를 얻는다.
3. 그 주소는 0번지에서 시작된 값이므로 그 주소값 자체가 해당 멤버 변수의 offset이 된다.

간결하고 멋진 방법으로 구조체의 멤버 변수의 offset을 계산하는 코드다!  
나아가서 해당 코드를 아래와 같이 매크로로 만들 수 있다.

```c++
#define OffsetOf(T, m) (size_t)&((T*)nullptr)->m - (size_t)nullptr

// 호출 할 때는 이런식으로..
OffsetOf(VertexLayout, normal);
```

위 코드에서 (size_t)nullptr를 빼주는 이유는 nullptr가 물론 (void*) 0 로써 0번지를 가리키는 값 이겠지만 명시적어놔 봤다. nullptr 대신 임의의 위치에서 시작한다면 해당 빼기 연산을 꼭 해줘야 할 것이다.

cstddef 헤더에 같은 동작을 하는 offsetof() 매크로가 있다.

아래는 c++스럽게 매크로를 이용하지 않고 템플릿을 이용해 OffsetOf 매크로와 동일한 동작을 하는 코드다.

```c++
template<typename T, typename U>
constexpr size_t offsetOf(U T::* m)
{
    return (size_t)&(((T*)nullptr)->*member) - (size_t)nullptr;
}

// 호출 할 때는 이런식으로..
offsetOf(&VertexLayout::normal);
```

## Member variable alignment, 멤버 변수 정렬

위 VertexLayout 구조체의 경우는 멤버 변수들이 연속적으로 메모리 상에 위치해 있는다. 아래와 같은 경우라면 멤버 변수가 메모리 상에 어떤식으로 정렬(align) 되어 있을까?

```c++
struct T
{				 
    int a;  	// 4 bytes
    bool b; 	// 1 byte
    float c;	// 4 bytes
    double d;	// 8 bytes
};
```

VertexLayout 구조체처럼 모든 멤버가 연속적으로 존재해서 offset들이 a는 0, b는 4, c는 5, d는 9 일까? 실제로 OffsetOf로 찍어보면..

```c++
int main()
{
    printf("%d ", OffsetOf(T, a));
    printf("%d ", OffsetOf(T, b));
    printf("%d ", OffsetOf(T, c));
    printf("%d ", OffsetOf(T, d));
    printf("\n%d", sizeof T);
  
    return 0;
}
// 출력 결과
// 0 4 8 16
// 24
```

결과는 예상과 다르게 나왔다. 그 이유는 컴파일러가 최적화를 위해 멤버 사이에 패딩(padding)을 넣었기 때문이다. 규칙은 아래와 같다.

*컴파일러는 멤버 변수가 메모리 상에서 구조체의 시작점을 기준으로 각 멤버 변수 바이트 크기의 배수 주소에서 변수가 위치되도록 패딩을 넣는다. 또한 구조체 사이즈가 멤버 변수중 가장 바이트수가 큰 놈의 배수로 끝나도록 마지막에 패딩을 넣는다.*

위 T 구조체로 예로 들면 다음과 같다.

- a는 0번지에서 4바이트 차지 -> offset = 0
- b는 4번지에서 1바이트 차지 -> offset = 4
- c는 float이므로 4바이트 즉 4의 배수 지점에서 시작 되어야 하기 때문에 가장 가까운 배수, 8번지에서 4바이트 차지 -> offset = 8
  - 5, 6, 7 번지는 padding이 들어감
- d는 double이므로 8바이트 즉 8의 배수 지점에서 시작 되어야 하기 때문에 가장 가까운 배수 16번지에서 8바이트 차지 -> offset = 16
  - 12, 13, 14, 15 번지는 padding이 들어감
- 멤버 변수중 바이트가 제일 큰 놈은 8바이트이고 현제 구조체가 8의 배수 24에서 딱 끝나기 때문에 뒷쪽에 padding을 넣지 않는다. -> size = 24bytes

이런 규칙대로 컴파일러는 구조체 멤버 사이사이에 패딩을 넣는데 그 이유는 CPU 메모리 엑세스 패턴과 관련하여 최적화 하기 위함이라고 한다. 위와 같이 잘 데이터가 정렬되어 있으면 *naturally aligned* 라고 표한하고 아니라면 *misaligned* 라고 표현 한다고 한다. misaligned된 데이터도 동작하긴 하지만 퍼포먼스 차이가 상당하다.(정말로 심하게 차이난다) 그러므로 pragma pack 같은 전처리 지시자를 이용해서 packing, alignment를 프로그래머가 컨트롤 할 수도 있지만 메모리가 정말 부족한 환경이 아니라면 그대로 두는게 좋다.

* https://docs.microsoft.com/en-us/cpp/cpp/alignment-cpp-declarations?view=msvc-170

## 정리

Alignment 내용과 관련해서 중요한 부분은 위에 적힌 규칙이 일반적이지만 컴파일러마다 다를 수 있고 같은 컴파일러를 써도 컴파일하는 환경마다 다를수 있다는 것이다.  
그래서 동일한 로컬 머신에서 구조체 또는 객체를 serialization해서 파일에 썻다가 불러오는것 같은 경우는 간단하게 바이너리로 write하고 memcpy로 불러오는 식으로 빠르게 처리할 수 있겠지만, 저장된 파일이 다른 환경에서 불러와 지는 경우라면 위 내용들을 고려해야 될 것이다.

