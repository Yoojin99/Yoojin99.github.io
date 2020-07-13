---
title:  "[C#] Nullable Type - \"?\" 에 대해"
excerpt: "Chatbot 공부 시작"

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - C#
  
tags:
  - 공부
  - C#
last_modified_at: 
---

[C#] Nullable type, "?"에 대해
========================

C# 코드를 보다보니 
```C#
int?
```
와 같은 것을 보게 되었다.

C#에서는 Null을 가질 수 없는 타입들에게 특별히 Null을 가질 수 있게 해주는 nullable type이라는 개념을 도입했다.

대표적으로 값 타입들이 null을 가질 수 없는 int, 구조체, double, bool등의 타입에서 사용이 된다.

## 1. Nullable Type 이란? 

Nullable Type이란 Null을 가질 수 없는 데이터 타입을 Null을 가질 수 있는 타입으로 만든 새로운 타입이다.

(클래스와 같은 reference type(참조 타입)은 이미 null 체크가 가능하기 때문에 따로 nullable type으로 만들지 않아도 된다.)

이런 값 타입들에 대해 "**값이 없다**"를 표현해주기 위해 Nullable Type이 개발 되었다고 한다.

따라서 위에 나온
```C#
int?
```
와 같은 키워드는 int를 nullable int type으로 변경하여 값이 할당 되었는지 아닌지 확인 할 수 있게 된 것이다.

## 2. Nullable Type의 선언 방법, 속성과 메서드, 주의점

### 1. 선언 방법

1. Nullable<T> 변수명 (ex : Nullable<int> num;)
2. T? 변수명 (ex : int? num;)

### 2. 속성과 메서드

#### 1. HasValue 속성

1. 값이 있는 경우 : true
2. 값이 없는 경우(Null): false

#### 2. Value 속성

1. 값이 있는 경우 : 할당된 값
2. 값이 없는 경우(Null) : Exception 발생

#### 3. GetValueOrDefault() 메서드

1. 값이 있는 경우 : 할당된 값 반환
2. 값이 없는 경우(Null) : 기존 타입의 default 값 반환

### 3. 주의점

Nullable은 중복이 불가능하다. 

```C#
Nullable<Nullable<int>> num; //불가능
```
* **int, int?는 엄연히 다른 타입이므로 명시적 형 변환이 필요하다.**
* Value를 접근할 때는 HasValue로 체크 한 후에 접근하는 것이 안전하다.
