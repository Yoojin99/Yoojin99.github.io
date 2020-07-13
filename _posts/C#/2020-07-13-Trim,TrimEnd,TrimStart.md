---
title:  "[C#] Trim, TrimEnd, TrimStart(공백제거, 문자제거)"
excerpt: ""

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

[C#] Trim, TrimEnd, TrimStart(공백제거, 문자제거)
========================

Trim, TrimStart, TrimEnd 함수는 C#에서 공백을 제거하며 원하는 문자(들)까지 제거할 수 있게 해주는 함수이다.

## 1. Trim(), TrimStart(), TrimEnd() 정의와 사용법

* public string Trim() : 현재 문자열의 앞쪽, 뒤쪽 공백을 모두 제거한 문자열을 반환한다.
* public string TrimStart() : 현재 문자열의 앞쪽 공백을 모두 제거한 문자열을 반환한다.
* public string TrimEnd() : 현재 문자열의 뒤쪽 공백을 모두 제거한 문자열을 반환한다.

Trim, TrimStart, TrimEnd는 String 클래스에 있는 멤버함수(메서드)이다.

얘네는 맨 앞, 뒤의 공백을 제거하는 애들이어서 특정 문자 사이에 있는 공백은 제거할 수 없다. 사이사이에 있는 공백을 제거하려면
replace 메서드로 가능하다.

```C#
string str = "  안녕 하 세요   ";

string str1 = str.Trim();
string str2 = str.TrimStart();
string str3 = str.TrimEnd();
```

와 같이 사용한다.

이렇게 Trim을 했다고 해서 원본 스트링이 변하거나 하는 것은 아니다.



