---  
layout: post  
title: "[iOS] - Singleton Pattern Swift로 구현하기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `Singleton Pattern을 Swift에서 구현해보자.`  

---

객체를 단 하나만 생성해야 하는 것을 보장하는 Singleton Pattern을 swift에서 구현하는 것은 굉장히 쉽다.

```swift
class Singleton {
  static let singleton = Singleton() // static 변수를 선언한다.
  
  private init() {} // 생성자를 private으로 선언한다. 이러면 외부에서 Singleton()으로 인스턴스를 생성하지 못한다.
}
```


다른 Singleton Pattern 관련 게시글을 보면서 공부하다가 이른 초기화와 늦은 초기화가 있는 것을 알게 되었는데,
위의 코드를 처음 봤을 때는 swift로 이른 초기화로 객체를 생성하는 것 같아 그럼 늦은 초기화는 어떻게 구현하는 건가 싶어서 찾아봤는데,
swift에서는 static으로 생성한 변수들은 모두 기본적으로 lazy하게 초기화된다고 한다. 

`static` 프로퍼티는 "타입 프로퍼티"이고, 오로지 한 번만 초기화되는 것을 의미한다. 이거는 static이 글로벌 전역 변수로 동작하기 때문에 lazily하게 이루어진다.
즉 lazy하게 초기화 되는 것을 의미한다. 그리고 Swift의 공식 문서에서는 아래와 같이 얘기한다.

> Global constants and variables are always computed lazily, in a similar manner to Lazy Stored Properties. Unlike lazy stored properties, global constants and variables do not need to be marked with the lazy modifier.

전역 상수와 변수는 항상 lazily하게 계산되고, 이는 Lazy 저장 프로퍼티와 비슷하게 동작한다. 하지만 Lazy 저장 프로퍼티와는 다르게, 전역 상수와 변수는 `lazy` 라는 키워드가 붙을 필요가 없다라고 말하고 있다..

이런 암묵적인 lazy 초기화가 일어나는 이유는 Swift Blog에서 또 말하고 있다.

> it allows custom initializers, startup time in Swift scales cleanly with no global initializers to slow it down, and the order of execution is completely predictable.

이것은 커스텀 초기화를 가능하게 하고, Swift는 실행한 초기에 글로벌 이니셜라이저없이 깔끔하게 확장되며, 실행 순서는 완전히 예측가능하다고 말하고 있다.

따라서 앱의 처음 실행했을 때 불필요하게 지연되는 시간을 줄이깅 위해 이렇게 디자인 한 것이다. 

참고

* https://stackoverflow.com/questions/34667134/implicitly-lazy-static-members-in-swift
