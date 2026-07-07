
Concurrent 문백 간에 공유될 때 위험이나 data race 없이 공유될 수 있는 thread-safe 한 타입

값들은 공유되는 mutable 상태가 없을 수 있고, 상태를 보호할 수 있음 (lock / 특정한 actor 에서만 접근 가능하도록 강제할 수 있음)

Sendable 로 마킹될 수 있는 타입들

- value 타입
- mutable 저장 공간이 없는 reference type
- 상태에 대해 내부적으로 접근을 관리하는 reference type
- function, closure

`Sendable` 프로토콜은 required 메서드나 프로퍼티가 필요없지만, 컴파일 타임에 강제되는 의미적인 요구사항이 있음. 

`Sendable` 에 대한 conformance 는 타입 정의부 내에 같은 파일로 정의되어야 함

컴파일러 강제 없이 `Sendable` 을 따르고 있음을 나타내려면 `@unchecked Sendable` 을 씀. 다만 unchecked senable type 에 대한 검증은 개발자의 몫. (e.g. lock 이나 queue 를 써서 상태에 대한 모든 접근을 보호)

# Sendable Structures and Enumerations

`Sendable` protocol 의 요구사항을 만족시키려면 enum / struct 는 오직 sendable member / associated value 를 가지고 있어야 함. (필수조건)

아래의 경우 struct / enum 은 `Sendable` 을 암묵적으로 따름. 다른 경우에는 `Sendable` 을 명시적으로 표시해야 함

- Frozen struct / enum
- public 이 아니거나 `@usableFromInline` 으로 표시되지 않은 struct / enum 

```swift
  // ✅ 자동 Sendable — internal(기본) + 멤버 전부 Sendable (필수조건)
struct Foo {
	let title: String
	var count: Int
}

// ❌ 자동 아님 — public 으로 선언됨
  public struct Bar {
      let title: String
      var count: Int
  }
  
  // 명시 필요:
  public struct Bar2: Sendable {   // 이렇게 직접 선언해야 함
      let title: String
      var count: Int
  }

  // ✅ 자동 Sendable — public이어도 @frozen이면 (B) 충족
  @frozen public struct Baz {
      let title: String
  }
```

*public 이면 Sendable 이 암묵적으로 안되는 이유 : 내부 구현을 보고 암묵적 `Sendable` 을 추론하게 되면, public 타입에서 내부 구현은 언제든 바뀌기 때문에 그걸 근거로 외부 계약을 만들면 위험함 (내부 구현 변경의 자유, 우연한 Sendable 이 충돌함)*

```swift
// 라이브러리 v1, 자동 Sendable 이라 가정
public User {
	public let name: String
}

// 앱 개발자
func login(_ user: User) {
	Task {
		print(user.name) // User 타입이 sendable 일 것이라 생각하고 씀
	}
}

// 라이브러리 v2
// ⚠️ non-sendable 이 됨!
public struct User {
	public let name: String
	let cache: NSMutableDictionary // non Sendable (내부 구현)
}

// ❗️ 내부 구현을 바꾼 것이 외부 사용처에 조용히 data race 를 심어버림, 컴파일러가 잡아줄 수 없음 (모듈이 분리 compile 되기 때문)
```


# Sendable Actors

모든 actor type 은 암묵적으로 `Sendable` 을 채택함. Actor type 의 mutable 상태에 대한 모든 접근은 순차적으로 수행되는 것이 보장되기 때문

# Sendable Classes

아래의 경우에 클래스는 `Sendable`

- `final` 로 표시됨
- immutable 이면서 sendable 인 저장 프로퍼티만 포함하고 있음
- superclass 가 없거나 `NSObject` 를 superclass 로 갖고 있음

`@MainActor` 로 표시된 클래스는 상태에 대한 모든 접근을 main actor 가 조정하기 때문에 암묵적으로 sendable. 이런 클래스들은 mutable 하면서 nonsendable 한 저장 프로퍼티를 가질 수 있음

# Sendable Functions and Closures

`@Sendable` attribute 를 붙여서 function / closure 를 sendable 이라고 표시할 수 있음. 이들이 capture 하는 값들은 무조건 sendable 이어야 함

Sendable closure 를 예상하고 있는 문맥에서 이미 요구사항을 만족하는 closure 는 암묵적으로 `Sendable` 을 따름. e.g. `Task.detached(priority:operation:)`

Type annoation 의 일부로 `@Sendable` 을 명시적으로 쓰거나 클로저의 파라미터 부분에 `@Sendable` 을 작성할 수 있음

```swift
let sendableClosure = { @Sendable (number: Int) -> String in
	if number > 12 { return "More than a dozen" }
	return "Less than a dozen"
}
```

# Sendable Tuples

Tuple 의 모든 요소가 sendable 이어야 `Sendable`.

# sending

## Region

Non-sendable 값들이 서로 참조로 엮여 있을 수 있는 덩어리 단위. 컴파일러가 묶어서 추적

```swift
let a = SomeClass()
let b = a // b 가 a 와 같은 인스턴스를 참조함
let c = Container(child: a) // c 내부에서 a 를 참조함
```

a, b, c 는 같은 region 으로 묶임. 이 중 하나를 다른 thread 에서 건드리면 다른 값들도 영향을 받음

**Compiler 의 규칙 : 하나의 region 은 하나의 격리 도메인 (현재 task / 특정 actor / 특정 task) 에서만 접근 가능해야 함**

## 문제

```swift
func doSomthing(with cat: Cat) {
	Task {
		... cat.map { ... } // Task 가 cat 을 capture 
	}
	// 여기에서도 (현재 함수에서도) 여전히 cat 을 process 할 수 있음
}

// ❗️ Passing closure as a 'sending' parameter risks causing data races between code in the current task and concurrent execution of the closure
```

`cat` 에 동시에 접근할 수 있는 곳

1. 현재 함수 - 함수 내부에서 사용할 수 있음
2. 함수 안에서 만든 `Task`- 나중에 비동기로 실행되며 `cat` 에 접근

⚠️ 같은 region (`cat`) 이 두 격리 도메인에 걸쳐있는 상황. 둘이 동시에 접근하면 data race

## sending == "region 을 떼어낸다"

`sending` 을 파라미터에 붙이는 것 == "이 값을 함수 안으로 전달하면, 호출한 쪽은 그 값을 더 이상 쓰지 않겠다고 약속함"

```swift
let cat = Cat()
doSomething(cat) // sending 으로 넘김
print(cat.name) // ❗️ 에러 : 이미 넘긴 값을 다시 사용
```

## 해결

```swift
func feed(cat: sending Cat) {
    Task {
        let catName = cat.name
    }

    print(cat.name)
}
```

* https://developer.apple.com/documentation/swift/sendable
