---
title:  "[C++] Namespace 사용법, 사용 이유"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - cpp
  
tags:
  - 공부
  - C++
  - namespace
last_modified_at: 2020-10-18
---

C++은 객체지향 언어이다. C는 절차지향 언어이다.

객체지향 언어는 이식성이 좋다는 게 특징이다. 쉽게 말하면 이미 잘 만들어진 기능 A를 다른 프로그램 B에 사용하고 싶을 때 함수충돌, 변수 충돌 문제 없이 사용할 수 있다는 것이다.
이렇게 할 수 있는 이유는 클래스와 namespace 덕분에 가능하다.

namespace는 작은 모듈화라고 생각하면 된다. 같은 이름이라도 모듈을 다르게 해서 구분할 수 있게 하는 것이다. namespace의 경우 대규모 프로젝트에서 유용하게 쓰일 것이다.
대규모 프로젝트를 진행할 때, 많은 사람이 협업을 할 것이고 모든 변수와 함수의 이름을 다 다르게 짓는 것은 사실상 불가능한 일이다. 따라서 이들을 서로 구분지을 수 있는 모듈화가 필요할 것이다.

예를 들어 사람 A 가 

```c++
void printThis() {std::cout<<"HelloA"<<std::endl;}
```

라는 함수를 작성했고,

사람 B가

```c++
void printThis() {std::cout<<"HelloB"<<std::endl;}
```

라고 함수의 이름을 겹치게 작성했다고 해보자. 이렇게 같은 이름의 함수를 작성했을 때 namespace로 이들을 구분 지을 수 있다.

```c++
namespace A
{
  void printThis() {std::cout<<"HelloA"<<std::endl;}
}

namespace B
{
  void printThis() {std::cout<<"HelloB"<<std::endl;}
}
```

출력은 다음과 같이 하면 된다.

```c++
int main() {
  A::printThis();
  B::printThis();
}
```

이때 코드에서 사용된 `::`를 **범위지정연산자(scope resolution operator)**라고 한다. 말 그대로 이름 공간을 지정할 때 사용하는 연산자이다.

namespace를 이용할 때 선언하는 부분과 정의하는 부분을 구분지을 수 있다.

```c++
#include<iostream>
 
namespace IamA{ void printOut();}
 
namespace IamB{ void printOut();}
 
void main() {
    IamA::printOut();
    IamB::printOut();
}
 
void IamA::printOut() { std::cout << "출력!" << std::endl; }
 
void IamB::printOut() { std::cout << "출력!" << std::endl; }
```

