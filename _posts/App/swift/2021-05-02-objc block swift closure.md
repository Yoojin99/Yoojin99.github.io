---  
layout: post  
title: "Swift - Swift의 Closure와 Obj-c의 Block 차이"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  
  
> `Swift의 Closure와 Objective-C의 Block은 어떤 차이가 있는가?`  

---

아직 Swift도 제대로 알지 못하고, Objective-C도 제대로 알지 못해서 헷갈리는 부분이 한 두 군데가 아니다. 나는 함수형 프로그래밍에 익숙하지 않은데,
Objective-C의 블럭을 공부할 때 처음 배우는 개념이라 생소했는데, Swift에서 등장하는 클로저도 처음 접했을 때는 어렵게 느껴졌다.

Objective-C의 블럭은 비동기 처리에 많이 사용된다는 것을 들었고, Swift의 클로저(특히 escaping closure)도 비동기 처리에 많이 사용된다고 한다. 
교육을 받을 때도 obj-c의 블럭과 swift의 클로저가 비슷한 부분이 많다고 들었는데, 그럼 대체 둘은 어떤 차이를 가지고 있는 걸까?

Apple의 "Using Swift with Cocoa and Objective-C" iBook에서 

> Swift의 클로저와 obj-c의 블록은 비교가 상호 호환이 가능해서, Swift의 클로저를 Object-C의 블럭을 기대하는 메서드에 넘겨줄 수 있다.
> Swift의 클로저와 함수는 같은 타입을 가지고 있기 때문에, Swift 함수의 이름을 전달해도 된다.
> 클로저는 블럭과 마찬가지로 비슷한 의미의 capture(값 획득)을 가지고 있지만 한 부분에서 다르다 : 변수는 복사되기 보다는 변경이 가능하다.
> 다른 말로, Objective-C의 `__block` 의 동작은 Swift에서 변수의 기본적인 동작과 동일하다.

라고 나와있다는데 아직도 무슨 말인지 헷갈린다. 잘 이해는 안되지만 클로저와 블럭은 비슷하고, 값 획득에서 무언가 차이가 있다는 말인것 같다.

이 둘의 컨셉은 거의 동일하나 클로저 내부에서 현재 scope에 존재하는 **값 타입 변수**들을 캡쳐해서 사용할 때 기본동작이 반대로 되어있다고 한다. 
반면 클래스 인스턴스와 같은 참조 타입 변수들은 항상 참조 copy가 일어나기 때문에 두 언어를 사용할 때 차이점에 신경쓰지 않아도 된다.

## Swift Closure의 capture

클로저에서 변수를 capture할 때는 명시적인 capture list를 작성하지 않으면 value type 변수임에도 불구하고 reference capturerk dlfdjsksek.

```swift
var anInteger = 42
let testClosure = {
    // anInteger는 capture되는 순간 reference copy됨
    print("Integer is: \(anInteger)")
}
anInteger = 84
testClosure()
// Prints "Integer is 84"
```

따라서 reference copy 대신 value copy를 하기 위해서는 capture list를 만들어서 변수를 명시해야 한다. value capture된 값은 const
value type 형태로 capture가 되고 closure안에서 변경이 불가능해진다.

```swift
var anInteger = 42
let testClosure = { [anInteger] in
    // anInteger는 capture되는 순간 value copy됨
    print("Integer is: \(anInteger)")
}
anInteger = 84
testClosure()
// Prints "Integer is 42"
```

## Objective-C의 capture

Obj-C의 block일 경우 value type이 capture될 때 기본동작이 value copy다.

```objective-c
int anInteger = 42;
void (^testBlock)(void) = ^{
     // anInteger는 capture되는 순간 value copy됨
    NSLog(@"Integer is: %i", anInteger);
};
anInteger = 84;
testBlock();
// Prints "Integer is: 42"
```

이를 reference copy로  변경하려면 `__block` 키워드를 capture할 변수 선언시에 명시해주면 된다.

```objective-c
__block int anInteger = 42;
void (^testBlock)(void) = ^{
     // anInteger는 caputure되는 순간 reference copy됨
    NSLog(@"Integer is: %i", anInteger);
};
anInteger = 84;
testBlock();
// Prints "Integer is: 84"
```


출처

* https://www.letmecompile.com/swift-closure-vs-objective-c-block/
* https://stackoverflow.com/questions/26374792/difference-between-block-objective-c-and-closure-swift-in-ios
