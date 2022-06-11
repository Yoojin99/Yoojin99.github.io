---  
layout: post  
title: "Swift 정리 - 19. 타입 캐스팅"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  

[Swift.org:Inheritance](https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html)

**타입 캐스팅은 인스턴스의 타입을 확인하거나 다른 클래스의 인스턴스로서 다루기 위한 방법이다.** Swift에서의 타입 캐스팅은 `is`, `as` operator로 구현한다.

# Type Casting을 위한 클래스 계층 정의

**클래스와 자식 클래스들의 계층에서 특정 클래스 인스턴스의 타입을 확인하거나 인스턴스를 같은 계층 내의 다른 클래스로 캐스팅하기 위해 타입 캐스팅을 사용한다.**

```swift
class MediaItem {
    var name: String
    init(name: String) {
        self.name = name
    }
}

class Movie: MediaItem {
    var director: String
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
}

class Song: MediaItem {
    var artist: String
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}

let library = [
    Movie(name: "Casablanca", director: "Michael Curtiz"),
    Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
    Movie(name: "Citizen Kane", director: "Orson Welles"),
    Song(name: "The One And Only", artist: "Chesney Hawkes"),
    Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
]
// the type of "library" is inferred to be [MediaItem]
```

위 코드에서 `library` 배열은 배열 리터럴로 정의됐다. Swift의 타입 체커는 `Movie`와 `Song`이 공통 부모 클래스 `MediaItem`을 가지고 있다는 것을 알아낸다. 만약 이 배열 내의 아이템을 하나씩 돌면 `MediatItem` 타입으로 접근된다. 
각 요소의 타입 그 자체로 접근하고 싶을 때는 타입을 확인하거나 다운 캐스팅할 수 있다.

# Checking Type

**Type check operator `is`** 를 사용해서 인스턴스가 특정 자식 클래스의 타입인지 확인한다. 만약 인스턴스가 자식 클래스 타입의 인스턴스라면 `true`를 리턴하고 아닐 시 `false` 를 리턴한다.

```swift
var movieCount = 0
var songCount = 0

for item in library {
    if item is Movie {
        movieCount += 1
    } else if item is Song {
        songCount += 1
    }
}

print("Media library contains \(movieCount) movies and \(songCount) songs")
// Prints "Media library contains 2 movies and 3 songs"
```

**Meta Type을 사용해서 타입을 확인할 수도 있다.** 메타 타입은 **타입의 타입**이다. 즉 타입 자체가 하나의 타입으로 또 표현할 수 있다는 뜻이다.

클래스, 구조체, 열거형 이름 뒤에 `.Type`을 붙이면 메타 타입을 의미한다. 프로토콜 타입의 메타 타입은 `.Protocol`을 붙인다.

**`self`를 사용해서 타입을 값처럼 표현할 수 있다.** `SomeClass.self`는 클래스의 인스턴스를 가리키는 것이 아니라 `SomeClass` 타입 자체를 값으로 표현한 값을 의미한다.

```swift
protocol SomeProtocol {}
class SomeClass {}

let someClassType: SomeClass.Type = SomeClass.self
let someProtocolType: SomeProtocol.Protocol = SomeProtocol.self
```

프로그램 실행 중에 인스턴스의 타입을 표현한 값을 확인하고 싶다면 `type(of:)` 함수를 사용한다.

```
class Top {}
class Mid: Top {}
class BottomA: Mid {}

let top = Top()
let mid = Mid()
let bottomA = BottomA()

print(type(of: mid) == Top.self) // false
print(type(of: bottomA) == Top.self) // false
print(type(of: bottomA) == Mid.self) // false
```

# Downcasting

특정 클래스 타입의 상수나 변수가 가리키는 인스턴스가 실제로는 자식 클래스의 인스턴스일 수 있다. 이때 타입 캐스트 연산자 `as?`, `as!`를 사용해서 자식 클래스 타입으로 downcast할 수 있다. 클래스 계층에서
상위에 있는 부모 클래스의 타입을 아래에 있는 자식 클래스의 타입으로 캐스팅한다 해서 Downcasting이라 하는 것이다.

* `as?` : Downcasting이 실패할 수 있기 때문에 optional 값을 리턴. Downcast가 성공할 지 확실하지 않을 때 사용한다. 실패할 경우 nil을 리턴한다.
* `as!` : Downcast 후 값을 강제로 unwrapping 함. Downcast가 성공할 것이라고 확신할 때 사용한다. 실패할 경우 런타임 에러가 발생한다.

```swift
for item in library {
    if let movie = item as? Movie {
        print("Movie: \(movie.name), dir. \(movie.director)")
    } else if let song = item as? Song {
        print("Song: \(song.name), by \(song.artist)")
    }
}

// Movie: Casablanca, dir. Michael Curtiz
// Song: Blue Suede Shoes, by Elvis Presley
// Movie: Citizen Kane, dir. Orson Welles
// Song: The One And Only, by Chesney Hawkes
// Song: Never Gonna Give You Up, by Rick Astley
```

**캐스팅은 인스턴스를 수정하거나 값을 바꾸지 않는다.** 인스턴스는 같은 상태로 남아있고, 단순히 캐스팅 된 타입의 인스턴스로서 다뤄지고 접근되는 것이다.

# Any, AnyObject Type Casting

Swift는 타입이 명확하지 않은 두 가지 특별한 타입을 제공한다.

* `Any` : 모든 타입의 인스턴스를 나타낼 수 있다. 함수 타입도 포함된다.
* `AnyObject` : 클래스 타입의 인스턴스만 나타낼 ㅅ ㅜ있다.

`Any`, `AnyObject`는 이들이 제공하는 특징과 기능이 필요할 때만 사용한다.

```swift
var things: [Any] = []

things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
things.append({ (name: String) -> String in "Hello, \(name)" })

for thing in things {
    switch thing {
    case 0 as Int:
        print("zero as an Int")
    case 0 as Double:
        print("zero as a Double")
    case let someInt as Int:
        print("an integer value of \(someInt)")
    case let someDouble as Double where someDouble > 0:
        print("a positive double value of \(someDouble)")
    case is Double:
        print("some other double value that I don't want to print")
    case let someString as String:
        print("a string value of \"\(someString)\"")
    case let (x, y) as (Double, Double):
        print("an (x, y) point at \(x), \(y)")
    case let movie as Movie:
        print("a movie called \(movie.name), dir. \(movie.director)")
    case let stringConverter as (String) -> String:
        print(stringConverter("Michael"))
    default:
        print("something else")
    }
}

// zero as an Int
// zero as a Double
// an integer value of 42
// a positive double value of 3.14159
// a string value of "hello"
// an (x, y) point at 3.0, 5.0
// a movie called Ghostbusters, dir. Ivan Reitman
// Hello, Michael
```

> Any 타입은 옵셔널 타입을 포함한 어떤 타입의 값이든 나타낼 수 있다. Swift는 만약 `Any`를 받기로 기대한 곳에 옵셔널 값을 넘기면 경고를 출력한다. 만약 옵셔널 값을 `Any` 값으로 사용하고 싶으면 옵셔널을 `as` operator를 사용해서 `Any`로 캐스팅한다.

```swift
let optionalNumber: Int? = 3
things.append(optionalNumber)        // Warning
things.append(optionalNumber as Any) // No warning
```
