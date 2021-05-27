---  
layout: post  
title: "[iOS] - 앱에 URL Sceheme 적용하기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `앱에 URL Scheme을 적용해보자.`  

---

앱에 URL Scheme을 적용한다는 것은 특정 형태의 URL로 내 앱을 호출할 수 있다는 말이다. Scheme이라는 것은 url에서 `https://~~`의 `http`에 해당한다.
내 앱의 Scheme이 예를 들어 `hello`라면, 사파리같은 곳에서 `hello://~~/~~`이런 형태로 url을 입력했을 때 OS에서 해당 scheme를 가지고
실행할 수 있는 앱을 찾아서 실행시키는 것이다.

## 개요

커스텀 url scheme는 앱 안에 있는 자원에 대해 참조할 수 있는 방법을 제공한다. 사용자는 예를 들어 이메일 안에 있는 커스텀 url을 클릭해서 내 앱의 특정 기능을 실행시킬 수 있다.

다른 앱 또한 특정한 컨텍스트 데이터를 가지고 내 앱을 실행 가능하다. 

커스텀 URL scheme가 deep linking의 형태를 띄고 있으므로, universal link를 사용하는 것이 좋다. 애플은 일반적인 시스템 앱의 scheme를 가지고 있다. `mailto`, `tel`, `sms`, `facetime` 같은 것들이 있다.

**주의**

URL Sceheme는 앱에 공격 벡터(해커가 컴퓨터나 네트워크에 접근하기 위해 사용하는 경로/방법)를 제공할 수 있으므로, 모든 URL 파라미터가 유효한지 검사하고 유효하지 않은 URL들은 차단해야 한다.
추가로, 사용자의 데이터를 손상시키지 않는 한에서의 행동들만 허용해라. 예를 들어, 다른 앱이 사용자에 대한 민감 정보에 접근하거나 내용을 지우게 하면 안된다. 코드를 이용해서 URL을 처리할 때,
테스트 케이스가 적절하지 않은 URL에 대해서도 테스트할 수 있게 해야 한다.

커스텀 URL Scheme을 지원하려면

1. 앱의 URL의 형태를 정의한다.
2. Scheme를 등록해서 시스템이 적절한 URL을 내 앱으로 연결시킬 수 있게 해야 한다.
3. 앱이 받는 URL을 처리한다.

URL은 무조건 커스텀 scheme 이름으로 시작해야 한다. 앱이 지원하는 옵션에 대한 파라미터를 추가할 수 있다. 예를 들어 이름과 사진을 보여줄 인덱스를 포함하는 URL을 받는 사진 앱이 있다고 해보자.
그러면 아래와 같은 URL이 들어올 수 있다.

```
myphotoapp:albumname?name="albumname"
myphotoapp:albumname?index=1
```

클라이언트는 scheme를 기반으로 한 URL로 앱의 `UIApplication`의 `open(_:options:completionHandler:)` 메서드를 실행시켜서 앱을 열어달라고 요청한다.
클라이언트는 앱이 URL을 열 때 시스템한테 클라이언트에 알려달라고 요청할 수 있다.

```swift
let url = URL(string: "myphotoapp:Vacation?index=1")

UIApplication.shared.open(url!) { (result) in
    if result {
       // The URL was delivered successfully!
    }
}
```

### URL Scheme 등록하기

URL scheme 등록은 어떤 URL이 앱을 열지를 명시하는 것이다. Xcode의 프로젝트 설정의 Info 탭에서 scheme를 등록한다.

![image](https://user-images.githubusercontent.com/41438361/119793768-d8591c80-bf11-11eb-85b1-e3640c16ea57.png)

* URL Scheme 부분에는 URL에 사용할 접두어를 정의한다.
* 앱의 역할을 선택한다
* 앱의 identifier를 정의한다.

제공한 identifier는 같은 scheme를 가지는 다른 앱들과 내 앱을 구별할 수 있게 해준다. 유일성을 보장하기 위해서, 회사 도메인과 앱 이름을 함친 DNS 문장을 명시한다.
(자유긴 하다.) 어떤 URL scheme는 시스템적인 측면에서 앞 뒤가 바뀌게 된다. 시스템은 잘 알려진 URL을 일치하는 시스템 앱으로 연결하고 `http` 기반의 잘 알려진 URL을 특정 앱으로 연결한다.(유튜브같은 것)
참고록 애플이 지원하는 scheme는 [여기](https://developer.apple.com/library/archive/featuredarticles/iPhoneURLScheme_Reference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007899) 
서 확인이 가능하다.

### URL 입력 들어온 것 처리하기

다른 앱이 내 커스텀 scheme를 포함한 URL을 열면, 시스템은 내 앱을 실행시키고 필요할 경우 맨 앞의 화면에 노출시킨다. 시스템은 앱 delegate의 `application(_:open:options)` 메서드를 호출해서 URL을 앱에 전달한다.
URL의 내용을 파싱하고 적절한 행동을 취하게 하는 코드를 추가한다. URL이 제대로 파싱되었는지 보장하기 위해, `NSURLComponents` API를 사용해서 url 내의 요소들을 추출하자.
내 앱을 실행시킨 다른 앱에 대한 정보와 같이 추가적인 정보를 얻으려면 시스템에서 제공된 options 딕셔너리를 이용한다.

```swift
func application(_ application: UIApplication,
                 open url: URL,
                 options: [UIApplicationOpenURLOptionsKey : Any] = [:] ) -> Bool {

    // Determine who sent the URL.
    let sendingAppID = options[.sourceApplication]
    print("source application = \(sendingAppID ?? "Unknown")")

    // Process the URL.
    guard let components = NSURLComponents(url: url, resolvingAgainstBaseURL: true),
        let albumPath = components.path,
        let params = components.queryItems else {
            print("Invalid URL or album path missing")
            return false
    }

    if let photoIndex = params.first(where: { $0.name == "index" })?.value {
        print("albumPath = \(albumPath)")
        print("photoIndex = \(photoIndex)")
        return true
    } else {
        print("Photo index missing")
        return false
    }
}
```

시스템은 또한 앱이 지원하는 커스텀 파일 타입을 열기 위해 앱 delegate의 `application(_:open:options)`를 사용한다.

앱이 `Scenes`에 맞춰져 있고 앱이 실행되지 않은 상태에서는 시스템은 URL을 실행 이후에 URL을 `scene(_:willConnectTo:options` delegate 메서드에 전달하고,
앱이 메모리에서 실행되고 있거나 suspended된 상태에서 URL을 열 때는 `scene(_:openURLContexts:)`에 전달한다.

```swift
func scene(_ scene: UIScene, 
           willConnectTo session: UISceneSession, 
           options connectionOptions: UIScene.ConnectionOptions) {

    // Determine who sent the URL.
    if let urlContext = connectionOptions.urlContexts.first {

        let sendingAppID = urlContext.options.sourceApplication
        let url = urlContext.url
        print("source application = \(sendingAppID ?? "Unknown")")
        print("url = \(url)")

        // Process the URL similarly to the UIApplicationDelegate example.
    }

    /*
     *
     */
}
```
