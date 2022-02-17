---  
layout: post  
title: "[iOS] - UIApplication.shared.open으로 url 리소스 열기"  
subtitle: ""  
categories: app
tags: app-ios uiapplication url
comments: true  
header-img: 

---  
  
> `UIApplication.shared.open()`는 어떻게 쓸까?  

---

## 정의

`UIApplication.shared.open(_:options:completionHandler:)`는 인스턴스 메서드로, **특정 URL에 있는 리소스를 비동기적으로 연다.**

iOS 10.0 이후로 사용 가능하다. 정의를 보면 아래와 같다.

```swift
func open(_ url: URL, 
  options: [UIApplication.OpenExternalURLOptionsKey : Any] = [:], 
  completionHandler completion: ((Bool) -> Void)? = nil)
```

* 파라미터
  * url : URL. 이 URL로 식별된 리소스는 현재 앱에 위치할 수도 있고 다른 앱에 의해 제공되는 것일 수도 있다. UIKit은 많은 일반적인 스킴들을 지원하는데, `http`, `https`, `tel`, `facetime`, `mail` 같은 것이 스킴에 포함된다. 또한 기기에 설치된 앱과 연관된 커스텀 URL 스킴을 적용할 수도 있다.
  * options : URL을 열 때 사용할 옵션 dictionary다. 이 딕셔너리의 키가 될 수 있는 것들의 목록을 보려면 `UIApplication.OpenExternalURLOptionsKey`를 참고하면 된다.
  * completionHandler : 결과를 받아 실행할 블럭이다. URL을 여는데 성공했는지, 실패했는지를 알고 싶다면 이 파라미터에 값을 제공하면 된다. 이 블럭은 앱의 메인 쓰레드에서 비동기적으로 실행된다. 블럭은 리턴 값이 없고, `success`라는 URL이 성공적으로 열렸는지를 나타내는 Boolean 값을 인자로 갖고 있다.. 

즉 정리하자면 이 메서드로 특정 리소스를 열 수 있다. 만약 명시된 URL scheme이 다른 앱에 의해 다뤄지고 있으면 iOS는 해당 앱을 열고 URL을 앱에 전달할 것이다. (다른 실행시킨다는건 다른 앱을 화면에 띄우는 것을 의미한다.)
만약 명시된 scheme을 다루는 앱이 없다면 completion handler의 `success` 파라미터가 false로 설정되어 호출된다.

URL을 다룰 수 있는 앱이 설치되었는지를 판단하기 위해 이 메서드를 호출하기 전에 `canOpenURL(_:)`을 호출하면 된다. 

## 병행성

위에서 봤듯이 completion handler를 사용해서 동기적으로 작동하는 코드에서 이 메서드를 호출할 수 있고, 또는 아래와 같이 정의된 비동기 메서드로 호출해도 된다.

```swift
func open(_ url: URL, options: [UIApplication.OpenExternalURLOptionsKey : Any] = [:]) async -> Bool
```

참고로 Swift에서 동기와 비동기 코드에 대한 정보를 더 보려면 [Calling Objective-C APIs Asynchronously](https://developer.apple.com/documentation/swift/calling_objective-c_apis_asynchronously)를 보자.

## Options

위에서 url을 열 때 설정할 수 있는 option들이 있다고 했다. 이 옵션에는 어떤 것들을 넣을 수 있는지 보자. 두 가지가 있는데, URL 옵션과 사용자가 탭하는 것을 집계할 수 있는 옵션이 있다.

* `universalLinksOnly` : URL이 오직 유요한 universal link이며 이 URL을 열 수 있는 앱이 있을 때만 URL을 연다. 키의 값에는 Boolean 값을 포함한 NSNumber 객체가 될 수 있다.
* `eventAttribution` : Private Click Measurement를 위해 tap 이벤트 기여 데이터를 브라우저에 보낼 때 사용하는 객체다.
