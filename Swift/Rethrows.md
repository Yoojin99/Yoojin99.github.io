
`rethrows` - 함수 A 가 throwing 하는 함수 B 를 파라미터로 받을 때 사용. 

**"자기 자신의 에러" 를 전달하는 함수가 아닌 함수 파라미터로 전달된 곳에서 발생한 에러를 "전달" 할 때 사용**

* 함수 B 가 throw - 함수 A 도 throwing 함수가 됨
* 함수 B 가 throw X - 함수 A 도 throw 하지 않음

```swift
// throwing func
func authFail(_ string: String) throws -> Bool {
	throw "Failed"
}

// non throwing func
func authSuccess(_ string: String) -> Bool {
	return true
}
```

위 두가지 방법을 모두 사용할 수 있는 auth 함수 생성

```swift
// throwing 하는 함수가 들어올 수 있기 때문에 authUser 도 throws 로 처리
func authUser(method: (String) throws -> Bool) throws {
	try method("test")
	print("Success")
}
```

사용할 때

```swift
// 항상 do-catch 문으로 에러 핸들링을 해줘야 함
do {
	try authUser(method: authFail)
} catch {

}
```

## Rethrows 키워드

* 인자로 전달된 함수가 throw 인 경우 - 바깥 함수도 throw
* 인자로 전달된 함수가 throw 하지 않는 경우 - 바깥 함수도 throw 하지 않음

위 두 케이스를 구분해서 작성할 수 있게 해줌

Rethrows 로 다시 작성된 `authUser`

```swift
func authUser(method: (String) throws -> Bool) rethrows {
	try method("test")
	print("Success")
}
```

사용처

```swift
// ✅ throw 하는 함수를 전달한 경우 - 일반적인 에러 핸들링
do {
	try authUser(method: authFail)
} catch {

}

// ✅ throw 하지 않는 함수를 전달한 경우 - 일반 throw 하지 않는 함수처럼 작성
authUser(method: authSuccess)
```

## map, flatMap, compactMap

![[Pasted image 20260701163830.png]]

`compactMap`, `flatMap`, `reduce` 등의 sequence 를 transform 하는 메서드의 정의에서 `rethrows` 를 사용함

`compactMap` 예시

```swift
func compactMap<ElementOfResult>(_ transform: (Self.Element) throws -> ElementOfResult?) rethrows -> [ElementOfResult]
```

```swift
let array: [Int?] = [1, nil, 2, 4]

func transformNonthrowing(_ number: Int?) -> String? {
	guard let number else { return nil }
	return (number.isMultiple(of: 2) ? "Even" : "Odd")
}

func transformThrowing(_ number: Int?) throws -> String {
	guard let number else { throw "number is nil" }
	return (number.isMultiple(of: 2) ? "Even" : "Odd")
}

// non throw 하는 메서드를 넘길 경우에 try 없이 호출될 수 있음
let result1 = array.compactMap(transformNonthrowing(_:))

// throw 하는 메서드를 넘길 경우 try 붙여서 홏 ㄹ
let result2 = try array.compactMap(transformThrowing(_:))
```

`map` 도 원래는 `rethrows` 를 사용하고 있었으나 Swift 6.0 Typed throws 이후에 rethrowing 하는 것 대신에 제네릭 코드에서 특정 에러 타입을 throw 할 수 있도록 아래와 같이 정의가 바뀜

https://github.com/swiftlang/swift/pull/69771

```swift
func map<T, E>(_ transform: (Self.Element) throws(E) -> T) throws(E) -> [T] where E: Error
```

callback 에서 발생한 에러는 `map` 을 호출한 곳에 전파되고, callback 이 에러를 던지지 않는 경우에 `E` 는 `Never` 가 됨