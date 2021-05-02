---  
layout: post  
title: "Swift - Escaping clousre, 탈출 클로저"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  
  
> `탈출 클로저는 무엇이며 어떨 때 사용하는 것인가?`  

---


## Escaping Clousre

탈출 클로저는 **함수의 전달인자로 전달한 클로저가 함수의 종료 후에 호출될때 클로저가 함수를 escape한다고 표현**한다. 이렇게 함수의 인자가 함수의 영역을
탈출해서 함수 밖에서 사용할 수 있는 개념은 기존의 scope 개념을 무시한다. 왜냐하면 함수에서 선언된 로컬 변수(클로저)가 로컬 변수의 영역을 뛰어넘어 함수
밖에서도 유효하기 때문이다.

```swift
class ViewModel {
  var completionHandler: (() -> Void)? = nil
  
  func fetchData(completion: @escaping() -> Void) {
    completionHandler = completion
  }
}
```

위 코드에서 볼 수 있듯이 completion은 함수의 실행이 종료되기 전에 실행되지 않기 때문에 함수 밖에서(escaping) 실행되는 클로저다.

### escaping

`@escaping`이 붙어 있어도 non-escaping 클로저를 인자를 넣을 수 있다.

```swift
func runClosure(closure: @escaping () -> Void) {
  closure()
}
```

하지만 escaping 클로저를 `@escaping` 없이 사용하면 컴파일 에러가 난다.

이렇게 escaping, non-escaping 클로저를 나눠서 사용하는 이유는 컴파일러의 퍼포먼스와 최적화 때문이다. non-escaping 클로저는 클로저의 실행이 언제 종료되는지 알기 때문에, 때에 따라 클로저에서 사용하는 특정 객체에 대한 retain, release 등의 처리를 생략해 객체의 라이프싸이클을 효율적으로 관리할 수 있다.

하지만 escaping 클로저는 함수 밖에서 실행되기 때문에 클로저가 함수 밖에서도 적절히 실행되는 것을 보장하기 위해 클로저에서 사용하는 객체에 대한 추가적인
reference cycle 관리 등을 해줘야 한다. 이 부분이 컴팡일러의 퍼포먼스와 최적화에 영향을 끼치기 때문에 swift는 필요할 때만 escaping 클로저를 사용하도록 해놓았다.

### self

`self`를 언급하는 escaping closure는 `self`가 클래스의 인스턴스를 가리킬 때 특별히 주의해야 한다. Escaping closure에서 `self`를 capturing 하는 것은 강한 참조 사이클을 생성하기 쉽다. 

일반적으로, 클로저는 클로저의 body 안에서 변수를 사용함으로써 값을 획득하지만, 이 경우에는 명시적으로 해줄 필요가 있다. 만약 `self`를 획득하고 싶다면, 사용할 때 `self`를 명시적으로 적어주거나 클로저의 획득 리스트에 `self`를 포함시켜야 한다. 명시적으로 `self`를 적으면 의도를 표현할 수 있고, 
참조 사이클이 없는 것을 확인할 수 있게 리마인드 할 수도 있다. 예를 들어 아래 코드에서 `someFunctionWIthEscapingClosure(_:)`에 전달된
클로저는 `self`를 명시적으로 적고 있다. 반대로, `someFUnctionWithNonescapingClosure(_:)`에 전달된 클로저는 nonescaping closure이므로 `self`를 암묵적으로 사용할 수 있다.

```swift
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}

let instance = SomeClass()
instance.doSomething()
print(instance.x)
// Prints "200"

completionHandlers.first?()
print(instance.x)
// Prints "100"
```

아래 코드는 `self`를 클로저의 획득 리스트에 포함시켜서 이를 획득하고, `self`를 암묵적으로 언급하는 코드다.

```swift
class SomeOtherClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { [self] in x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
```

만약 `self`가 구조체가 열거형의 인스턴스라면 항상 `self`를 암묵적으로 사용할 수 있다. 하지만, `self`가 구조첸나 열거형의 인스턴스일 때 `self`에 대한 변경가능한 참조는 획득하지 못한다. 구조체나 열거형은 공유되는 변경가능성을 허용하지 않는다.

```swift
struct SomeStruct {
    var x = 10
    mutating func doSomething() {
        someFunctionWithNonescapingClosure { x = 200 }  // Ok
        someFunctionWithEscapingClosure { x = 100 }     // Error
    }
}
```

위에서 `someFUnctionWithEscapingClosure`에서 에러가 난 이유는 mutating 메서드 안에서 `self`는 변경가능하기 때문에 에러가 난 것이다. 이거는
클로저가 구조체에 대한 변경가능한 참조를 획득하지 못하는 규칙을 위반한다.

### 

클로저의 Escaping은 A 함수가 종료된 이후에 B 함수가 실행되도록 함수를 작성할 수 있다는 점에서 유용하다. 즉 **함수 사이에 실행 순서를 정할 수 있다.**
함수의 실행 순서를 보장할 수 있는 것은 비동기 함수의 경우도 포함하기 때문에 중ㅇ하다. 서버에서 json 형식의 데이터를 가져와 화면에 이를 출력하는 것을 생각해보자.
이때 HTTP 통신을 위해 Alamofire라이브러리를 사용한다.

```swift
Alamofire.request(urlRequest).responseJSON { response in 
  //handle response
}
```

위 메서드는 서버로 request를 전송한다. 그리고 GET 방식으로 Json 형식의 데이터를 받아온다. 그 결과를 Response 객체를 통해 바을 수 있다.
일반적으로 서버에 request를 전송하고 response를 받는 함수들은 비동기로 작동하여 request를 보낸 직후 반환되는데, response가 request 결과를 기다리게
하는 형태로 함수를 작성할 수 있도록 하는 것이 바로 Escaping clousre다. responseJSON 메서드의 파라미터를 보면 아래와 같이 되어 있다.

```swift
@discardableResult
    public func responseJSON(
        queue: DispatchQueue? = nil,
        options: JSONSerialization.ReadingOptions = .allowFragments,
        completionHandler: @escaping (DataResponse<Any>) -> Void)
        -> Self
    {

    }
```

여기서 queue와 options는 기본값이 지정되어 있어서 값을 따로 지정하지 않아도 되고, completionHandler에는 값을 줘야 한다. completionHandler는
escaping closure 형태로 작성되어 있어서 responseJSON 함수가 종료되고(완전히 서버로부터 값을 가져온 상태) 실행되는 것이다. 그 부분이 { response in } 부분이다.

### Async inside Async

Escaping closure는 HTTP 통신에서 completionHandler로 많이 사용된다. 서버에 요청하는 Restful API 기반의 Request들은 앱의 여러군데에서 사용되기 때문에
따로 클래스를 만들어 사용하는 것이 유용하다. 아래서는 하나의 class 안에서 통신 메서드들을 static 함수 형태로 관리한다.

```swift
class Server {
  static getPerson() {
    // do sth
  }
}
```

위 코드는 static 메서드로 getPerson()을 작성했기 때문에 `Server.getPerson()` 형태로 호출할 수 있다. 
앞에서 서버에서 json 정보를 가져와 앱 화면을 보여주는 경우세어, 이 메서드를 static 함수 형태로 관리하려면 어떻게 해야 할까? 데이터를 받아오는 것과
데이터로 화면을 업데이트 하는 것 모두 비동기로 이루어져야 한다. 또한 데이터를 받아온 상태에서 화면을 업데이트 하는 것을 **보장**해야 한다. 
안그러면 데이터가 다 받아지지 않았는데 화면을 업데이트 해서 크래시가 날 수 있다. 따라서 이때는 두 개의 Escaping Closure를 함께 사용한다.

```swift
class Server {
  static var persons: [Person] = []
  // 2
  static getPerson(completion: @escaping (Bool, [Person]) -> Void) {
    Alamofire.request(urlRequest).responseJSON { response in
      persons.append(데이터)
      DispatchQueue.main.async {
        // 3
        completion(true, persons)
      }
    }
  }
}

// 1
Server.getPerson { (isSucess, persons) in
  // 4
  if isSuccess {
    //update UI
  }
}
```

1. 필요한 데이터를 Server의 정적 함수 `getPerson(completion:)`을 통해 호출한다.
2. Alamofire를 통해 서버로 `Request`를 전송하고, `responseJson`은 EscapingCLousre이므로 `{ response in }` 부분은 결과가 모두 들어온 이후에 실행된다.
3. `responseJson`의 `completionHandler` 블럭이 실행되고, 화면 업데이트를 위해 서버에서 받아온 데이터를 처음 호출했던 곳으로 보내려고 `getPerson(completion:)`의 `completion`을 호출한다. 이때 화면 업데이트는 Main 쓰레드에서 이뤄져야 하므로 `completion`은 escaping closure 형태를 취한다.
4. 호출된 `completion`으로 `getPerson(completion:)` 메서드의 `completion` 블럭이 실행된다. 이때 통신이 잘 되었는지 확인하는 Bool을 `isSuccess`로 넘기고, 데이터를 persons로 넘겼다.


출처

* https://hcn1519.github.io/articles/2017-09/swift_escaping_closure
* https://jusung.github.io/Escaping-Closure/
