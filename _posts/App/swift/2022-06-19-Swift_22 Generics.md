---  
layout: post  
title: "Swift 정리 - 22. Generics"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  

[Swift.org:Generics](https://docs.swift.org/swift-book/LanguageGuide/Generics.html)

**Generic 코드는 어떤 타입에도 유연하게 대응할 수 있는 재사용 가능한 함수, 타입을 작성할 수 있게 해준다.** 깔끔하고 추상화된 방법으로 중복을 제거하고 의도를 나타낼 수 있는 코드를 작성할 수 있다.

제네릭은 Swift의 가장 강력한 기능 중 하나로, Swift 표준 라이브러리의 많은 부분이 제네릭 코드로 작성되었다. `Array`, `Dictionary` 타입도 제네릭 컬렉션이다. 이 두 컬렉션에 저장할 수 있는 타입은 제한적이지 않다.

# Generics가 해결하는 문제

두 `Int` 값을 교체하는 제네릭하지 않은 하수가 있다.

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

**문제는 이 함수가 `Int` 값에만 사용할 수 있다는 것이다.** 만약 `String`, `Double` 타입의 값들에도 이 함수를 쓰고 싶다면 아래와 같이 **추가로 함수를 작성해야 한다.**

```swift
func swapTwoStrings(_ a: inout String, _ b: inout String) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

각 함수의 구현부를 보면 굉장히 비슷한 것을 볼 수 있다. 차이는 함수들이 인자로 받는 값들의 타입이다. 이 함수들을 어떤 타입이라도 받을 수 있는 하나의 함수로 만들면 더 좋을 것이다. 이를 제네릭 코드로 작성할 수 있다.

# Generic Functions

위에서 본 함수를 제네릭 버전으로 작성하면 아래와 같다.

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

일반적인 형태는 아래와 같다.

```swift
타입이름<타입 매개변수>
함수 이름<타입 매개변수>(함수 매개변수)
```

함수의 제네릭 버전은 *placeholder* 타입 이름(여기에서는 `T`)을 *actual* 타입 이름(`Int`, `String`, `Double`) 대신에 사용했다. Placeholder 타입 이름은 `T`가 무엇이 되어야 할지 명시하지 않지만 `a`, `b`가 모두 같은 타입 `T`여야 한다고 명시한다.
`T`를 대체할 실제 타입은 함수가 호출될 때 결정된다.

제네릭 함수와 그렇지 않은 함수의 또다른 차이점은 제네릭 함수의 이름 뒤에 placeholder 타입 이름이 괄호 안에 들어가서 붙는다는 것이다. 괄호는 `T`가 함수 정으에서 placeholder 타입 이름이라는 것을 알려준다. 

```swift
// 실제 사용

var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
// someInt is now 107, and anotherInt is now 3

var someString = "hello"
var anotherString = "world"
swapTwoValues(&someString, &anotherString)
// someString is now "world", and anotherString is now "hello"
```

# Type Parameters

위 함수에서 placeholder 타입 T`는 타입 파라미터다. 타입 파라미터는 함수 이름 뒤에 괄호를 붙이고 괄호 안에 placeholder 타입 이름을 써줘서 명시한다. (e.g. `<T>`)
  
타입 파라미터는 다음의 용도로 사용할 수 있다.
  
* 함수의 파라미터들의 타입
* 함수의 리턴 타입
* 타입 annotation
  
타입 파라미터는 실제 함수가 호출될 때 실제 타입으로 교체된다. 또한 하나 이상의 타입 파라미터를 괄호 안에 쉼표로 구분해서 여러 번 작성할 수 있다.

## Type Parameter Name
  
대부분의 경우 타입 파라미터 이름은 설명력 있는 이름을 가진다. (`Dictionary<Key, Value>`, `Array<Element>`) 이를 통해 코드를 읽는 사람이 타입 파라미터와 사용된 곳에서와의 관계를 이해할 수 있다.
  
만약 의미 있는 관계가 없다면 한 글자로 표현해도 된다. (`T`, `U` 등)
  
> 항상 타입 파라미터의 이름을 지을 때 upper camel case로 지어서 이게 타입이라는 정보를 표현해야 한다.
  
# Generic Types
  
Generic 함수에 추가로 Generic 타입도 정의할 수 있다. 이는 어떤 타입과도 동작하는 클래스, 구조체, 열거형이 될 수 있다.
  
`Stack`이라는 제네릭 컬렉션 타입을 정의할 것이다. 
