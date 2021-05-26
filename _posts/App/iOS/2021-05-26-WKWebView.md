---  
layout: post  
title: "[iOS] - WKWebView, 웹뷰 사용하기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `WKWebView가 무엇인지 보고 사용해보자.`  

---

## WKWebView

WKWebView는 in-app 브라우저와 같은 상호작용 가능한 웹 컨텐츠를 화면에 출력하는 객체다. 즉 브라우저를 앱에서 띄운다고 생각하면 된다.

### 정의

```swift
@interface WKWebView : UIView
```

WKWebView는 UIView를 상속 받고 있으니 View의 일종이다.

### 개요

WKWebView 객체는 앱 UI에서 통합된 웹 컨텐츠를 매끄럽게 보여주기 위해 사용하는 플랫폼-네이티브 뷰이다. 웹 뷰는 웹 브라우징을 지원하며, HTML, CSS, javaScript
컨텐츠를 앱의 네이티브 뷰와 같이 보여준다. 웹 기술을 사용하는 것이 네이티브 뷰를 사용해서 앱을 스타이일하는 것보다 더 가독성이 좋을 때 이 것을 사용하는 것이 좋다.
예를 들어, 앱의 컨텐츠가 자주 바뀔 때 사용할 수 있다.

웹 뷰는 delegate 객체들을 통해 navigation과 UX를 관리한다. 사용자가 웹 컨텐츠의 링크를 클릭하거나, 페이지 이동에 영향을 미치는 컨텐츠와 상호작용할 때 이에 반응하기 위해
navigation delegate를 사용한다. 예를 들어, 특정 조건이 성립되지 않았을 때까지 사용자가 새로운 컨텐츠로 이동하는 것을 막고 싶을 수 있다. 
웹 컨텐츠와 상호작용하는 것에 반응하기 위해 UI Delegate를 사용해서 경고 메세지, 컨텍스트 메뉴와 같은 네이티브 UI 객체를 화면에 띄운다.

즉 웹 뷰에서 화면 전환과 같이 이동하는 것들과 관련이 있는 것은 navigation delegate, 화면에 경고창과 같은 특정 UI를 띄우는 것은 UI Delegate와 관련이 있다는 소리다.

*`WKWebView`는 iOS8 이후 버전에서 `UIWebView`를 대체하고 있고, macOS 10 이후 버전에서 `WebView`를 대체하고 있다.*

뷰 계층에 WKWebView 객체를 코드나 인터페이스 빌더를 이용해서 뷰 계층에 임베딩하면 된다. 인터페이스 빌더에서는 data detector, media playback, interaction behavior
와 같은 많은 설정을 제공하지만, 더 많은 내용을 설정하고 싶다면 `WKWebViewConfiguration` 객체를 이용해서 코드 상으로 커스터마이징 하면 된다.
예를 들어 이 객체를 이용해서 custom URL 스킴을 위한 handler를 명시하거나 웹 컨텐츠에 따로 설정을 적용할 수 있다.

웹 뷰가 스크린에 뜨기 전에, `URLRequest` 구조체를 이용해서 웹 서버에서 컨텐츠를 불러오거나 로컬 파일이나 HTML string에서 바로 컨텐츠를 불러와야 한다.
웹 뷰는 초기 load request의 일부로 이미지나 비디오 같이 임베딩 된 리소스를 자동으로 불러온다. 그리고 뷰의 바운딩 사각형 안에 컨텐츠를 렌더링해서 출력한다.

아래 예제는 `WKWebView` 객체를 이용해서 default view를 바꾼 것이다.

```swift
import UIKit
import WebKit
class ViewController: UIViewController, WKUIDelegate {
    
    var webView: WKWebView!
    
    override func loadView() {
        let webConfiguration = WKWebViewConfiguration()
        webView = WKWebView(frame: .zero, configuration: webConfiguration)
        webView.uiDelegate = self
        view = webView
    }
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let myURL = URL(string:"https://www.apple.com")
        let myRequest = URLRequest(url: myURL!)
        webView.load(myRequest)
    }}

```

웹 뷰는 웹 컨텐츠에 뜨는 전화번호를 자동으로 Phone link로 바꾼다. 사용자가 Phone link를 터치하면 Phone app이 실행되고 번호가 표시되기 된다. Default 
data detector 행위를 바꾸기 위해 `WKWebViewConfiguration`객체를 사용해라.

또한 `setMagnification:centeredAtPoint:`를 이용해서 처음 웹 뷰에 뜰 때 웹 컨텐츠의 사이즈를 설정할 수 있다. 그리고 사용자는 제스처를 이용해서 크기를 조정할 수 있다.








dddd
