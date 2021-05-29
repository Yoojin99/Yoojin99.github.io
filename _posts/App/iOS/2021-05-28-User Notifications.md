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

Notification을 생성해서 사용자 디바이스에 푸시할 것이다.

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
 


ddfsdfa;

