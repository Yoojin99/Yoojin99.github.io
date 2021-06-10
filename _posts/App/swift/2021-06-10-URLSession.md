---  
layout: post  
title: "Swift - URLSession"  
subtitle: ""  
categories: app
tags: app-swift URLSession
comments: true  
header-img: 
---  
  
> ``  

---

## URLSession

URLSession은 클래스로, 연관된 네트워크 데이터 전송 작업들을 조정하는 객체다.

```swift
class URLSession : NSObject
```

URLSession 클래스와 연관된 다른 클래스들은 **URL이 가리키는 엔드포인트에서 데이터를 다운로드하고 업로드하는 API를 제공**한다. 이 API를 사용해서
앱이 실행되고 있지 않거나 iOS에서는 앱이 suspended 되었을 때 뒷단(background)에서 다운로드를 할 수 있다. Redirection이나 작업이 완료된 것과 같은
이벤트를 받고, 인증을 지원할 때 URLSessionDelegate와 URLSessionTaskDelegate를 사용할 수 있다.

*URLSession API는 복잡한 방식으로 같이 작동하는 여러개의 다른 클래스를 포함하고 있다. 그래서 API를 사용하기 전에 [URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system)
을 읽어보는 것이 좋다. 참고로 앞에 있는 링크에 있는 내용도 이 글에서 정리했다.*

앱은 하나 이상의 URLSession 인스턴스를 생성하고, 각각은 연관된 데이터 전송 작업의 그룹을 조정한다. 예를 들어, 웹 브라우저를 만들고 있을 때, 앱은 한 탭이나 윈도우마다
하나의 세션, 혹은 상호작용을 하기 위해서나 background에서 일어나는 다운로드를 위해 하나의 세션을 생성할 수 있다. 각 세션에서, 앱은 작업들을 추가하고,
각각은 특정 URL에 대한 request를 나타낸다.

### URL Session의 타입

주어진 URL 세션의 작업들은 하나의 host를 만들기 위한 동시에 발생할 수 있는 연결의 최대 개수, 연결들이 cellular 네트워크를 사용할 수 있는지나, 이런 
연결 행위를 정의하는 공통의 세션 설정 객체를 공유하고 있다. 

URLSession은 일반적인 request들을 위한 싱글톤인 shared 세션을 가지고 있다. (얘는 설정 객체를 가지고 있지 않다.) 이건 내가 생성하는 세션처럼
커스터마이징할 수 있는 것은 아니지만, 내가 굉장히 제한된 요구사항을 가지고 있을 때 좋은 시작점을 제공한다. 이 세션은 공유된 클래스 메서드를 사용해서 접근할 수 있다.
다른 종류의 세션에 대해서는, 아래의 세 종류의 설정 중 하나로 URLSession을 생성한다.

* Default 세션은 공유된 세션과 굉장히 비슷하게 행동하지만, 내가 직접 구성할 수는 없다. 이 세션은 데이터를 점진적으로 획득하기 위해 delegate를 default 세션에 할당하는 게 가능하다.
* 임시 세션은 공유된 세션과 비슷하지만, 캐시, 쿠키, 혹은 디스크에 대한 인증 정보를 쓰지 않는다.
* 백그라운드 세션은 앱이 실행중이지 않을 때 컨텐츠를 업로드하고 다운로드하는 것을 가능하게 해준다.

각 설정 타입을 생성하는 것에 있어서 더 자세한 내용을 보려면 [URLSessionConfiguration](https://developer.apple.com/documentation/foundation/urlsessionconfiguration) 클래스의 
[세션 설정 객체 생성하기](https://developer.apple.com/documentation/foundation/urlsessionconfiguration#1660412) 부분을 보면 된다.

### URL Session 작업의 타입

세션에서, 서버에 데이터를 업로드할 수도 있고, 이후 디스크의 파일이나 메모리의 하나 이상의 [NSData](https://developer.apple.com/documentation/foundation/nsdata) 객체로 데이터를 서버에서 받는 작업들을 생성한다.
URLSession API는 네 가지 타입의 작업들을 제공한다.

* Data task. 데이터 작업들은 NSData 객체를 사용해서 데이터를 주고 받는다. 데이터 작업들은 짧고 종종 서버와 상호작용하는 request에 맞춰져 있다.
* Upload task. 업로드하는 작업들은 데이터 작업과 비슷하지만, 데이터를 전송하고, 앱이 실행중이지 않을 때 background에서 업로드 하는 것을 지원한다.
* Download task. 다운로드하는 작업들은 데이터를 파일의 형태로 받고, 앱이 실행중이지 않을 때 background에서 다운로드하고 업로드하는 것들을 지원한다.
* WebSocket task. 웹소켓 작업들은 [RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455)에 정의된 웹소켓 프로토콜을 이용해서 TCP, TLS위에서 메세지를 교환한다.

### Session Delegate 사용하기

세션에 있는 작업들은 공통의 delegate 객체를 공유한다. 이 delegate를 구현해서 아래를 포함한 다양한 이벤트들이 발생할 때, 정보를 제공하고 받는다.

* 인증 실패
* 데이터를 서버로부터 받았을 때
* 데이터를 캐싱할 수 있게 될 때

Delegate에 의해 제공되는 기능들이 필요하지 않다면, 세션을 생성할때 delegate에 `nil`을 전달하면 된다.

**세션 객체는 앱이 종료되거나 명시적으로 세션을 위반하지 않을때까지 delegate에 강한 참조를 획득하고 있다. 만약 세션을 위반하지 않았다면, 앱이 멈출때까지 메모리에 누수가 발생할 것이다.**

세션에서 생성한 각 작업들은 URLSessionTaskDelegate에 정의된 메서드들을 이용해서 세션의 delegate를 호출한다. 작업에 특정적인 별도의 delegate를 
배치해서 세션 delegate에 callback이 도달하기 전에 이를 가로챌 수 있다.

### 비동기성과 URL Session

대부분의 네트워킹 API와 같이, URLSession API는 비동기적인 성질이 강하다. 내가 호출하는 메서드에 따라, 아래의 세 가지 방법 중 한 방법으로 데이터를 앱에 반환한다.

* 만약 Swift를 사용하고 있다면, 공통의 작업을 수행하기 위해 `async` 키워드가 붙여진 메서드를 사용할 수 있다. 예를 들어, `data(from:delegate:)`는 데이터를 가져오고, `download(from:delegate:)`는 파일을 다운로드 한다. 부르는 시점에서 전송작업이 완료될때까지 실행을 멈추기 위해 `await` 키워드를 사용한다.

sssss

출처

* https://developer.apple.com/documentation/foundation/urlsession
* https://developer.apple.com/documentation/foundation/url_loading_system
