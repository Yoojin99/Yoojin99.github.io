---  
layout: post  
title: "Swift 개발자를 위한 KISS 원리"  
subtitle: ""  
categories: app
tags: app-swift URLSession
comments: true  
header-img: 
---  
  
> ``  

---

[원문](https://medium.com/@smalam119/the-kiss-principle-in-coding-with-swifty-examples-841993b6c653)

## KISS가 무엇인가?

KISS는 **Keep it simple, stupid**의 약자이다. 공식적으로는 **Keep It Simple and Straightforward**이다. 이 원리는
시스템은 나중에도 내부적인 것들을 이해하기 쉽게 디자인 되어야 한다는 것을 의미한다. 결과적으로, 최소한의 노력을 들여서 수정을 할 수 있게 된다.
이 원리는 Kelly Johnson이라는 공학자가 고안했다. 리드 엔지니어로서 jet aircraft를 만들때, Kelly는 디자이너들에게 공학 교육과 기본적인 도구가 있는 어떤 사람이라도
위급 상황에 이를 수리할 수 있도록 시스템을 간단하게 만들라고 가이드했다. 만약 시스템이 너무 잘 동작하지만 내부 구조를 알거나 수리하기 어렵다면, 이 디자인은
KISS 원리를 따르지 않는 것이다. 이 원리는 시스템은 디자인하는 분야들에 관련이 있고 유용하다. 

## 이 글이 작성된 이유

주로 소프트 엔지니어들은 코드를 작성하고, 유닛/UI 테스트를 작성하고, 구조를 결정하고 그 외에도 다른 많은 것들을 한다. 코딩이 소프트 엔지니어링에서 가장 근본적인 것이기 때문에
이 글도 여기에 초점을 맞출 것이다. 이해하기 어렵고 긴 코드를 같은 동작을 하게끔 하면서 더 간단하고 간결한 코드로 바꿀 수 있는지 알아보려고 한다. 이렇게 함으로써
KISS 원리를 따르도록 코드를 바꾸는 법을 알아볼 것이다. 이 원리는 5가지의 다른 상황에서의 코드를 볼 것이다. 

## KISS의 동기

SOLID, DRY, YAGNI, KISS 등과 같은 소프트웨어 개발 원리는 쉽게 읽고, 확장되고, 유지될 수 있는 좋은 코드를 작성하기 위한 목적을 위해 만들어졌다. KISS는
특히 명시적으로 간단한 코드를 작성하는 것을 촉구한다. 이렇게 하면 수많은 장점이 있다.

* 협업을 쉽게 만든다: 코드를 작성하는 것은 일이지만, 좋은 코드를 작성하는 것은 책임감의 문제다. 만약 누군가가 간단한 코드를 작성하면, 동료 코더들은 그것을 쉽게 읽고 이해할 수 있다. 이를 통해 협업이 쉬워진다.
* 자기애 상승: 개발자들은 자주 버그를 수정하거나 새로운 기능 추가 등을 위해 그들이 작성한 코드를 다시 봐야 한다. 그들이 만약 코드를 충동적이고 어렵게 썼다면 자신의 코드를 이해하는데 어려움을 겪을 것이다. 그래서 세분화 되고 직관적인 코드를 작성하면 스스로를 대견하게 여길 수 있을 것이다.
* 더 좋은 코드: 간단한 코드는 꼭 좋은 코드가 아니지만, 좋은 코드의 특징이 될 수 있다. 만약 간결하게 만들 수 있다면, 더 좋은 코드를 만들기 위한 한 단계를 더 나간 것이다. Occam's Razor에 따르면, "한 이론을 설명하는 가장 간단한 이론이 더 복잡한 설명보다 정확할 가능성이 높다." 이를 염두했을 때, 코드를 작성하기 위한 여러 가지 방법이 있다면 가장 간단한 방법이 가장 정확할 것이다. Occam's Razor 같은 철학과 문학과 깊게 관련있는 것을 프로그래밍과 연결하는 것이 이상하게 느껴질 수도 있지만, 이미 이 사이의 간극을 줄이는 연구들이 있었다. [1](https://shekhargulati.com/2019/05/30/mental-models-for-software-engineers-occams-razor/)[2](https://naveenkumarmuguda.medium.com/occams-razor-in-software-development-56ee3e8b8ce8)
* 에러가 덜 날 코드: 가끔 개발자들은 먼저 생각하지 않고 불필요한 추상화를 추가하는 경향이 있다. 아마 코드를 재사용성 있고 모듈화하기 위해서 일 것이다. 이를 위해 추가적인 코드가 덧붙여진다. 이 코드의 목적은 단지 추상화를 수용하기 위함이다. 하지만 실제 비즈니스 로직을 가진 코드는 흐릿한 곳 깊이 있을 것이다. 며칠 후 새로운 요구사항이 왔고, 기존의 추상화된 것을 대체하지 않는 이상 동일하게 작동할 수 없다고 해보자. 이 대체하는 것은 요구사항에 적용가능한 추상화를 적용한 새로운 코드를 생성하고 기존의 요구사항이 깨지지 않도록 보장한다. 이런 코드들은 주로 조건문들인 경우가 많으며, 만약 잘 다뤄지지 않았을 때 버그가 있는 상황을 포함할 수도 있다. 간단한 코드는 추상화의 반댓말이 아니다. 그러나, 추상화는 목적이 있어야 하고 어디에나 적용이 가능해야 한다. 이렇게 함으로써 에러에 취약하게 되지 않을 것이다.

## "말은 쉬운데, 그럼 간단한 코드를 보여줘라"

이제 몇 접근법을 버고 간단한 코드가 가동성과 확장성을 향상시킬 수 있는지를 볼 것이다.

### Shorthands

코드를 덜 복잡하게 만드는 방법 중 하나는 가능한 곳에 줄일 수 있는 표현을 쓰는 것이다. 그 중 하나는 삼항 연산자(?:)가 될 것이다. 굉장히 고급스럽지만, 종종 무시된다. 아래의 코드를 보자.

```swift
if isSuccess {
    label.textColor = .black
} else {
    label.textColor = .red
}
```
위의 코드에서, boolean 값의 응답 플래그가 백엔드 서버에서 받아져서 `isSuccess` 변수에 저장되었다고 해보자. 값을 토대로, 라벨의 텍스트 색깔이 업데이트 되고, 사용자에게 메세지를 보여줄 것이다. 여기에서 **if-else**문이 사용되었다.

이 문장은 아래와 같이 삼항 연산자를 사용해서 작성될 수 있고, 동일한 작업을 하도록 할 수 있다.

```swift
label.textColor = isSucess ? .black : .red
```

두 번째 코드가 더 간결하고 읽기 쉽다. 4줄을 한 줄로 줄인 것이다. 적은 수의 코드가 더 좋은 코드임에는 논란의 여지가 있지만 이 경우에서는 if-else 문을 사용하는 것보다 좋다. 여기에서는 읽기 어려운 코드로 이어질 수 있는 삼항 연산자들은 중첩해서 사용하면 안된다.

Nil coalescing, Shorthand parameter syntax, Add then assign to(+=)와 같은 shorthand들도 있다.

### 적당한 것으로 대체하기

KISS 원리를 따를 때 기억해야 할 것은 무엇을 하기위해 필요 이상의 것을 하지 않는 것이다. 예를 들어 메서드를 정의할 때, 누군가는 연산 프로퍼티를 이용하는 것에 대해 먼저 생각할 수가 있다. 만약 연산 프로퍼티로 충분하다면 메서드를 사용할 필요가 없다. 
아래의 코드에서는 5년 후의 사람의 나이를 리턴하는 `getAgeAfterFiveYears()`라는 메서드가 있다.

```swift
struct Person {
  let name: String
  let age: Int
  var gender: GenderType?
  
  func getAgeAfterFiveYears() -> Int { age + 5 }
}
```

이 메서드는 연산 프로퍼티로 대체될 수 있다. 연산 프로퍼티는 값을 저장하지 않는 특별한 종류의 프로퍼티이다. 대신에, 이것들은 다른 프로퍼티의 값을 연산해서 이를 리턴한다.

```swift
var ageAfterFiveYears: Int { age + 5 }
```

이제 메서드와 연산 프로퍼티는 같은 값을 리턴한다. 하지만 호출자에게는 연산 프로퍼티는 메서드가 아닌 단지 다른 프로퍼티일 뿐이다.

```swift
let person = Person(name: "Yoojin", age: 99)
person.getAgeAfterFiveYears()
person.ageAfterFiveYears
```

만약 메서드가 굉장히 간단해서 예를 들어 아무 인자를 받지 않고 많은 작업을 하지 않고 메서드같지도 안핟면, 이는 연산 프로퍼티로 만들기 좋은 후보가 된다. 여기서 연산 프로퍼티는 적절하며,
KISS 원리를 따를 뿐만 아니라 흔히 Swifty 하게 코드를 작성했다고 할 수 있다. 다르게 작성해도 충분한 방법들을 찾아보는 것은 항상 옳다.

### Syntactic Sugar

![image](https://user-images.githubusercontent.com/41438361/122697262-ed5e6c80-d27f-11eb-8507-32686c80aa29.png)

Syntactic Sugar란 컴퓨터 공학에서 무언가를 더 읽거나 표현하기 쉽게 하는 문법을 의미한다. 
프로그래밍 언어는 개발자가 장황한 코드를 작성하는 것을 피할 수 있게 syntactic sugar를 이미 제공하고 있다. 위에서 정의한 Person 구조체를 보도록 하자.
Person 객체들을 담는 배열이 있다고 해보자. 이제 "chole"라는 이름을 가진 사람이 배열에서 몇 번째 인덱스인지를 찾을 것이다. 아래는 그 방법 중 하나이다.

```swift
var index: Int?

for i in 0..<people.count {
  if people[i].name == "chole" {
    index = i
    break
  }
}

print(index)
```

이를 위한 다른 방법은 Swift 언어 자체에서 제공하는 배열 collection 타입의 `firstIndex(where:)` 메서드를 사용하는 것이다.

```swift
let index = people.firstIndex { $0.name == "chole" }
print(index)
```

Trailing closure, property wrapper, if-let과 같이 Swift에서 제공하는 Syntactic Sugar들이 있다.

*참고로 firstIndex(where:) 메서드로 대체했다고 시간 복잡도가 줄어들지는 않는다.*

### Higher-order Functions

Higher-order 함수들은 하나 이상의 함수를 인자나 리턴 값으로 받는 함수들이다. 이것들은 **Functional Programming, FP**에 속하는 개념이다. 어떤 언어들은 다른 언어들로부터 빌려와서 호환이 되는 요소들이 있어 공생가능하다.
Swift도 콜렉션 타입이 higher-order 함수들을 지원하기 때문에 그 중 하나이다. **Sorted, Map, FlatMap, Reduce** 등이 있다. 이런 고차원 함수들은 코드ㅇ를 간결하고 간단하게 만들 수 있다.
위에서 언급했던 사람들 배열에서 이름 배열이 나와야 한다고 가정해보자. 먼저 아래의 방법으로 구현하는 것을 생각해 볼 수 있을 것이다.

```swift
var names = [String]()

for person in people {
  names.append(person.name)
}

print(names)
```

FP 다운 방법으로 하자면 아래와 같다.

```swift
var names = people.map{ $0.name }
print(names)
```

이건 syntactic sugar 처럼 보일 수 있지만, 완전히 다른 것이다. 고차원 함수들은 실제 상황에서 더 유연하게 사용될 수 있다. 예를 들어, Map은 콜렉션의 각 요소마다
동일한 연산을 한다. 이 연산은 클로저를 인자로 전달해 제공한다.

### 통일성

가끔 확장 가능한 코드를 위해 간결함을 희생하는 것이 더 좋을 때도 있다. 코드에 더 긍정적으로 작용하면서 대체가능한 변화를 만들기 위한 기회비용이라고 생각하면 된다. 아래의 예제를 보자.

```swift
func showTower(title: String, description: String, levels: Int) {
  print(title)
  print(description)
  print(levels)
}

let difficulty = "novice"

if difficulty == "novice" {
    showTower(title: "Novice", description: "All Too Easy!", levels: 8)
} else if difficulty == "warrior" {
    showTower(title: "Warrior", description: "You Will Die Mortal!", levels: 10)
} else if difficulty == "master" {
    showTower(title: "Master", description: "You Will Never Win!", levels: 12)
} else {
    showTower(title: "Unknown", description: "Unknown!", levels: -1)
}
```

위의 코드에서는 `difficulty`라는 상수의 값에 따라 타워에 대한 정보를 출력하고 있다. 여기에서 분명히 if-else 문은 간결해보이지않고, 문장 일치 비교가 안전하지도 않다. 또한 
새로운 difficulty 값이 들어왔을 때 컴파일러가 알아서 새로운 else-if 문을 추가하지도 않으므로 미래 지향적이지도 않다. 개발자가 이를 직접 염두해두고 있어야 한다.

다행히, **raw 값을 갖는 Enum, Typealias, Tuple**, 그리고 연산 프로퍼티로 위의 문제를 해결할 수 있다.

```swift
enum Difficulty: String {
  typealias DifficultyContent = (title: String, description: String, levels: Int)
  
  case novice
  case warrior
  case master
  case unknown
  
  var content: DifficultyContent {
    switch self {
    case .novice:
        return (title: "Novice", description: "All Too Easy!", levels: 8)
    case .warrior:
        return (title: "Warrior", description: "You Will Die Mortal!", levels: 10)
    case .master:
        return (title: "Master", description: "You Will Never Win!", levels: 12)
    case .unknown:
        return (title: "Unknown", description: "Unknown!", levels: -1)
    }
  }
}

func showTower(for difficulty: Difficulty) {
    print(difficulty.content.title)
    print(difficulty.content.description)
    print(difficulty.content.levels)
}

let difficulty = Difficulty(rawValue: "novice")

showTower(for: difficulty ?? .unknown)
```

이제 어떤 종류의 difficulty 타입이 새로 나온다고 해도, Difficulty enum에 추가하면 된다. 컴파일러는 switch 문에서 이 case가 커버될 때까지 에러를 던질 것이고,
코드는 실행되지 않을 것이다. 그래서, 코더는 실수를 남기지 않을 것이다.

## 마지막으로

Swift 개발자로서, KISS를 다음과 같이 제안한다. "Keep it simple and Swifty" :)



ddddddssssss
