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
  
`Stack`이라는 제네릭 컬렉션 타입을 정의할 것이다. 스택은 push, pop이라는 동작을 정의하고 있다.
    
먼저 제네릭하지 않은 버전의 스택을 구현하는 방법은 다음과 같다.
    
```swift
struct IntStack {
    var items: [Int] = []
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
}
```

이를 제네릭 버전으로 구현하게 되면 다음과 같다.

```swift
struct Stack<Element> {
    var items: [Element] = []
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
```

제네릭 버전에서는 실제 타입인 `Int` 대신에 타입 파라미터 `Element`를 사용했다. `Element`는 나중에 제공될 타입의 placeholder 이름을 정의한다. 
    
```swift
var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")
stackOfStrings.push("cuatro")
// the stack now contains 4 strings

let fromTheTop = stackOfStrings.pop()
// fromTheTop is equal to "cuatro", and the stack now contains 3 strings
```

## Extending a Generic Type

제네릭 타입을 확장할 때, extension 정의에 타입 파라미터 리스트를 제공하지 않는다. 대신, 원래 타입을 정의한 곳에서 쓴 타입 파라미터 리스트를 확장한 곳에서 쓸 수 있다.

```swift
extension Stack {
    var topItem: Element? {
        return items.isEmpty ? nil : items[items.count - 1]
    }
}
```

# Type Constraints

제네릭 함수와 제네릭 타입에서 사용할 수 있는 타입에 제약을 두면 유용할 때가 있다. 타입 제약은 타입 파라미터가 특정 클래스를 상속하거나, 특정 프로토콜을 준수해야 한다는 것을 명시한다.

## Type Constraint Syntax

타입 파라미터 이름 뒤에 콜론을 붙이고, 클래스나 프로토콜 이름을 써서 타입 제약을 추가한다.

```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}
```

`T` 타입 파라미터는 `SomeClass`를 상속해야하고, `U`는 `SomeProtocol`을 준수해야 한다는 제약을 표현했다.

## Type Constraints 사용

제네릭하지 않은 함수가 있다. 배열 내에서 특정 값을 찾는 함수인데, String인 경우에만 사용할 수 있다.

```swift
func findIndex(ofString valueToFind: String, in array: [String]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

이를 제네릭 버전으로 작성하면 아래와 같이 작성할 수 있다. 하지만 이 버전은 컴파일 되지 않는다.

```swift
// compile error
func findIndex<T>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

문제는 `==` 이부분인데, Swift의 모든 타입이 `==` 연산을 통해 같은지 확인할 수 있는 것은 아니기 때문이다. `==` 연산자를 통해 같은지 확인할 수 있게 보장하는 좋은 방법은 `==` 연산을 구현하게 요구하는 프로토콜 `Equatable`을 준수하게 하는 것이다.

따라서 위 함수를 아래와 같이 다시 작성할 수 있다.

```swift
func findIndex<T: Equatable>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

`T: Equatable`은 어떤 타입 `T`는 `Equatable` 프로토콜을 준수해야 한다라는 조건을 말한다. 이제 String이 아닌 다른 타입에도 이 함수를 쓸 수 있다.

```swift
let doubleIndex = findIndex(of: 9.3, in: [3.14159, 0.1, 0.25])
// doubleIndex is an optional Int with no value, because 9.3 isn't in the array
let stringIndex = findIndex(of: "Andrea", in: ["Mike", "Malcolm", "Andrea"])
// stringIndex is an optional Int containing a value of 2
```

# Associated Types

프로토콜을 정의할 때, 하나 이상의 연관 타입을 정의하는 게 유용할 때가 있다. 연관 타입은 프로토콜에서 사용되는 타입의 placeholder 이름을 제공한다. 연관 타입의 실제 타입은 프로토콜이 채택될때까지 명시되지 않는다. 연관 타입은 `associatedtype` 키워드를 붙여서 명시한다.

## Associated Types 사용

연관 타입 `Item`을 가진 프로토콜 `Container`가 있다.

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

이 프로토콜에서는 어떻게 아이템을 저장할지나 어떤 타입을 사용할 수 있는지 명시하지 않는다. 오직 세 개의 기능과 어떤 타입이 제공되어야 한다는 점만 나타낸다.

프로토콜을 준수하는 타입은 저장할 값들의 타입을 명시해야 한다. 위 프로토콜을 채택하는 제네릭하지 않은 버전을 작성하면 아래와 같다.

```swift
struct IntStack: Container {
    // original IntStack implementation
    var items: [Int] = []

    mutating func push(_ item: Int) {
        items.append(item)
    }

    mutating func pop() -> Int {
        return items.removeLast()
    }

    // conformance to the Container protocol
    typealias Item = Int

    mutating func append(_ item: Int) {
        self.push(item)
    }

    var count: Int {
        return items.count
    }

    subscript(i: Int) -> Int {
        return items[i]
    }
}
```

다시 이를 제네릭 버전으로 바꾸면 아래와 같다.

```swift
struct Stack<Element>: Container {
    // original Stack<Element> implementation
    var items: [Element] = []

    mutating func push(_ item: Element) {
        items.append(item)
    }

    mutating func pop() -> Element {
        return items.removeLast()
    }

    // conformance to the Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }

    var count: Int {
        return items.count
    }

    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

## Extending an Existing Type to Specify an Associated Type

이미 존재하는 타입이 프로토콜을 준수하게 할 수 있다. 프로토콜에 연관 타입이 있어도 마찬가지다.

```swift
extension Array: Container {}
```

`Array`에 존재하는 `append()` 메서드와 서브스크립트를 통해 Swift가 `Item`의 타입을 유추할 수 있게 해준다.

## Adding Constraints to an Associated Type

프로토콜의 연관 타입에 타입 제약을 추가할 수도 있다.

```swift
protocol Container {
    associatedtype Item: Equatable
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

## Using a Protocol in Its Asoociated Type's Constraints

프로토콜은 내부 요구사항에서 또 사용될 수 있다.

```swift
protocol SuffixableContainer: Container {
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}
```

이 프로토콜에서 `Suffix`는 연관 타입이다. `Suffix`는 두 개의 제약을 가지고 있다.

* `SuffixableContainer` 프로토콜 준수
* `Item` 타입이 컨테이너의 `Item` 타입과 같아야 한다.

```swift
extension Stack: SuffixableContainer {
    func suffix(_ size: Int) -> Stack {
        var result = Stack()
        for index in (count-size)..<count {
            result.append(self[index])
        }
        return result
    }
    // Inferred that Suffix is Stack.
}
var stackOfInts = Stack<Int>()
stackOfInts.append(10)
stackOfInts.append(20)
stackOfInts.append(30)
let suffix = stackOfInts.suffix(2)
// suffix contains 20 and 30
```

`Stack`에서 `Suffix` 연관타입은 `Stack`이므로 `suffix` 연산은 다른 `Stack`을 리턴한다.

# Generic Where Clauses

타입 제약은 제네릭 함수, 서브스크립트, 타입과 관련된 타입 파라미터에 제약을 추가할 수 있게 해준다. 이를 generic where 문으로 정의할 수 있다. 

Generic `where` 문은 연관 타입이 특정 프로토콜을 준수해야 하거나 특정 타입 파라미터와 연관 타입이 같다는 것을 명시할 수 있게 해준다.

```swift
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.Item == C2.Item, C1.Item: Equatable {

        // Check that both containers contain the same number of items.
        if someContainer.count != anotherContainer.count {
            return false
        }

        // Check each pair of items to see if they're equivalent.
        for i in 0..<someContainer.count {
            if someContainer[i] != anotherContainer[i] {
                return false
            }
        }

        // All items match, so return true.
        return true
}
```
                                         
# Extensions with a Generic Where Clause

`where`을 extension의 일부로 사용할 수도 있다.

 ```swift
 extension Stack where Element: Equatable {
    func isTop(_ item: Element) -> Bool {
        guard let topItem = items.last else {
            return false
        }
        return topItem == item
    }
}
```
    
만약 Element가 `Equatable`하지 않은 스택에서 `isTop(_:)` 를 호출하면 컴파일 에러가 발생하게 된다.

```swift
struct NotEquatable { }
var notEquatableStack = Stack<NotEquatable>()
let notEquatableValue = NotEquatable()
notEquatableStack.push(notEquatableValue)
notEquatableStack.isTop(notEquatableValue)  // Error
```

프로토콜 extension에서도 whrere문을 사용할 수 있다.

```swift
extension Container where Item: Equatable {
    func startsWith(_ item: Item) -> Bool {
        return count >= 1 && self[0] == item
    }
}
```

# Contextual Where Clauses

구현 내부에서 제네릭 타입 제약이 없는 곳에서 정의 부분에 `where` 문을 사용할 수 있다. 

```swift
extension Container {
    func average() -> Double where Item == Int {
        var sum = 0.0
        for index in 0..<count {
            sum += Double(self[index])
        }
        return sum / Double(count)
    }
    func endsWith(_ item: Item) -> Bool where Item: Equatable {
        return count >= 1 && self[count-1] == item
    }
}
let numbers = [1260, 1200, 98, 37]
print(numbers.average())
// Prints "648.75"
print(numbers.endsWith(37))
// Prints "true"
```

위 코드를 아래와 같이 다시 쓸 수도 있다. 두 개의 extension을 생성하고 각각에 `where` 문을 붙인다.

```swift
extension Container where Item == Int {
    func average() -> Double {
        var sum = 0.0
        for index in 0..<count {
            sum += Double(self[index])
        }
        return sum / Double(count)
    }
}
extension Container where Item: Equatable {
    func endsWith(_ item: Item) -> Bool {
        return count >= 1 && self[count-1] == item
    }
}
```

# Associated Types with a Generic Where Clause

Subscript도 제네릭할 수 있고 `where`문을 포함할 수 있다.

```swift
extension Container {
    subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Iterator.Element == Int {
            var result: [Item] = []
            for index in indices {
                result.append(self[index])
            }
            return result
    }
}
```
