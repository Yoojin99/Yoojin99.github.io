---
title:  "[C++] 전처리기 preprocessor"
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
  - preprocessor
last_modified_at: 2021-01-08
---

전처리기는 말 그대로 프로그램을 컴파일 할 때 **컴파일 직전**에 실행되는 별도의 프로그램이다.
전처리기가 실행되면 각 코드 파일에서 지시자(#로 시작해서 줄바꿈으로 끝나는 코드)를 찾는다.

전처리기는 다음과 같은 역할/기능을 한다.

1. 컴파일러 실행 직전 단순히 텍스트를 조작하는 치환 역할
2. 디버깅에 도움을 준다.
3. 파일의 중복 포함을 방지한다.

## Include

`#include` 를 하면 전처리기는 include된 파일의 내용을 지시자의 위치에 복사한다.(전방 선언에 사용한다.)

```cpp
#incldue <filename>
#include "filenmae"
```

## Macro

`#define` 지시자를 이용해서 매크로를 만들수 있다. 매크로는 **입력을 출력으로 변환하는 방식을 정의하는 규칙이다.**

매크로에는

1. 객체와 유사한 매크로(object-like macro)
2. 함수와 유사한 매크로(function-like macro)가 있다.

함수와 유사한 매크로는 함수처럼 작동한다.

객체와 유사한 매크로는 아래 방법 중 하나로 정의한다.

```cpp
#define identifier
#define identifier substitution_text
```

첫 번째 줄에서는 대체할 텍스트가 없지만, 두 번째 줄에서는 대체할 텍스트가 있다. 지시자는 명령어가 아니기 때문에
세미콜론 `;` 으로 끝나지 않는다.

### 대체 텍스트가 있는 객체와 유사한 매크로

이 지시자에서 identifier는 말 그대로 substitution_text로 대체된다. 식별자(identifier)는 일반적으로 공백을 나타나는 밑줄을 사용하여
모두 대문자로 입력한다.

```cpp
#define IMSI_NUMBER 7

std::cout << "임시 숫자: " << IMSI_NUMBER << std::endl;
```

전처리기는 위의 코드를 아래로 치환한다.

```cpp
#define IMSI_NUMBER 7

std::cout << "임시 숫자: " << 7 << std::endl;
```

### 대체 텍스트가 없는 객체와 유사한 매크로

객체와 유사한 매크로는 대체 텍스트 없이 정의할 수 있다.

```cpp
#define HELLO
```

이 전처리 지시자는 조건부 컴파일을 하기 위해 사용된다.

## 조건부 컴파일

조건부 컴파일 전처리 지시자를 사용하면 컴파일할 조건이나 컴파일하지 않을 조건을 지정할 수 있다.

`#ifdef` 지시자를 사용하면 전처리기가 이전에 `#`이 정의되었는지 아닌지 확인한다. 만약 정의되었다면 
`#ifdef`와 `#endif` 사이의 코드가 컴파일 된다. 아니라면 코드가 무시된다.

```cpp
#define FIRST_NUM

#ifdef FIRST_NUM
std::cout << "First" << std::endl;
#endif

#ifdef SECOND_NUM
std::cout << "Second" << std::endl;
#endif
```

위에서는 `FIRST_NUM`이 정의되어서 `std::cout << "First" << std::endl;`는 컴파일된다.
하지만 `SECOND_NUM`은 정의되지 않아 `std::cout << "Second" << std::endl;`는 무시된다.

`#ifndef`는 `ifdef`의 반대이다. 식별자가 아직 정의되지 않았는지 확인한다.

```cpp
#ifndef SECOND_NUM
std::cout << "Second" << std::endl;
#endif
```

이러면 Second가 출력된다.
