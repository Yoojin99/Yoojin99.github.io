---  
layout: post  
title: "[iOS] - WKWebView 새 창 열기, 닫기, `target="_blank"` 지원하기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `WKWebView에서 새 창을 열고 닫고, 새 창을 열 때 target이 blank로 처리된 경우 처리하는 법을 알아보자.`  

---

웹 뷰를 이용해서 개발을 하다가 새 윈도우 창을 열 때 blank는 지원하는 것이냐는 질문을 받고 blank가 뭔지 몰라
당황했던 기억이 있다. 구글링을 하다보니 정말 기본중에 기본... 이었는데, WKWebView에서 새 윈도우 창을 열고, 닫게 하는 방법과 
또 이 blank는 무엇인지 공부한 내용을 정리해보려고 한다.

## window.open()

새로운 윈도우 창을 띄울 때 `window.open()`이라는 코드를 사용한다. 이 때 옵션도 같이 설정할 수 있다.

보통 JS에서 새로운 창을 띄우려면 아래와 같이 코드를 작성할 것이다.

```javascript
<script type="text/javascript">
function openPop(){
    var popup = window.open('http://www.naver.com', '네이버팝업', 'width=700px,height=800px,scrollbars=yes');
}
</script>

<body>
   <a href="#none" target="_blank" onclick="openPop()">팝업</a>
</body>
```

위 코드는 링크를 클릭했을 때 네이버 창 팝업을 띄우는 것이다. 

### 팝업 여는 위치, target

이때 팝업을 띄울 위치를 설정할 수 있다.

* `target="_blank"` : 팝업을 새 창에서 연다. default 값이다.
* `target="_parent"` : 부모 창에서 팝업이 열린다.
* `target="_self"` : 현재 페이지에서 팝업이 열린다.
* `target="_top"` : 현재 페이지에서의 최상위 페이지에서 팝업이 열린다.

```js
<!--팝업을 여는곳 -->
 <a href="#none" target="_blank" onclick="openPop()">팝업</a>
```

그래서 target을 blank로 설정하면 새 빈 페이지의 윈도우 창을 띄우고 그 창에서 해당 링크로 이동하겠다는 뜻이다.

## iOS WebView에서 새 윈도우 창/팝업 열고 닫기

iOS에서 WKWebView를 사용하고 있을 때, 특정 메서드를 오버라이딩하지 않으면 `window.open()`으로 새로운 창을 열려고 할 때
열리지 않는다. 따라서 WKUIDelegate의 메서드를 오버라이딩 해줘야 한다.

이때 새 윈도우 창이나 팝업을 열고 닫는 방법은 크게 두 가지가 있다.

### 1. SubView 추가하기

내가 사용한 방법이다. 그나마 이 방법을 이용하는 것이 간단했고, 상위 뷰에서 새로운 윈도우 창을 열 때마다 하위 뷰를 더한다는 점에서
구조가 직관적이라고 생각했다. 아래의 두 메서드 모두 `WKUIDelegate`의 메서드이다. 

```swift
// 새 윈도우 창/팝업 열기
func webView(_ webView: WKWebView, createWebViewWith configuration: WKWebViewConfiguration, for navigationAction: WKNavigationAction, windowFeatures: WKWindowFeatures) -> WKWebView? {
    let newWebView = WKWebView(frame: webView.bounds, configuration: configuration)
    newWebView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
    newWebView.uiDelegate = self
    newWebView.navigationDelegate = self 
    webView.addSubview(newWebView) // 새 하위 뷰를 만들어서 추가한다.

    return newWebView
}

// 윈도우 창/팝업을 닫을 때
func webViewDidClose(_ webView: WKWebView) {
    webView.removeFromSuperview() // 상위 뷰에서 뷰를 제거한다.
}
```

### 2. JavaScript injection 사용하기

이 방법을 쓰려면 `WKNavigationDelegate`를 구현해야 한다. 위의 방법은 `WKUIDelegate`만 구현해도 됐었다.
여기에서는 `window.open()`을 했을 때, 새로운 윈도우를 여는 것 대신에 같은 위도우에서 새로운 url 주소를 로드하는 방식으로 구현한다.
또 `window.close()`를 했을 때 웹 뷰 컨트롤러가 dismiss 될 것이다.

```swift
func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
    webView.evaluateJavaScript("window.open = function(open) { return function (url, name, features) { window.location.href = url; return window; }; } (window.open);", completionHandler: nil)
    webView.evaluateJavaScript("window.close = function() { window.location.href = 'myapp://closewebview'; }", completionHandler: nil)

    if let url = navigationAction.request.url?.absoluteString, url.contains("myapp://closewebview") {
        self.dismiss(animated: true, completion: nil)
        decisionHandler(.cancel)
        return
    }

    decisionHandler(.allow)
}
```

그리고 `<a href="http://a_hyper_link" target="_blank">Display name</a>`를 지원하기 위해서 , `WKUIDelegate`의 메서드를 아래와 같이 구현한다.

```swift
func webView(_ webView: WKWebView, createWebViewWith configuration: WKWebViewConfiguration, for navigationAction: WKNavigationAction, windowFeatures: WKWindowFeatures) -> WKWebView? {
  if !(navigationAction.targetFrame?.isMainFrame ?? false) {
    webView.load(navigationAction.request)
  }
  return nil
}
```

참고로 이를 테스트하기 위한 사이트가 있으니 참고하면 아주 편할 것 같다.(https://facebook-login-demo-71f0c.firebaseapp.com/)


출처

* https://lnsideout.tistory.com/entry/jQuery-windowopen-%ED%8C%9D%EC%97%85-%EC%99%84%EB%B2%BD%EA%B0%9C%EB%85%90%EC%A0%95%EB%A6%AC
* https://stackoverflow.com/questions/25713069/why-is-wkwebview-not-opening-links-with-target-blank
* https://blog.nextzy.me/ios-handling-popup-new-window-inside-web-view-c9c91b23ac2b

