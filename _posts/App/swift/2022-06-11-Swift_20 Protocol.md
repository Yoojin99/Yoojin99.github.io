---  
layout: post  
title: "Swift 정리 - 20. Protocol"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  

[Swift.org:Inheritance](https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html)

Protocol은 **특정 기능을 할 수 있는 메서드, 프로퍼티, 요구사항의 청사진을 정의한다.** 이 요구사항을 실제로 구현하기 위해 클래스, 구조체, 열거형이 프로토콜을 *채택*할 수 있다. 프로토콜의 요구사항을 만족하는 타입은 프로토콜을 *준수*한다고 한다.

프로토콜을 준수하는 클래스가 구현해야 하는 요구사항을 명시하는 것에 더해 **프로토콜을 확장해서 이런 요구사항이나 추가 기능을 구현할 수 있다.**

# Protocol 문법

```swift
protocol SomeProtocol {
    // protocol definition goes here
}
```

프로토콜을 채택하기 위해서는 특정 타입 뒤에 콜론을 붙이고 프로토콜의 이름을 적는다. 여러 프로토콜을 채택할 수 있다.

```swift
struct SomeStructure: FirstProtocol, AnotherProtocol {
    // structure definition goes here
}
```

클래스에게 부모 클래스가 있다면 부모클래스 이름을 프로토콜 전에 써줘야 한다.

```swift
class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
}
```

# 프로퍼티 요구사항

프로토콜은 conforming 타입이 인스턴스 프로퍼티나 타입 프로퍼티를 구현하게 요구할 수 있다. 저장/연산 프로퍼티인지는 명시하지 않고, 이름과 타입만을 따르도록 요구한다. 또한 각 프로퍼티가 gettable/gettable and settable인지를 명시한다.

프로토콜에서 프로퍼티가 gettable&settable 하다고 명시했다면 프로토콜을 채택한 타입에서 해당 프로퍼티를 상수 저장 프로퍼티나 읽기 전용 연산 프로퍼티로 구현할 수 없다. 만약 프로토콜이 프로퍼티를 gettable하다고 명시했다면 채택하는 타입에서 어떤 타입으로 구현해도 상관 없다.
즉 프로토콜에서 명시한 기능을 구현하면 프로토콜을 잘 준수한 것이다.

프로토콜에서 프로퍼티는 항상 `var` 키워드로 명시한다. Gettable/settalbe인지는 프로퍼티 정의 뒤에 표시한다.

```swift
protocol SomeProtocol {
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
}

protocol AnotherProtocol {
    // 타입 프로퍼티를 정의할 때는 항상 `static` 키워드를 붙여야 한다. 클래스에서 이를 구현할 때 class/static인지는 상관없다.
    static var someTypeProperty: Int { get set }
}
```

# Method 요구사항

프로토콜은 프로토콜을 채택하는 클래스에서 구현할 인스턴스, 타입 메서드를 명시할 수 있다. 프로토콜에서 메서드를 정의할 때는 중괄호와 메서드 body를 쓰지 않는다. Variadic 파라미터를 사용할 수는 없지만 기본 값은 정의할 수 없다.

타입 메서드의 경우에는 `static` 키워드를 붙여야 한다.

```swift
protocol SomeProtocol {
    static func someTypeMethod()
}

protocol RandomNumberGenerator {
    func random() -> Double
}

class LinearCongruentialGenerator: RandomNumberGenerator {
    var lastRandom = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0
    func random() -> Double {
        lastRandom = ((lastRandom * a + c)
            .truncatingRemainder(dividingBy:m))
        return lastRandom / m
    }
}
let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.3746499199817101"
print("And another one: \(generator.random())")
// Prints "And another one: 0.729023776863283"
```

# Mutating Method 요구사항

인스턴스를 수정하는 메서드를 구현해야할 때가 있다. 메서드가 값 타입(구조체, 열거형인 경우)이면 메서드의 `func`키워드 전에 `mutating` 키워드를 붙여서 인스턴스나 인스턴스의 프로퍼티를 수정할 수 있음을 명시한다.

프로토콜에 이런 메서드를 정의할 때도 `mutating` 키워드를 붙인다.

> 만약 프로토콜에 인스턴스 메서드를 정의할 때 `mutating` 키워드를 붙였는데 클래스에서 이 프로토콜을 채택했다면 클래스에서 구현할 때는 `mutating`을 붙이지 않아도 된다. Mutating 키워드는 값타입에서만 사용된다.

```swift
protocol Togglable {
    mutating func toggle()
}

enum OnOffSwitch: Togglable {
    case off, on
    mutating func toggle() {
        switch self {
        case .off:
            self = .on
        case .on:
            self = .off
        }
    }
}
var lightSwitch = OnOffSwitch.off
lightSwitch.toggle()
// lightSwitch is now equal to .on
```

# Initializer 요구사항

프로토콜은 특정 이니셜라이저를 구현하게 요구할 수 있다. 메서드를 정의할 때와 마찬가지로 중괄호를 붙이지 않는다.

```swift
protocol SomeProtocol {
    init(someParameter: Int)
}
```

## 클래스

프로토콜에서 정의된 이니셜라이저 요구사항을 클래스에서 구현할 때 지정/편의 이니셜라이저로 구현할 수 있다. 모두 `required` modifier 를 붙여야 한다.

```swift
class SomeClass: SomeProtocol {
    required init(someParameter: Int) {
        // initializer implementation goes here
    }
}
```

`required` 키워드를 붙여 이 클래스의 자식 클래스도 이니셜라이저를 구현하게 해준다.

> `final` 클래스는 자식 클래스를 가질 수 없기 때문에 `required` 키워드를 붙이지 않아도 된다.

만약 자식 클래스에서 부모 클래스의 지정 이니셜라이저를 오버라이딩하면서 프로토콜에 있는 이니셜라이저를 구현할 때 `required`, `override` 키워드를 동시에 붙인다.

```swift
protocol SomeProtocol {
    init()
}

class SomeSuperClass {
    init() {
        // initializer implementation goes here
    }
}

class SomeSubClass: SomeSuperClass, SomeProtocol {
    // "required" from SomeProtocol conformance; "override" from SomeSuperClass
    required override init() {
        // initializer implementation goes here
    }
}
```

## Failable Initializer 요구사항

프로토콜은 실패 가능한 이니셜라이저를 정의할 수 있다. 프로토콜에서 정의한 실패가능한 이니셜라이저를 실패하지 않는 이니셜라이저로 구현할 수도 있다.

# 타입으로서의 프로토콜

프로토콜은 스스로 기능을 정의하지 않는다. 대신 코드에서 타입으로서 프로토콜을 사용할 수 있다. 프로토콜을 타입으로 사용하는 것을 *existential type*이라고도 하는데, "프로토콜을 준수하는 T라는 타입이 exist"하다라는 말에서 왔다.

프로토콜을 타입으로 사용할 수 있는 곳은 다음과 같다.

* 함수, 메서드, 이니셜라이저의 인자/리턴 타입
* 상수, 변수, 프로퍼티 타입
* 배열, 딕셔너리, 다른 컨테이너의 아이템의 타입

```swift
class Dice {
    let sides: Int
    let generator: RandomNumberGenerator
    init(sides: Int, generator: RandomNumberGenerator) {
        self.sides = sides
        self.generator = generator
    }
    func roll() -> Int {
        return Int(generator.random() * Double(sides)) + 1
    }
}

var d6 = Dice(sides: 6, generator: LinearCongruentialGenerator())
for _ in 1...5 {
    print("Random dice roll is \(d6.roll())")
}
// Random dice roll is 3
// Random dice roll is 5
// Random dice roll is 4
// Random dice roll is 5
// Random dice roll is 4
```

# Delegation

**Delegation은 클래스나 구조체가 자신의 책임을 다른 타입의 인스턴스에게 위임/떠넘기는 것을 가능하게 하는 디자인 패턴이다.** 다른 인스턴스에게 위임할 책임을 캡슐화한 프로토콜을 정의해서 이 디자인 패턴을 구현할 수 있다. 위임을 통해 특정 소스의 구체적인 타입을 알지 않고도 특정 행동을 할 수 있게 할 수 있다.

```swift
protocol DiceGame {
    var dice: Dice { get }
    func play()
}
protocol DiceGameDelegate: AnyObject {
    func gameDidStart(_ game: DiceGame)
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int)
    func gameDidEnd(_ game: DiceGame)
}

class SnakesAndLadders: DiceGame {
    let finalSquare = 25
    let dice = Dice(sides: 6, generator: LinearCongruentialGenerator())
    var square = 0
    var board: [Int]
    init() {
        board = Array(repeating: 0, count: finalSquare + 1)
        board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
        board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
    }
    
    // 이 delegate에 특정 동작을 대신 실행하게 한다.
    weak var delegate: DiceGameDelegate?
    
    func play() {
        square = 0
        delegate?.gameDidStart(self)
        gameLoop: while square != finalSquare {
            let diceRoll = dice.roll()
            delegate?.game(self, didStartNewTurnWithDiceRoll: diceRoll)
            switch square + diceRoll {
            case finalSquare:
                break gameLoop
            case let newSquare where newSquare > finalSquare:
                continue gameLoop
            default:
                square += diceRoll
                square += board[square]
            }
        }
        delegate?.gameDidEnd(self)
    }
}

class DiceGameTracker: DiceGameDelegate {
    var numberOfTurns = 0
    func gameDidStart(_ game: DiceGame) {
        numberOfTurns = 0
        if game is SnakesAndLadders {
            print("Started a new game of Snakes and Ladders")
        }
        print("The game is using a \(game.dice.sides)-sided dice")
    }
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int) {
        numberOfTurns += 1
        print("Rolled a \(diceRoll)")
    }
    func gameDidEnd(_ game: DiceGame) {
        print("The game lasted for \(numberOfTurns) turns")
    }
}

let tracker = DiceGameTracker()
let game = SnakesAndLadders()
// game은 tracker가 DiceGameTracker타입이라는 것을 알 필요가 없다
game.delegate = tracker
game.play()
// Started a new game of Snakes and Ladders
// The game is using a 6-sided dice
// Rolled a 3
// Rolled a 5
// Rolled a 4
// Rolled a 5
// The game lasted for 4 turns
```

# Extension으로 Protocol Conformance 추가하기

특정 타입의 소스 코드에 접근할 수 없어도 해당 타입이 새로운 프로토콜을 채택하고 준수하게 타입을 확장할 수 있다. 

```swift
protocol TextRepresentable {
    var textualDescription: String { get }
}

// extension으로 프로토콜을 채택하고 준수하게 했다.
extension Dice: TextRepresentable {
    var textualDescription: String {
        return "A \(sides)-sided dice"
    }
}
```

## 조건에 부합할 때 프로토콜 준수하게 하기

제네릭 타입은 특정 조건 하에서만 프로토콜의 요구사항을 만족하게 할 수 있다. 타입을 확장해서 제네릭 타입이 프로토콜을 준수하게 할 때 제약들을 쓰면 된다.

```swift
extension Array: TextRepresentable where Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
let myDice = [d6, d12]
print(myDice.textualDescription)
// Prints "[A 6-sided dice, A 12-sided dice]"
```

## Extension으로 프로토콜 채택 정의하기

타입이 이미 프로토콜의 요구사항을 충족하고 있는데 프로토콜을 채택하고 있지 않을 때 빈 extension 문으로 프로토콜을 채택하게 할 수 있다. 단순히 프로토콜의 요구사항을 충족한다고 프로토콜을 채택하고 있는 것은 아니기 때문에 명시적으로 프로토콜을 채택하고 있다고 명시해야 한다.

```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}
extension Hamster: TextRepresentable {}

let simonTheHamster = Hamster(name: "Simon")
let somethingTextRepresentable: TextRepresentable = simonTheHamster
print(somethingTextRepresentable.textualDescription)
// Prints "A hamster named Simon"
```

## Synthesized Implementation

Swift는 여러 케이스에서 자동으로 `Equatable`, `Hashable`, `Comparable` 프로토콜을 준수하는 구현을 제공한다. 이를 통해 프로토콜을 준수하기 위해 코드를 작성하지 않아도 된다.

### Equatable

* `Equatable`을 구현한 구조체가 저장 프로퍼티만 갖고 있을 때
* `Equatable`을 구현한 열거형이 연관 값만 가질 때
* 연관 값을 가지지 않는 열거형

일 경우 `Equatable`의 synthesized implementation을 제공한다.

```swift
struct Vector3D: Equatable {
    var x = 0.0, y = 0.0, z = 0.0
}

let twoThreeFour = Vector3D(x: 2.0, y: 3.0, z: 4.0)
let anotherTwoThreeFour = Vector3D(x: 2.0, y: 3.0, z: 4.0)
if twoThreeFour == anotherTwoThreeFour {
    print("These two vectors are also equivalent.")
}
// Prints "These two vectors are also equivalent."
```

### Hashable

* `Hashable`을 구현한 구조체가 저장 프로퍼티만 갖고 있을 때
* `Hashable`을 구현한 열거형이 연관 값만 가질 때
* 연관 값을 가지지 않는 열거형

일 경우 `Hashable`의 synthesized implementation을 제공한다.

### Comparable

원시 값을 갖지 않는 열거형일 경우 `Comparable`의 synthesized implementation을 제공한다. 만약 열거형에 연관 타입이 있다면 모두 `Comparable` 프로토콜을 준수해야 한다.

```swift
enum SkillLevel: Comparable {
    case beginner
    case intermediate
    case expert(stars: Int)
}
var levels = [SkillLevel.intermediate, SkillLevel.beginner,
              SkillLevel.expert(stars: 5), SkillLevel.expert(stars: 3)]
for level in levels.sorted() {
    print(level)
}
// Prints "beginner"
// Prints "intermediate"
// Prints "expert(stars: 3)"
// Prints "expert(stars: 5)"
```

# Protocol Inheritance

프로토콜은 하나 이상의 다른 프로토콜을 상속해서 요구사항을 추가할 수 있다.

```swift
protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
    // protocol definition goes here
}
```

# Class-Only Protocols

프로토콜이 클래스 타입에서만 채택되게 할 수 있다. `AnyObject` 프로토콜을 채택하게 하면 된다.

```swift
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
```

# Protocol Composition

여러 프로토콜을 한 번에 준수하게 할 수 있다. 프로토콜 조합은 새로운 프로토콜 타입을 만들지 않는다.

`SomeProtocol & AnotherProtocol`과 같이 작성하고, 여러 개의 프로토콜을 나열 할 수 있다. 여기에 하나의 클래스 타입을 추가할 수도 있다.

```swift
protocol Named {
    var name: String { get }
}
protocol Aged {
    var age: Int { get }
}
struct Person: Named, Aged {
    var name: String
    var age: Int
}
func wishHappyBirthday(to celebrator: Named & Aged) {
    print("Happy birthday, \(celebrator.name), you're \(celebrator.age)!")
}
let birthdayPerson = Person(name: "Malcolm", age: 21)
wishHappyBirthday(to: birthdayPerson)
// Prints "Happy birthday, Malcolm, you're 21!"
```

# Protocol Conformance ghkrdls

`is`, `as` operator를 사용해서 프로토콜을 준수하는지 확인하고, 특정 프로토콜로 캐스팅할 수 있다.

* `is` : 인스턴스가 프로토콜을 준수하면 `true`, 아닐 경우 `false` 리턴
* `as?` : 프로토콜 타입의 옵셔널 값을 리턴. 프로토콜을 준수하지 않을 경우 `nil`
* `as!` : 프로토콜 타입으로의 다운캐스팅 후 forced-unwrapping. 실패할 경우 런타임 에러

```swift
protocol HasArea {
    var area: Double { get }
}

class Circle: HasArea {
    let pi = 3.1415927
    var radius: Double
    var area: Double { return pi * radius * radius }
    init(radius: Double) { self.radius = radius }
}

class Country: HasArea {
    var area: Double
    init(area: Double) { self.area = area }
}

class Animal {
    var legs: Int
    init(legs: Int) { self.legs = legs }
}

let objects: [AnyObject] = [
    Circle(radius: 2.0),
    Country(area: 243_610),
    Animal(legs: 4)
]

for object in objects {
    if let objectWithArea = object as? HasArea {
        print("Area is \(objectWithArea.area)")
    } else {
        print("Something that doesn't have an area")
    }
}
// Area is 12.5663708
// Area is 243610.0
// Something that doesn't have an area
```

# Optional Protocol 요구사항

프로토콜에 **optional requirement**를 정의할 수 있다. 이 요구사항은 프로토콜을 채택하는 타입에서 구현되지 않아도 된다. 프로토콜 정의에서 `optional` 키워드를 붙인다. 이 선택적인 요구사항을 정의한 프로토콜은 `objc` 속성이 부여되어야 한다.

```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}

class Counter {
    var count = 0
    var dataSource: CounterDataSource?
    func increment() {
        // dataSource에서 `increment`를 구현했는지 알 수 없기 때문에 옵셔널 체이닝을 사용한다.
        if let amount = dataSource?.increment?(forCount: count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement {
            count += amount
        }
    }
}
```

# Protocol Extensions

 프로토콜은 메서드, 이니셜라이저, 서브스크립트, 연산 프로퍼티에 대한 구현을 제공하기 위해 확장될 수 있다. 이를 통해 각 타입에서 개별적으로 구현하지 않고 default 동작을 프로토콜에서 제공할 수 있다.
 
 ```swift
 extension RandomNumberGenerator {
    func randomBool() -> Bool {
        return random() > 0.5
    }
}

let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.3746499199817101"
print("And here's a random Boolean: \(generator.randomBool())")
// Prints "And here's a random Boolean: true"
 ```
 
 ## Default Implementation 제공하기
 
 만약 프로토콜을 채택한 타입에서 이미 default 구현이 있는 메서드 등을 따로 구현한다면 프로토콜에서 정의한 default 구현을 덮어쓰게 된다.
 
 ```swift
 extension PrettyTextRepresentable  {
    var prettyTextualDescription: String {
        return textualDescription
    }
}
 ```
 
 ## Protocol Extension에 제약 추가
 
 프로토콜 extension을 정의할 때, 프로토콜을 준수하는 타입이 특정 조건을 만족할 때 extension 내부의 메서드와 프로퍼티를 사용할 수 있게 할 수 있다.. 이를 `where` 를 사용해서 프로토콜 extension에 정의한다.
 
 아래 코드는 요소들이 `Equatable` 프로토콜을 준수하는 컬렉션에서 사용할 수 있는 메서드를 `Collection` 프로토콜 extension에 정의한 것이다.
 
 ```swift
 extension Collection where Element: Equatable {
    func allEqual() -> Bool {
        for element in self {
            if element != self.first {
                return false
            }
        }
        return true
    }
}
 ```
