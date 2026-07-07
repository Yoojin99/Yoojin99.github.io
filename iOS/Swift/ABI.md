
* ABI (Application Binary Interface) - compile 이후 바이너리끼리 약속하는 인터페이스, **컴파일된 프로그램끼리 대화하는 규칙**
* API (Application Programming Interface) - 개발자가 사용하는 인터페이스

```swift
// API
func map<T>(_ transform: (Element) -> T) -> [T]

// 개발자는 아래만 알면 됨
array.map { }

// ABI - 컴파일하면 예를 들어 아래와 같은 symbol 이 생김
_$s5MyLib3addyS2i_SitF
```

e.g.

* Apple : Swift.framework 에 `Array.map` 을 배포함
* 나 : `numbers.map {}` 을 사용해서 앱 컴파일

내 앱에서는 "`Array.map` symbol 을 호출해라" 라는 코드만 있음
실제 구현은 OS 안 Swift 라이브러리에 있음

⏰ 시간 지난 후

* Apple : `Array.map` 삭제할게
* 내 앱 : 실행할 때 `Array.map` 어디 있음? => `dyld: Symbol not found`

ABI 가 깨진 것

### Compiler / Language / Standard Library / Runtime

#### 1. Swift Language

문법. e.g. `-swift-version 5`

```swift
if let x {}

async / await
```

#### 2. Swift Compiler

Source code 를 기계어로 바꿈. `swiftc`. e.g. `Swift 5.0 compiler`, `Swift 5.2 compiler`, `Swfit 6 compiler`

#### 3. Swift Standard Library

`Array`, `Dictionary`, `String` 등의 타입이 들어있는 라이브러리. e.g `libswiftCore.dylib`

#### 4. Runtime

프로그램 실행 중 필요한 기능. e.g. ARC, type metadata, protocol witness table, existential.

Standard Library 와 함께 대부분 `libswiftCore.dylib` 에 들어있음

### OS 안 Swift 라이브러리?

Apple OS 에는 Swift Runtime, Standard Library 의 상단 부분이 OS 와 함께 들어있음

- Swift 5 이후 (ABI Stability 이후) : OS 안에 Swift 라이브러리가 포함되어 있음 (e.g. `libswiftCore.dylib`, `libswiftFoundation.dylib`,...). 

내 앱

```swift
let a = [1, 2, 3].map { $0 * 2 }
```

컴파일 시 아래와 같은 코드만 들어감

```
Array.map 호출
```

실행시

```
App
 │
 ▼
libswiftCore.dylib // map 구현이 OS 안에 존재
 │
 ▼
Array.map 실행
```

- Swift 5 이전 : OS 에 Swift 가 없음. 앱 생성시 아래와 같이 앱 안에 Swift Runtime 을 같이 넣어서 배포, Swift 앱 용량이 컸음

```
MyApp.app

libswiftCore.dylib
libswiftFoundation.dylib
...
```

❗️즉 Swift 는 ABI 를 마음대로 바꿀 수 있었고, 버전 간 ABI 호환을 전혀 보장하지 않았음

e.g. (Swift 4.0 컴파일러 - Swift Runtime 4.0) 조합만 지원하면 됐고, (Swift 4.0 컴파일러 - Swift Runtime 4.1) 조합은 지원 안해도 됐음 => 그래서 앱 안에 Swift Runtime 을 같이 넣었던 것

결론 : OS 가 `libswiftCore.dylib` 를 제공하면 수만개의 앱이 그 라이브러리를 사용하는데, 애플이 마음대로 삭제할 수 없음


# Binary Compatibility

**특정 버전의 Swift compiler 로 빌드된 앱이 다른 버전의 compiler 로 빌드된 라이브러리와 소통할 수 있는 것**

e.g.

Swift 5.0 으로 빌드된 앱

* Swift 5 표준 라이브러리가 설치된 시스템에서 실행 가능
* Swift 5.1 설치된 시스템 => 실행 가능
* Swift 6 설치된 시스템 => 실행 가능

Apple OS 에 ABI stability 가 적용됐다 == 해당 OS 에서 배포된 앱들이 Swift standard library 를 임베딩할 필요가 없음 (Swift runtime, standard library 는 OS 에 넣어짐) => 앱 용량이 줄어듦 

* https://www.swift.org/blog/abi-stability-and-more/
