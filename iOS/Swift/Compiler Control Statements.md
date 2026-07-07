
* Preprocess : 컴파일 전에 처리하는 작업
* Compiler Control Statements (컴파일러 제어문) : 컴파일러의 동작을 변경할 수 있게 하는 문장
	1. conditional compilation block
	2. line control statement
	3. compile-time diagnostic statement

# 1. Conditional compilation block

컴파일 조건에 따라서 컴파일이 되는 것. 

```swift
#if 컴파일 조건문
	// code
#endif

#if 조건문1
	조건문1 이 true 일 때 컴파일 되는 코드
#elseif 조건문2
	조건문2 이 true 일 때 컴파일 되는 코드
#else 조건문3
	조건문3 이 true 일 때 컴파일 되는 코드
#endif

// 흔히 사용하는 
#if DEBUG
#endif
```

컴파일 조건은 컴파일 시점에 검증되기 때문에 컴파일 시점에 조건문이 true 인 경우에만 내부에 있는 코드가 컴파일되고 실행됨 (if 문과는 다름). 

e.g. Swift 5 compiler / Swift Language 4.2 모드 에서 코드를 컴파일할 경우 print 문이 전부 다 출력됨


```swift
#if compiler(>=5)
print("Compiled with the Swift 5 compiler or later")
#endif
#if swift(>=4.2)
print("Compiled in Swift 4.2 mode or later")
#endif
#if compiler(>=5) && swift(<5)
print("Compiled with the Swift 5 compiler or later in a Swift mode earlier than 5")
#endif
// Prints "Compiled with the Swift 5 compiler or later".
// Prints "Compiled in Swift 4.2 mode or later".
// Prints "Compiled with the Swift 5 compiler or later in a Swift mode earlier than 5".
```

## Compilation Condition

조건문에 올 수 있는 것들

* custom flag : 빌드 세팅에서 정의하는 이름. e.g. `DEBUG` 
* 플랫폼/아키텍처 : OS, 아키텍처 검사. e.g. `os(iOS)`, `arch(arm64)`
* 언어/컴파일러 버전 : 버전 검사. e.g. `swift(>=6.0)`, `compiler(>=5.9)`
* 그 외 : 모듈/환경 검사. e.g. `canImport(UIKit)`, `targetEnvironment(simulator)`

### Custom compilation flag

```swift
#if DEBUG // custom flag
#endif
```

#### Build Settings

Build Settings > Swift Compiler - Custom Flags > Active Compilation Conditions

![[Pasted image 20260611164718.png]]

* 의미
	* **빌드 configuration - custom flag 목록**
	* e.g. `Debug` configuration : `DEBUG` 플래그가 활성화되어 있음

![[Pasted image 20260611165810.png]]

DEBUG configuration 의 경우 개발자가 따로 별도 설정을 하지 않아도 `$(inherited)` 로 `DEBUG` 가 자동으로 설정되기 때문에 `#if DEBUG` 를 바로 쓸 수 있는 것.

만약 아래와 같이 `$(inherited)` 구문을 아예 지워버린다면 DEBUG configuration 이더라도 `#if DEBUG` 구문에서 작성한 코드를 호출할 경우 에러가 남

![[Pasted image 20260611170059.png]]

### 플랫폼 / 아키텍처

```swift
#if os(iOS)
	// iOS 전용 코드
#elseif os(macOS)
	// macOS 전용 코드
#endif
```

```swift
#if arch(arm64)
	// arm 64 전용 코드 e.g. iPhone
#elseif arch(x86_64)
	// x86_64 전용 코드 e.g. iOS Simulator
#endif
```

## Import

어떤 모듈이 import 될 수 있는지 확인

```swift
#if canImport(UIKit)
import UIKit
#endif
```

# 2. Line Control Statement

진단/디버깅 목적으로 소스 코드의 위치를 변경하고 싶을 때 사용. 컴파일되고 있는 소스코드와 줄 번호 / 파일 이름이 다른 코드 줄 번호 / 파일 이름 을 명시할 때 사용

```swift
#sourceLocation(file: <파일 경로>, line: <줄 번호>)
#sourceLocation()
```

# 3. Compile-Time Diagnostic Statement

Swift 5.9 이전에는 `#warning` `#error` 문장을 사용했는데, 이제는 Swift 표준 라이브러리에 macro 로 제공됨

```swift
#warning("이건 warning 입니다") // warning 표시
#error("이건 error 입니다") // error 표시
```


* 참고
	* https://docs.swift.org/swift-book/documentation/the-swift-programming-language/statements/#Conditional-Compilation-Block


