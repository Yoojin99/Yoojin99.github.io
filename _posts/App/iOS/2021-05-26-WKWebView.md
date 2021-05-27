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
컨텐츠를 앱의 네이티브 뷰와 같이 보여준다. 웹 기술을 사용하는 것이 네이티브 뷰를 사용해서 앱을 스타일링하는 것보다 더 가독성이 좋을 때 이 것을 사용하는 것이 좋다.
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

### Web Content를 통해 Navigation 관리하기

WKWebView는 두 페이지 간에 링크를 통해 이동하는 것, 앞으로 뒤로가기 버튼 등등을 포함한 브라우징 경험을 제공한다. 사용자가 컨텐츠의 링크를 클릭하면, 웹 뷰는 브라우저처럼 행동하고 그 링크의 컨텐츠를 화면에 출력한다. Navigation을 비활성하거나, navigation 행동을 커스터마이징 하고 싶으면 웹 뷰에 navigation delegate를 제공한다. 이 프로토콜은 `WKNavigationDelegte`를 말한다. Navigation delegate를 이용해서 웹 뷰의  navigation 행동을 수정하거나, 새로운 컨텐츠의 로딩 상황을 추적할 수 있다.

또한 WKWebView의 메서드를 이용해서 컨텐츠 간의 이동을 코드로 설정할 수 있거나 앱 인터페이스의 다른 부분으로 이동을 할 수 있다. 예를 들어 UI가 앞으로 뒤로 가기 버튼이 있다면, 이 버튼에 각각 `goForward:`와 `goBack:` 메서드를 제공해서 일치하는 웹 이동이 일어나게 웹 뷰를 설정한다. `canGoBack`과 `canGoForward` 프로퍼티를 이용해서 버튼을 활성화 시킬지 비활성화 시킬지 결정할 수 있다.

### Native UI 요소 화면에 출력하기 : WKUIDelegate

WKUIDelegate는 프로토콜로, 웹페이지 대신에 navitve UI 요소를 출력하는 메서드들이 있다.

웹 뷰 UI delegate 는 새로운 창을 열거나, 사용자가 요소를 클릭할 때 출력되는 기본 메뉴 아이템의 행동을 추가하거나, 다른 UI 관ㄹㄴ된 작업을 수행하기 위해 이 프로토콜을 구현했다. 이 메서드들은 JavaScript나 다른 플러그인 된 컨텐츠를 다루는 것의 결과로 실행될 수 있다. 기본 웹 뷰 구현은 웹 뷰 하나에 한 윈도우를 가정하고 있으므로, 이를 따르지 않는 다른 UI는 이 것을 구현해야 한다.

이 WKUIDelegate에서 선언한 메서드 중 유의해서 봐야 할 메서드가 3개가 있다. JavaScript로 팝업을 띄웠을 때 원래는 Web View에서는 아무 동작도 하지 않지만, 아래의 3개의 메서드를 구현해서 JavaScript로 팝업을 띄울 때 webview에서도 똑같이 팝업이 뜨게 설정할 수 있다.

#### JavaScript alert() 띄우기

JavaScript 상에서 `alert()`을 했을 때 이 alert popup을 앱에서도 똑같이 띄우기 위해서는 아래 메서드를 구현한다.

```swift
optional func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void)
```

이 중에서도 completionHandler는 이 alert 패널이 해제되고 난 후에 호출되는 핸들러다.

구현 예시는 아래와 같다.

```swift
func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void) {
    let alert = UIAlertController(title: message, message: nil, preferredStyle: .alert) // alertController
    let ok = UIAlertAction(title: "OK", style: .default, handler: { (action) in 
        completionHandler()
    }) // 확인 버튼의 텍스트를 설정하고, 이를 눌렀을 때 completionHandler를 실행시키는 action을 지정한다.
    alert.addAction(ok) // alertController에 action을 추가한다.

    self.present(alert, animated: true, completion: nil) // alert Controlller를 띄운다.
}
```

그리고 이것을 테스트해보면 아래와 같이 팝업이 잘 뜨는 것을 확인할 수 있다. 이 메서드를 구현하지 않으면 팝업이 앱에서 뜨지 않는다.

![image](https://user-images.githubusercontent.com/41438361/119765089-7ab2d900-beed-11eb-82dd-34ee4e76951f.png)

#### JavaScript confirm() 띄우기

이번에는 아래 메서드를 구현한다.

```swift
optional func webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (Bool) -> Void)
```

completionHandler가 인자로 받는 bool 값은 사용자가 'ok'를 누르면 true, 'cancel'을 누르면 false를 넣어준다.

나는 아래와 같이 구현했다.

```swift
func webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (Bool) -> Void) {
    let alert = UIAlertController(title: message, message: nil, preferredStyle: .alert)
    alert.addAction(UIAlertAction(title: "OK", style: .default, handler: { (action) in completionHandler(true)
    })) // ok 누를 때
    alert.addAction(UIAlertAction(title: "Cancel", style: .default, handler: { (action) in completionHandler(false)
    })) // cancel 누를 때

    self.present(alert, animated: true, completion: nil)
}
```

그러면 아래와 같이 화면에 confirm 팝업이 뜬다.

![image](https://user-images.githubusercontent.com/41438361/119765430-3bd15300-beee-11eb-9b04-0f240f8ba1ef.png)

#### JavaScript prompt() 띄우기

```swift
optional func webView(_ webView: WKWebView, runJavaScriptTextInputPanelWithPrompt prompt: String, defaultText: String?, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (String?) -> Void)
```

위 메서드를 아래와 같이 구현했다.

```swift
 func webView(_ webView: WKWebView, runJavaScriptTextInputPanelWithPrompt prompt: String, defaultText: String?, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (String?) -> Void) {
    let alert = UIAlertController(title: "", message: nil, preferredStyle: .alert)
    alert.addTextField{ (textField) in
        textField.text = defaultText
    } // 텍스트 필드 안에 기본으로 있는 텍스트 설정

    alert.addAction(UIAlertAction(title: "OK", style: .default, handler: { (action) in
        guard let text = alert.textFields?.first?.text else {
            completionHandler(defaultText)
            return
        }
        completionHandler(text)
    })) // ok 눌렀을 때

    alert.addAction(UIAlertAction(title: "Cancel", style: .default, handler: { (action) in completionHandler(nil)
    })) // cancel 눌렀을 때

    self.present(alert, animated: true, completion: nil)
}
```

화면은 아래와 같이 나온다.

![image](https://user-images.githubusercontent.com/41438361/119765913-09742580-beef-11eb-9d5a-98adfa68f35e.png)


### WebKit이 Content를 불러올 수 있는지 판단하기 : handlesURLScheme

WKWebKit에는 `handlesURLScheme`이라는 클래스 함수(타입 메서드)가 있다.

이 함수는 WebKit이 특정 URL scheme를 지원하는지를 판단해서 bool 값을 리턴한다.

```swift
class func handlesURLScheme(_ urlScheme: String) -> Bool
```

위의 `urlScheme` 에는 url scheme을 넣으면 된다. 만약 WebKit이 URL scheme을 지원하면 true를 반환하고, 그렇지 않으면 false를 반환한다.

예를들어 아래와 같이 코드를 작성했다고 하자.

```swift
print(WKWebView.handlesURLScheme("http")) // true
print(WKWebView.handlesURLScheme("InvalidURLScheme")) // false
```

WKWebView는 "http"라는 url scheme를 제공하기 때문에 true를 반환하고, "InvalidURLScheme"은 내가 임의로 만든 현재 유효하지 않은 scheme이므로 false를 반환한다.

### 웹페이지 사이에 이동을 관리하기 : WKNavigationDelegate

WKNavigationDelegate는 프로토콜로, 화면 이동을 허용할 것인지 차단할 것인지에 대한 메서드와 화면 이동 요청을 추적하는 것에 대한 메서드들이 정의되어 있다.

웹 뷰의 메인 프레임에서의 변화를 조정하기 위해 WKNaviationDelegate의 메서드를 구현하면 된다. 사용자가 웹 컨텐츠를 navigate하려고 할 때, 웹 뷰는 모든 전환을 관리하기 위해 navigation delegate를 이용한다. 예를 들어, 이 ㅁ세더들을 사용해서 컨텐츠간 이동하는 특정 링크를 제한할 수 있다. 

