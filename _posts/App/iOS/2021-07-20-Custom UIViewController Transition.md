---  
layout: post  
title: "[iOS] - Custom UIViewController Transition, 커스텀 뷰컨트롤러 이동 구현하기"  
subtitle: ""  
categories: app
tags: app-ios
comments: true  
header-img: 

---  
  
> `커스텀으로 UIViewController가 전환되도록 구현해보자.`  

---

iOS에는 기본적으로 멋진 view controller transition을 제공한다. Push, pop, cover vertically와 같은 전환이 있지만 이를 커스텀으로 만들 수 있다.

## Transitioning API

들어가기 전에 transitioning API를 알고 있어야 한다. Transitioning API는 프로토콜의 collection이다. 아래 그림은
API의 메인 컴포넌트들이다.




* 출처
  * https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started
  
