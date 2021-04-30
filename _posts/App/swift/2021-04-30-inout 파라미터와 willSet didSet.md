---  
layout: post  
title: "Swift - 프로퍼티 감시자가 있는 프로퍼티를 inout으로 전달했을때 항상 willSet didSet이 호출되는 이유"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  
  
> `프로퍼티 감시자가 있는 프로퍼티를 함수의 입출력 매개변수(inout)로 전달했을 때 왜 항상 willSet과 didSet 감시자를 호출하는가?`  

---

결론부터 얘기하자면 함수 내부에서 값이 변경되든 그렇지 않든 함수가 종료되는 시점에 값을 다시 쓰기 때문이다.

먼저 in-Out 파라미터에 대해 보도록 하겠다.

## In-Out 파라미터

In-out 파라미터는 아래와 같이 전달된다.

1. 함수가 호출되었을 때, 변수의 값이 복사된다.
2. 함수 내에서 변수의 값이 수정된다.(아닐 수도 있다. 그런데 내부에서 값을 수정하지 않는다면 in-out을 굳이 사용할 필요가 없을 것이다.)
3. 함수가 return할 때, 함수의 복사한 값을 원래의 변수에 할당한다.

이런 동작들은 copy-in-copy-out이나 call by value result(값으로 호출)로 알려져 있다. 예를 들어 연산 프로퍼티나 감시자가 있는 프로퍼티를 in-out 파라미터로 넘겨주게 된다면,
getter는 함수 호출의 일부로 같이 불려질 것이고 setter는 함수 return의 일부로 같이 불려질 것이다.

최적화를 위해, 만약 변수가 메모리의 물리적 주소에 저장된 값이라면 같은 메모리 주소는 함수 내부와 밖에서 동시에 사용된다. 즉 주소를 함수 외부, 내부에서 공유한다는 소리다. 이런 최적화 된 행위는 
call by reference로 알려져 있고, 이는 복사하는 것의 오버헤드를 없애면서 copy-in-copy-out 모델의 요구사항을 충족한다.

함수에서 in-out 변수로 전달된 값을 원래의 값이 현재의 스코프에서 접근이 가능하다 하더라도 접근하지 말아야 한다. 원래의 값에 접근하는 것은 값에 동시 접근하는 것이고, 이것은 스위프트의 메모리 배제 보장을
위반하는 것이다. 같은 이유로, 여러개의 In-out 파라미터로 같은 값을 전달하면 안된다.

참고

* https://docs.swift.org/swift-book/ReferenceManual/Declarations.html#ID545
