---  
layout: post  
title: "[iOS] - Universal Link"  
subtitle: ""  
categories: app
tags: app-ios universal link
comments: true  
header-img: 

---  

# Universal Link

## Universal Link란?

[WWDC 2019](https://developer.apple.com/videos/play/wwdc2020/10098/)에 나온 유니버셜 링크의 정의는

**웹, 앱에 모두 속하는 컨텐츠를 표현하는 HTTPS, HTTP URL이다.** 

Universal link는 **웹 브라우저 대신 앱에서 컨텐츠를 열게 해준다.** 이 말은 **앱 내의 컨텐츠로 연결할 수 있는 링크**라고도 할 수 있다. Universal link를 사용해서 앱 내의 특정 컨텐츠를 열 수 있다.

<img width="511" alt="image" src="https://user-images.githubusercontent.com/41438361/172602089-8b587a0d-b267-46cf-83cb-ac0230bf2f1b.png">

사용자가 universal link를 탭하거나 클릭할 때, 시스템은 Safari나 웹 사이트를 통해 라우팅하지 않고 바로 앱으로 리다이렉트한다. Universal link가 표준 HTTP / HTTPS 링크이기 때문에, 한 URL이 웹사이트와 앱에 동시에 적용될 수 있다.

* 앱이 설치되어 있지 않은 경우 : 시스템이 Safari에서 URL을 연다.
* 앱이 설치되어 있는 경우 : 웹사이트가 자기 대신에 앱이 URL을 열게 허용하는지를 검증하기 위해 웹 서버에 저장된 파일을 확인한다. 이 웹사이트와 앱 사이의 연결을 정의하는 파일은 서버에만 저장할 수 있다.

Universal link를 생성하기 위해서는

1. Associated Domains entitlement라는 특별한 자격을 앱에 추가하고
2. 웹 서버에 JSON 파일을 추가한다.

이 entitlement는 웹 서버의 도메인 이름을 언급하고, 웹 서버는 앱의 application identifier를 언급한다. 이를 통해 앱과 웹사이트 사이에 안전한 양방향 연결을 생성할 수 있고, 이를 통해 웹사이트 대신에 앱이 작업할 수 있게 된다.

앱은 universal link를 통해 소통할 수 있다. **Universal link를 사용하면 Third-party 서버 없이 다른 앱이 적은 양의 데이터를 내 앱에 직접 보낼 수 있게 된다.**

URL query 문자열을 통해 앱이 다루는 파라미터 등을 정의할 수 있다. 예를 들어 사진 앱에서 화면에 표시할 앨범과 사진의 인덱스 정보를 포함하는 파라미터를 아래와 같이 명시할 수 있다.

```
https://myphotoapp.example.com/albums?albumname=vacation&index=1
https://myphotoapp.example.com/albums?albumname=wedding&index=17
```

다른 앱들은 domain, path, parameter에 따라 URL을 만들고, 다음 메서드들을 호출해서 내 앱에서 URL을 열어달라고 요청한다.

* iOS, tvOS : `UIApplication`의 `open(_:options:completionHandler:)` 
* watchOS : `WKExtension`의 `openSystemURL(_:)`
* SwiftUI : `openURL`

URL을 통해 내 앱을 열도록 요청하는 앱은 내 앱이 URL을 열 때 시스템이 알리게도 요청할 수 있다.

아래 코드는 앱이 iOS와 tvOS에서 universal link를 호출하는 코드다.

```swift
if let appURL = URL(string: "https://myphotoapp.example.com/albums?albumname=vacation&index=1") {
    UIApplication.shared.open(appURL) { success in
        if success {
            print("The URL was delivered successfully.")
        } else {
            print("The URL failed to open.")
        }
    }
} else {
    print("Invalid URL specified.")
}
```

## Universal Link 지원하기

Universal link를 지원하려면 아래의 두 단계를 거친다.

1. 앱과 웹사이트 사이의 양방향 연결을 생성하고 앱이 처리할 수 있는 URL을 명시한다.
2. Universal link를 통해 앱이 띄워질 때 시스템이 제공한 user activity 객체를 처리하기 위해 app delegate를 업데이트한다.

사용자가 사파리 내의 웹사이트나 `WKWebView`에서 링크를 클릭하게 되면 앱이 켜지는데, 플랫폼에 따라 아래의 함수들을 호출한다.

* iOS : `open(_:options:completionHandler:)`
* watchOS : `openSystemURL(_:)`
* macOS : `open(_:withApplicationAt:configuration:completionHandler:)`
* SwiftUI : `openURL`

> 앱이 위 메서드를 사용해서 웹사이트로 이동하게 해놨다면 앱이 열리지 않을 것이다.

만약 사파리의 웹사이트에서 universal link를 클릭하면 사용자가 브라우저 내에서 작업하고 싶다는 것으로 생각해 링크를 사파리에서 연다. 만약 다른 도메인에서 universal link를 터치하면 링크를 앱에서 열게 된다.

## 1. 앱-웹사이트 양방향 연결 생성 & 앱이 처리할 수 있는 URL 명시

Associated Domain이라는 개념이 나오는데, 이름을 봤을 때 도메인들을 연관지어주는 것임을 짐작할 수 있다. **Associated Domain은 도메인과 앱 사이에 안전한 연결을 생성해서 웹사이트의 권한이나 기능들을 앱에서 공유할 수 있게 해준다.** 예를들어 온라인 소매업자는 웹사이트와 함께 앱을 제공할 수 있는데 이럴 때 사용할 수 있다.

공유된 웹 권한, universal link, Handoff, App Clip이 모두 associated doamin을 사용한다. Associated domain은 universal link의 기반이 되어 웹사이트의 컨텐츠를 앱에서 보여줄 수 있게 하는 기능을 제공한다. 만약 앱이 없는 사용자의 경우 네이티브 앱 대신에 같은 정보를 웹 브라우저에서 볼 수 있다.

웹사이트와 앱을 연결하려면 

1. 웹사이트에 associated domain 파일이 었어야 하고
2. 앱에 적절한 entitlement가 있어야 한다.

웹사이트의 `apple-app-site-association` 파일에 있는 앱들은 일치하는 [Associated Domains Entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains)가 있어야 한다.

### 웹사이트에 Associated Domain 파일 추가

사용자가 앱을 설치하면 시스템은 associated domain 파일을 다운로드하는 걸 시도하고 entitlement 내의 domain을 검증한다.

> 만약 사이트가 여러 개의 subdomain(e.g. `example.com`, `www.example.com`, `support.example.com`)을 사용하면 각각이 `Associated Domains Entitlement`에 자신의 엔트리를 가지고 있어야 하고, 각각이 자신만의 `apple-app-site-association` 파일을 가지고 있어야 한다.

웹사이트에 associated domain 파일을 추가하려면 extension 없이 `apple-app-site-association` 파일을 생성한다. 그리고 도메인에서 제공할 서비스들에 맞게 파일에 JSON 코드를 입력한다. 
Universal link를 사용하려면 `applink` 서비스 내에 내 도메인을 위한 app identifier의 목록을 명시해야 한다.

```json
{
  "applinks": {
      "details": [
           {
             "appIDs": [ "ABCDE12345.com.example.app", "ABCDE12345.com.example.app2" ],
             "components": [
               {
                  "#": "no_universal_links",
                  "exclude": true,
                  "comment": "Matches any URL whose fragment equals no_universal_links and instructs the system not to open it as a universal link"
               },
               {
                  "/": "/buy/*",
                  "comment": "Matches any URL whose path starts with /buy/"
               },
               {
                  "/": "/help/website/*",
                  "exclude": true,
                  "comment": "Matches any URL whose path starts with /help/website/ and instructs the system not to open it as a universal link"
               },
               {
                  "/": "/help/*",
                  "?": { "articleNumber": "????" },
                  "comment": "Matches any URL whose path starts with /help/ and which has a query item with name 'articleNumber' and a value of exactly 4 characters"
               }
             ]
           }
       ]
   },
   "webcredentials": {
      "apps": [ "ABCDE12345.com.example.app" ]
   },

    "appclips": {
        "apps": ["ABCED12345.com.example.MyApp.Clip"]
    }
}
```

`appIDs`와 `apps` 키는 웹사이트와 함께 사용할 수 있는 앱의 application identifier를 명시한다. 이 Application identifier는 아래의 형태로 작성해야 한다.

```
<Application Identifier Prefix>.<Bundle Identifier>
```

`details` 딕셔너리는 `applinks` 서비스 타입에만 사용된다. `components` 키는 값으로 딕셔너리들의 배열을 갖고 있는데, 각 딕셔너리에서는 URL에 대한 패턴 매칭을 제공한다. `applinks` 를 작성하는 자세한 방법을 보려면 [Apple Developer 문서](https://developer.apple.com/documentation/bundleresources/applinks)를 확인하면 된다.

Association 파일을 만들면 사이트의 `.well-known` 디렉토리에 위치시킨다. 파일의 URL은 아래의 형태를 띄고 있어야 한다.

```
https://<fully qualified domain>/.well-known/apple-app-site-association
```

파일을 유효한 인증과 함께 `https://`러 host해야 하고 리다이렉션은 없어야 한다.

### 앱에 Associated Domains Entitlement 추가

Xcode에서 target의 Signing & Capabilities 탭을 열고, Associated Domains capability를 추가한다. 만약 없는 상태라면 [Associated Domains Entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains)를 앱에 추가하고, app ID에 associated domains 기능을 추가하면 된다.

<img width="1102" alt="image" src="https://user-images.githubusercontent.com/41438361/172723871-1918dc4d-c29a-4c87-b990-2ea22f9d29d6.png">

> watchOS 앱에서는 꼭 WatchKit Extension 타겟에 Associated Domains capability를 추가해야 한다.

Entitlement에 domain을 추가하려면 Domain 표 하단의 Add 버튼을 클릭한다. 그리고 앱이 지원할 서비스를 위한 적절한 prefix와 사이트의 domain을 입력한다. 요구되는 subdomain과 top-level domain만 포함하도록 해야 한다. path, query 문, 슬래시 `/`를 붙이지 않게 주의한다.

<img width="1080" alt="image" src="https://user-images.githubusercontent.com/41438361/172724309-45d21f1e-675c-46ef-a8b8-4950e750ed10.png">

앱과 credential을 공유하는 도메인을 추가한다. `appclips`를 제외한 서비스에서 `*.` 로 domain의 prefix를 써서 모든 subdomain과 일치하게 할 수 있다. 위에서 작성한 도메인은 아래의 형태를 띄고 있어야 한다.

```
<service>:<fully qualified domain>
```

iOS 14, macOS 11부터는 앱이 웹 서버의 `apple-app-site-association` 파일에 접근하기 위해 서버쪽에 요청을 바로 보내지 않고, 특정 도메인에 종속된 Apple-managed content delivery network(CDN)에 이런 요청들을 보낸다.

만약 앱을 개발할 때 웹 서버가 공용 인터넷에 접근할 수 없다면 alternate mode 기능을 사용해서 CDN을 거치게 하고, private한 도메인에 연결하게 할 수 있다. Alernate mode는 아래의 query 문을 associated domain의 entitlement에 추가하면 된다. 이 Associated Domains Entitlement를 작성하는 자세한 방법은 [Apple Developer 문서](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains)에서 확인하면 된다.

```
<service>:<fully qualified domain>?mode=<alternate mode>
```
 
## 2. 시스템이 제공한 user activity 객체를 처리하기 위해 app delegate를 업데이트

위에서 앱-웹 간의 양방향 연결을 설정했다.

사용자가 universal link를 터치하면, 시스템은 앱을 열고 `NSUSerActivity` 객체를 전달한다. 이 객체를 처리해서 앱이 어떻게 열릴 것인지, 어떤 작업을 할 것인지 정할 수 있다.

여기에서는 `activityType`이 `NSUserActivityTypeBrowsingWeb`으로 설정된 `NSUserActivity` 객체를 받을 때 app delegate가 처리하는 동작을 구현한다.

> Universal link는 앱에 attack vector를 제공할 가능성이 있으니 모든 URL 파라미터를 검증하고, 이상한 URL은 사용하지 말아야 한다. 추가로 가능한 동작들을 제한해서 사용자의 데이터로 이상한 처리를 하지 않도록 예방해야 한다. 예를 들어 universal link가 컨텐츠를 바로 제거할 수 있거나 사용자의 민감한 정보에 접근할 수 없게 해야 한다. 

### App Delegate가 Universal link에 대응하게 하기

Universal link를 터치해서 시스템이 앱을 열면 앱은 `activityType`이 `NSUserActivityTypeBrowsingWeb`인 `NSUserActivity` 객체를 받게 된다. Activity 객체의 `webpageURL` 프로퍼티를 사용자가 접근하는 HTTP, HTTPS URL을 포함한다. `NSURLComponents` API를 사용해서 URL의 요소를 추출할 수 있다.

```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool
{
    // Get URL components from the incoming user activity.
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
        let incomingURL = userActivity.webpageURL,
        let components = NSURLComponents(url: incomingURL, resolvingAgainstBaseURL: true) else {
        return false
    }

    // Check for specific URL components that you need.
    guard let path = components.path,
    let params = components.queryItems else {
        return false
    }    
    print("path = \(path)")

    if let albumName = params.first(where: { $0.name == "albumname" } )?.value,
        let photoIndex = params.first(where: { $0.name == "index" })?.value {

        print("album = \(albumName)")
        print("photoIndex = \(photoIndex)")
        return true

    } else {
        print("Either album name or photo index missing")
        return false
    }
}
```

만약 app이 `Scenes`에 맞춰져 있고 앱이 실행중이지 않을 때 시스템은 실행 이후에 universal link를 `scene(_:willConnectTo:options:)` delegate  메서드에 전달하고,
앱이 실행중이거나 메모리에서 suspended 됐을 때는 `scene(_:continue:)`에 전달한다.

```swift
func scene(_ scene: UIScene, willConnectTo
           session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    
    // Get URL components from the incoming user activity.
    guard let userActivity = connectionOptions.userActivities.first,
        userActivity.activityType == NSUserActivityTypeBrowsingWeb,
        let incomingURL = userActivity.webpageURL,
        let components = NSURLComponents(url: incomingURL, resolvingAgainstBaseURL: true) else {
        return
    }

    // Check for specific URL components that you need.
    guard let path = components.path,
        let params = components.queryItems else {
        return
    }
    print("path = \(path)")

    if let albumName = params.first(where: { $0.name == "albumname" })?.value,
        let photoIndex = params.first(where: { $0.name == "index" })?.value {
        
        print("album = \(albumName)")
        print("photoIndex = \(photoIndex)")
    } else {
        print("Either album name or photo index missing")
    }
}
```

## Universal Link 테스트

* https://search.developer.apple.com/appsearch-validation-tool/
* https://limitless-sierra-4673.herokuapp.com/

위 두 사이트를 사용해서 universal link가 잘 작동하는지 테스트할 수 있다.


* 참조
* https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content?preferredLanguage=occ
* https://developer.apple.com/videos/play/wwdc2020/10098/?time=31
* https://developer.apple.com/documentation/Xcode/supporting-associated-domains
* https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app
* https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains
* https://gist.github.com/anhar/6d50c023f442fb2437e1#general-link-resources
* https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html
