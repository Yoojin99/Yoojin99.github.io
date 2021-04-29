---  
layout: post  
title: "Swift - method parameter는 mutable/let인가?"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  
  
> `함수의 파라미터가 mutable한가?`  

---

문득 아래와 같은 코드를 작성하다 에러가 나는 것을 확인할 수 있었다. 참고로 밑의 SomeInfoStruct에는 구조체를 넣어줘서 값 타입의 파라미터를 받을 수 있게 했다.

```swift
func changeBasicInfo(_ info: SomeInfoStruct) {
    info.age = 1 // Cannot assign to property: 'info' is a 'let' constant
}
```

`Cannot assign to property: 'info' is a 'let' constant`라는 에러가 떴다. info는 let 상수이니 info의 프로퍼티 값을 변겨알 수 없다는 말이다.

구글링을 하면서 찾은 것들은 아래와 같다. 몇몇 코드는 파라미터 앞에 `var`를 붙여서 파라미터를 var 변수로 받을 수 있게 해놓았는데, 이는 **Swift3**에서 더 이상 지원하지 않는다. 

해결 방법은 두 가지로 생각할 수 있다.

1. `inout` 키워드를 사용해서 다른 언어에서 pointer를 넘겨주는 것처럼 사용하면 된다. 
2. 함수 내부에서 새로운 var 변수를 하나 생성해서 파라미터의 값을 넘겨준다.

### 1. inout 키워드 사용

`inout` 키워드는 함수 안에서 파라미터를 수정할 수 있게 하고, 함수 밖에서도 해당 수정들이 유지되도록 할 수 있게 한다.

```swift
func changeBasicInfo(_ info: inout SomeInfoStruct) {
  info.age = 1
}

changeBasicInfo(&변수)
```

### 2. 새로운 var 변수 하나 생성

물론 이 경우 함수 안에서 copiedInfo를 만들어 수정했지만, 원래 파라미터로 넘겨준 변수가 수정되는 것은 아니다. 단지 copiedInfo가 파라미터로 넘겨준 애의 값을 복사해서 새로운 것이 하나 더 생성된 것이다.

```swift
func changeBasicInfo(_ info: SomeInfoStruct) {
  var copiedInfo = info
  copiedInfo.age = 1
}
```

### 추가

위에서 오류나던 코드에서 SomeInfoStruct 부분을 참조 타입의 클래스로 바꾸면 에러가 나지 않는다. 그 이유는 **값 타입을 함수에 파라미터로 넘겨줄 때, 해당 파라미터들은 상수이므로 변경될 수 없다.**

```swift
func changeBasicInfo(_ info: SomeClass) {
    info.age = 1 
}
```


참고

* https://www.hackingwithswift.com/example-code/language/what-are-inout-parameters
* https://stackoverflow.com/questions/24077880/swift-make-method-parameter-mutable/24078011#24078011
