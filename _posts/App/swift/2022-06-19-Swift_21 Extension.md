---  
layout: post  
title: "Swift 정리 - 21. Extension"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  

[Swift.org:Extension](https://docs.swift.org/swift-book/LanguageGuide/Extensions.html)


**Extension은 클래스, 구조체, 열거형, 프로토콜에 새로운 기능을 추가한다.** 원래 소스 코드에 접근할 수 없을 때 타입을 확장할 수 있게 해주는 능력이 있다.(이를 retroactive modling이라 한다.)

Swift의 extension이 할 수 있는 기능은 다음과 같다.

* 인스턴스/타입 연산 프로퍼티 추가
* 인스턴스/타입 메서드 정의
* 이니셜라이저 제공
* subscript 정의
* 중첩 타입 정의
* 이미 존재하는 타입이 프로토콜을 준수하게 할 수 있다.

Swift에서는 프로토콜을 확장할 수도 있다.

> Extension은 새로운 기능을 추가할 수는 있지만 이미 존재하는 기능을 override 할 수는 없다.

## Inheritance vs Extension

||상속|익스텐션|
|확장|수직 확장|수평 확장|
|사용|클래스 타입|클래스, 구조체, 프로토콜, 제네릭 등 모든 타입|
|재정의|가능|불가|

## Extension Syntax

`extension` 키워드를 붙여 정의한다.

```swift
extension SomeType {
    // new functionality to add to SomeType goes here
}
```

Extension을 사용해 하나 이상의 프로토콜을 채택할 수 있다. Protocol conformance를 추가하려면 프로토콜 이름을 나열하면 된다.

```swift
extension SomeType: SomeProtocol, AnotherProtocol {
    // implementation of protocol requirements goes here
}
```

## Extension으로 추가할 수 있는 것들

### Computed Properties

Extension은 인스턴스/타입 연산 프로퍼티를 이미 존재하는 타입에 추가할 수 있다.

```swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}

let oneInch = 25.4.mm
print("One inch is \(oneInch) meters")
// Prints "One inch is 0.0254 meters"
let threeFeet = 3.ft
print("Three feet is \(threeFeet) meters")
// Prints "Three feet is 0.914399970739201 meters"
```

위 연산 프로퍼티들은 읽기 전용 연산프로퍼티로 `get`키워드 없이 표현된 걸 볼 수 있다. 

타입 연산 프로퍼티를 추가하려면 `static` 키워드를 사용해서 추가할 수 있다.

> Extension은 연산 프로퍼티를 추가할 수 있지만 저장 프로퍼티를 추가하거나, 이미 존재하는 프로퍼티에 프로퍼티 감시자를 붙일 수 없다.

### Initializers

Extension을 통해 새로운 이니셜러이저를 추가할 수 있다. 

**클래스에는 새로운 편의 이니셜라이저를 추가할 수 있지만 새로운 지정 이니셜라이저나 디이니셜라이저를 추가할 수 없다.** 지정 이니셜라이저와 디이니셜라이저는 항상 클래스의 원래 구현부에서 정의되어야 한다.

모든 저장 프로퍼티에 기본 값을 제공하고 아무런 커스텀 이니셜라이저가 없는 값 타입의 extension에 새로운 이니셜라이저를 추가하면 기본 이니셜라이저나 멤버 와이즈 이니셜라이저를 쓸 수 있다. 

만약 다른 모듈에서 정의된 구조체의 extension에 이니셜라이저를 추가하면 이니셜라이저 내부에서 `self`에 접근할 수 없다.

```swift
struct Size {
    var width = 0.0, height = 0.0
}

struct Point {
    var x = 0.0, y = 0.0
}

struct Rect {
    var origin = Point()
    var size = Size()
}
```

모든 구조체가 기본 값을 제공하고 있기 때문에 자동으로 기본 이니셜라이저(default initializer)와 멤버와지즈 이니셜라이저를 사용할 수 있다.

```swift
extension Rect {
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

Extension을 통해 새로운 이니셜라이저를 정의했는데, 이 이니셜라이저에서는 자동으로 생성된 멤버와이즈 이니셜라이저를 호출했다.

## Methods

Extension은 새로운 인스턴스/타입 메서드를 추가할 수 있다.

```swift
extension Int {
    func repetitions(task: () -> Void) {
        for _ in 0..<self {
            task()
        }
    }
}

3.repetitions {
    print("Hello!")
}
// Hello!
// Hello!
// Hello!
```

### Mutating Instance Methods

`self`나 프로퍼티를 바꾸는 구조체/열거형 메서드는 메서드를 `mutating`으로 표시해야 한다.

```swift
extension Int {
    mutating func square() {
        self = self * self
    }
}
var someInt = 3
someInt.square()
// someInt is now 9
```

## Subscripts

Extension은 새로운 subscript를 추가할 수 있다.

```swift
extension Int {
    subscript(digitIndex: Int) -> Int {
        var decimalBase = 1
        for _ in 0..<digitIndex {
            decimalBase *= 10
        }
        return (self / decimalBase) % 10
    }
}
746381295[0]
// returns 5
746381295[1]
// returns 9
746381295[2]
// returns 2
746381295[8]
// returns 7
```

## Nested Types

Extension은 새로운 중첩 타입을 추가할 수 있다.

```swift
extension Int {
    enum Kind {
        case negative, zero, positive
    }
    var kind: Kind {
        switch self {
        case 0:
            return .zero
        case let x where x > 0:
            return .positive
        default:
            return .negative
        }
    }
}

func printIntegerKinds(_ numbers: [Int]) {
    for number in numbers {
        switch number.kind {
        case .negative:
            print("- ", terminator: "")
        case .zero:
            print("0 ", terminator: "")
        case .positive:
            print("+ ", terminator: "")
        }
    }
    print("")
}

printIntegerKinds([3, 19, -27, 0, -6, 0, 7])
// Prints "+ + - 0 - 0 + "
```
