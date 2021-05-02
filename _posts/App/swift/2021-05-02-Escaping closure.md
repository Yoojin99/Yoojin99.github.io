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

위 코드에서 볼 

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
