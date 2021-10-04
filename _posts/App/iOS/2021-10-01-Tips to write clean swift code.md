---  
layout: post  
title: "[iOS] - 5 Tips to Write Clean Swift Code"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: 

---  
  
> `Clean한 swift code르 작성하는 팁`  

---

[https://betterprogramming.pub/5-tips-to-write-clean-swift-code-2ef287a11500](https://betterprogramming.pub/5-tips-to-write-clean-swift-code-2ef287a11500)


## 1. Positive Guard 사용하기

`guard`문을 사용하는 이유는 

struct KeyValueStorage<T: Codable>: Storage {
  var memory: [String: T] = [:]
  func get(key: String) -> T? {
    return self.memory[key]
  }

  mutating func set(value: T, for key: String) {
    self.memory[key] = value
  }
}

1. Optional을 unwrapping하기 위해
2. 선제 조건이 있을 때

사용한다.

예를 들어 코드를 아래와 같이 작성했다고 해보자. `isEmpty`에 `!`를 붙여 부정임을 표시했다.

```swift
guard !pieces.isEmpty else {
    return 
}
```

이런 코드는 두 가지 문제가 있다.

1. 코드 작성 시 `!`를 빼먹을 실수를 할 수 있다.
2. 딱 보고 바로 어떤 의미인지 알기 어렵다. 물론 위의 예제는 간단해서 바로 이해하기 쉽지만, `특정조건`앞에 `!`를 붙였을 때 바로 이해하기 어려울 떄가 있다.

이를 해결하기 위해서 extension을 만들 수도 있다.

```swift
extension Collection {
  var isNotEmpty: Bool {
    return !self.isEmpty
  }
}

// ...
guard pieces.isNotEmpty else {
  return
}
```

이 방식을 이용하면 코드를 더 가독성 있게 만들 수 있다.

## Leverage Type Inference

Swift는 타입 추론을 할 수 있고, 컴파일러가 이를 해주기 때문에 우리는 코드를 작성할 떄 모든 타입을 칠 필요가 없다. 이런 특징은 사용자가 임의로 정의한 타입, 특히 enum을 사용할 때 빛난다. 예를들어 `Result<Int, Myerror>` 타입을 리턴하는 함수가 있다고 해보자.

```swift
enum CustomModule {
    enum Error: Swift.Error {
        case oddValue
    }
    
    static func doubleEven(value: Int) -> Result<Int, Error> {
        guard value.isMultiple(of: 2) else {
            return .failure(.oddValue) // 8
        }
        return .success(value * 2) // 10
    }
}

let res = CustomModule.doubleEven(value: 4)
switch res { // 16
case .success(let value):
    print(value)
case .failure(let error):
    print(error)
```

이 코드에서는 여러 레벨의 참조를 패키징하고 있다.

* 8줄: 컴파일러는 리턴 타입(`Result.failure`)와 `Error`(`CustomModule.Error`)를 모두 참조하고 있다.
* 10줄: 컴파일러는 리턴 타입의 `success`를 추론할 것이다.
* 16줄: switch에서 우리는 두 경우의 타입을 언급하고 있다.

컴파일러는 system type을 마스킹할 때에도 타입을 올바르게 추론할 수 있다. 두 번째 줄의 `Error`도 `Swift.Error` 타입과 같은 이름을 가지고 있다. 하지만 specifiicity 규칙 때문에, `doubleEven` 함수는 자동으로 에러의 올바른 타입을 가리킨다.

이런 매커니즘은 불필요한 코드 작성 없이 의미있고 가동성 있는 코드를 작성하는 것을 가능하게 한다.

## Protocol Witnesses

Protocol witnesses는 프로토콜을 대신할 흥미로운 대체제다. 이들은 표준 protocol-oriented programming(POP)를 value-oriented 접근 방법으로 대체한다.

이를 통해 얻을 수 있는 장점이 있는데,

1. 프로토콜을 구현하기 위해 요구되는 struct의 수를 감소시킨다.
2. 제네릭을 단순화한다.
3. Protocols with Associated Types(PAT)를 단순화한다.

예를 들어 `Codable` 데이터 타입을 위한 매니저를 구현해야 하고, 이를 persistence 레이어에 저장하고 싶다고 해보자. 일반적인 POP 접근은 아래와 같을 것이다.

```swift
protocol Storage {
  associatedtype T: Codable

  func get(key: String) -> T?
  mutating func set(value: T, for key: String)
}

struct Manager<T: Codable> {
  var storage: Storage
  
  init(storage: Storage) {
    self.storage = storage
  }
}
```

하지만 여기서는 Storage가 가진 associated 타입 때문에 “Protocol ’Storage’ can only be used as a generic constraint because it has Self or associated type requirements” 라는 에러가 발생한다.

이를 해결하기 위해서 `Storage` 프로토콜을 구현한 구현체가 있어야 했다. 예를 들어 메모리에 있는 값을 가지고 있을 `KeyValueStorage`를 구현할 수 있을 것이다.

```swift
struct KeyValueStorage<T: Codable>: Storage {
  var memory: [String: T] = [:]
  func get(key: String) -> T? {
    return self.memory[key]
  }

  mutating func set(value: T, for key: String) {
    self.memory[key] = value
  }
}
```

만약 매니저가 메모리 저장소를 `UserDefaults`를 사용하는 걸로 바꿔야 한다고 하면 어떻게 할까?
이제 `Storage` 프로토콜을 구현한 또 다른 새로운 struct를 생성해야 한다. 또한 새로운 프로토콜 구현체를 사용하기 위해 `Manager`의 내부를 바꿔야 한다. 이는 개방 폐쇄 원칙에 어긋난다. 

Witnesses를 이용해서, 이 문제들을 해결할 수 있다. Witness는 클로저를 가지고 있는 struct이다. 클로저를 프로토콜 함수 정의처럼 생각할 수 있을 것이다. 이 구조체를 위해 값을 생성할 때, 우리는 구현체를 제공해야 한다.

이제 위에서 언급한 문제를 해결해보자. 먼저, `Storage` witness를 아래와 같이 정의한다.

```swift
struct Storage<T: Codable> {
    // 1. closures that defines the implementation of the witness
    private let _get: (_ key: String) -> T?
    private let _set: (_ value: T, _ key: String) -> Void
    
    // 2. initializer that let us create different versions of the witnesses
    init (
        get: @escaping (_ key: String) -> T?,
        set: @escaping (_ value: T, _ key: String) -> Void
    ) {
        self._get = get
        self._set = set
    }
    
    // 3. nicer apis with albels for the parameters
    func get(key: String) -> T? {
        return self._get(key)
    }
    
    func set(value: T, for key: String) {
        self._set(value, key)
    }
}
```

이 코드는 굉장히 적은 줄 수에 많은 의미를 담고 있다. 이는 witness의 일반적인 구조다.

1. Witness가 제공하는 클로저는 API를 정의한다.
2. Witness의 이니셜라이저에 다양한 구현체를 전달할 수 있다.
3. 여러 함수들이 클로저를 실행시킨다. 이 단계는 필수적이진 않지만, 호출하는 대상에게 더 좋은 API를 제공하는데 도움이 된다. 예를 들어 변수에 라벨을 다는 데 사용할 수 있다.

이제, `Storage` 구조체를 적절한 클로저로 초기화 해서 `KeyValueStorage`를 만들 수 있다. 또 다른 클로저와 변수를 가지고 `UserDefaultStorage`를 만들 수도 있다.

```swift
struct SomeCodableThing: Codable {}

class Manager<T: Codable> {
    let storage: Storage<T>
    
    
    init(storage: Storage<T>) {
        self.storage = storage
    }
}

// in-memory 저장소를 사용하는 매니저 생성
var memory: [String: SomeCodableThing] = [:]
let memoryManager = Manager<SomeCodableThing>(storage: Storage(get: { key in
    return memory[key]
}, set: { value, key in
    memory[key] = value
}))

// user-default backed storage를 사용하는 매니저 생성
let userDefaults = UserDefaults.standard
let userDefaultManager = Manager<SomeCodableThing>(storage: Storage(get: { key in
    return userDefaults.object(forKey: key) as? SomeCodableThing
}, set: { value, key in
    userDefaults.setValue(value, forKey: key)
}))
```

이를 통해 다른 struct를 정의하지 않고도 다양한 저장소와 매니저를 만들 수 있게 된다.

매니저와 메모리 저장소의 정의는 변하지 않는다. 위의 구조를 따르면 개방-폐쇄 원칙에도 어긋나지 않는다. 변하는 것은 오직 매니저에 전달되는 저장소의 타입이다. 이런 접근 방법은 DI(Dependency Injection)의 기반이 된다. 매니저의 구현을 걱정할 필요없이 매니저에 다른 구현체들을 주입할 수 있기 때문이다.

이런 접근 방법은 클린 코드의 중요한 원친인 DI 원친을 따른다. 프로토콜을 추상화된 기능을 위해 사용했음에도 불구하고, 첫 번째 예제는 프로토콜에 의존하지 않고 `KeyValueStorage`에 의존한다. Protocol witness 접근법을 사용하면, 매니저는 `get`과 `set`을 제공하는 구조체에만 의존하고, 그 구현 내용은 전혀 신경쓰지 않는다.

## Use Factory Methods

Protocol witness가 멋지고, 개방-폐쇄 원칙, DIP를 지킬 수 있게 해준다 해도, 코드가 아주 깔끔하지는 않다고 할 수 있다. 그리고 원래의 프로토콜 기반 접근 법에 의해 작성된 코드보다 읽기도 힘들다.

하지만 이런 문제를 해결할 수 있는 방법이 있다. 코드를 리팩토링하고 몇 전역 팩토리 메서드를 제공하는 것이다. 

가장 핵심이 되는 아이디어는 구조체에 몇 기본 구현을 생성하기 위해 `extension`을 만드는 것이다. 그리고, 코드의 여러 부분에서 이를 재사용 한다. 

```swift
extension Storage {
    static var keyValue: Self {
        var memory: [String: T] = [:]
        
        return .init { key in
            return memory[key]
        } set: { (value, key) in
            memory[key] = value
        }
    }
    
    static func userDefaults(using userDefaults: UserDefaults = .standard) -> Self
    {
        return .init { key in
            return userDefaults.object(forKey: key) as? T
        } set: { (value, key) in
            userDefaults.setValue(value, forKey: key)
        }

    }
}
```

이 코드에서, 굉장히 적은 줄의 코드로 다른 `Storage` witness를 어떻게 생성하는지 볼 수 있다. `memory` 딕셔너리는 `static` factory에서
capture된다. 의존성 역전은 `UserDefaults`에도 적용될 수 있다. `manager`에 `userDefualts` 파라미터를 전달함으로써, 주입된 의존성에 의존성을 주입할 수 있다!

사용할 때, 다른 `Storage` 구현체를 전달해서 다른 매니저들을 초기화할 수 있다. 

```swift
let manager1 = Manager<Something>(storage: .keyValue)

let manager2 = Manager<Int>(storage: .userDefaults())

let customUserDefaults = UserDefaults(suiteName: "custom-suite")!
let manager3 = Manager<String>(storage: .userDefaults(using: customUserDefaults))
```

## Use High Order Functions and Keypaths

반복문과 같이 읽기 힘들고 이해하기 어려운 것도 없을 것이다. 가끔 아래와 같이 코딩할 때가 있다.

```swift
struct SDK {
    var name: String
    var version: String
    var platform: String
}

let sdks: [SDK] = [
  .init(name: "device", version: "13.7", platform: "mobile"),
  .init(name: "simulator", version: "13.7", platform: "mobile"),
  .init(name: "device", version: "14.4", platform: "mobile"),
  .init(name: "simulator", version: "14.4", platform: "mobile"),
  .init(name: "tvOS", version: "3.5", platform: "tv"),
  .init(name: "tvSimulator", version: "3.5", platform: "tv")
]

var candidate: SDK? = nil

for sdk in sdks {
    if sdk.name.lowercased().contains("simulator") && sdk.platform == "mobile" {
        if let cand = candidate, sdk.version > cand.version {
            candidate = sdk
        } else if candidate == nil {
            candidate = sdk
        }
    }
}
```

아래 부분의 for in 반복문을 보면, simulator이면서 가장 높은 버전을 가진 애를 추출하려고 하는 것을 알 것이다.

하지만 이를 더 간결하게 표현할 수 있는 방법은 없을까? 바로 함수형 프로그래밍 접근 방법으로 이를 해결해볼 것이다.

먼저, 구조체에 몇가지 기능을 추가해야 한다. 우리는 Swift extension으로 이를 보장할 수 있다.

```swift
extension SDK: Comparable {
    var isSimulator: Bool {
        return self.name.lowercased().contains("simulator")
    }
    
    var isMobile: Bool {
        return self.platform == "mobile"
    }
    
    static func < (lhs: SDK, rhs: SDK) -> Bool {
        return lhs.version < rhs.version
    }
}
```

그리고 `for-loop`대신에 collection의 함수 `filter`, `max`를 이용해서 이를 대체한다.

```swift
let candidate2 = sdks.filter(\.isMobile)
    .filter(\.isSimulator)
    .max()
```

이제 더 간단한 코드로 같은 결과를 얻어낼 수 있다. 또 이 코드는 설명적이기까지 하다. 처음 보는 사람도 이해하기 쉬운 구조를 가지고 있다. 

