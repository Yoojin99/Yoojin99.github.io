---  
layout: post  
title: "[iOS] - User Notificaation, 푸시 알림 띄우기"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `앱에 notification를 띄우는 법을 알아보자.`  

---

이 글은 Notification을 앱에서 띄우기 위한 개념에 대해서만 정리했다. 실제로 띄우는 것을 구현하려면 [이 글](https://yoojin99.github.io/app/Push-%EC%95%8C%EB%A6%BC-%EB%9D%84%EC%9A%B0%EA%B8%B0/)을 보자.

애플에는 user notification을 띄울 수 있는 [프레임워크](https://developer.apple.com/documentation/usernotifications)가 있다.
Notification와 push를 따로 말하는 이유는 통상적으로 화면 상단에 뜨는 앱 알림을 'push'라고 일반적으로 말하긴 했지만 앱 내부에서 알림을 띄우는 것과,
앱 외부(서버)에서 알림을 보내는 것을 구분하기 위해서다. Push는 앱 외부에서 알림을 보내는 경우에 push라고 한다.

먼저 이 프레임 워크부터 보도록 하자.

## User Notifications

Notification 는 서버에서 보낼 수도 있고, 앱 내에서도 보낼 수 있다.

### 개요

사용자에게 보여지는 알림은 앱이 구동중이든 아니든 중요한 정보를 사용자에게 전달한다. 예를 들어 스포츠 앱은 사용자에게 가장 좋아하는 팀의 점수를 푸시로 알려줄 수도 있다.
Notification은 또한 앱에게 정보를 다운로드하고 인터페이스를 업데이트하도록 할 수 있다. Notification은 alert를 띄우고, 소리를 낼 수 있고, 앱 아이콘을 뱃지로 보여줄 수 있다.

![image](https://user-images.githubusercontent.com/41438361/119909980-1a757300-bf91-11eb-9c91-8b38f68748b7.png)

앱 내에서나 서버에서 알림을 발생시킬 수 있다. 앱 내부에서 알림을 띄운다면, 앱은 notification content를 생성하고 알림을 발생시킬 시간이나 지역과 같은 조건을 명시한다.
앱 외부에서 보내는 notification(push)은 notification을 보내기 위해 회사 서버를 사용하고, Apple Push Notification service(APNs)가 사용자
기기로의 알림 전달을 다룬다.

이 프레임 워크는 아래의 것들을 할 때 사용한다.

* 앱이 지원하는 notification의 타입을 정의한다.
* Notification 타입과 관련있는 커스텀 action을 정의한다.
* 앱 내부에서 알림을 띄우는 경우에 알림을 띄울 시간을 정한다.
* 이미 전달된 알림을 처리한다.
* 사용자가 선택하는 action에 반응한다.

원래 의도는 local(앱 내부)와 remote(서버) notification을 시간에 맞게 보내는 것이지만, 보장되지는 않는다. PushKit 프레임워크는 VoIP와 watchOS에서 특정 notification 타입ㅇ 대해
더 시간적으로 잘 맞춰주는 전달 기술을 채택하고 있다. 

## 알림 띄울 권한 요청하기

Notification을 띄울 때 alert, sound, 앱 아이콘 badge를 띄울 수 있는 권한을 요청하는 것이다.

### 개요

Local/remote notification은 alert을 띄우고, 소리를 내고, 앱 아이콘 배지를 보여줘서 사용자의 주의를 끈다. 이런 상호작용은 앱이 동작하고 있지 않거나
백그라운드에서 돌아가고 있을 때 발생한다. Notification은 사용자에게 앱과 관련된 정보를 보여줄 게 있다고 알려준다. 사용자가 알림 기반의 상호작용을 좋아하지 않을 수 있으므로 알림을 띄울 권한을 요청해야 한다.

![image](https://user-images.githubusercontent.com/41438361/119912120-c5882b80-bf95-11eb-8213-733a4bd2f19a.png)

### 문맥에 맞게 명시적으로 승인을 요청하기

권한 허용을 요청하기 위해, 공유된 `UNUserNotificationCenter` 인스턴스를 가져와서 `requestAuthorization(options:completionHandler:)` 메서드를 호출해라.
앱이 채택하는 모든 상호작용 타입을 명시해야 한다. 예를 들어, alert을 띄우고, 앱 아이콘 배지를 추가하거나 소리를 낼 권한을 요청할 수 있다.

```swift
let center = UNUserNotificationCenter.current()
center.requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
    
    if let error = error {
        // Handle the error here.
    }
    
    // Enable or disable features based on the authorization.
}
```

처음 권한을 요청할 때, 시스템은 사용자가 허용했는지 거절했는지를 답하도록 하고 사용자의 응답을 기억한다. 이후에 또 권한 요청을 한다면 사용자에게 띠워지지 않을 것이다.
사용자가 인증이 필요한 이유를 이해할 수 있게 요청을 문맥에 맞게 해야한다. 리마인더 알림을 보내는 작업 트래킹 앱은 사용자가 첫 작업을 설정한 이후에 요청을 날릴 수 있다.
문맥에 맞게 권한을 요청하는 것은 단순히 처음 앱을 실행했을 때 권한을 요청하는 것보다 더 나은 UX를 제공한다. 이러면 사용자는 알림이 제공되는 목적을 더 쉽게 이해할 수 있다.

### 알림을 보내는 시도를 하기 위해 Provisional Authorization 사용하기

알림을 보내겠다는 권한을 명시적으로 요청할 때, 사용자는 앱이 보내는 알림을 보기도 전에 권한을 허용하거나 거절해야 한다. 권한을 요청하기 전에 context를
잘 설정했다고 해도 사용자는 이를 결정하기 힘들 수도 있다.

시도적으로 알림을 보내기 위해 provisional authorization을 사용해라. 사용자는 알림을 보고 이를 허용할지 안할지 결정할 수 있다.

시스템은 이런 알림을 조용히 보낸다-소리나 배너를 띄우지 않고, 잠금 화면에 띄우지 않는다. 대신 notification center의 기록에 띄운다. 이런 알림은
사용자가 계속 알림을 받을 건지 아니면 끌지를 선택하게 하는 버튼을 포함시킬 수 있다.

![image](https://user-images.githubusercontent.com/41438361/119913430-e1410100-bf98-11eb-954b-f4bd5eaa33a2.png)

만약 사용자가 Keep 버튼을 클릭하면, 시스템은 알림을 눈에 띄게 할지 아니면 조용히 하게 할지 선택하게 한다. 만약 사용자가 눈에 띄게 하는 알림을 선택했다면,
앱은 provisional authorization에 포함된 모든 권한을 얻게 된다. 사용자가 조용한 알림을 선택했다면, 시스템은 앱이 알림을 보낼 수 있게 허용은 하지만 alert, sound, 앱 아이콘 배지를 띄우지 않게 한다.
이 경우 알림은 오직 notification center history에만 뜨게 된다.

만약 사용자가 Turn Off 버튼을 눌렀을 때는 시스템은 추가적인 알림을 보내기 위해 권한을 요청하는 것을 거절하기 전에 이 선택을 확인한다.

Provisional authorization을 보내기 위해, 알림을 보낼 권한을 요청할 때 `provisional` 옵션을 추가하면 된다.

```swift
let center = UNUserNotificationCenter.current()
center.requestAuthorization(options: [.alert, .sound, .badge, .provisional]) { granted, error in
    
    if let error = error {
        // Handle the error here.
    }
    
    // Provisional authorization granted.
}
```

명시적으로 권한을 요청하는 것과 다르게, 이 코드는 사용자에게 알림을 받을 권한을 요청하지 않는다. 대신 이 메서드를 처음 호출할 때, 자동으로 권한을 획득한다.
하지만 사용자가 명시적으로 keep하거나 turn off 하기 전까지 권한 상태는 `UNAuthorizationStatus.provisional`이다. 사용자가 권한 상태를 언제든지 바꿀 수 있기 때문에,
local notification을 보내기 전에 상태를 계속 확인해야 한다.

추가로, provisional authorization을 요청했다면, 처음 앱이 실행됐을 때 승인을 요청할 수 있다. 사용자는 이 알림을 실제로 받았을 때 알림을 킬지 아니면 끌 지 확인할 수 있다.

### 현재 승인된 권한을 바탕으로 notification 커스텀하기

항상 local notification을 보내기 전에 앱의 권한 상태를 체크해야 한다. 사용자는 어느 때든 앱의 권한 설정을 바꿀 수 있다. 또한 앱과 상호작용하는 타입을 바꿀 수 있고, 이는 앱이 보내는
notification의 수나 타입을 바꿔야 할 수도 있다.

사용자에게 가장 좋은 경험을 제공하기 위해, notification center의 `getNotificationSettings(completionHandler:)`를 호출해서 현재 알림 설정을 불러온다.
그리고 이 설정들을 기반으로 알림을 커스터마이징한다.

```swift
let center = UNUserNotificationCenter.current()
center.getNotificationSettings { settings in
    guard (settings.authorizationStatus == .authorized) ||
          (settings.authorizationStatus == .provisional) else { return }

    if settings.alertSetting == .enabled {
        // Schedule an alert-only notification.
    } else {
        // Schedule a notification with a badge and sound.
    }
}
```

위 코드는 앱이 권한이 없을 때 알림을 스케줄링 하는 것을 방지하기 위해 guard 절을 사용했다. 그리고 코드는 허용된 상호작용 타입을 기반으로 notification을 설정한다.
여기서는 alert-based 알림을 가능한 사용하는 것으로 보인다.

하지만 앱이 어떤 종류의 상호작용이 허용되지 않았음에도 alert, sound, badge를 포함한 알림을 보내고 싶을 수 있다. 시스템은 `UNNotificationSetting`의 인스턴스의
`notificationCenterSetting` 프로퍼티가 `UNNotificationSetting.enabled`로 설정되어 있는 한 Notification Center에 알림을 띄운다.
Notification center delegate의 `userNotificationCenter(_:willPresent:withCompletionHandler:)` 메서드는 앱이 화면에 노출되었을 때도
알림을 받을 수 있고, alert, sound, badge 정보에 여전히 접근할 수 있다.

## Remote Notification

### Remote Notification Server 세팅하기

Notification을 생성해서 사용자 디바이스에 푸시할 것이다. 이를 세팅하려면 크게 두 가지 작업을 해야 한다.

1. 서버 쪽 설정하기
2. 보안 연결 설정하기(서버와 APNs)

#### 개요

Remote notification(push notification)을 사용해서 앱이 동작하고 있지 않을 때에도 앱을 사용하는 기기에 작은 양의 데이터를 푸시한다. 앱은
사용자에게 중요한 정보를 제공하기 위해 notification을 사용한다. 예를 들어, 메세지 서비스는 새로운 메세지가 왔을 때 원격 notification을 보낼 수 있다.

Remote notification을 전달하는 것은 몇 개의 핵심 요소를 포함한다.

* 서버(provider server)
* Apple Push Notification service(APNs)
* 사용자 기기
* 사용자 기기에서 돌아가고 있는 앱

원격 알림은 회사의 서버로부터 시작된다. 어떤 알림을 보낼 것인지 선택해서 사용자들에게 보낸다. 알림을 보낼 때가 되면, 알림 데이터와 사용자 기기를 식별하는
고유의 식별자를 가지고 request를 생성한다. 그리고 이 요청을 사용자 기기로 알림을 전달하는 것을 관리하는 APNs로 보낸다. 알림이 받아지면, 사용자 기기의
 os는 사용자 상호작용을 다루고 이를 앱으로 보낸다.
 
![image](https://user-images.githubusercontent.com/41438361/120070068-257cf000-c0c4-11eb-81db-84f20d1ce5a5.png)

개발자는 provider 서버를 세팅하고 사용자 기기의 알림을 다루도록 앱을 설정할 책임이 있다. 애플은 사용자에게 알림을 보여주는 것을 포함해서 그 사이에 있는 모든 것을 관리한다.
서버와 통신하고 적절한 정보를 제공하는 사용자 기기에서 실행되는 앱이 꼭 있어야 한다. 

#### 알림을 위한 커스텀 환경 설정하기

몇개의 필수 작업을 수행해서 remote notification server를 설정하자. 

* 사용자 기기에서 실행되는 앱의 인스턴스에서 device token을 받고, 이 토큰을 사용자 계정과 관련짓는 코드를 작성하자. [앱을 APNs에 등록해야 한다.](https://developer.apple.com/documentation/usernotifications/registering_your_app_with_apns)
* 알림을 사용자에게 언제 보낼지 결정하고, 알림 payload를 생성하는 코드를 작성한다. [Remote notification을 생성한다.](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification)
* Payload를 포함하는 POST request를 생성하고, 이를 HTTP/2 연결로 보낼 코드를 작성한다. [Notification request를 APNs로 보낸다.](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns)
* Token 기반의 인증을 위해 토큰을 일시적으로 재생성한다. [Token 기반의 연결을 APNs에 설정한다.](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server)

#### APNs에 신뢰되는 연결을 설정해야 한다.

Provider 서버와 APNs는 연결을 안전하게 설정해야 한다. 이를 설정하려면 [GeoTrust Global CA root certificate](https://www.digicert.com/kb/digicert-root-certificates.htm) (2021/3/29까지)와 [AAA Certificate Services root certificate](https://sectigo.com/knowledge-base/detail/AAA-Certificate-Services-Root-2028/kA03l00000117cL) (2021/3/29 부터) 를 각각 provider server에 다운받아야 한다.

만약 provider server가 macOS라면, GeoTrust Global CA root certificate는 기본적으로 키체인 안에 있다. 만약 provider server가 macOS 10.14나 이후 버전을 실행한다면, AAA Certificate Services root certificate는 키체인에 기본적으로 있다. 다른 시스템에서는 이 인증들을 다운받아야 한다.

알림을 보내기 위해, provider server는 token-based나 certificate-based 의 신뢰 연결을 HTTP/2와 TLS를 이용해서 APNs와 설정해야 한다.
각각의 기술은 장단점이 있기 때문에, 본인의 상황에 맞는 것을 쓰면 된다.

* [Token-based 연결 설정하기](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/establishing_a_token-based_connection_to_apns)
* [Certificate-based 연결 설정하기](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/establishing_a_certificate-based_connection_to_apns)

#### APNs 가 제공하는 것이 무엇인지 알기

APNs는 알림을 제공하기 위한 것들을 한다.

* APNs는 사용자 기기와 공인된, 암호화된, 그리고 지속적인 IP 연결을 관리한다.
* APNS는 현재 오프라인인 기기를 위해 알림을 저장할 수 있다. 그리고 online이 될 때 저장된 알림을 전달한다.
* APNs 같은 번들 ID를 가진 알림들을 합칠 수 있다.

이제 위에서 언급한 작업들을 하나씩 해보겠다.

### 앱을 APNs에 등록하기

Apple Push Notification service(APNs)와 통신해서 앱을 식별하는 고유한 디바이스 토큰을 받는다.

#### 개요

APNs는 알림을 기기로 보내기 전에 사용자 기기의 주소를 무조건 알아야 한다. 이 주소는 기기와 앱을 모두 고유하게 식별할 수 있는 디바이스 토큰의 형태를 따른다. 실행될 때, 앱은 APNs와 통신해서 기기 토큰을 받아 이를 나중에 provider server에 보낸다. 서버는 이후 보내는 알림에 이 토큰을 포함시킨다.

*기기 토큰은 한 앱에서만 사용가능하고 다른 앱이 같은 기기에 설치되어 있다고 해도 다른 앱에서는 사용할 수 없다. 두 앱 모두 자신만의 고유한 디바이스 토큰이 있고 이를 Provider server에 전달한다.*

#### Push Notifications Capability 활성화하기

요구되는 자격을 부여하기 위해, Xcode 프로젝트에서 Push Notifications capability를 아래와 같이 활성화시킨다. 프로젝트 설정의 Signing&Capabilities에서 왼쪽 윗부분에 있는 + Capability 버튼을 누르면 설정할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/120070928-d6d15500-c0c7-11eb-9328-2fe255e52159.png)

이 옵션을 활성화 시키면 iOS에서는 [APS Environment Entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/aps-environment) 를 앱에 추가하는 것이다. macOS에서는 [APS Environment (macOS) Entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_aps-environment)를 추가한다.

*개발자 계정에서는 프로젝트에 할당된 앱 ID에도 push notification service를 활성화해야 한다. 개발자 계정을 설정하려면 [Developer Account](https://idmsa.apple.com/IDMSWebAuth/signin?appIdKey=891bd3417a7776362562d2197f89480a8547b108fd934911bcbea0110d07f757&path=%2Faccount%2F&rv=1#/overview/) 페이지에서 설정할 수 있다.*

#### 앱을 등록하고 앱의 디바이스 토큰 받기

APNs에 앱을 등록하고 현재 기기에 있는 앱의 주소를 알 수 있는 고유한 디바이스 토큰을 받는다. Provider server는 기기에 알림을 보내기 전에 무조건 이 토큰을 받아야 한다. 

Apple에서 제공된 API를 이용해서 앱이 실행될 때마다 앱을 동록하고 디바이스 토큰을 받는다. 등록 프로세스는 플랫폼에 따라 별 차이가 없다.

* iOS/tvOS : 디바이스 토큰을 받기 위해 `UIApplication`의 `registerForRemoteNotifications()` 메서드를 호출한다. 정상적으로 등록이 되었으면, 앱 delegate의 `application(_:didRegisterForRemoteNotificationsWithDeviceToken:)` 메서드를 사용해서 토큰을 받는다.
* macOS : 디바이스 토큰을 받기 위해 `NSApplication`의 `registerForRemoteNotifications()` 메서드를 호출한다. 정상적으로 등록이 되었다면, 앱 delegate의 `application(_:didRegisterForRemoteNotificationsWithDeviceToken:)` 메서드를 사용한다.

APNs에 성공적으로 등록된 것을 관리하는 것에 추가로, `application(_:didFailToRegisterForRemoteNotificationsWithError:)` 메서드를 구현해서 등록이 실패했을 때를 대비한다. 사용자의 기기가 네트워크에 연결되어 있지 않거나, APNs가 무언가의 이유로 연결할 수 없을 때, 혹은 앱이 code-siging 자격을 갖추고 있지 않을 때 등록이 실패할 수 있다. 등록이 실패하면 플래크를 세우고 잠시 후에 다시 시도해본다.

아래 코드는 remote notifications을 위해 등록하고 일치하는 토큰을 받기 위한 iOS 앱 delegate 메서드의 예시를 보여준다. `sendDeviceTokenToServer` 메서드는 앱이 데이터를 provider server에 보내기 위해 사용하는 커스텀 메서드다.

```swift
func application(_ application: UIApplication,
           didFinishLaunchingWithOptions launchOptions:
           [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
   // Override point for customization after application launch.
               
   UIApplication.shared.registerForRemoteNotifications() // APNs에 등록하는 프로세스를 시작한다. 성공하면 바로 밑의 메서드에 디바이스 토큰을 전달한다. 
   return true
}

func application(_ application: UIApplication,
            didRegisterForRemoteNotificationsWithDeviceToken 
                deviceToken: Data) { // 여기서 전달받은 토큰을 원격 알림을 생성하기 위해 서버에 전달해야 한다. 
   self.sendDeviceTokenToServer(data: deviceToken)
}

func application(_ application: UIApplication,
            didFailToRegisterForRemoteNotificationsWithError  
                error: Error) { // 등록이 실패하면 이게 호출된다.
   // Try again later.
}
```

*토큰을 로컬 저장소에 절대로 캐싱하면 안된다. APNs는 사용자가 앱을 새 기기에 설치할 때나 사용자가 os를 재설치할 때, 사용자가 백업에서 기기를 복원할 때 새 토큰을 발행한다. 만약 시스템이 이때마다 토큰을 제공하게 한다면, 업데이트 된 토큰을 받는 것이 확실해야 한다.*

#### 토큰을 Provider Server에 전달하기

디바이스 토큰을 받으면, 앱에서 provider server로 네트워크 연결을 설정한다. 이 디바이스 토큰과 사용자를 식별하기 위한 추가 정보를 보안 연결을 통해 서버에 전달한다. 예를 들어, 사용자의 로그인 이름이나 그 외의 것들이 포함될 수 있다. 네트워크에 보내는 어떤 정보라도 암호화되어야 한다.

Provider 서버에서, 알림을 보내기 위해 접근할 수 있는 안전한 곳에 토큰을 저장한다. 알림을 생성할 때, 서버는 알림을 특정 디바이스에 보낼 수 있어야 한다.
그래서 알림이 사용자 계정에 연결되었다면, 디바이스 토큰을 사용자의 계정 정보와 함께 저장해야 한다. 사용자가 여러개의 기기를 가지고 있을 수 있기 때문에, 여러개의 디바이스 토큰을 다루는 것에 대한 준비도 해야 한다.

## 알림 관리

### UNUserNotificationCenter : 앱이나 앱 extension에 알림과 관련된 동작들을 관리하는 객체

앱과 앱 extension에서 알림과 관련된 모든 행위들을 관리하기 위해 공유된 `UNUserNotificationCenter` 객체를 사용한다. 이 객체는 아래의 경우에 사용한다.

* 알림을 띄울 때 alert, sound, icon badge와 같이 사용자와 상호작용할 권한을 요청하기. (권한은 모든 사용자 상호작용에서 요청된다.) [위](https://developer.apple.com/documentation/usernotifications/asking_permission_to_use_notifications)에서도 잠깐 언급했다.
* 알림이 전달되었을 때 앱이 지원하는 알림 타입과 사용자가 수행할 커스텀 행위를 선언한다. [여기](https://developer.apple.com/documentation/usernotifications/declaring_your_actionable_notification_types)에서 actionable한 알림 타입을 선언하는 것에 대해 더 볼 수 있다.
* 앱에 알림을 전달하는 시간을 설정한다. [여기](https://developer.apple.com/documentation/usernotifications/scheduling_a_notification_locally_from_your_app)에서 확인할 수 있다.
* APNs에 의해 전달된 원격 알림의 payload를 처리한다. [여기](https://developer.apple.com/documentation/usernotifications/handling_notifications_and_notification-related_actions)에서 더 확인할 수 있다.
* 이미 전달된 알림과 Notification Center에 표시된 알림을 관리한다.
* 커스텀 알림 타입과 관련있는 사용자 선택 행위를 관리한다. [여기](https://developer.apple.com/documentation/usernotifications/handling_notifications_and_notification-related_actions)에서 더 확인할 수 있다.
* 앱에 알림과 관련된 세팅을 불러온다.

전달된 알림과 알림과 관련된 행위를 다루기 위해 `UNUserNotificationCenterDelegate`를 채택한 객체를 생성하고 `delegate` 프로퍼티를 이 delegate와 상호작용하는 작업이 실행되기 전에 이 객체에 설정한다. Delegate 객체를 delegate에 정보를 리턴하는 메서드를 호출한 이후에 할당하면 오류가 발생한다.

### UNUserNotificationCenterDelegate : 들어오는 알림이나 알림과 관련된 행위를 다루는 인터페이스

이 프로토콜의 메서드를 사용해서 알림에서 사용자가 선택하는 행위를 다루고, 앱이 현재 화면에 떠있을 때 오는 알림을 처리한다. 객체에서 이 메서드를 구현한 후에는 
이 객체를 공유된 `UNUserNotificationCenter` 객체의 `delegate` 프로퍼티에 할당한다. 사용자 알림 센터 객체는 적절한 시점에 delegate의 메서드를 호출한다.

*앱이 실행되기 전에 delegate 객체를 `UNUserNotificationCenter` 객체에 할당해야 한다. 예를 들어 iOS 앱에서는, 엡 delegate의 ` application(_:willFinishLaunchingWithOptions:)` 나 ` application(_:didFinishLaunchingWithOptions:)` 메서드에서 할당해야 한다. 이 메서드가 호출된 이후에 delegate를 할당하는 게 알림을 놓치게 할 수 있다.*

### Notification과 Notification과 관련된 행위 다루기

앱의 커스텀 액션을 다루는 것을 포함해서 시스템의 알림 인터페이스와 사람의 상호작용에 응답하기.

#### 개요

알림은 주로 사용자에게 정보를 제공하는 방법이지만, 앱은 이것에 반응할 수 있다. 예를 들어, 아래와 같은 것들에 응답하고 싶을 수 있다.

* 알림 인터페이스에서 사용자가 선택한 행위
* 앱이 화면에서 실행되고 있을 때 알림이 도착했을 경우
* 조용한 알림 [background update를 앱에 push 하기](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/pushing_background_updates_to_your_app)에서 더 확인할 수 있다.
* PushKit 프레임워크와 관련된 알림(VoIP나 complication-related 알림)

#### 사용자 선택 행위 관리하기

Actionable 알림은 사용자가 알림 인터페이스에서 바로 응답할 수 있게 해준다. 알림 컨텐츠에 추가로, actionable 알림은 사용자가 할 수 있는 행위를 나타내는 버튼을 하나 이상 표시한다. 한 버튼을 터치하면 앱을 화면 앞으로 불러오지 않고도 선택된 행위를 앱에 할 수 있게 한다. 만약 앱이 actionable 알림 타입을 지원한다면, 관련된 행위를 다뤄야 할 것이다.

![image](https://user-images.githubusercontent.com/41438361/120091451-42f29e00-c146-11eb-809e-6ee901365200.png)

앱이 지원하는 카테고리를 선언하는 시점에 actionable 알림 타입을 실행시에 선언해야 한다. [Actionable 알림 타입 선언하기](https://developer.apple.com/documentation/usernotifications/declaring_your_actionable_notification_types)에서 더 확인할 수 있다.


공유된 `UNUserNotificationCenter` 객체의 delegate 객체의 선택된 행위들을 관리한다. 사용자가 특정 행위를 선택할 경우, 시스템은 앱을 백그라운드에서 실행시키고 delegate의 `userNotificationCenter(_:didReceive:withCompletionHandler:)` 메서드를 실행시킨다. 응답 객체의 `actionIdentifier` 프로퍼티에서 값을 앱의 액션 중 하나와 일치시킨다. Action identifier가 시스템에서 정의된 액션 중 하나를 일치시킬 수 있도록 준비시켜야 한다. 시스템은 사용자가 알림을 무시하거나 앱을 실행시킬때 특별한 행위를 보고한다.

아래 예시는 미팅 초대와 관련된 행위들을 처리한다. ACCEPT_ACTION과 DECLINE_ACTION 문자열은 미팅 초대에 적절한 응답을 생성하는 앱에서 정의된 액션을 정의한다. 만약 사용자가 앱에서 정의된 액션 중 하나를 선택하지 않는다면, 메서드는 사용자가 앱을 실행시키기 전까지 초대를 저장하고 있는다.

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter,
            didReceive response: UNNotificationResponse,
            withCompletionHandler completionHandler: 
               @escaping () -> Void) {
   // Get the meeting ID from the original notification.
   let userInfo = response.notification.request.content.userInfo
        
   if response.notification.request.content.categoryIdentifier ==
              "MEETING_INVITATION" {
      // Retrieve the meeting details.
      let meetingID = userInfo["MEETING_ID"] as! String
      let userID = userInfo["USER_ID"] as! String
            
      switch response.actionIdentifier {
      case "ACCEPT_ACTION":
         sharedMeetingManager.acceptMeeting(user: userID, 
               meetingID: meetingID)
         break
                
      case "DECLINE_ACTION":
         sharedMeetingManager.declineMeeting(user: userID, 
               meetingID: meetingID)
         break
                
      case UNNotificationDefaultActionIdentifier,
           UNNotificationDismissActionIdentifier:
         // Queue meeting-related notifications for later
         //  if the user does not act.
         sharedMeetingManager.queueMeetingForDelivery(user: userID,
               meetingID: meetingID)
         break
                
      default:
         break
      }
   }
   else {
      // Handle other notification types...
   }
        
   // Always call the completion handler when done.
   completionHandler()
}
```

#### 앱이 화면상에서 실행되고 있을 때 알림 관리하기

만약 앱이 가장 앞에서(화면에 노출된 상태에서) 실행되고 있을 때 알림이 온다면, 시스템은 알림을 앱에 바로 전달한다. 알림을 받으면, 알림의 payload로 하고 싶은 것을 아무거나 할 수 있다. 예를 들어, 알림에 포함된 새로운 정보를 반영하기 위해 앱의 인터페이스를 업데이트 할 수도 있다. 스케줄 된 알림을 설정하거나 이런 알림들을 수정할 수 있다.

알림이 도착하면, 시스템은 `UNUserNotificationCenter` 객체의 delegate의 ` userNotificationCenter(_:willPresent:withCompletionHandler:)` 메서드를 호출한다. 이 메서드를 사용해서 알림을 처리하고 시스템이 내가 이게 전달되기를 원한다는 것을 알게 해준다. 아래의 예제는 캘린더 앱에서 이 메서드를 사용하는 것을 보여준다. 초대가 오면, 앱은 새로운 초대를 앱의 인터페이스에서 보여주기 위해 커스텀 메서드인 `queueMeetingForDelivery`를 호출한다. 앱은 또한 `sound` 값을 completion handler에 전달해서 알림의 사운드를 틀게 할지 시스템에게 물어본다. 다른 알림 타입에 대해서는 메서드는 알림이 소리가 나지 않게 한다.

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter,
         willPresent notification: UNNotification,
         withCompletionHandler completionHandler: 
            @escaping (UNNotificationPresentationOptions) -> Void) {
   if notification.request.content.categoryIdentifier == 
            "MEETING_INVITATION" {
      // Retrieve the meeting details.
      let meetingID = notification.request.content.
                       userInfo["MEETING_ID"] as! String
      let userID = notification.request.content.
                       userInfo["USER_ID"] as! String
            
      // Add the meeting to the queue.
      sharedMeetingManager.queueMeetingForDelivery(user: userID,
            meetingID: meetingID)

      // Play a sound to let the user know about the invitation.
      completionHandler(.sound)
      return
   }
   else {
      // Handle other notification types...
   }

   // Don't alert the user for other types.
   completionHandler(UNNotificationPresentationOptions(rawValue: 0))
}
```

앱을 PushKit으로 등록했다면, PushKit-타입을 겨냥하는 알림들은 앱에 바로 전해지며 사용자에게 절대로 보여지지 않는다. 만약 앱이 화면에 띄워져 있거나 백그라운드에서 실행이 된다면, 시스템은 알림을 처리할 시간을 앱에게 준다. 만약 앱이 실행되고 있지 않으면, 시스템은 앱을 백그라운드에서 실행시켜서 알림을 처리할 수 있게 한다. PushKit 알림을 전송하기 위해서, provider server는 무조건 알림의 토픽을 적절한 타겟(앱의 완료)에 설정해야 한다. 
