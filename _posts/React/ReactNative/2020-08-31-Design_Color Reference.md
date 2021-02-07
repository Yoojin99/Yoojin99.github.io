---
title:  "React Native - Design - Color Reference"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - react
  
tags:
  - react-rn
  - React Native
  
last_modified_at: 
---

RN에서의 component들은 JavaScript를 이용해서 스타일링 된다. 색깔 property들은 CSS가 웹에서 동작하는 방식과 비슷하다.

## Color API

RN는 여러 색깔 API를 제공한다.

* PlatformColor는 플랫폼의 색깔 시스템을 참조할 수 있게 한다.
* DynamicColorIOS는 iOS 특정적이고 light와 dark 모드에서 어떤 색깔을 사용할 지 결정할 수 있게 한다.

## 색깔 표기법

### RGB

RN은 16진법과 함수 표기법 모두를 `rgb()`와 `rgba()`로 지원한다.

* `#f0f` (#rgb)
* `#ff00ff` (#rrggbb)
* `#f0ff` (#rgba)
* `#ff00ff00` (#rrggbbaa)
* `rgb(255, 0 ,255)`
* `rgba(255,0,255, 1.0)`

### Hue Saturation Lightness (HSL)

RN는 함수 표기법으로 `hsl()`과 `hsla()`를 지원한다.

* `hsl(360, 100%, 100%)`
* `hsla(360, 100%, 100%, 1.0)`

### Color ints

RN은 또한 `int` 값으로 색을 지원한다.

* `0xff00ff00`

### Named colors

RN에서 색의 이름 문자열을 값으로 사용할 수도 있다.

RN은 오직 소문자 이름만 취급한다. 

`transparent`는 `rgba(0,0,0,0)`의 줄임말이다.

#### Color keywords

이름이 있는 색은 CSS3/SVG 명시를 따른다.
