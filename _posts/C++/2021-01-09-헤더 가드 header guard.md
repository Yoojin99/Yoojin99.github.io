---
title:  "[C++] 헤더 가드"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - C++
  
tags:
  - 공부
  - C++
  - 헤더 가드
last_modified_at: 2021-01-09
---

## 중복 정의 문제

식별자는 하나의 정의만 가질 수 있다.  따라서 변수 식별자를 두 번 이상 정의한 프로그램에서는 오류가 발생한다.

함수 또한 두 번 이상 정의하면 컴파일 오류가 발생한다. 헤더 파일이 두 번 이상 포함될 경우에도 문제가 될 수 있다.

예를 들어 math.h는 아래와 같다.

```cpp
int getSquareSides() {
  return 4;
}
```

geometry.h라는 헤더파일은 아래와 같다.

```cpp
#include "math.h"
```

그리고 main.cpp가 아래와 같다고 하자.

```cpp
#include "math.h"
#include "geometry.h"

int main() {
  return 0;
}
```

위의 main.cpp에서는 "math.h"를 복사하고, "math.h"를 포함한 "geometry.h"를 복사한다.

모든 지시자를 해결한 main.cpp는 아래와 같다.

```cpp
int getSquareSides() {
  return 4;
}

int getSquareSides() {
  return 4;
}

int main() {
  return 0;
}
```

## 헤더 가드

헤더 가드(header guard)라는 매커니즘으로 위의 문제를 해결 할 수 있다.

헤더 가드는 조건부 컴파일 지시자를 사용한다.

```cpp
#ifndef SOME_UNIQUE_NAME
#define SOME_UNIQUE_NAME

#endif
```

위 헤더 파일이 include 되면 `SOME_UNIQUE_NAME`이 정의되었는지 확인한다.

헤더 파일을 만약 include 하지 않은 상태라면 `SOME_UNIQUE_NAME은 정의되어 있지 않기 때문에
이것을 정의하고 헤더 파일의 내용을 include 한다. 하지만 이전에 헤더 파일을 include 했다면
이미 정의되어 있기 때문에 헤더 파일 내용이 무시된다.

그니까 헤더파일이 정의되어 있지 않다면 정의를 하는 것이고, 정의되어 있다면 #endif를 만날 때까지 그 사이의
코드를 무시하는 것이다. 이 매커니즘을 **Include guard**라고 부른다.

모든 헤더 파일에는 헤더 가드가 있어야 한다.

`SOME_UNIQUE_NAME` 은 일반적으로 `_H`가 붙은 헤더 파일의 이름을 사용한다.

math.h는 아래와 같이 작성해준다.

```cpp
#ifndef MATH_H
#define MATH_H

int getSquareSides() {
  return 4;
}

#endif
```

## #pragma once

현재 많은 최신 컴파일러는 `#pragma` 지시자를 이용한 단순한 헤더 가드를 지원한다.

헤더 가드와 같은 기능을 하고, 짧다는 것이 장점이다. 

하지만 이 지시자는 비주얼 스튜디오 종속적인 키워드라, 보편적인 코드를 만들고 싶다면 헤더 가드를 사용하는 것이 좋다.
